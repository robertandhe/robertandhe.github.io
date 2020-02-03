---
layout: post
title: "数据挖掘比赛特征工程技巧"
subtitle: 'data mining feature engineering'
date:       2019-10-10 12:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - competition
---

# <center>Feature Engineering</center>
>More data beats clever algorithms, but better data betas more data - by Peter Norving     

## 1. 类别特征
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

## 2. 数值特征
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
7. Row statistics   
   Create statistics on a row of data.   
   Number of NaN's, Number of 0's, Number of negative values, Mean, Max, Min, Skewness etc.   

## 3. 时间特征     
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

## 4. 空间特征  
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

## 5. Label Engineering   
Can treat a label/target/dependent varaibles as a feature of the data and vice versa.   
Log-transform: y->log(y+1)|exp(y_pred) - 1
Square-transform   
Box-Cox transform   
Create a score, to turn binary target in regression.   
Train regressor to predict a feature not available in test set.


# <center>techniques from discussion</center>
1. NAN processing  
[link](https://www.kaggle.com/c/ieee-fraud-detection/discussion/108575#latest-641841)  
If you give np.nan to LGBM, then at each tree node split, it will split the non-NAN values and then send all the NANs to either the left child or right child depending on what’s best. Therefore NANs get special treatment at every node and can become overfit. By simply converting all NAN to a negative number lower than all non-NAN values (such as - 999),
```python
df[col].fillna(-999, inplace=True)
```
then LGBM will no longer overprocess NAN
2. 特征选择  
[link](https://www.kaggle.com/c/ieee-fraud-detection/discussion/111308)
- forward feature selection (using single or groups of features)
- recursive feature elimination (using single or groups of features)
- permutation importance[link](https://www.kaggle.com/paultimothymooney/feature-selection-with-permutation-importance)
- adversarial validation
- correlation analysis
- time consistency
- client consistency
- train/test distribution analysis
  

