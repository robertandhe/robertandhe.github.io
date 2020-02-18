---
layout: post
title: "数据挖掘比赛常用函数"
subtitle: 'data mining function'
date:       2019-10-10 12:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - competition
---

## 设种子
种子保证了实验结果的可重复性，便于前后比较
```python
def seed_everything(seed=0):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
```
## 数据压缩
数据压缩能缩小数据占用空间，加快运行速度，常用的一个函数如下：   
```python
def reduce_memory_usage(df, verbose=True):
numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
start_mem = df.memory_usage().sum() / 1024**2    
for col in df.columns:
    col_type = df[col].dtypes
    if col_type in numerics:
        c_min = df[col].min()
        c_max = df[col].max()
        if str(col_type)[:3] == 'int':
            if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                df[col] = df[col].astype(np.int8)
            elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                df[col] = df[col].astype(np.int16)
            elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                df[col] = df[col].astype(np.int32)
            elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                df[col] = df[col].astype(np.int64)  
        else:
            if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                df[col] = df[col].astype(np.float16)
            elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                df[col] = df[col].astype(np.float32)
            else:
                df[col] = df[col].astype(np.float64)    
end_mem = df.memory_usage().sum() / 1024**2
if verbose: print('Mem. usage decreased to {:5.2f} Mb ({:.1f}% reduction)'.format(end_mem, 100 * (start_mem - end_mem) / start_mem))
return df
```
## 冗余特征
去除变化小，缺失值太多，极端分布特征  
```python
one_value_cols = [col for col in train.columns if train[col].nunique() <= 1]
one_value_cols_test = [col for col in test.columns if test[col].nunique() <= 1]

many_null_cols = [col for col in train.columns if train[col].isnull().sum() / train.shape[0] > 0.95]
many_null_cols_test = [col for col in test.columns if test[col].isnull().sum() / test.shape[0] > 0.95]

big_top_value_cols = [col for col in train.columns if train[col].value_counts(dropna=False, normalize=True).values[0] > 0.95]
big_top_value_cols_test = [col for col in test.columns if test[col].value_counts(dropna=False, normalize=True).values[0] > 0.95]

cols_to_drop = list(set(one_value_cols + one_value_cols_test + many_null_cols + many_null_cols_test + big_top_value_cols + \
                        big_top_value_cols_test))
```  
## 相关性分析
观察哪些变量之间有相关性[link](https://www.kaggle.com/cdeotte/eda-for-columns-v-and-id)
```python
def make_corr(Vs,Vtitle=''):
    cols = ['TransactionDT'] + Vs
    plt.figure(figsize=(15,15))
    sns.heatmap(train[cols].corr(), cmap='RdBu_r', annot=True, center=0.0)
    if Vtitle!='': 
        plt.title(Vtitle,fontsize=14)
    else: plt.title(Vs[0]+' - '+Vs[-1],fontsize=14)
    plt.show()
```
## 变量分组思想
Example:按照nan分组[link](https://www.kaggle.com/cdeotte/eda-for-columns-v-and-id)
```python
nans_df = train.isna()
nans_groups={}
i_cols = ['V'+str(i) for i in range(1,340)]
for col in train.columns:
    cur_group = nans_df[col].sum()
    try:
        nans_groups[cur_group].append(col)
    except:
        nans_groups[cur_group]=[col]
del nans_df; x=gc.collect()

for k,v in nans_groups.items():
    print('####### NAN count =',k)
    print(v)
```
分组后再在组内进行相关性分析，按照相关性把变量分到各个子集，在子集内挑选具有代表性的变量，例如unique等。
```python
grps = [[1],[2,3],[4,5],[6,7],[8,9],[10,11]]
def reduce_group(grps,c='V'):
    use = []
    for g in grps:
        mx = 0; vx = g[0]
        for gg in g:
            n = train[c+str(gg)].nunique()
            if n>mx:
                mx = n
                vx = gg
            #print(str(gg)+'-'+str(n),', ',end='')
        use.append(vx)
        #print()
    print('Use these',use)
reduce_group(grps)
```
## 图形分布  
借助pandas和seaborn可快速绘制优美图形  
Example 1：直接通过DataFrame.plot绘图 
```python
df_train['TransactionAmt'].apply(np.log).plot(kind='hist', bins=100, figsize=(15, 5),
                                            title='Distribution of Log Transaction Amt')
```
![eda_fig1](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-05-post/eda_fig1.png)
Example 2: seaborn绘图 
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 6))
fig.suptitle("Count of transaction and mean of fraud by days and weekday",fontsize=20)
sns.countplot(df_train['weekday'], ax=ax1)
ax1t = ax1.twinx()
perc_fraud_weekday = df_train.groupby('weekday')['isFraud'].mean()
perc_fraud_weekday = perc_fraud_weekday.reset_index()
sns.pointplot(x='weekday', y='isFraud', data=perc_fraud_weekday, ax=ax1t)
ax1t.set_ylim([0, 0.05])
sns.countplot(df_train['days'], ax=ax2)
ax2t = ax2.twinx()
perc_fraud_days = df_train.groupby('days')['isFraud'].mean()
perc_fraud_days = perc_fraud_days.reset_index()
sns.pointplot(x='days', y='isFraud', data=perc_fraud_days, ax=ax2t)
ax2t.set_ylim([0, 0.05])
```
![eda_fig2](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-05-post/eda_fig2.png)
Example 3:
```python
df_train.groupby('ProductCD')['TransactionID'].count().sort_index().plot(x='ProductCD', kind='barh', figsize=(15, 3), color=color_pal, title='Count of Observations by ProductCD')
```
![eda_fig3](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-05-post/eda_fig3.png)
其余常用函数为distplot, barplot, countplot, boxplot, violinplot
## 异常值  
Example 1：quantile函数
```python
df_train[['card1', 'card2', 'card3', 'card5']].quantile([.1, .25, .5, .75, .9, .99])
```
Example 2： 箱线图
```python
sns.boxplot(x='DT_M', y='D2', hue='isFraud', data=train_df, palette='hls')
```
![eda_fig4](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-05-post/eda_fig4.png)
## 缺失值  
数值型特征一般用均值，加权均值，中位数填充；分类特征一般可用众数填充。  
在比赛中，lightgbm自身可对缺失值较好处理，故仅对card2-card6进行众数填充，用于后面构造虚拟身份特征。
## 后处理
玄学！ 
在IEEE-Fraud-Detection比赛中，关键在于识别欺诈客户，而不是欺诈交易。所以最终预测结果中，同一客户应该具有相同的
欺诈概率值,取所有预测概率值的平均或者分位数。
## 特征工程函数
values_normalization
```python
def values_normalization(dt_df, periods, columns, enc_type='both'):
    for period in periods:
        for col in columns:
            new_col = col +'_'+ period
            dt_df[col] = dt_df[col].astype(float)  

            temp_min = dt_df.groupby([period])[col].agg(['min']).reset_index()
            temp_min.index = temp_min[period].values
            temp_min = temp_min['min'].to_dict()

            temp_max = dt_df.groupby([period])[col].agg(['max']).reset_index()
            temp_max.index = temp_max[period].values
            temp_max = temp_max['max'].to_dict()

            temp_mean = dt_df.groupby([period])[col].agg(['mean']).reset_index()
            temp_mean.index = temp_mean[period].values
            temp_mean = temp_mean['mean'].to_dict()

            temp_std = dt_df.groupby([period])[col].agg(['std']).reset_index()
            temp_std.index = temp_std[period].values
            temp_std = temp_std['std'].to_dict()

            dt_df['temp_min'] = dt_df[period].map(temp_min)
            dt_df['temp_max'] = dt_df[period].map(temp_max)
            dt_df['temp_mean'] = dt_df[period].map(temp_mean)
            dt_df['temp_std'] = dt_df[period].map(temp_std)
            
            if enc_type=='both':
                dt_df[new_col+'_min_max'] = (dt_df[col]-dt_df['temp_min'])/(dt_df['temp_max']-dt_df['temp_min'])
                dt_df[new_col+'_std_score'] = (dt_df[col]-dt_df['temp_mean'])/(dt_df['temp_std'])
            elif enc_type=='norm':
                 dt_df[new_col+'_std_score'] = (dt_df[col]-dt_df['temp_mean'])/(dt_df['temp_std'])
            elif enc_type=='min_max':
                dt_df[new_col+'_min_max'] = (dt_df[col]-dt_df['temp_min'])/(dt_df['temp_max']-dt_df['temp_min'])

            del dt_df['temp_min'],dt_df['temp_max'],dt_df['temp_mean'],dt_df['temp_std']
    return dt_df
