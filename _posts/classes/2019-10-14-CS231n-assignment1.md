---
layout: post
title: "CS231n assignment1"
subtitle: 'CS231n assignment'
date:       2019-10-14 21:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - CS231n
  - machine learning
---
主要是总结几个函数
1. np.bincount
在knn分类时，先找到k近邻，根据k近邻标签中的多数赋予测试样本。其实就是一个k近邻标签的众数，但numpy没有现成的众数函数。在这里采用np.bincount函数。
```python
y_pred[i] = np.argmax(np.bincount(closest_y))
```
根据[文档](https://docs.scipy.org/doc/numpy/reference/generated/numpy.bincount.html)解释：
>Count number of occurrences of each value in array of non-negative ints.
The number of bins (of size 1) is one larger than the largest value in x. If minlength is specified, there will be at least this number of bins in the output array (though it will be longer if necessary, depending on the contents of x). Each bin gives the number of occurrences of its index value in x. If weights is specified the input array is weighted by it, i.e. if a value n is found at position i, out[n] += weight[i] instead of out[n] += 1.

可看出这就是一个桶排序算法，桶数比数组最大值大1。瞬间感觉学好数据结构，横跨各个语言都不怕！
2. plt.errorbar
在cross-validation找knn最优k时，用5折交叉验证。对每个k，有5个分类器，5个准确率。
```python
accuracies_mean = np.array([np.mean(v) for k,v in sorted(k_to_accuracies.items())])
accuracies_std = np.array([np.std(v) for k,v in sorted(k_to_accuracies.items())])
plt.errorbar(k_choices, accuracies_mean, yerr=accuracies_std)
```
按照各个k下5折交叉验证的准确率均值，方差画图error bar图形。
![plterrorbar](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-14/plterrbar.png)

未完待续。。。