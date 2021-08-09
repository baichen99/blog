---
title: python数据处理笔记
date: 2021-02-03 11:26:52
tags:
- 数据处理
- python
- pandas
- numpy
categories:
- 机器学习

---

## Pandas

### dataframe.loc[index, column]

根据index, column来返回对应数据。index为行，column为列。

[ioc使用](https://blog.csdn.net/chenKFKevin/article/details/62049060)

```python
df = pd.DataFrame([[1, 2], [4, 5], [7, 8]], index=['cobra', 'viper', 'sidewinder'], columns=['max_speed', 'shield'])
print(df)
print('-'*20)
print(df.loc['cobra'])
```

```
            max_speed  shield
cobra               1       2
viper               4       5
sidewinder          7       8
--------------------
max_speed    1
shield       2
Name: cobra, dtype: int64
```

### dataframe.drop

[官方文档](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.drop.html)

drop会将数据删除一列，并返回一个副本

## Numpy

[reshape(-1, 1)](https://blog.csdn.net/lxlong89940101/article/details/84314703)

### numpy.argmax(a, axis=None, out=None)



Parameters:
a : *array_like*
数组
axis : *int, 可选*
默认情况下，索引的是平铺的数组，否则沿指定的轴。
out : *array, 可选*
如果提供，结果以合适的形状和类型被插入到此数组中。
Returns:
index_array : *ndarray of ints*
索引数组。它具有与a.shape相同的形状，其中axis被移除。

```
>>> a = np.arange(6).reshape(2,3)
>>> a
array([[0, 1, 2],
       [3, 4, 5]])
>>> np.argmax(a)
5
>>> np.argmax(a, axis=0)#0代表列
array([1, 1, 1])
>>> np.argmax(a, axis=1)#1代表行
array([2, 2])
>>>
>>> b = np.arange(6)
>>> b[1] = 5
>>> b
array([0, 5, 2, 3, 4, 5])
>>> np.argmax(b) # 只返回第一次出现的最大值的索引
1
```

### ndarray from ragged nested sequences)

[Numpy VisibleDeprecationWarning (ndarray from ragged nested sequences)](https://stackoverflow.com/questions/63097829/debugging-numpy-visibledeprecationwarning-ndarray-from-ragged-nested-sequences)

## tensorflow

### PIL, Image, Tensor转换

[Pytorch中Tensor与各种图像格式的相互转化](https://oldpan.me/archives/pytorch-tensor-image-transform)

### one-hot编码

keras.utils.to_categorical(y, num_classes=None, dtype='float32')


```python
> labels
array([0, 2, 1, 2, 0])
# `to_categorical` 将其转换为具有尽可能多表示类别数的列的矩阵。
# 行数保持不变。
> to_categorical(labels)
array([[ 1.,  0.,  0.],
       [ 0.,  0.,  1.],
       [ 0.,  1.,  0.],
       [ 0.,  0.,  1.],
       [ 1.,  0.,  0.]], dtype=float32)
```

### tensorflow多维张量计算

[https://blog.csdn.net/weixin_42445581/article/details/82791811](tensorflow多维张量计算)

### load model 

[当model中有自定义层的时候 load出错](https://github.com/tensorflow/tensorflow/issues/43498)

###  

## Sklearn

- [pandas 下的 one hot encoder 及 pd.get_dummies() 与 sklearn.preprocessing 下的 OneHotEncoder 的区别](https://blog.csdn.net/lanchunhui/article/details/72870358)
