---
layout: post
title: "logistic回归算法代码实现"
subtitle: ''
date:     2020-02-04 12:48:00
author: "mudux"
music: false
music-id: 187937
mathjax: true
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - machine learning
---

&emsp;[上个博客](https://hexinlin.top/2020/02/03/logistic/)详细介绍了logistic回归理论知识，这里讲解如何实现logistic回归。

## 1. CPP实现

### 1.1 logistic.h文件
```c++
#include <iostream>
#include <vector>
#include <cmath>

class LogisticRegression{
public:
    LogisticRegression(double learningRate, int iteration);

    ~LogisticRegression();

    std::vector<double> sigmoid(std::vector<double>& input);

    double sigmoid(double);
    
    std::vector<double> fit(std::vector<std::vector<double>>& x, std::vector<int>& y);

    std::vector<double> batchGradDescent(std::vector<std::vector<double>>& x, std::vector<int>& y);

    std::vector<int> predict(std::vector<std::vector<double>>& testData);

    std::vector<std::vector<double>> predict_proba(std::vector<std::vector<double>>& x);

    double score(std::vector<std::vector<double>>& x, std::vector<int>& y);

    std::vector<double> get_weights();
 
private:
    std::vector<double> weights;
    double learningRate;
    int iteration;
};


LogisticRegression::LogisticRegression(double learningRate=0.001, int iteration=500)
{
    this->learningRate = learningRate;
    this->iteration = iteration;
}

LogisticRegression::~LogisticRegression()
{
    
}

std::vector<double> LogisticRegression::sigmoid(std::vector<double>& input)
{
    int m = input.size();
    std::vector<double> res(m, 0.0);
    for (int i = 0; i < m; ++i)
        res[i] = 1.0 / (1 + exp(-input[i]));
    return res;
}


double LogisticRegression::sigmoid(double in)
{
    return 1.0 / (1 + exp(-in));
}

std::vector<double> LogisticRegression::fit(std::vector<std::vector<double>>& x, std::vector<int>& y)
{
    std::vector<double> weights = batchGradDescent(x, y);
    return weights;
}

std::vector<double> LogisticRegression::batchGradDescent(std::vector<std::vector<double>>& x, std::vector<int>& y)
{
    int m = x.size(), n = x[0].size();
    weights = std::vector<double>(n, 1.0);
    for (int ite = 0; ite < iteration; ++ite)
    {
        std::vector<double> t(m, 0.0);
        for (int i = 0; i < m; ++i)
        {
            double sum = 0.0;
            for (int j = 0; j < n; ++j)
                sum += x[i][j] * weights[j];
            t[i] = sum;
        }
        std::vector<double> h = sigmoid(t);
        std::vector<double> err(m, 0.0);
        for (int i = 0; i < m; ++i)
            err[i] = h[i] - y[i];
        for (int i = 0; i < n; ++i)
        {
            double sum = 0.0;
            for (int j = 0; j < m; ++j)
                sum += x[j][i] * err[j];
            weights[i] = weights[i] - learningRate * sum;
        }
    }
    return weights;
}

std::vector<int> LogisticRegression::predict(std::vector<std::vector<double>>& testData)
{
    int m = testData.size(), n = testData[0].size();
    if (n != weights.size()){
        std::cerr << "Fatal error! unequal length of test data and weights" << std::endl;
        exit(1);
    }
    std::vector<int> ans(m, 0);
    for (int i = 0; i < m; ++i)
    {
        double sum = 0.0;
    
        for (int j = 0; j < n; ++j)
            sum += testData[i][j] * weights[j];
        // sum = sigmoid(sum);
        if (sum >= 0.0)
            ans[i] = 1;
        else
            ans[i] = 0;
    }
    return ans;
}

std::vector<std::vector<double>> LogisticRegression::predict_proba(std::vector<std::vector<double>>& x)
{
    int m = x.size(), n = x[0].size();
    if (n != weights.size()){
        std::cerr << "Fatal error! unequal length of test data and weights" << std::endl;
        exit(1);
    }
    std::vector<double> t(m, 0);
    for (int i = 0; i < m; ++i)
    {
        double sum = 0.0;
        for (int j = 0; j < n; ++j)
            sum += x[i][j] * weights[j];
        t[i] = sum;
    }
    std::vector<std::vector<double>> ans(m, std::vector<double>(2, 0.0));
    for (int i = 0; i < m; ++i)        
    {
        ans[i][0] = 1.0 - t[i];
        ans[i][1] = t[i];
    }
    return ans;
}


double LogisticRegression::score(std::vector<std::vector<double>>& x, std::vector<int>& y)
{
    std::vector<int> pred = predict(x);
    int cnt = 0, m = x.size();
    for (int i = 0; i < m; ++i)
        if (pred[i] == y[i])
            ++cnt;
    return cnt / (double)m;
}

std::vector<double> LogisticRegression::get_weights()
{
    return weights;
}
```

### 1.2 测试一下
```c++
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
#include <iterator>
#include "logistic.h"

using namespace std;

int main()
{
    ifstream in(".\\horseColicTraining.txt");
    if (!in.is_open()){
        cerr << "fatal error！ unable to open file" << endl;
        return 0;
    }
    vector<vector<double>> data;
    stringstream ins;
    string s;
    while (getline(in, s)){
        ins << s;
        vector<double> row;
        double t;
        while (ins >> t){
            row.push_back(t);
        }
        data.push_back(row);
        ins.clear();
    }
    in.close();
    vector<vector<double>> trainX;
    vector<int> trainY;
    for (int i = 0; i < data.size(); ++i)
    {
        trainY.push_back((int)data[i].back());
        trainX.push_back(data[i]);
        trainX[i].pop_back();
    }
    
    in.open(".\\horseColicTest.txt");
    if (!in.is_open()){
        cerr << "fatal error！ unable to open file" << endl;
        return 0;
    }
    vector<vector<double>> data2;
    while (getline(in, s)){
        ins << s;
        vector<double> row;
        double t;
        while (ins >> t){
            row.push_back(t);
        }
        data2.push_back(row);
        ins.clear();
    }
    in.close();
    vector<vector<double>> testX;
    vector<int> testY;
    for (int i = 0; i < data2.size(); ++i)
    {
        testY.push_back((int)data2[i].back());
        testX.push_back(data2[i]);
        testX[i].pop_back();
    }

    LogisticRegression lr = LogisticRegression();
    lr.fit(trainX, trainY);
    vector<int> pred = lr.predict(testX);
    cout << "Predict labels:" << endl;
    for (auto& a : pred)
        cout << a << " ";
    cout << endl;
    vector<vector<double>> prob = lr.predict_proba(testX);

    double acc = lr.score(testX, testY);
    cout << "Accuracy:" << acc << endl;

    vector<double> weights = lr.get_weights();
    cout << "Weights:" << endl;
    for (double& w : weights)
        cout << w << " ";
    cout << endl;
    return 0;
}  
```

输出为：

![cppTest](https://gitee.com/alston972/MarkDownPhotos/raw/master/2020-02-03/cppTest.PNG)

## 2. Python实现
### 2.1 logistic.py文件
```python
import numpy as np


class LogisticRegressionClassifier():
    def __init__(self, learingRate=0.001, iteration=500, method='stochastic'):
        self.learingRate = learingRate
        self.iteration = iteration
        self.method_ = 'stochastic'
        self.x = None
        self.y = None

    def sigmoid(self, inX):
        return 1.0 / (1 + np.exp(-inX))

    def fit(self, x, y):
        if self.method_ == 'stochastic':
            return self.stocGradDescent(x, y)
        if self.method_ == 'batch':
            return self.batchGradDescent(x, y)
            
    def batchGradDescent(self, x, y):
        _, n = np.shape(x)
        weights = np.ones((n, 1))
        for _ in range(self.iteration):
            h = self.sigmoid(np.dot(x, weights))
            error = h - y
            weights = weights - self.learingRate * np.dot(x.T, error)
        self.weights_ = weights
        return weights
    
    
    def stocGradDescent(self, x, y):
        m, n = np.shape(x)
        weights = np.ones((n, 1))
        for j in range(self.iteration):
            dataIndex = range(m)
            for i in range(m):
                alpha = 4 / (1.0 + j + i) + self.learingRate
                randIndex = np.random.randint(0, len(dataIndex))
                h = self.sigmoid(np.dot(x[randIndex, :], weights))
                error = h - y[randIndex, :]
                t = x[randIndex, ].T.reshape((-1, 1))
                weights = weights - alpha * error * t
        self.weights_ = weights
        return weights

    
    def predict(self, testData):
        self.check_is_fitted('weights_')
        prob = self.sigmoid(np.dot(testData, self.weights_))    
        m, n = np.shape(testData)
        pred = np.zeros((m, 1))
        for i in range(m):
            if prob[i] >= 0.5:
                pred[i] = 1
            else:
                pred[i] = 0
        return pred


    def predict_proba(self, testData):
        self.check_is_fitted('weights_')
        probPos = self.sigmoid(np.dot(testData, self.weights_)) 
        probNeg = 1 - probPos
        prob = np.concatenate([probNeg, probPos], axis=1)
        return prob
        
    def get_weights(self):
        self.check_is_fitted('weights_')
        return self.weights_

    def check_is_fitted(self, attributes, msg=None, all_or_any=all):
        if msg is None:
           msg = ("This %(name)s instance is not fitted yet. Call 'fit' with "
                   "appropriate arguments before using this method.")
        if not hasattr(self, 'fit'):
            raise TypeError("This is not an estimator instance.")
        if not isinstance(attributes, (list, tuple)):
            attributes = [attributes]
        if not all_or_any([hasattr(self, attr) for attr in attributes]):
            raise TypeError(msg % {'name': type(self).__name__})

    
    def score(self, testData, y):
        pred = self.predict(testData)
        return np.mean(pred == y)
```

### 2.2 测试一下
logisticTest.py文件
```python
import numpy as np
import pandas as pd
from logistic import LogisticRegressionClassifier

trainData = pd.read_csv('./logistic/horseColicTraining.txt', header=None, delimiter='\t', sep=' ')
trainData = trainData.values
trainX = trainData[:, :-1]
trainY = trainData[:, -1].reshape((-1, 1))

testData = pd.read_csv('./logistic/horseColictest.txt', header=None, delimiter='\t', sep=' ')
testData = testData.values
testX = testData[:, :-1]
testY = testData[:, -1].reshape((-1, 1))

lr = LogisticRegressionClassifier(0.001, 1000)
weights = lr.fit(trainX, trainY)
acc = lr.score(testX, testY)
print("Accuracy:", acc)
```

输出为：

![pythonTest](https://gitee.com/alston972/MarkDownPhotos/raw/master/2020-02-03/pythonTest.PNG)

## 3. sklearn库实现
skLogistic.py文件
```python
import pandas as pd
from sklearn.linear_model import LogisticRegression

trainData = pd.read_csv('./logistic/horseColicTraining.txt', header=None, delimiter='\t', sep=' ')
trainData = trainData.values
trainX = trainData[:, :-1]
trainY = trainData[:, -1].reshape((-1, 1))

testData = pd.read_csv('./logistic/horseColictest.txt', header=None, delimiter='\t', sep=' ')
testData = testData.values
testX = testData[:, :-1]
testY = testData[:, -1].reshape((-1, 1))

lr = LogisticRegression(C=0.001, max_iter=500)
lr.fit(trainX, trainY)
acc = lr.score(testX, testY)
print("Accuracy:", acc)
```
输出为：

![skLR](https://gitee.com/alston972/MarkDownPhotos/raw/master/2020-02-03/skLR.PNG)

&emsp;**可以看出自编CPP和PYTHON实现准确率一样，权重也差不多，符合预期，sklearn库实现上有少许差别，但结果基本差不多，符合预期。**

## 4. Reference
机器学习实战-第5章  
[sklearn-LR](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html#sklearn.linear_model.LogisticRegression)  