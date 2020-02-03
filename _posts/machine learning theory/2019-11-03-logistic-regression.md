---
layout: post
title: "logistic regression理论总结"
subtitle: 'logistic regression'
date:       2019-11-03 19:12:00
author: "mudux"
music: false
music-id: 187937
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - machine learning
---

>logistic regression(LR)回归从名字上看是回归算法，但其实是一个分类算法。是机器学习中最常用的算法之一。LR回归用来估计事物发生的概率，如用户购买商品的概率，广告被点击的概率。


# 概率预测问题
&emsp;在线性判别模型如PLA下，我们直接预测测试样本是pass/fail, positive/negative，etc。假设现在想要预测事件发生的概率而不仅仅是发生
或者不发生。但是在训练模型时，我们只有hard binary classification的数据集，因为数据标记只有发生或不发生而不是发生/不发生的概率，没有办法
直接根据概率来建模。所以需要新的模型来解决概率预测模型。
# logistic回归模型
&emsp;在统计学中，用对数模型(logit model)求事件发生的概率。 事件的几率（odds）是$p=P\left({Y=1}\right)$和$p=P\left( {Y=0}\right)$的比值。假设对数几率是输入的线性函数，即

<!-- $$
\log \frac{P\left({Y = 1\left|X\right.}\right)}{1-P\left({Y=1\left|X\right.}\right)} = w^T x
$$ -->

$$
\begin{equation}
\log \frac{P\left({Y = 1\left|X\right.}\right)}{1-P\left({Y=1\left|X\right.}\right)} = w^T x
\end{equation}
$$

&emsp;由此可求出事件发生的概率为：

$$
\begin{equation}
P\left( {Y = 1\left| X \right.} \right) = \frac{\exp \left( {w^T x} \right)}{1 + \exp \left( {w^T x} \right)}
\end{equation}
$$

$$
\begin{equation}
P\left( {Y = 0\left| X \right.} \right) = \frac{1}{1 + \exp \left( {w^T x} \right)}
\end{equation}
$$

&emsp;转换可得：

$$
\begin{equation}
P\left( {Y = 1\left| X \right.} \right) = \frac{1}{1 + \exp \left( { - w^T x} \right)}
\end{equation}
$$

