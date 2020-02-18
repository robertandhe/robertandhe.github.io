---
layout: post
title: "2019年终总结"
date:   2020-01-02 10:04:00
author: "mudux"
# header-style: text
header-img: "img/2019summary.jpg"
header-mask: 0.1
catalog: false
tags:
  - life
---

新的一年开始了，是时候写2019年年终总结了。
## 两篇文章
&emsp;投了两篇文章，一篇中文EI期刊已中，一篇EI会议已投，这个应该是稳中的，顺利达到毕业要求。
## 两个比赛
&emsp;参加了两个kaggle比赛。
&emsp;第一个是[IEEE-CIS Fraud Detection](https://www.kaggle.com/c/ieee-fraud-detection)，是关于信用卡交易欺诈检测的问题，最终排名是148/6381,Top 3%，银牌。  
&emsp;第二个是[ASHRAE - Great Energy Predictor III](https://www.kaggle.com/c/ashrae-energy-prediction)，这个是关于能源消耗预测的问题，最终排名是10/3594,Top 1%,金牌。
## 两个项目
&emsp;在实验室做了两个项目。  
&emsp;第一个项目是关于不平衡学习的问题。主要是实现了各不平衡学习算法，如SMOTE,Borderline-SMOTE,MWMOTE,ADASYN等，并和自己提出的LPGOS过采样算法做了比较。同时实现了不平衡学习迁移分类算法TrWSBoost，属于TrAdaboost的改良。这个项目是基于WxPython实现的UI界面。  
&emsp;第二个项目是关于效能评估的问题。利用Qt来实现，分为几大模块:
- 数据预处理，对无标签数据利用Kmeans, DBSCAN或者层次聚类算法实现自动标注，对标注后数据按照信息熵进行离散化。
- 指标优化，利用粗糙集算法进行属性约简。
- 综合评估，分为两大类，第一类是层次分析法，通过评判矩阵计算各属性权重，再进行预测。第二类是分类算法，实现了SVM，BP神经网络分类算法，同时能设定隐含层网络节点个数，载入网络权重等。
- 可信性分析，利用训练数据和已预测数据进行可信性分析，包括模型可信度，样本置信度和综合可信性。
- 报告生成，利用QAxObject实现了word报告自动生成。
- 数据库使用，具有数据库连接设置，数据存储查询等功能。

## 一项专利
提交了一个发明专利申请，等后续消息。

## 知识进阶
- 林轩田老师机器学习基石，机器学习技法已看完，受益良多，感谢台大的知识共享；
- 常用数据结构学习，leetcode刷题接近四百道；
- 推荐系统实战，暂时还没能用上；
- 流畅的Python，看了不到十章，不得不说是本好书，适合python进阶；
- 数据库学习，能用的地步。

## 2020愿景
- 找个好实习；
- 找个好工作；
- 身体健康；
- 流畅的python看完；
- 外出旅游一到两次，今年忙于各种事情；
- 比赛拿个奖金。

