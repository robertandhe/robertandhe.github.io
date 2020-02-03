---
layout: post
title: "IEEE Fraud Detection Top team guide"
subtitle: 'ieee fraud detection'
date:       2019-10-09 12:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - technique
---

# 1st place solution
## discussion1
[link-discussion](https://www.kaggle.com/c/ieee-fraud-detection/discussion/111284#latest-644768)
第一名在discussion中提出了几个重点：
1. 时间不是最重要的  
因为欺诈行为的特征不会短时间内随时间变化，而是欺诈的人在随着时间变换。训练集、Public LB和Private LB在时间上是分离的，从下图可见，在Private LB中仅有16.4%的clients在训练集中出现过，68.2%的clients没有在训练集中出现，另外15.4%不确定。
![top1-discussion-train-test-time](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/ieee-fraud-detection-top-team-share/top1-discussion-train-test-time.png)
> 注：每条线表示每个client，左端点是初次交易时间，右端点是最后一次交易时间    
2. Magic Feature  
根据vesta公司工作人员对数据集的解释
>The logic of our labeling is define reported chargeback on the card as fraud transaction (isFraud=1) and transactions posterior to it with either user account, email address or billing address directly linked to these attributes as fraud too. If none of above is reported and found beyond 120 days, then we define as legit transaction (isFraud=0).  
比赛不是在预测交易行为，重点在于识别欺诈客户，classifying clients (credit cards) instead of transactions。
3. Fraudulent Clients  
top1队伍利用card、addr1和D1来辨别每个客户。D1是当前交易时间和开卡时间的间隔，D3表示当前交易时间和上笔交易时间的间隔，
而TransactionDay(从TransactionDT求出)是当前交易日期，所以``D1n = TransactionDay - D1``表示开卡时间，``D3n = TransactionDay - D3``表示上笔交易时间。下图是2134个欺诈客户中``client=2988694``的例子。
![top1-discussion-fraudulent-clients](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/ieee-fraud-detection-top-team-share/top1-discussion-fraudulent-client.png)
4. Preventing Overfitting
测试集中68.2%的UID没有在训练集中出现过，为了防止在训练集和测试集上过拟合，不能把client UID当作特征。同时其他有利于识别client UID的特征如D, V, ID也不能用。
相反，必须根据UID创建aggregated group features聚合特征，例如取所有的C,M列，``new_features = df.groupby('uid')[CM_columns].agg(['mean'])``。

## kernel1
EDA for Columns V and ID[link](https://www.kaggle.com/cdeotte/eda-for-columns-v-and-id)  
这个kernel的主要思想是对V和ID特征降维。
首先按照NAN个数把它们分组，在组内按照相关性分成各个子集，在子集内按照unique的个数挑选一个代表性变量。

## kernel2
在这个kernel中，top1队伍给出了Public LB达到0.962081的代码[link](https://www.kaggle.com/cdeotte/xgb-fraud-with-magic-0-9600).
1. 数据类型处理  
把所有数值变量转换为float32类型，把所有名义变量转换为category类型。
2. D columns normalize  
D各个列表示从某一时间点的``time deltas``，把这些列转化会它们原来的时间点``D15n = Transaction_Day - D15 and Transaction_Day = TransactionDT/(24*60*60)``，否则各个D列的值就是随时间增长，容易带来train，test的不一致性。
3. 几个特征工程函数  
使用了五个特征工程函数    
frequency encoding
```python
def encode_FE(df1, df2, cols):
    for col in cols:
        df = pd.concat([df1[col],df2[col]])
        vc = df.value_counts(dropna=True, normalize=True).to_dict()
        vc[-1] = -1
        nm = col+'_FE'
        df1[nm] = df1[col].map(vc)
        df1[nm] = df1[nm].astype('float32')
        df2[nm] = df2[col].map(vc)
        df2[nm] = df2[nm].astype('float32')
        print(nm,', ',end='')
```
label encoding
```python
def encode_LE(col,train=X_train,test=X_test,verbose=True):
    df_comb = pd.concat([train[col],test[col]],axis=0)
    df_comb,_ = df_comb.factorize(sort=True)
    nm = col
    if df_comb.max()>32000: 
        train[nm] = df_comb[:len(train)].astype('int32')
        test[nm] = df_comb[len(train):].astype('int32')
    else:
        train[nm] = df_comb[:len(train)].astype('int16')
        test[nm] = df_comb[len(train):].astype('int16')
    del df_comb; x=gc.collect()
    if verbose: print(nm,', ',end='')
```
group aggregation
```python
def encode_AG(main_columns, uids, aggregations=['mean'], train_df=X_train, test_df=X_test, 
              fillna=True, usena=False):
    # AGGREGATION OF MAIN WITH UID FOR GIVEN STATISTICS
    for main_column in main_columns:  
        for col in uids:
            for agg_type in aggregations:
                new_col_name = main_column+'_'+col+'_'+agg_type
                temp_df = pd.concat([train_df[[col, main_column]], test_df[[col,main_column]]])
                if usena: temp_df.loc[temp_df[main_column]==-1,main_column] = np.nan
                temp_df = temp_df.groupby([col])[main_column].agg([agg_type]).reset_index().rename(
                                                        columns={agg_type: new_col_name})

                temp_df.index = list(temp_df[col])
                temp_df = temp_df[new_col_name].to_dict()   

                train_df[new_col_name] = train_df[col].map(temp_df).astype('float32')
                test_df[new_col_name]  = test_df[col].map(temp_df).astype('float32')
                
                if fillna:
                    train_df[new_col_name].fillna(-1,inplace=True)
                    test_df[new_col_name].fillna(-1,inplace=True)
                
                print("'"+new_col_name+"'",', ',end='')
```
combine features
```python
def encode_CB(col1,col2,df1=X_train,df2=X_test):
    nm = col1+'_'+col2
    df1[nm] = df1[col1].astype(str)+'_'+df1[col2].astype(str)
    df2[nm] = df2[col1].astype(str)+'_'+df2[col2].astype(str) 
    encode_LE(nm,verbose=False)
    print(nm,', ',end='')
```
group aggregation nunique
```python
def encode_AG2(main_columns, uids, train_df=X_train, test_df=X_test):
    for main_column in main_columns:  
        for col in uids:
            comb = pd.concat([train_df[[col]+[main_column]],test_df[[col]+[main_column]]],axis=0)
            mp = comb.groupby(col)[main_column].agg(['nunique'])['nunique'].to_dict()
            train_df[col+'_'+main_column+'_ct'] = train_df[col].map(mp).astype('float32')
            test_df[col+'_'+main_column+'_ct'] = test_df[col].map(mp).astype('float32')
            print(col+'_'+main_column+'_ct, ',end='')
```  
4. 初步特征工程
```python
X_train['cents'] = (X_train['TransactionAmt'] - np.floor(X_train['TransactionAmt'])).astype('float32')
X_test['cents'] = (X_test['TransactionAmt'] - np.floor(X_test['TransactionAmt'])).astype('float32')
encode_FE(X_train,X_test,['addr1','card1','card2','card3','P_emaildomain'])
encode_CB('card1','addr1')
encode_CB('card1_addr1','P_emaildomain')
encode_FE(X_train,X_test,['card1_addr1','card1_addr1_P_emaildomain'])
encode_AG(['TransactionAmt','D9','D11'],['card1','card1_addr1','card1_addr1_P_emaildomain'],['mean','std'],usena=True)
```
5. 时间一致性   
对每个特征，根据第一个月这个唯一的特征训练模型去预测最后一个月，希望训练集和测试集auc大于0.5，如果差别大说明
这个特征时间不一致，应该舍弃。这个队伍总共训练了242个模型.  
```python
# FAILED TIME CONSISTENCY TEST
cols = list( X_train.columns )
cols.remove('TransactionDT')
for c in ['D6','D7','D8','D9','D12','D13','D14']:
    cols.remove(c)
for c in ['C3','M5','id_08','id_33']:
    cols.remove(c)
for c in ['card4','id_07','id_14','id_21','id_30','id_32','id_34']:
    cols.remove(c)
for c in ['id_'+str(x) for x in range(22,28)]:
    cols.remove(c)
```  
到这里xgboost模型Public LB达到0.951005。    
6. Magic Feature-UID  
利用card1，add1，D1n构建unique client id.  
```python
X_train['day'] = X_train.TransactionDT / (24*60*60)
X_train['uid'] = X_train.card1_addr1.astype(str)+'_'+np.floor(X_train.day-X_train.D1).astype(str)
X_test['day'] = X_test.TransactionDT / (24*60*60)
X_test['uid'] = X_test.card1_addr1.astype(str)+'_'+np.floor(X_test.day-X_test.D1).astype(str)
```      
7. Group Aggregation Features      
```python
encode_FE(X_train,X_test,['uid'])
# AGGREGATE 
encode_AG(['TransactionAmt','D4','D9','D10','D15'],['uid'],['mean','std'],fillna=True,usena=True)
# AGGREGATE
encode_AG(['C'+str(x) for x in range(1,15) if x!=3],['uid'],['mean'],X_train,X_test,fillna=True,usena=True)
# AGGREGATE
encode_AG(['M'+str(x) for x in range(1,10)],['uid'],['mean'],fillna=True,usena=True)
# AGGREGATE
encode_AG2(['P_emaildomain','dist1','DT_M','id_02','cents'], ['uid'], train_df=X_train, test_df=X_test)
# AGGREGATE
encode_AG(['C14'],['uid'],['std'],X_train,X_test,fillna=True,usena=True)
# AGGREGATE 
encode_AG2(['C13','V314'], ['uid'], train_df=X_train, test_df=X_test)
# AGGREATE 
encode_AG2(['V127','V136','V309','V307','V320'], ['uid'], train_df=X_train, test_df=X_test)
# NEW FEATURE
X_train['outsider15'] = (np.abs(X_train.D1-X_train.D15)>3).astype('int8')
X_test['outsider15'] = (np.abs(X_test.D1-X_test.D15)>3).astype('int8')
```
到这里xgboost模型Public LB达到0.960205。          
8. 后处理               
对每一client给出相同概率  
```python  
X_test['isFraud'] = sample_submission.isFraud.values
X_train['isFraud'] = y_train.values
comb = pd.concat([X_train[['isFraud']],X_test[['isFraud']]],axis=0)
uids = pd.read_csv('/kaggle/input/ieee-submissions-and-uids/uids_v4_no_multiuid_cleaning..csv',usecols=['TransactionID','uid']).rename({'uid':'uid2'},axis=1)
comb = comb.merge(uids,on='TransactionID',how='left')
mp = comb.groupby('uid2').isFraud.agg(['mean'])
comb.loc[comb.uid2>0,'isFraud'] = comb.loc[comb.uid2>0].uid2.map(mp['mean'])
uids = pd.read_csv('/kaggle/input/ieee-submissions-and-uids/uids_v1_no_multiuid_cleaning.csv',usecols=['TransactionID','uid']).rename({'uid':'uid3'},axis=1)
comb = comb.merge(uids,on='TransactionID',how='left')
mp = comb.groupby('uid3').isFraud.agg(['mean'])
comb.loc[comb.uid3>0,'isFraud'] = comb.loc[comb.uid3>0].uid3.map(mp['mean'])
sample_submission.isFraud = comb.iloc[len(X_train):].isFraud.values
sample_submission.to_csv('sub_xgb_96_PP.csv',index=False)
```
到这里xgboost模型Public LB达到0.961847。     



# 2nd place solution
[link](https://www.kaggle.com/c/ieee-fraud-detection/discussion/111321?utm_medium=email&utm_source=intercom&utm_campaign=competition-recaps-ieee-2019)
1. Data Cleaning
观察所有特征train vs. test分布图，例如
![top2-dis1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/ieee-fraud-detection-top-team-share/top2-discussion1.PNG)
红色为训练集数据，蓝色为测试集数据，左边横轴为card1，右边横轴为card1的频率编码。
可看出频率编码后训练集测试集分布更均衡。所以把card1转化为频率。
2. Cross Validation  
Cross validation for time series is always tricky.  
run these folds, using months starting with with 0. The | indicates the train/val split
0 | 2 3 4 5 6
0 1 | 3 4 5 6
0 1 2 | 4 5 6
0 1 2 3 | 5 6
3. Feature Engineering
先用permuation importance进行特征选择。
client id   
```python
data['uid1']` =  (data.day - data.D1).astype(str) +'_' + \
            data.P_emaildomain.astype(str)
data['uid2'] =  (data.card1.astype(str) +'_' + \
            data.addr1.astype(str) +'_' + \
            (data.day - data.D1).astype(str) +'_' + \
            data.P_emaildomain.astype(str))
```
根据D1n和P_emaildomain构造uid1，再根据uid1，card1，addr1构造uid2.  
聚合特征, col为uid1 or uid2.  
```python
def add_gr(data, col):
    cols = data.columns
    gr = data.groupby(col)
    data[col+'_count'] = gr.TransactionID.transform('count').astype('int32')
    data[col+'_next_dt'] = gr.TransactionDT.shift(-1)
    data[col+'_next_dt'] -= data.TransactionDT
    data[col+'_mean_dt'] = gr[col+'_next_dt'].transform('mean').astype('float32')
    data[col+'_std_dt'] = gr[col+'_next_dt'].transform('std').astype('float32')
    data[col+'_median_dt'] = gr[col+'_next_dt'].transform('median').astype('float32')

    data[col+'_next_amt'] = gr.TransactionAmt.shift(-1)
    data[col+'_mean_amt'] = gr.TransactionAmt.transform('mean').astype('float32')
    data[col+'_std_amt'] = gr.TransactionAmt.transform('std').astype('float32')
    data[col+'_median_amt'] = gr.TransactionAmt.transform('median').astype('float32')
    new_cols = list(set(data.columns) - set(cols) -set([col+'_next_dt']))
    return new_cols
```
对每一client找出下次交易时间，再计算时间差，取mean，std和median。
再对每一个client找出下次交易金额，计算交易金额mean，std和median。
4. Posting processing
