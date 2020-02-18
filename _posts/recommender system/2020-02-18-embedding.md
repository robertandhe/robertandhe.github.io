---
layout: post
title: "tensorflow embedding使用"
subtitle: ''
date:     2020-02-18 18:44:00
author: "mudux"
music: false
music-id: 187937
mathjax: true
autoNumber: true
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - tensorflow
---

&emsp;最近在学习推荐系统时，在很多模型如FM，FFM和DeepFM中对离散特征都使用了embedding方法，把稀疏特征映射为密集向量。这里总结下tensorflow中embedding的使用。

### 1. embedding_lookup等同于one-hot相乘
```python
embedding = tf.get_variable(name='embedding',shape=[4,4],dtype=tf.float32,initializer=tf.random_uniform_initializer)
cat_feature = tf.constant([2,3,1,0])
get_embedding1 = tf.nn.embedding_lookup(embedding, cat_feature)
cat_feature_one_hot = tf.one_hot(cat_feature,depth=4)
get_embedding2 = tf.matmul(cat_feature_one_hot,embedding)

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    embedding1,embedding2 = sess.run([get_embedding1,get_embedding2])
    print(embedding1)
    print(embedding2)
```
输出为：  
```
[[0.54882884 0.23241234 0.41473126 0.9396106 ]
 [0.5108671  0.01206315 0.553378   0.84107137]
 [0.5031127  0.3810377  0.7152388  0.06897962]
 [0.69333184 0.15539801 0.40856004 0.9438031 ]]
[[0.54882884 0.23241234 0.41473126 0.9396106 ]
 [0.5108671  0.01206315 0.553378   0.84107137]
 [0.5031127  0.3810377  0.7152388  0.06897962]
 [0.69333184 0.15539801 0.40856004 0.9438031 ]]
 ```

 &emsp;可以看出embedding_lookup等同于embedding矩阵乘以one-hot向量。

### 2. embedding联合gather单维索引  

 &emsp;通过gather获取对应位置embeddding。
 ```python
embedding = tf.constant(
    [
        [0.21,0.41,0.51,0.11],
        [0.22,0.42,0.52,0.12],
        [0.23,0.43,0.53,0.13],
        [0.24,0.44,0.54,0.14]
    ],dtype=tf.float32)
index_a = tf.Variable([2, 3, 1, 0])    
gather_a = tf.gather(embedding, index_a)
gather_a_axis1 = tf.gather(embedding,index_a,axis=1)
with tf.Session() as sess:
    sess.run(tf.compat.v1.global_variables_initializer())
    print(sess.run(gather_a))
    print(sess.run(gather_a_axis1))
 ```
输出为：  
```
[[0.23 0.43 0.53 0.13]
 [0.24 0.44 0.54 0.14]
 [0.22 0.42 0.52 0.12]
 [0.21 0.41 0.51 0.11]]
[[0.51 0.11 0.41 0.21]
 [0.52 0.12 0.42 0.22]
 [0.53 0.13 0.43 0.23]
 [0.54 0.14 0.44 0.24]]
 ```

### 3. embedding联合gather多维索引
 ```python
 a = tf.Variable([[1, 2, 3, 4, 5], [6, 7, 8, 9, 10], [11, 12, 13, 14, 15]])
index_a = tf.Variable([2])

b = tf.get_variable(name='b',shape=[3,3,2],initializer=tf.random_normal_initializer)
index_b = tf.Variable([[0,1,1],[2,2,0]])

with tf.Session() as sess:
    sess.run(tf.compat.v1.global_variables_initializer())
    print(sess.run(tf.gather_nd(a, index_a)))
    print(sess.run(b))
    print(sess.run(tf.gather_nd(b, index_b)))
 ```
 输出为：  
``` 
[11 12 13 14 15]
[[[ 0.01930716 -0.62254894]
  [ 1.3473835   1.840278  ]
  [-0.85062003  1.1773872 ]]

 [[ 0.07472491 -1.1163161 ]
  [-1.9266038   0.7734673 ]
  [ 0.5189785  -0.4816973 ]]

 [[ 0.65923125  1.0562596 ]
  [-0.71715087 -1.7175527 ]
  [ 0.80931383 -0.84878135]]]
[1.840278   0.80931383]
```


### 4. 多值离散特征的embedding_lookup_sparse
&emsp;在之前的模型中，每个离散特征只有一个取值，因此在处理的过程中，将原始数据处理成了两个文件，一个记录特征的索引，一个记录特征的值。  
&emsp;但是当某一个离散特征有多个取值时如何获取embedding？例如很多人都喜欢NBA球队，有的人喜欢勇士，有的人喜欢湖人，也有人同时喜欢骑士，猛龙，马刺。对于这种多值离散特征如何获取embedding？  
&emsp;根据DeepFM的思想，需要将每一个field的特征转换为定长的embedding，即使有多个取值，也要变换为定长的embedding。  
&emsp;tf.nn.embedding_lookup_sparse就是处理这种问题的。
```python
tf.nn.embedding_lookup_sparse(
    params,
    sp_ids,
    sp_weights,
    combiner=None,
    max_norm=None,
    name=None
)
```
参数意义为：  
**params**:embedding矩阵；  
**sp_ids**:NxM的SparseTensor，N为batch size，其实就是索引；  
**sp_weights**:权重，可为SparseTensor或者None,为None时表示各个权重为1。如果不是None则必须和sp_ids有同样的shape和indices;  
**combiner**:每行id融合函数，可为"mean","sqrtn"和"sum"  ；
**max_norm**:embedding融合前clip阈值。  
&emsp;这个函数假设在sp_ids在每一行至少有一个id。并且所有id值在[0, p0)之间，p0是params行数。  
&emsp;例如：
```python
a = tf.SparseTensor(indices=[[0, 0],[0, 1],[1, 2],[1,3]], values=[1, 2, 2, 3], dense_shape=[2, 4])
b = tf.sparse_tensor_to_dense(a)

embedding = tf.constant(
    [
        [0.21,0.41,0.51,0.11],
        [0.22,0.42,0.52,0.12],
        [0.23,0.43,0.53,0.13],
        [0.24,0.44,0.54,0.14]
    ],dtype=tf.float32)

embedding_sparse = tf.nn.embedding_lookup_sparse(embedding, sp_ids=a, sp_weights=None, combiner='sum')

with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    print(sess.run(embedding_sparse))
    print(sess.run(b))
```
输出为：  
```
[[0.45       0.85       1.05       0.25      ]
 [0.47       0.87       1.0699999  0.26999998]]
[[1 2 0 0]
 [0 0 2 3]]
 ```