```
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
## lightgbm
```python
lgb_params = {
                'objective':'binary',
                'boosting_type':'gbdt',
                'metric':'auc',
                'n_jobs':-1,
                'learning_rate':0.01,
                'num_leaves': 2**8,
                'max_depth':-1,
                'tree_learner':'serial',
                'colsample_bytree': 0.7,
                'subsample_freq':1,
                'subsample':0.7,
                'n_estimators':80000,
                'max_bin':255,
                'verbose':-1,
                'seed': SEED,
                'early_stopping_rounds':100, 
        } 

def make_predictions(tr_df, tt_df, features_columns, target, lgb_params, NFOLDS=2):
    folds = KFold(n_splits=NFOLDS, shuffle=True, random_state=SEED)

    X,y = tr_df[features_columns], tr_df[target]    
    P,P_y = tt_df[features_columns], tt_df[target]  

    tt_df = tt_df[['TransactionID',target]]    
    predictions = np.zeros(len(tt_df))
    
    for fold_, (trn_idx, val_idx) in enumerate(folds.split(X, y)):
        print('Fold:',fold_)
        tr_x, tr_y = X.iloc[trn_idx,:], y[trn_idx]
        vl_x, vl_y = X.iloc[val_idx,:], y[val_idx]
            
        print(len(tr_x),len(vl_x))
        tr_data = lgb.Dataset(tr_x, label=tr_y)

        if LOCAL_TEST:
            vl_data = lgb.Dataset(P, label=P_y) 
        else:
            vl_data = lgb.Dataset(vl_x, label=vl_y)  

        estimator = lgb.train(
            lgb_params,
            tr_data,
            valid_sets = [tr_data, vl_data],
            verbose_eval = 200,
        )   
        
        pp_p = estimator.predict(P)
        predictions += pp_p/NFOLDS

        if LOCAL_TEST:
            feature_imp = pd.DataFrame(sorted(zip(estimator.feature_importance(),X.columns)), columns=['Value','Feature'])
            print(feature_imp)
        
        del tr_x, tr_y, vl_x, vl_y, tr_data, vl_data
        gc.collect()
        
    tt_df['prediction'] = predictions
    
    return tt_df
```
## xgboost
```python
oof = np.zeros(len(X_train))
preds = np.zeros(len(X_test))

clf = xgb.XGBClassifier( 
    n_estimators=2000,
    max_depth=12, 
    learning_rate=0.02, 
    subsample=0.8,
    colsample_bytree=0.4, 
    missing=-1, 
    eval_metric='auc',
    # USE CPU
    #nthread=4,
    #tree_method='hist' 
    # USE GPU
    tree_method='gpu_hist' 
)

h = clf.fit(X_train.loc[idxT,cols], y_train[idxT], 
    eval_set=[(X_train.loc[idxV,cols],y_train[idxV])],
    verbose=50, early_stopping_rounds=100)

oof[idxV] += clf.predict_proba(X_train[cols].iloc[idxV])[:,1]
preds += clf.predict_proba(X_test[cols])[:,1]/skf.n_splits

feature_imp = pd.DataFrame(sorted(zip(clf.feature_importances_,cols)), columns=['Value','Feature'])
plt.figure(figsize=(20, 10))
sns.barplot(x="Value", y="Feature", data=feature_imp.sort_values(by="Value", ascending=False).iloc[:50])
plt.title('XGB95 Most Important Features')
plt.tight_layout()
plt.show()
```