&emsp;上式即是sigmoid函数，sigmoid分布如下图所示：
![sigmoid](https://gitee.com/alston972/MarkDownPhotos/raw/master/2019-10-15/sigmoid.png){:height="50%" width="50%"}
&emsp;在下文中，均用$\theta$表示sigmoid分布，即
$$\theta \left( s \right) = \frac{1}{1 + {e^{ - s}}}$$
logistic regression模型即是用
$h\left( x \right) = \theta \left( {w^T}x \right) = \frac{1}{1 + \exp \left( { - {w^T}x} \right)}$
近似$P\left( {y = 1\left| x \right.} \right)$，事件发生的几率越大，$P\left({Y=1}\right)越大$。  


# 与线性回归关系
逻辑斯的回归（logistic regression）与线性回归（linear regression）都是一种广义线性模型（generalized linear model）。
LR回归假设因变量y服从伯努利分布，线性回归假设因变量y服从高斯分布。去除sigmoid映射函数，LR算法就是一个线性回归。


# 损失函数
&emsp;根据y标签是否是1/-1或者1/0,存在两种不同形式的损失函数。第一种是来自李航的《统计学习方法》，第二种是来自林轩田老师的机器学习基石。
### 第一种损失函数
&emsp;在这里设标签取值为1/0，设：$P\left( {y = 1\left| x \right.} \right) = \pi \left( x \right),&emsp;P\left( {y = 0\left| x \right.} \right) = 1 - \pi \left( x \right)$。设给定数据集$T={\left( {x_1},{y_1} \right),\left( {x_2},{y_2} \right),...,\left( {x_N},{y_N} \right)}$。用极大似然估计法估计模型参数，从而得到逻辑斯蒂回归模型。  
&emsp;似然函数为

$$
\begin{equation}
\prod\limits_{i = 1}^N {
{[{\pi \left( {x_i} \right)}]}^{y_i}
{\left[ {1 - \pi \left( {x_i} \right)} \right]}^{1 - {y_i}}
}
\end{equation}
$$

&emsp;对数似然函数为  

<!-- \begin{align} 公式无编号-->

$$
\begin{align}
L\left( w \right) &= \sum\limits_{i = 1}^N {\left[ {y_i}\log \pi \left( {x_i} \right) + \left( {1 - {y_i}} \right)\log \left( 1 - \pi \left( {x_i} \right) \right) \right]}\\
&= \sum\limits_{i = 1}^N {\left[ {y_i}\log \frac{\pi \left( {x_i} \right)}{1 - \pi \left( {x_i} \right)} + \log \left( {1 - \pi \left( {x_i} \right)} \right) \right]}\\
&= \sum\limits_{i = 1}^N {\left[ {y_i}\left( {w \cdot {x_i}} \right) - \log \left( {1 + \exp \left( {w^T {x_i}} \right)}\right)\right]}
\end{align}
$$

&emsp;似然函数的反即为损失函数

$$
\begin{equation}
E\left( w \right) = \sum\limits_{i = 1}^N {\left[ {\log \left( {1 + \exp \left( {w^T {x_i}} \right)} \right) - {y_i}\left( {w^T {x_i}} \right)} \right]}
\end{equation}
$$

### 第二种损失函数
&emsp;在这里设标签为1/-1。由logistic函数性质，$P\left( {y =  - 1\left| x \right.} \right) = 1 - P\left( {y = 1\left| x \right.} \right) = 1 - h\left( x \right) = h\left( { - x} \right)$。设给定数据集$T={\left( {x_1},{y_1} \right),\left( {x_2},{y_2} \right),...,\left( {x_N},{y_N} \right)}$。其中${y_1} = 1,{y_2} =  - 1,...,{y_N} =  - 1$。  
&emsp;似然函数为

$$
\begin{equation}
L\left( h \right) = P\left( {x_1} \right)h\left( {x_1} \right) \times P\left( {x_2} \right)\left( {1 - h\left( {x_2} \right)} \right) \times ... \times P\left( {x_N} \right)\left( {1 - h\left( {x_N} \right)} \right)
\end{equation}
$$

&emsp;在上式中，$P\left( {x_1} \right),P\left( {x_2} \right),...,P\left( {x_N} \right)$与参数$w$无关。似然函数可简化为  
<center>$$L\left( h \right) = h\left( {x_1} \right) \times \left( {1 - h\left( {x_2} \right)} \right) \times ... \times \left( {1 - h\left( {x_N} \right)} \right)$$</center>

&emsp;即

$$
\begin{equation}
L\left( h \right) = \prod\limits_{i = 1}^N {h\left( {y_i}{x_i} \right)}
\end{equation}
$$

&emsp;求解$w$最大化似然函数

$$
\begin{equation}
\mathop {\max }\limits_W likelihood\left( w \right) \propto \prod\limits_{i = 1}^N {\theta \left( {y_i}{w^T}{x_i} \right)}
\end{equation}
$$

&emsp;即

$$
\begin{equation}
\mathop {\max }\limits_w \log \prod\limits_{i = 1}^N {\theta \left( {y_i}{w^T}{x_i} \right)} 
\end{equation}
$$

&emsp;损失函数为似然函数的相反数

$$
\begin{align}
E\left( w \right) &= \sum\limits_{i = 1}^N { - \log \theta \left( {y_i}{w^T}{x_i} \right)}\\
&= \sum\limits_{i = 1}^N {\log \left( {1 + \exp \left( { - {y_i}{w^T}{x_i}} \right)} \right)} 
\end{align}
$$

&emsp;在上述公式中，公式(9) $err\left( {w,x,y} \right) = {\left[ {\log \left( {1 + \exp \left( {w^T {x_i}} \right)} \right) - {y_i}\left( {w^T {x_i}} \right)} \right]}$和公式(15) $err\left( {w,x,y} \right) = \log \left( {1 + \exp \left( { - yw^Tx} \right)} \right)$为两种形式的**交叉熵损失(cross-entropy loss)**。

### 两种损失函数分析
&emsp;在李航的书中，logistic损失函数为

$$
\begin{equation}
E_1\left( w \right) = {\log \left( {1 + \exp \left( {w^T{x_i}} \right)} \right) - {y_i}\left( {w^T{x_i}} \right)}
\end{equation}
$$

&emsp;在林轩田老师的课程中，logistic损失函数为

$$
\begin{equation}
E_2\left( w \right) = {\log \left( {1 + \exp \left( { - {y_i}{w^T}{x_i}} \right)} \right)} 
\end{equation}
$$

&emsp;当y为负，即$y_1=0$,$y_2=-1$时

$$
\begin{equation}
E_1\left( w \right) = {\log \left( {1 + \exp \left( {w^T}{x_i} \right)} \right)} 
\end{equation}
$$

$$
E_2\left( w \right) = {\log \left( {1 + \exp \left( {w^T}{x_i} \right)} \right)} 
$$

&emsp;当y为正，即$y_1=1$,$y_2=1$时

$$
\begin{equation}
E_1\left( w \right) = {\log \left( {1 + \exp \left( -{w^T}{x_i} \right)} \right)} 
\end{equation}
$$

$$
\begin{equation}
E_2\left( w \right) = {\log \left( {1 + \exp \left( -{w^T}{x_i} \right)} \right)} 
\end{equation}
$$

&emsp;所以两种损失函数是等价的，但从形式上来看第二种更加优美。

# 损失函数梯度
### 第一种梯度

$$
\begin{equation}
E_1\left( w \right) = {\log \left( {1 + \exp \left( {w^T{x_i}} \right)} \right) - {y_i}\left( {w^T{x_i}} \right)}
\end{equation}
$$

$$
\begin{equation}
\frac{\partial {E_1}\left( w \right)}{\partial w} = \sum\limits_{i = 1}^N 
{\left[ \frac{\exp \left( {w^T}{x_i} \right)}{1 + \exp \left( {w^T}{x_i} \right)}{x_i} - {y_i}{x_i} \right]} 
\end{equation}
$$

### 第二种梯度

$$
\begin{equation}
E_2\left( w \right) = {\log \left( {1 + \exp \left( { - {y_i}{w^T}{x_i}} \right)} \right)} 
\end{equation}
$$


$$
\begin{align}
\frac{\partial {E_2}\left( w \right)}{\partial w} &= \sum\limits_{i = 1}^N {\frac{\exp \left( { - {y_i}{w^T}{x_i}} \right)}{1 + \exp \left( { - {y_i}{w^T}{x_i}} \right)}} \left( { - {y_i}{x_i}} \right)\\
&= \sum\limits_{i = 1}^N {\theta \left( { - {y_i}{w^T}{x_i}} \right)} \left( { - {y_i}{x_i}} \right)
\end{align}
$$

&emsp;第二种形式更加优雅。

# 决策函数
&emsp;LR回归的预测值在(0, 1)区间内，根据具体的问题可以设置不同的阈值，一般以0.5为阈值，即

<center>${y_{pred}} = 1$, if $h\left( x \right) = P\left( {y = 1\left| x \right.} \right) > 0.5$</center>

&emsp;注意这里$h\left( x \right) = 0.5$时，$w^Tx=0$,即对数几率为0，几率为1，表示事件发生和不发生的概率相等，刚好为决策边界。
