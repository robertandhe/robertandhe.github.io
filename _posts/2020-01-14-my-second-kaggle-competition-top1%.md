---
layout: post
title: "My Second Kaggle Competition Top 1%"
subtitle: 'kaggle competition'
author: "mudux"
date: 2020-01-14
# header-style: text
catalog: true
header-img: "img/kaggleCompetition2.jpg"
header-mask: 0.1
tags:
  - kaggle
  - competition
  - machine learning
---

# <center>ASHRAE-Great Energy Predictor III</center>
> 写在前面：第二次参加kaggle比赛，比赛名次为10/3614。这次还是一个数据挖掘的比赛，相比于上个欺诈检测的比赛学到了东西就少了些，而且也没那么有趣，但还是有所收获。感谢给力的队友Clancy Lee。

## 1. 比赛介绍
### 1.1 比赛背景
ASHRAE协会组织的关于能源消耗预测的比赛。  
Founded in 1894, [ASHRAE](https://www.ashrae.org/) serves to advance the arts and sciences of heating, ventilation, air conditioning refrigeration and their allied fields. ASHRAE members represent building system design and industrial process professionals around the world. With over 54,000 members serving in 132 countries, ASHRAE supports research, standards writing, publishing and continuing education - shaping tomorrow’s built environment today.
### 1.2 评价标准
比赛评价标准为Root Mean Squared Logarithmic Error(RMSLE). 

$$
\begin{equation}
\varepsilon  = \sqrt {\frac{1}{n}\sum\limits_{i = 1}^n {\left( {\log \left( p_i + 1\right) - \log \left( a_i + 1\right)} \right)}^2}
\end{equation}
$$  

其中：  
$\varepsilon$表示RMSLE(score)；  
$n$表示测试集大小；  
$p_i$表示预测值；  
$a_i$表示实际值；  
$log(x)$表示对数。  


### 1.3 数据集介绍
train.csv:训练集数据，包括：
- building_id - Foreign key for the building metadata.
- meter - The meter id code. Read as {0: electricity, 1: chilledwater, 2: steam, 3: hotwater}. Not every building has all meter types.
- timestamp - When the measurement was taken
- meter_reading - The target variable. Energy consumption in kWh (or equivalent). Note that this is real data with measurement error, which we expect will impose a baseline level of modeling error. UPDATE: as discussed here, the site 0 electric meter readings are in kBTU

building_metadata.csv:关于各个建筑信息的文件。
- site_id - Foreign key for the weather files.
- building_id - Foreign key for training.csv
- primary_use - Indicator of the primary category of activities for the building based on EnergyStar property type definitions
- square_feet - Gross floor area of the building
- year_built - Year building was opened
- floor_count - Number of floors of the building

weather_[train/test].csv:天气数据
- site_id
- air_temperature - Degrees Celsius
- cloud_coverage - Portion of the sky covered in clouds, in oktas
- dew_temperature - Degrees Celsius，露温。
- precip_depth_1_hr - Millimeters
- sea_level_pressure - Millibar/hectopascals
- wind_direction - Compass direction (0-360)
- wind_speed - Meters per second

test.csv:测试数据
- row_id - Row id for your submission file
- building_id - Building id code
- meter - The meter id code
- timestamp - Timestamps for the test data period

sample_submission.csv

## 2. 数据预处理
&emsp;这个比赛特征工程我们做的不多，主要是在于数据预处理。
### 2.1数据压缩
```python
from pandas.api.types import is_datetime64_any_dtype as is_datetime
from pandas.api.types import is_categorical_dtype

def reduce_mem_usage(df, use_float16=False):
    """ iterate through all the columns of a dataframe and modify the data type
        to reduce memory usage.        
    """
    start_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage of dataframe is {:.2f} MB'.format(start_mem))
    
    for col in df.columns:
        if is_datetime(df[col]) or is_categorical_dtype(df[col]):
            # skip datetime type or categorical type
            continue
        col_type = df[col].dtype
        
        if col_type != object:
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
                if use_float16 and c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
        else:
            df[col] = df[col].astype('category')

    end_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage after optimization is: {:.2f} MB'.format(end_mem))
    print('Decreased by {:.1f}%'.format(100 * (start_mem - end_mem) / start_mem))
    
    return df
```
### 2.2 异常值
&emsp;主要做了几个工作：
- 移除site0异常值，因为site0数据为0的太多；
- 时长超过48小时的0读数，；
- 电力消耗为0的读数，可以不用水，不用蒸汽，但不用电现在基本不可能了；
- 对building 1099异常大的数，可能为错误读数，也可去除。
[参照这里](https://www.kaggle.com/purist1024/ashrae-simple-data-cleanup-lb-1-08-no-leaks)  

### 2.3 天气数据时间校正
&emsp;根据天气数据可发现许多数据每天的最高温不是出现在中午附近，明显有问题，后面发现应该是记录天气是是用的本地时间，而比赛举办方当作UTC时间给出了，所以应该进行校正，具体就是假设每天最高温出现在下午两点左右，根据最高温校正，[参照这里](https://www.kaggle.com/patrick0302/locate-cities-according-weather-temperature)。  

### 2.4 气温数据插值
&emsp;在这个比赛中温度数据肯定是非常重要的，但天气数据中有许多缺失值，需要进行插值，插值的原理就是天气不会剧烈变化，前后应该有连贯性。具体使用的函数就是
```python
weather_df = weather_df.groupby('site_id').apply(lambda  group: group.interpolate(limit=200, limit_direction='both'))
```
和
```python
def fill_weather_dataset(weather_df):
    
    # Find Missing Dates
    time_format = "%Y-%m-%d %H:%M:%S"
    start_date = datetime.datetime.strptime(weather_df['timestamp'].min(),time_format)
    end_date = datetime.datetime.strptime(weather_df['timestamp'].max(),time_format)
    total_hours = int(((end_date - start_date).total_seconds() + 3600) / 3600)
    hours_list = [(end_date - datetime.timedelta(hours=x)).strftime(time_format) for x in range(total_hours)]

    missing_hours = []
    for site_id in range(16):
        site_hours = np.array(weather_df[weather_df['site_id'] == site_id]['timestamp'])
        new_rows = pd.DataFrame(np.setdiff1d(hours_list,site_hours),columns=['timestamp'])
        new_rows['site_id'] = site_id
        weather_df = pd.concat([weather_df,new_rows])

        weather_df = weather_df.reset_index(drop=True) 
    weather_df['timestamp'] = pd.to_datetime(weather_df['timestamp'])
    weather_df = timestamp_align(weather_df)
    # Add new Features
    #     weather_df["datetime"] = pd.to_datetime(weather_df["timestamp"])
    weather_df["datetime"] = weather_df["timestamp"]
    weather_df["day"] = weather_df["datetime"].dt.day
    weather_df["week"] = weather_df["datetime"].dt.week
    weather_df["month"] = weather_df["datetime"].dt.month
    
    # Reset Index for Fast Update
    weather_df = weather_df.set_index(['site_id','day','month'])

    air_temperature_filler = pd.DataFrame(weather_df.groupby(['site_id','day','month'])['air_temperature'].mean(),columns=["air_temperature"])
    weather_df.update(air_temperature_filler,overwrite=False)

    # Step 1
    cloud_coverage_filler = weather_df.groupby(['site_id','day','month'])['cloud_coverage'].mean()
    # Step 2
    cloud_coverage_filler = pd.DataFrame(cloud_coverage_filler.fillna(method='ffill'),columns=["cloud_coverage"])

    weather_df.update(cloud_coverage_filler,overwrite=False)

    due_temperature_filler = pd.DataFrame(weather_df.groupby(['site_id','day','month'])['dew_temperature'].mean(),columns=["dew_temperature"])
    weather_df.update(due_temperature_filler,overwrite=False)

    # Step 1
    sea_level_filler = weather_df.groupby(['site_id','day','month'])['sea_level_pressure'].mean()
    # Step 2
    sea_level_filler = pd.DataFrame(sea_level_filler.fillna(method='ffill'),columns=['sea_level_pressure'])

    weather_df.update(sea_level_filler,overwrite=False)

    wind_direction_filler =  pd.DataFrame(weather_df.groupby(['site_id','day','month'])['wind_direction'].mean(),columns=['wind_direction'])
    weather_df.update(wind_direction_filler,overwrite=False)

    wind_speed_filler =  pd.DataFrame(weather_df.groupby(['site_id','day','month'])['wind_speed'].mean(),columns=['wind_speed'])
    weather_df.update(wind_speed_filler,overwrite=False)

    # Step 1
    precip_depth_filler = weather_df.groupby(['site_id','day','month'])['precip_depth_1_hr'].mean()
    # Step 2
    precip_depth_filler = pd.DataFrame(precip_depth_filler.fillna(method='ffill'),columns=['precip_depth_1_hr'])

    weather_df.update(precip_depth_filler,overwrite=False)
    
    weather_df = weather_df.reset_index()
    
    weather_df = aggregation_temperature(weather_df, 'site_id', ['dew_temperature', 'air_temperature'], ['month'], ['mean'])
    
    weather_df = weather_df.drop(['datetime','day','week','month'],axis=1)
    
    def get_meteorological_features(data):
        def calculate_rh(df):
            df['relative_humidity'] = 100 * (np.exp((17.625 * df['dew_temperature']) / (243.04 + df['dew_temperature'])) / np.exp((17.625 * df['air_temperature'])/(243.04 + df['air_temperature'])))
        def calculate_fl(df):
            flike_final = []
            flike = []
            # calculate Feels Like temperature
            for i in range(len(df)):
                at = df['air_temperature'][i]
                rh = df['relative_humidity'][i]
                ws = df['wind_speed'][i]
                flike.append(feels_like(at, rh, ws))
            for i in range(len(flike)):
                flike_final.append(flike[i].f)
            df['feels_like'] = flike_final
            del flike_final, flike, at, rh, ws
        calculate_rh(data)
        calculate_fl(data)
        return data

    weather_df = get_meteorological_features(weather_df)
    
    return weather_df
```
&emsp;这里使用了[meteocalc package](https://pypi.org/project/meteocalc/)进行天气特征工程。  

### 2.5 数据转换
&emsp;log1p转换，meter_reading和square_feet读数值都大，进行对数变换。
## 3. 特征工程
1. 天气聚合特征
```python
def aggregation_temperature(df, site, cols, periods, agg_types):
    for col in cols:
        for period in periods:
            for agg_type in agg_types:
                new_col_name = col+'_'+site+'_'+period+'_'+agg_type
                temp_df = df.groupby([site, period])[col].agg([agg_type]).reset_index().rename(columns={agg_type:new_col_name})
                correspond_id = site+'_'+period

                temp_df[correspond_id] = temp_df[site].astype(str) + '_' + temp_df[period].astype(str)
                df[correspond_id] = df[site].astype(str) + '_' + df[period].astype(str)
                
                temp_df.drop([site, period], axis=1, inplace=True)
                temp_df[new_col_name] = temp_df[new_col_name].astype(np.float16)
                temp_df = temp_df.set_index(correspond_id)
                temp_df = temp_df[new_col_name].to_dict()

                df[new_col_name] = df[correspond_id].map(temp_df).astype(np.float16)
                del df[correspond_id], temp_df
                gc.collect()
    return df
weather_df = aggregation_temperature(weather_df, 'site_id', ['dew_temperature', 'air_temperature'], ['month'], ['mean'])    
```

&emsp;在每个site内求气温和露温的月均值。
1. 天气滞后特征
```python
def add_lag_feature(weather_df, window=3):
    group_df = weather_df.groupby('site_id')
    cols = ['air_temperature', 'cloud_coverage', 'dew_temperature', 'precip_depth_1_hr', 'sea_level_pressure', 'wind_direction', 'wind_speed']
    rolled = group_df[cols].rolling(window=window, min_periods=0)
    lag_mean = rolled.mean().reset_index().astype(np.float16)
    lag_max = rolled.max().reset_index().astype(np.float16)
    lag_min = rolled.min().reset_index().astype(np.float16)
    lag_std = rolled.std().reset_index().astype(np.float16)
    for col in cols:
        weather_df[str(col) + '_mean_lag_' + str(window)] = lag_mean[col]
        weather_df[str(col) + '_max_lag_' + str(window)] = lag_max[col]
        weather_df[str(col) + '_min_lag_' + str(window)] = lag_min[col]
        weather_df[str(col) + '_std_lag_' + str(window)] = lag_std[col]
```
&emsp;我们的本地测试就只有这几个特征有效。

### 4.模型训练
&emsp;因为这个问题的特殊性，可以按照meter_type，site,KFold等训练模型，为了增加模型的多样性，我们分开训练自己的模型。我是按照meter_type来就是4个模型，Clancy_Lee按照GroupKFold，12，1，2月假设为一个季度，3，4，5为一个季度，6，7，8和9，10，11为另外两个季度。

### 5.模型融合
&emsp;因为这个比赛存在数据泄露，所以可以根据泄露数据本地测试各个模型的好坏。我们训练个4个结果，融合时进行了一个网格搜索，所有各个模型的最优权重。
```python
test_df['pred1'] = sample_submission1.meter_reading
test_df['pred2'] = sample_submission2.meter_reading
test_df['pred3'] = sample_submission3.meter_reading
test_df['pred4'] = sample_submission4.meter_reading
test_df['pred5'] = sample_submission5.meter_reading

leak_df['pred1_l1p'] = np.log1p(leak_df.pred1)
leak_df['pred2_l1p'] = np.log1p(leak_df.pred2)
leak_df['pred3_l1p'] = np.log1p(leak_df.pred3)
leak_df['pred4_l1p'] = np.log1p(leak_df.pred4)
leak_df['pred5_l1p'] = np.log1p(leak_df.pred5)
leak_df['meter_reading_l1p'] = np.log1p(leak_df.meter_reading)

all_combinations = list(np.linspace(0.18,0.23,6)) # 这里是融合5个模型的代码

l = [all_combinations, all_combinations, all_combinations, all_combinations,all_combinations]
all_l = list(itertools.product(*l)) + list(itertools.product(*reversed(l)))
filtered_combis = [l for l in all_l if l[0] + l[1] + l[2] + l[3] + l[4] > 0.98 and l[0] + l[1] + l[2] + l[3] + l[4]< 1.02]

best_combi = [] # of the form (i, score)
for i, combi in enumerate(filtered_combis):
    #print("Now at: " + str(i) + " out of " + str(len(filtered_combis))) # uncomment to view iterations
    if i % 1000 == 1:
        print(i / len(filtered_combis), "%")
    score1 = combi[0]
    score2 = combi[1]
    score3 = combi[2]
    score4 = combi[3]
    score5 = combi[4]
    v = score1 * leak_df['pred1'].values + score2 * leak_df['pred2'].values + score3 * leak_df['pred3'].values + + score4 * leak_df['pred4'].values + score5 * leak_df['pred5'].values
    vl1p = np.log1p(v)
    curr_score = np.sqrt(mean_squared_error(vl1p, leak_df.meter_reading_l1p))
    
    if best_combi:
        prev_score = best_combi[0][1]
        if curr_score < prev_score:
            best_combi[:] = []
            best_combi += [(i, curr_score)]
    else:
        best_combi += [(i, curr_score)]
            
score = best_combi[0][1]
print(score)

final_combi = filtered_combis[best_combi[0][0]]
```

## 6. 第一名做法
&emsp;[第一名队伍的方案](https://www.kaggle.com/c/ashrae-energy-prediction/discussion/124709)如下：
![scheme](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2020-01-14/1stSolution.PNG)
### 6.1 Processing
#### Remove anomalies
As others have noted, cleaning the data was very important in this competition. The assumption is that there are unpredictable and hence unlearnable anomalies in the data that, if trained on, degrade the quality of the predictions. We identified and filtered out three types of anomalies:
1. Long streaks of constant values
2. Large positive/negative spikes
3. Additional anomalies determined by visual inspection
  
We noticed that some of these anomalies were consistent across multiple buildings at a site. We validated potential anomalies using all buildings in a site--if an anomaly showed up at the same time at multiple buildings, we could be reasonably certain that this was indeed a true anomaly. This allowed us to remove anomalies that were not necessarily part of a long streak of constant values or a large spike.
#### Impute Missing Temperature Values
There were a lot of missing values in temperature metadata. We found that imputing the missing data using linear interpolation helped our models.
#### Local Time Zone Correlation
As noted in the competition forum, the timezone in the train/test data was different from the timezone in the weather metadata. We used the information in this [discussion post](https://www.kaggle.com/c/ashrae-energy-prediction/discussion/112841) to correct the timezones.
#### Target Transformations
Like most competitors, we started by predicting log1p(meter_reading). We also corrected the units for site 0 as per this [discussion post](https://www.kaggle.com/c/ashrae-energy-prediction/discussion/119261).
Near the end of the competition, we tried standardizing meter_reading by dividing by square_feet; i.e., we predicted log1p(meter_reading/square_feet). Isamu came up with the idea after reading this [discussion post](https://www.kaggle.com/c/ashrae-energy-prediction/discussion/122263) by Artyom Vorobyov. The models trained with the standardized target added diversity to our final ensemble and improved our score by about 0.002. If we had more time we would have liked to explore this idea further; for example, we could have tried to predict log1p(meter_reading)/square_feet or created features using the standardized targets.

### 6.2 Feature Engineering and Feature Selection
- Raw features from train/test, weather metadata, and building metadata
- Categorical interactions such as the concatenation of building_id and meter
- Time series features including holiday flags and time of day features
- Count (frequency) features
- Lag temperature features similar to those found in the public kernels
- Smoothed and 1st, 2nd-order differentiation temperature features using Savitzky-Golay filter (see the figure below)
- **Cyclic encoding of periodic features**; e.g., hour gets mapped to hour_x = cos(2*pi*hour/24) and hour_y = sin(2*pi*hour/24)
- **Bayesian target encoding** (see this [kernel](https://www.kaggle.com/mmotoki/hierarchical-bayesian-target-encoding))

### 6.3 Models
- 1 model per meter
- 1 model per site_id
- 1 model per (building_id, meter)

Our team tried different approaches to validation in this competition. Like other competitors, we tried K-Fold CV using consecutive months as the validation set. The following code shows one approach to getting validation months:
```python
import numpy as np
def get_validation_months(n):
  return [np.arange(i, i+n) % 12 + 1 for i in range(12)]

>>> get_validation_months(6)
[array([1, 2, 3, 4, 5, 6]),
 array([2, 3, 4, 5, 6, 7]),
 array([3, 4, 5, 6, 7, 8]),
 array([4, 5, 6, 7, 8, 9]),
 array([ 5,  6,  7,  8,  9, 10]),
 array([ 6,  7,  8,  9, 10, 11]),
 array([ 7,  8,  9, 10, 11, 12]),
 array([ 8,  9, 10, 11, 12,  1]),
 array([ 9, 10, 11, 12,  1,  2]),
 array([10, 11, 12,  1,  2,  3]),
 array([11, 12,  1,  2,  3,  4]),
 array([12,  1,  2,  3,  4,  5])]
```


## 7. 改进
- 模型还是不够丰富，可以融合Catboost,MLP,xgboost等；
- 特征工程太少，本地测试许多特征都没用，可能是数据清洗不到位。