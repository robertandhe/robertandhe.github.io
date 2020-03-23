---
layout: post
title: "My First Kaggle Competition Top 3%"
subtitle: 'First Blood'
author: "mudux"
date: 2019-10-05
# header-style: text
catalog: true
header-img: "img/ieee_cis_fraud3.jpg"
header-mask: 0.1
tags:
  - kaggle
  - competition
  - machine learning
---


# <center>IEEE-CIS Fraud Detection</center>

>写在前面：初次参加kaggle比赛，取得了top 3%的成绩，好于预期。在接近一个半月的比赛中，学到了许多之前光做理论没有机会接触的东西，也发现了一些不足，总体来说，收获很大。大佬们的开源共享精神让人心生敬仰。感谢队友clancy的交流进步。


## 1 比赛介绍   

### 1.1 比赛背景 
IEEE与Vesta联合举办信用卡交易欺诈检测， 为经典的数据挖掘领域的欺诈检测问题[link](https://www.kaggle.com/c/ieee-fraud-detection)。    
### 1.2 数据集描述
数据集包括五个文件train_transaction.csv, train_identitiy.csv, test_transaction.csv, test_identity.csv, sample_submission.csv
train_transaction.csv, train_identitiy.csv为训练集交易数据与对应的交易身份记录，test_transaction.csv, test_identity.csv为测试数据。
在训练集中，总交易数为590540，其中欺诈交易数为20663，符合不平衡分类问题特点。   
train_transaction文件包括394个特征，分别为：
- TransactionID: 交易唯一标识，与train_identity merge键值
- isFraud: label
- TransactionDT: timedelta from a given reference datetime, 以秒为单位
- TransactionAmt: transaction payment amount in USD
- ProductCD: product code, the product for each transaction, 分别为W, H, C, S, R
- card1-card6: payment card information, such as card type, card category, issue bank, country, etc
- addr1 & addr2: address
- dist1 & dist2: distance
- P_emaildomain & R_emaildomain: purchaseer and recipient email domain
- C1-C14: counting, such as how many addresses are found to be associated with the pament card, etc. The actual meaning is masked
- D1-D15: timedelta, such as days between previous transaction, days to issue, etc
- M1-M9: match, such as names on card and address, etc
- V1-V339: Vesta engineered rich features, including ranking, counting, and other relations.
在这些特征中，类别特征为：
ProductCD, card1-card6, addr1, addr2, P_emaildomain,R_emaildomain, M1-M9   

train_identitiy文件包括41个特征，分别为：
- TransactionID: 交易唯一标识
- id01-id38: network connection information(IP, ISP, Proxy,etc) and digital signature(UA/browser/os/version,etc) associated with transactions.
- DeviceType: mobile or desktop
- DeviceInfo: detail device information, such as SAMSUNG SM-G892A Build/NRD90M 

train_transaction有590540笔记录，train_identity仅有144233笔记录，所以有些交易没有对应的身份数据。
## 2 特征工程  
>More data beats clever algorithms, but better data betas more data - by Peter Norving     

很重要，最有创造力的工作 
### 2.1 类别特征
1. Onehot encoding   
   适用于线性模型，稀疏，极大扩充数据维度。在本次比赛中，采用lgb分类，为非线性树模型，所以没用onehot。  
   sklearn.preprocessing.OntHotEncoder()    
2. Hash encoding 
    散列表，避免onehot过度稀疏，但同一键值可散列到同一表示，导致冲突。   
    sklearn.feature_extraction.FeatureHasher()   
3. Label encoding   
   每个类别一个编码，适用于非线性树模型，不增加数据维度，本次比赛主要采用label encoding.   
   sklearn.preprocessing.LabelEncoder()   
4. Count encoding   
   用出现频率替换类别特征。适用于线性和非线性模型，对缺失值敏感， 缺点：不同变量，同一编码。
5. LabelCount encoding   
   特征按频率排序编码。 适用于线性和非线性模型，对缺失值不敏感，不同变量不同编码，即使频率一样，最优。
6. Target encoding   
   Encode categorical variables by their ratio of target.   
   Be careful to avoid overfit!   
   Form of stacking: single-variable model which outputs average target.   
   Add smooting to avoid setting variable encodings to 0. 
   Add random noise to combat overfit.   
7. Category embedding   
   Use a Neural Network to create dense embeddings from categorical variables.    
   Map categorical variables in a function approximation problem into Euclidean spaces.   
   Faster model training.   
   Less memory overhead.  
   Can give better accuracy that 1-hot encoded.  
   [related paper-Entity Embeddings of Categorical Variables](https://arxiv.org/abs/1604.06737).
8. Nan encoding   
   Give Nan values an explicit encoding instead of ignoring.  
   Be careful to avoid overfit!   
9. Polynomial encoding   
    Encode interactions between categorical variables.   
    相当重要！ 因为lgb树模型训练为贪心算法，只从现有特征寻找最优特征，不能发现交互特征，所以必须手工创建。    
    sklearn.preprocessing.PolynomialFeatures()   
10. Expansion encoding   
    Create multiple categorical variables from a single variable.   
    有些高序特征high cardinality features包含许多信息。 like user-agents, is mobile? is latest version? Operation system,
    Browser build etc.   
11. Consolidation encoding   
    Map different categorical variables to the same variable.   
    Spelling errors, slightly different job descriptions, full name vs. abbreviations.  
    Eg. Shell, shell, SHELL, Shell Gasoline-> Shell   

### 2.2 数值特征
1. Rounding  
   Round numerical variables.   
   Retain most significant features of the data. Sometimes too much precision is just noise.  
   Rounded variables can be treated as categorical variables. 
2. Binning   
   Put numerical variables into a bin and encode with bin-ID.  
   Binning can be set pragmatically, by quantiles, evenly, or use models to find optimal bins.   
   Can work gracefully with variables outside of ranges seen in the train set.  
3. Scaling  
   Scale to numerical variables into a certain range.  
   Standard(Z) Scaling -> sklearn.preprocessing.scale()
   MinMax Scaling -> sklearn.preprocessing.MinMaxScaler() 
   Root Scaling   
   Log Scaling -> np.log1p()
4. Imputation   
   Impute missing variables.   
   Mean: very basic.   
   Mean: More robust to outliers.   
   Ignoring: just postpones the problem, not bad to lgb classifier.  
   Using a model: Can expose algorithm bias.    
   sklearn.impute.SimpleImputer()  
5. Interactions  
   Specifically encodes the interactions between numerical variables.  
   Substraction, Addition, Multiplication, Divison.   
   Use: Feature selection by statistical tests, or trained model feature importances.  
   Application related.  
   Ignore: Human intuition; weird interactions can give signification improvement!    
6. Non-linear encoding for linear algo's   
   Hardcode non-linearitist to improve linear algorithms.  
   Polynomial kernel.   
   Leafcoding(random forest embeddings).   
   Genetic algorithms.   
   Locally Linear Embedding, Spectral Embedding, TSNE.   
8. Row statistics   
   Create statistics on a row of data.   
   Number of NaN's, Number of 0's, Number of negative values, Mean, Max, Min, Skewness etc.   

### 2.3 时间特征     
dates, lots of opportunity for major improments.   
1. Projecting to a circle   
    Turn single features, like day_of_week, into two coordinates on a circle.   
    Use for day_of_week, day_of_month, hour_of_day etc.   
2. Trendlines    
   Instead of encoding: total spend, encode things like: Spend in last week, Spend in last month, spend in last year.    
3. Closeness to major events   
   Hardcode categorical features like: date_3_days_before_holiday: 0/1.   
   National holidays, major sport events, weekends, firt Saturday of month etc.   
   These factors can have major influence on spending behavior.   

### 2.4 空间特征  
Spatial variables are variables that encode a location in space.  Examples include: GPS-coordinates, cities, countries, addresses.   
1. Categorizing location  
    K-means clustering, Raw latitude longitude, Convert Cities to latitude longitude,
    Add zip codes to streetnames.   
2. Closeness to hubs   
   Find closeness between a location to a major hub.    
   Small towns inherit some of the culture/context of nearby big cities.   
   Phone locatin can be mapped to nearby businesses and supermarkets.   
3. Spatial fradulent behavior   
   Location event data can be indicative of suspicious behavior.   
   Impossible travel speed: Multiple simultaneous transactions in different countries.   
   Spending in different town that home or shipping address.    
   Never spending at the same location.   

### 2.5 Label Engineering   
Can treat a label/target/dependent varaibles as a feature of the data and vice versa.   
Log-transform: y->log(y+1)|exp(y_pred) - 1
Square-transform   
Box-Cox transform   
Create a score, to turn binary target in regression.   
Train regressor to predict a feature not available in test set.   


## 3 比赛过程
### 3.1 比赛思路
 1. EDA
 2. 添加特征
 3. 交叉验证
 4. 线下测试
 5. 去除无效特征
 6. 线上测试
 重复上述步骤

### 3.2 EDA 观察数据分布，缺失值，异常值，特征相关性，与标签的关系，训练集与测试集关系  
在EDA过程中，基本都在使用pandas与seaborn，通过图形观察。
有几个常用技巧：
#### 3.2.1 设种子  
种子保证了实验结果的可重复性，便于前后比较
```python
def seed_everything(seed=0):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
```

#### 3.2.2 数据压缩  
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
#### 3.2.3 图形分布  
借助pandas和seaborn可快速绘制优美图形  
Example 1：直接通过DataFrame.plot绘图 
```python
df_train['TransactionAmt'].apply(np.log).plot(kind='hist', bins=100, figsize=(15, 5),
                                            title='Distribution of Log Transaction Amt')
```
![eda_fig1](https://gitee.com/alston972/MarkDownPhotos/raw/master/2019-10-05-post/eda_fig1.png)
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
![eda_fig2](https://gitee.com/alston972/MarkDownPhotos/raw/master/2019-10-05-post/eda_fig2.png)
Example 3:
```python
df_train.groupby('ProductCD')['TransactionID'].count().sort_index().plot(x='ProductCD', kind='barh', figsize=(15, 3), color=color_pal, title='Count of Observations by ProductCD')
```
![eda_fig3](https://gitee.com/alston972/MarkDownPhotos/raw/master/2019-10-05-post/eda_fig3.png)
其余常用函数为distplot, barplot, countplot, boxplot, violinplot
#### 3.2.4 异常值  
Example 1：quantile函数

```python
df_train[['card1', 'card2', 'card3', 'card5']].quantile([.1, .25, .5, .75, .9, .99])
```
Example 2： 箱线图

```python
sns.boxplot(x='DT_M', y='D2', hue='isFraud', data=train_df, palette='hls')
```

![eda_fig4](https://gitee.com/alston972/MarkDownPhotos/raw/master/2019-10-05-post/eda_fig4.png)
#### 3.2.5 缺失值  
数值型特征一般用均值，加权均值，中位数填充；分类特征一般可用众数填充。  
在本次比赛中，lightgbm自身可对缺失值较好处理，故仅对card2-card6进行众数填充，用于后面构造虚拟身份特征。

#### 3.2.6 冗余特征
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

#### 3.2.7 其它 

```python
plt.style.use('ggplot')
pd.set_option('display.max_columns', 12345)
color_pal = [x['color'] for x in plt.rcParams['axes.prop_cycle']]
```

DataFrame.info(), DataFrame.describe(), DataFrame.shape(), DataFrame.head(), DataFrame.groupby(), DataFrame[feature].isnull(), 
DataFrame.merge(), DataFrame.concat(), DataFrame.sort_values(), DataFrame.rename(columns={"":""}), DataFrame.to_dict(),
DataFrame.to_pickle(), pd.read_pickle()等。


### 3.3 构造特征  
#### 3.3.1 一些常用函数  
1. Count encoding  
```python
def frequency_encoding(train_df, test_df, columns, self_encoding=False):
    for col in columns:
        temp_df = pd.concat([train_df[[col]], test_df[[col]]])
        fq_encode = temp_df[col].value_counts(dropna=False).to_dict()
        if self_encoding:
            train_df[col] = train_df[col].map(fq_encode)
            test_df[col]  = test_df[col].map(fq_encode)            
        else:
            train_df[col+'_fq_enc'] = train_df[col].map(fq_encode)
            test_df[col+'_fq_enc']  = test_df[col].map(fq_encode)
    return train_df, test_df
```
2. 时间段内频率特征  
```python
def timeblock_frequency_encoding(train_df, test_df, periods, columns, 
                                 with_proportions=True, only_proportions=False):
    for period in periods:
        for col in columns:
            new_col = col +'_'+ period
            train_df[new_col] = train_df[col].astype(str)+'_'+train_df[period].astype(str)
            test_df[new_col]  = test_df[col].astype(str)+'_'+test_df[period].astype(str)

            temp_df = pd.concat([train_df[[new_col]], test_df[[new_col]]])
            fq_encode = temp_df[new_col].value_counts().to_dict()

            train_df[new_col] = train_df[new_col].map(fq_encode)
            test_df[new_col]  = test_df[new_col].map(fq_encode)
            
            if only_proportions:
                train_df[new_col] = train_df[new_col]/train_df[period+'_total']
                test_df[new_col]  = test_df[new_col]/test_df[period+'_total']

            if with_proportions:
                train_df[new_col+'_proportions'] = train_df[new_col]/train_df[period+'_total']
                test_df[new_col+'_proportions']  = test_df[new_col]/test_df[period+'_total']

    return train_df, test_df
```
3. 特征缩放 
```python
def values_normalization(dt_df, periods, columns):
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

            dt_df[new_col+'_min_max'] = (dt_df[col]-dt_df['temp_min'])/(dt_df['temp_max']-dt_df['temp_min'])
            dt_df[new_col+'_std_score'] = (dt_df[col]-dt_df['temp_mean'])/(dt_df['temp_std'])
            del dt_df['temp_min'],dt_df['temp_max'],dt_df['temp_mean'],dt_df['temp_std']
    return dt_df
```
4. 虚拟身份相关特征  
```python
def uid_aggregation(train_df, test_df, main_columns, uids, aggregations):
    for main_column in main_columns:  
        for col in uids:
            for agg_type in aggregations:
                new_col_name = col+'_'+main_column+'_'+agg_type
                temp_df = pd.concat([train_df[[col, main_column]], test_df[[col,main_column]]])
                temp_df = temp_df.groupby([col])[main_column].agg([agg_type]).reset_index().rename(
                                                        columns={agg_type: new_col_name})

                temp_df.index = list(temp_df[col])
                temp_df = temp_df[new_col_name].to_dict()   

                train_df[new_col_name] = train_df[col].map(temp_df)
                test_df[new_col_name]  = test_df[col].map(temp_df)
    return train_df, test_df
```
5. 本次测试函数   
```python 
def make_test_predictions(tr_df, tt_df, target, lgb_params, NFOLDS=3):
    features_columns = [col for col in tt_df.columns if col not in remove_features]
        
    folds = KFold(n_splits=NFOLDS)   
    under_sample_data = tr_df.iloc[under_sample_indices,:]
    tr_df = under_sample_data
    
    X,y = tr_df[features_columns], tr_df[target]    
    P,P_y = tt_df[features_columns], tt_df[target]  
    feature_importances_lgb = pd.DataFrame()
    feature_importances_lgb['feature'] = X.columns
    
    for col in list(X):
        if X[col].dtype=='O':
            X[col] = X[col].fillna('unseen_before_label')
            P[col] = P[col].fillna('unseen_before_label')

            X[col] = train_df[col].astype(str)
            P[col] = test_df[col].astype(str)

            le = LabelEncoder()
            le.fit(list(X[col])+list(P[col]))
            X[col] = le.transform(X[col])
            P[col]  = le.transform(P[col])

            X[col] = X[col].astype('category')
            P[col] = P[col].astype('category')
        
    tt_df = tt_df[['TransactionID',target]]    
    predictions = np.zeros(len(tt_df))
    tr_data = lgb.Dataset(X, label=y)
    vl_data = lgb.Dataset(P, label=P_y) 
    estimator = lgb.train(
            lgb_params,
            tr_data,
            valid_sets = [tr_data, vl_data],
            verbose_eval = 200,
        )   

    pp_p = estimator.predict(P)
    predictions += pp_p/NFOLDS
    tt_df['prediction'] = predictions
    if LOCAL_TEST:
        feature_imp = pd.DataFrame(sorted(
            zip(estimator.feature_importance(), X.columns),reverse=True), columns=['Value', 'Feature'])
        print(feature_imp)
    if LOCAL_TEST:
        print('Holdout AUC:', metrics.roc_auc_score(tt_df[TARGET], tt_df['prediction']))
    return tt_df
```
#### 3.3.2 添加时间特征  
本次比赛属于时间序列数据，可从TransactionDT构造时间特征，TransactionDT为从某一起始日期的时间戳。
比赛社区经检验认为起始日期为2017-11-30。    
```python
START_DATE = datetime.datetime.strptime('2017-11-30', '%Y-%m-%d')
from pandas.tseries.holiday import USFederalHolidayCalendar as calendar
dates_range = pd.date_range(start='2017-10-01', end='2019-01-01')
us_holidays = calendar().holidays(start=dates_range.min(), end=dates_range.max())
```
```python
for df in [train_df, test_df]:
    # Temporary
    df['DT'] = df['TransactionDT'].apply(lambda x: (START_DATE + datetime.timedelta(seconds = x)))
    df['DT_M'] = ((df['DT'].dt.year-2017)*12 + df['DT'].dt.month).astype(np.int8)
    df['DT_W'] = ((df['DT'].dt.year-2017)*52 + df['DT'].dt.weekofyear).astype(np.int8)
    df['DT_D'] = ((df['DT'].dt.year-2017)*365 + df['DT'].dt.dayofyear).astype(np.int16)
    
    df['DT_hour'] = (df['DT'].dt.hour).astype(np.int8)
    df['DT_day_week'] = (df['DT'].dt.dayofweek).astype(np.int8)
    df['DT_day_month'] = (df['DT'].dt.day).astype(np.int8)
    
    # possible solo feature
    df['is_december'] = df['DT'].dt.month
    df['is_december'] = (df['is_december']==12).astype(np.int8)
    
    # holidays
    df['is_holiday'] = (df['DT'].dt.date.astype('datetime64').isin(us_holidays)).astype(np.int8)
    
    # add features into remove features
    remove_features += ['DT','DT_M','DT_W','DT_D','DT_hour','DT_day_week','DT_day_month']
    
    # D9 column
    df['D9'] = np.where(df['D9'].isna(),0,1)
```
```python  
    # total transaction pre timeblocks
for col in ['DT_M','DT_W','DT_D']:
    temp_df = pd.concat([train_df[[col]], test_df[[col]]])
    fq_encode = temp_df[col].value_counts().to_dict()

    train_df[col+'_total'] = train_df[col].map(fq_encode)
    test_df[col+'_total']  = test_df[col].map(fq_encode)

    # We can't use it as solo feature
    remove_features.append(col+'_total')
```

#### 3.3.3 添加虚拟身份
比赛数据没有给出每笔交易对应的交易人员身份，需要自己构造虚拟身份。  
从card1-card6, addr1 & addr2, P_emaildomain & R_emaildomain创建身份。  
为了增强模型泛化能力，同样的身份应该同时出现在训练集和测试集中，对不满足的身份去噪：
```python
i_cols = ['card1']

for col in i_cols: 
    valid_card = pd.concat([train_df[[col]], test_df[[col]]])
    valid_card = valid_card[col].value_counts()
    valid_card_std = valid_card.values.std()
    valid_card = valid_card[valid_card>2]
    valid_card = list(valid_card.index)

    train_df[col] = np.where(train_df[col].isin(test_df[col]), train_df[col], np.nan)
    test_df[col]  = np.where(test_df[col].isin(train_df[col]), test_df[col], np.nan)

    train_df[col] = np.where(train_df[col].isin(valid_card), train_df[col], np.nan)
    test_df[col]  = np.where(test_df[col].isin(valid_card), test_df[col], np.nan)

for col in ['card2','card3','card4','card5','card6',]: 
    train_df[col] = np.where(train_df[col].isin(test_df[col]), train_df[col], np.nan)
    test_df[col]  = np.where(test_df[col].isin(train_df[col]), test_df[col], np.nan)
```
因为card1无缺失值，card2-card6有缺失值，对card2-card6进行众数填充：   
```python
i_cols = ['TransactionID','card1','card2','card3','card4','card5','card6']

full_df = pd.concat([train_df[i_cols], test_df[i_cols]])

## I've used frequency encoding before so we have ints here
## we will drop very rare cards
full_df['card6'] = np.where(full_df['card6']==30, np.nan, full_df['card6'])
full_df['card6'] = np.where(full_df['card6']==16, np.nan, full_df['card6'])

i_cols = ['card2','card3','card4','card5','card6']

for col in i_cols:
    temp_df = full_df.groupby(['card1',col])[col].agg(['count']).reset_index()
    temp_df = temp_df.sort_values(by=['card1','count'], ascending=False).reset_index(drop=True)
    del temp_df['count']
    temp_df = temp_df.drop_duplicates(keep='first').reset_index(drop=True)
    temp_df.index = temp_df['card1'].values
    temp_df = temp_df[col].to_dict()
    full_df[col] = np.where(full_df[col].isna(), full_df['card1'].map(temp_df), full_df[col])
    
    
i_cols = ['card1','card2','card3','card4','card5','card6']
for col in i_cols:
    train_df[col] = full_df[full_df['TransactionID'].isin(train_df['TransactionID'])][col].values
    test_df[col] = full_df[full_df['TransactionID'].isin(test_df['TransactionID'])][col].values
```


构造虚拟身份：   
```python
train_df['uid'] = train_df['card1'].astype(str)+'_'+train_df['card2'].astype(str)
test_df['uid'] = test_df['card1'].astype(str)+'_'+test_df['card2'].astype(str)

train_df['uid2'] = train_df['uid'].astype(str)+'_'+train_df['card3'].astype(str)+'_'+train_df['card4'].astype(str)
test_df['uid2'] = test_df['uid'].astype(str)+'_'+test_df['card3'].astype(str)+'_'+test_df['card4'].astype(str)

train_df['uid3'] = train_df['uid2'].astype(str)+'_'+train_df['addr1'].astype(str)+'_'+train_df['addr2'].astype(str)
test_df['uid3'] = test_df['uid2'].astype(str)+'_'+test_df['addr1'].astype(str)+'_'+test_df['addr2'].astype(str)

train_df['uid4'] = train_df['uid3'].astype(str)+'_'+train_df['P_emaildomain'].astype(str)
test_df['uid4'] = test_df['uid3'].astype(str)+'_'+test_df['P_emaildomain'].astype(str)

train_df['uid5'] = train_df['uid3'].astype(str)+'_'+train_df['R_emaildomain'].astype(str)
test_df['uid5'] = test_df['uid3'].astype(str)+'_'+test_df['R_emaildomain'].astype(str)

```

#### 3.3.4 与时间段内平均交易时间和交易峰值的差距   
突出反常交易时间   
```python
for df in [train_df, test_df]:
    df['bank_type'] = df['card3'].astype(str) +'_'+ df['card5'].astype(str)
remove_features.append('bank_type') 

encoding_mean = {
    1: ['DT_D','DT_hour','_hour_dist','DT_hour_mean'],
    2: ['DT_W','DT_day_week','_week_day_dist','DT_day_week_mean'],
    3: ['DT_M','DT_day_month','_month_day_dist','DT_day_month_mean'],
    }

encoding_best = {
    1: ['DT_D','DT_hour','_hour_dist_best','DT_hour_best'],
    2: ['DT_W','DT_day_week','_week_day_dist_best','DT_day_week_best'],
    3: ['DT_M','DT_day_month','_month_day_dist_best','DT_day_month_best'],   
    }

for col in ['card3','card5','bank_type']:
    for df in [train_df, test_df]:
        for encode in encoding_mean:
            encode = encoding_mean[encode].copy()
            new_col = col + '_' + encode[0] + encode[2]
            df[new_col] = df[col].astype(str) +'_'+ df[encode[0]].astype(str)

            temp_dict = df.groupby([new_col])[encode[1]].agg(['mean']).reset_index().rename(
                                                                    columns={'mean': encode[3]})
            temp_dict.index = temp_dict[new_col].values
            temp_dict = temp_dict[encode[3]].to_dict()
            df[new_col] = df[encode[1]] - df[new_col].map(temp_dict)

        for encode in encoding_best:
            encode = encoding_best[encode].copy()
            new_col = col + '_' + encode[0] + encode[2]
            df[new_col] = df[col].astype(str) +'_'+ df[encode[0]].astype(str)
            temp_dict = df.groupby([col,encode[0],encode[1]])[encode[1]].agg(['count']).reset_index().rename(
                                                                    columns={'count': encode[3]})

            temp_dict.sort_values(by=[col,encode[0],encode[3]], inplace=True)
            temp_dict = temp_dict.drop_duplicates(subset=[col,encode[0]], keep='last')
            temp_dict[new_col] = temp_dict[col].astype(str) +'_'+ temp_dict[encode[0]].astype(str)
            temp_dict.index = temp_dict[new_col].values
            temp_dict = temp_dict[encode[1]].to_dict()
            df[new_col] = df[encode[1]] - df[new_col].map(temp_dict)
```


#### 3.3.4 时间段内频率   

```python  
new_columns = ['uid','uid2','uid3','uid4','uid5']
periods = ['DT_M', 'DT_W', 'DT_D']
i_cols = ['card1','card2','card3','card5'] + new_columns
train_df, test_df = timeblock_frequency_encoding(train_df, test_df, periods, new_columns)


i_cols = ['bank_type']
periods = ['DT_M','DT_W','DT_D']
train_df, test_df = timeblock_frequency_encoding(train_df, test_df, periods, i_cols, 
                                 with_proportions=False, only_proportions=True)
```

#### 3.3.5 频率编码   
```python  
new_columns = ['uid','uid2','uid3','uid4','uid5']
i_cols = ['card1','card2','card3','card5'] + new_columns
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=False)  

i_cols = ['D'+str(i) for i in range(1,16)]
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=True)


i_cols = ['C'+str(i) for i in range(1,15)]
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=False)
```

#### 3.3.6 D1-D15 timedelta特征    
最重要！为距离上次交易时间差， 距离发卡时间等。  
```python
i_cols = ['D'+str(i) for i in range(1,16)]
uids = ['uid','uid2','uid3','uid4','uid5','bank_type']
aggregations = ['mean','std']

train_df, test_df = uid_aggregation(train_df, test_df, i_cols, uids, aggregations)
```

```python
for df in [train_df, test_df]:

    for col in i_cols:
        df[col] = df[col].clip(0) 
    
    # Lets transform D8 and D9 column
    # As we almost sure it has connection with hours
    df['D9_not_na'] = np.where(df['D9'].isna(),0,1)
    df['D8_not_same_day'] = np.where(df['D8']>=1,1,0)
    df['D8_D9_decimal_dist'] = df['D8'].fillna(0)-df['D8'].fillna(0).astype(int)
    df['D8_D9_decimal_dist'] = ((df['D8_D9_decimal_dist']-df['D9'])**2)**0.5
    df['D8'] = df['D8'].fillna(-1).astype(int)
```

```python 
i_cols.remove('D1')
i_cols.remove('D2')
i_cols.remove('D9')
periods = ['DT_D','DT_W','DT_M']
for df in [train_df, test_df]:
    df = values_normalization(df, periods, i_cols)
```


#### 3.3.7 C1-C14 特征  

```python
i_cols = ['C'+str(i) for i in range(1,15)] 
for df in [train_df, test_df]:
    for col in i_cols:
        max_value = train_df[train_df['DT_M']==train_df['DT_M'].max()][col].max()
        df[col] = df[col].clip(None,max_value) 
```  


#### 3.3.8 id01-id38特征    
表示交易对应id特征，包括网络连接类型，mobile/desktop, 手机品牌型号，浏览器厂商版本，屏幕长宽等。  
Expansion encoding here.  
```python
for df in [train_identity, test_identity]:
    ########################### Device info
    df['DeviceInfo'] = df['DeviceInfo'].fillna('unknown_device').str.lower()
    df['DeviceInfo_device'] = df['DeviceInfo'].apply(lambda x: ''.join([i for i in x if i.isalpha()]))
    df['DeviceInfo_version'] = df['DeviceInfo'].apply(lambda x: ''.join([i for i in x if i.isnumeric()]))
    
    ########################### Device info 2
    df['id_30'] = df['id_30'].fillna('unknown_device').str.lower()
    df['id_30_device'] = df['id_30'].apply(lambda x: ''.join([i for i in x if i.isalpha()]))
    df['id_30_version'] = df['id_30'].apply(lambda x: ''.join([i for i in x if i.isnumeric()]))
    
    ########################### Browser
    df['id_31'] = df['id_31'].fillna('unknown_device').str.lower()
    df['id_31_device'] = df['id_31'].apply(lambda x: ''.join([i for i in x if i.isalpha()]))


i_cols = [
          'DeviceInfo','DeviceInfo_device','DeviceInfo_version',
          'id_30','id_30_device','id_30_version',
          'id_31','id_31_device',
          'id_33',
         ]
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=True)
```   

#### 3.3.9 target encoding特征  
```python
for col in ['ProductCD','M4']:
    new_col_name = col + '_target_mean'
    temp_dict = train_df.groupby([col])[TARGET].agg(['mean']).reset_index().rename(
                                                        columns={'mean': new_col_name})
    temp_dict.index = temp_dict[col].values
    temp_dict = temp_dict[col+'_target_mean'].to_dict()

    train_df[new_col_name] = train_df[col].map(temp_dict)
    test_df[new_col_name]  = test_df[col].map(temp_dict)


for col in ['card4', 'card6']:
    new_col_name = col + '_target_mean'
    temp_dict = train_df.groupby([col])[TARGET].agg(['mean']).reset_index().rename(
                                                        columns={'mean': new_col_name})
    temp_dict.index = temp_dict[col].values
    temp_dict = temp_dict[col+'_target_mean'].to_dict()

    train_df[new_col_name] = train_df[col].map(temp_dict)
    test_df[new_col_name]  = test_df[col].map(temp_dict)
```


#### 3.3.10 TransactionAmt   
log scaling here.  
先去除极端值   
```python   
train_df['TransactionAmt'] = train_df['TransactionAmt'].clip(0,5000)
test_df['TransactionAmt']  = test_df['TransactionAmt'].clip(0,5000)


i_cols = ['TransactionAmt']
uids = ['bank_type']
aggregations = ['mean','std']

train_df, test_df = uid_aggregation(train_df, test_df, i_cols, uids, aggregations)
periods = ['DT_D','DT_W','DT_M']
for df in [train_df, test_df]:
    df = values_normalization(df, periods, i_cols)


for dist in ['dist1', 'dist2']:
    train_df['product_'+dist] = train_df['ProductCD'].astype(str)+'_'+train_df[dist].astype(str)
    test_df['product_'+dist] = test_df['ProductCD'].astype(str)+'_'+test_df[dist].astype(str)
    
i_cols = ['product_dist1', 'product_dist2']
periods = ['DT_D','DT_W','DT_M']
train_df, test_df = timeblock_frequency_encoding(train_df, test_df, periods, i_cols, 
                                             with_proportions=False, only_proportions=True)
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=True)


train_df['TransactionAmt'] = np.log1p(train_df['TransactionAmt'])
test_df['TransactionAmt'] = np.log1p(test_df['TransactionAmt'])  
```  


#### 3.3.11 ProductCD   
```python 
train_df['product_type'] = train_df['ProductCD'].astype(str)+'_'+train_df['TransactionAmt'].astype(str)
test_df['product_type'] = test_df['ProductCD'].astype(str)+'_'+test_df['TransactionAmt'].astype(str)

i_cols = ['product_type']
periods = ['DT_D','DT_W','DT_M']
train_df, test_df = timeblock_frequency_encoding(train_df, test_df, periods, i_cols, 
                                                 with_proportions=False, only_proportions=True)
train_df, test_df = frequency_encoding(train_df, test_df, i_cols, self_encoding=True)
```

#### 3.3.12 Trend特征    
```python 
for uid in ['uid3']:
    for df in [train_df, test_df]:
        for period in ['DT_M']:
            new_col_name = uid + '_' + period + '_mean_TransactionAmt_dist'
            temp_df = df.groupby([uid, period])['TransactionAmt'].agg(['mean']).reset_index()
            temp_df[new_col_name] = temp_df[uid].astype(str)+'_'+(temp_df[period]).astype(str)
            temp_df.index = temp_df[new_col_name].values
            temp_df = temp_df['mean'].to_dict()
            df[new_col_name] = df[uid].astype(str)+'_'+(df[period] - 1).astype(str)
            df[new_col_name] = df['TransactionAmt'] - df[new_col_name].map(temp_df)
```

#### 3.3.12 Spatial特征   
```python
for uid in ['uid']:
    for dist in ['dist1']:
        new_col_name = uid + '_' + 'DT_D' + '_' + 'ProductCD' + '_' + dist
        temp_min = train_df.groupby([uid]+['ProductCD', 'DT_D'])[dist].agg(['min']).reset_index()
        temp_min[new_col_name] = temp_min[uid].astype(str) + '_' + temp_min['DT_D'].astype(str) + '_' + temp_min['ProductCD'].astype(str)
        temp_max = train_df.groupby([uid]+['ProductCD', 'DT_D'])[dist].agg(['max']).reset_index()
        temp_max[new_col_name] = temp_max[uid].astype(str) + '_' + temp_max['DT_D'].astype(str) + '_' + temp_max['ProductCD'].astype(str)
        temp = temp_min.merge(temp_max, on = new_col_name)
        temp['gap'] = temp['max'] - temp['min']
        del temp['min'], temp['max']
        temp.index = temp[new_col_name].values
        temp = temp['gap'].to_dict()
        
        train_df[new_col_name] = train_df[uid].astype(str) + '_' + train_df['DT_D'].astype(str) + '_' + train_df['ProductCD'].astype(str)
        test_df[new_col_name] = test_df[uid].astype(str) + '_' + test_df['DT_D'].astype(str) + '_' + test_df['ProductCD'].astype(str)
        train_df[new_col_name] = train_df[new_col_name].map(temp)
        test_df[new_col_name] = test_df[new_col_name].map(temp)


for df in [train_df, test_df]:
    for addr in ['addr1', 'addr2']:
        new_col_name = 'uid_DT_D_ProductCD_' + addr + '_std'
        temp_df = df.groupby(['uid', 'DT_D', 'ProductCD'])[addr].agg(['std']).reset_index()
        temp_df[new_col_name] = temp_df['uid'].astype(str)+'_'+temp_df['DT_D'].astype(str)+'_'+temp_df['ProductCD'].astype(str)
        temp_df.index = temp_df[new_col_name].values
        temp_df = temp_df['std'].to_dict()
        df[new_col_name] = df['uid'].astype(str)+'_'+df['DT_D'].astype(str)+'_'+df['ProductCD'].astype(str)
        df[new_col_name] = df[new_col_name].map(temp_df)


for df in [train_df, test_df]:
    for addr in ['addr1', 'addr2']:
        new_col_name = 'uid2_DT_D_ProductCD_' + addr + '_std'
        temp_df = df.groupby(['uid2', 'DT_D', 'ProductCD'])[addr].agg(['std']).reset_index()
        temp_df[new_col_name] = temp_df['uid2'].astype(str)+'_'+temp_df['DT_D'].astype(str)+'_'+temp_df['ProductCD'].astype(str)
        temp_df.index = temp_df[new_col_name].values
        temp_df = temp_df['std'].to_dict()
        df[new_col_name] = df['uid2'].astype(str)+'_'+df['DT_D'].astype(str)+'_'+df['ProductCD'].astype(str)
        df[new_col_name] = df[new_col_name].map(temp_df)
```


#### 3.3.13 去除噪声    
去除噪声特征，too specific.   
```python
remove_features = [
    'TransactionID', 'TransactionDT',  # These columns are pure noise right now
    TARGET,
]

remove_features += ['DT','DT_M','DT_W','DT_D','DT_hour','DT_day_week','DT_day_month']

new_columns = ['uid','uid2','uid3','uid4','uid5']
remove_features += new_columns
``` 
最终共有824个特征



## 4 特征选择   
特征选择减少特征数量，降维，降低过拟合，提高模型泛化能力。
### 4.1 逐个筛选 
1. 去除变化小，缺失值太多，极端分布特征
2. 线下验证   
3. 检验同分布scipy.stats.ks_2samp()
4. Pearson相关系数 scipy.stats.pearsonr()仅对线性关系敏感，如果两个变量高度非线性相关，Perarson相关性也接近0
5. 最大信息系数minepy.MINE.compute_score()
6. 基于学习模型的特征排序 
利用机器学习算法，如RandomForestRegressor针对某个特征和目标变量建立预测模型。

### 4.2 批量筛选    
1. sklearn.feature_selection.RFECV()
1. sklearn.model_selection.chi2()  
2. sklearn.feature_selection.SelectKBest()
3. 降维sklearn.decomposition.PCA()等
4. sklearn.feature_selection.f_regression()批量相关性特征选择  

## 5 交叉验证   
常用的交叉验证方式有sklearn.model_selection.KFold, sklearn.model_selection.TimeSeriesSplit, 
sklearn.model_selection.GroupKFold. 本次比赛中，由于数据为时间序列数据，且训练集测试集在时间上分离，所以
选择GroupKFold交叉验证。以月份为组切分标准。  
## 6 分类器   
lgb
```python
lgb_params = {
                    'objective':'binary',
                    'boosting_type':'gbdt',
                    'metric':'auc',
                    'n_jobs':-1,
                    'learning_rate':0.007,
                    'num_leaves': 2**8,
                    'max_depth':-1,
                    'tree_learner':'serial',
                    'colsample_bytree': 0.5,
                    'subsample_freq':1,
                    'subsample':0.7,
                    'n_estimators':10000,
                    'max_bin':255,
                    'verbose':-1,
                    'seed': SEED,
                    'early_stopping_rounds':100, 
                } 


def make_predictions(tr_df, tt_df, features_columns, target, lgb_params, NFOLDS=2):
    
    folds = GroupKFold(n_splits=NFOLDS)

    X,y = tr_df[features_columns], tr_df[target]    
    P,P_y = tt_df[features_columns], tt_df[target]  
    split_groups = tr_df['DT_M']

    tt_df = tt_df[['TransactionID',target]]    
    predictions = np.zeros(len(tt_df))
    oof = np.zeros(len(tr_df))
    
    for fold_, (trn_idx, val_idx) in enumerate(folds.split(X, y, groups=split_groups)):
        print('Fold:',fold_)
        tr_x, tr_y = X.iloc[trn_idx,:], y[trn_idx]
        vl_x, vl_y = X.iloc[val_idx,:], y[val_idx]
            
        print(len(tr_x),len(vl_x))
        tr_data = lgb.Dataset(tr_x, label=tr_y)
        vl_data = lgb.Dataset(vl_x, label=vl_y)  

        estimator = lgb.train(
            lgb_params,
            tr_data,
            valid_sets = [tr_data, vl_data],
            verbose_eval = 200,
        )   
        
        pp_p = estimator.predict(P)
        predictions += pp_p/NFOLDS
        
        feature_importance['fold_'+str(fold_+1)] = estimator.feature_importance()
        
        oof_preds = estimator.predict(vl_x)
        oof[val_idx] = (oof_preds - oof_preds.min())/(oof_preds.max() - oof_preds.min())
        
        del tr_x, tr_y, vl_x, vl_y, tr_data, vl_data
        gc.collect()
        
    tt_df['prediction'] = predictions
    print('OOF AUC:', roc_auc_score(y, oof))
    if LOCAL_TEST:
        print('Holdout AUC:', roc_auc_score(tt_df[TARGET], tt_df['prediction']))
    
    return tt_df
```

## 7 ensemble   
在比赛中尝试了stacknet，gmean，blending融合，显示blending效果最佳。 
本次比赛评价指标为auc，预测概率值大小对结果没影响，重要的是排序，但应注意blending时应先对每个结果minmax，防止参照
标准不一。 

## 8 技巧  
- 最好中间结果备份；
- pd.read_csv速度慢，转为pickle；
- 数据压缩；
- 线下测试很重要;
- 根据RandomForestRegressor或者lgb.feature_importance()特征重要性构造特征；
- 训练lgb时类别特征`astype('category')`;
- post processing;
- 比赛中多看大佬kernel与discussion;
- 比赛结束学习top team的分享。


## 9 收获与不足
- 数据挖掘入门；
- 熟练使用pandas,seaborn，sklearn, plt；
- 初步上手lightgbm, xgboost；
- 对构造特征不够熟练；
- 没有post processing;
- 没有读相关paper;
- 针对这次比赛典型的不平衡学习问题，尝试了随机过采样方法但反而导致过拟合，或许有更好的过采样方法。


## 10 What to do next 
- catboost；
- 对业务的理解；
- 图像比赛? 
- 对机器学习算法的深入理解；
- 更大的实践项目。