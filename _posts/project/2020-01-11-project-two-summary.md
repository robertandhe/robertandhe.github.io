---
layout: post
title: "2019年12月项目2总结"
# subtitle: 'QT'
date:   2020-01-11 20:57:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: false
tags:
  - qt
  - technique
  - project
---
&emsp;最近这个月做了一个小项目，功能是用来做效能评估，即装备状态评估。界面如下：
![gui界面](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2020-01-11/gui.PNG)
主要包括四个模块：
- 数据预处理；功能包括数据标注，标注算法有3种，DBSCAN,KMeans和层次聚类法，聚类后PCA降到2维显示。为了第二个模块属性约简，这里要加数据离散化模块。
- 属性约简；应用粗糙集理论进行属性约简。
- 综合评估；其实就是分类算法，实现了层析分析法，SVM和BP神经网络。层析分析法导入专家评判数据，判断不一致性，找到最大特征值对应特征向量就是属性权重，即W参数，然后就是多元回归公式了。SVM没啥好说的，就是台大libsvm的c++代码改造。神经网络功能支持设定隐含层数目与每个隐含层结点个数和训练次数，也支持导入已经训练好的参数。训练完给出准确率，然后在预测集上做预测，这个结果要用于下个模块。
- 可信性分析：分别为模型可行性分析，样本置信度分析和综合可行性分析。模型可信性分析就是每个类别的可信性，样本置信度分析利用SVM求样本点到分类平面的距离，通过邻域内同类别和不同类别样本统计值算，综合可信性分析就是前面两个可信性的乘积。

还包括一些附加功能：
- word报告生成；设定好word模板，打好书签，利用书签位置插入文字，图片，表格等。
- 保存数据到数据库：程序初始化时连接到数据库，运行过程中自动保存。


## 自动生成word报告
&emsp;见[qt自动生成word报告](https://hexinlin.top/2020/01/11/qt-word-report/)

## mysql操作
&emsp;见[qt mysql 操作](https://hexinlin.top/2020/01/11/qt-mysql-operation/)

## neural network & SVM
&emsp;师兄的工作，不写了。

## 其余算法
没啥好写的

## 后续
- 考虑美化下界面；
- 后面应该还要加功能。

## reference
[qt命令行打包](https://blog.csdn.net/qq_41786131/article/details/89921488)   
[k-means code](https://github.com/marcoscastro/kmeans/blob/master/kmeans.cpp)  
[alglib package用于层次聚类](https://www.alglib.net/docs.php#refmanual)  
[alglib package doc](https://www.alglib.net/translator/man/manual.cpp.html#struct_ahcreport)   
[dbscan密度聚类](https://www.cnblogs.com/pinard/p/6208966.html)   
[dbscan github](https://github.com/bowbowbow/DBSCAN/blob/master/clustering.cpp)   
[pca github](https://github.com/liyanghua/Principal-component-analysis-/blob/master/pca.cpp)  
[TNT & JAMA package](https://math.nist.gov/tnt/download.html)  
[QT word文档操作实例](https://blog.csdn.net/qq_35192280/article/details/83021975)  
[C/C++连接MySQL数据库例子](https://blog.csdn.net/to_Baidu/article/details/53819829)  
[Qt5.4下连接Mysql,QSqlDatabase: QMYSQL driver not loaded but available](https://blog.csdn.net/tenlee/article/details/43614241)  
[Qt的Mysql数据库表操作](https://blog.csdn.net/fanyun_01/article/details/53958275) 
