---
title: 神经网络：dropout 正则化
date: 2021-02-19 09:47:05
tags:
- dropout
- 神经网络
categories:
- 神经网络
mathjax: true
---

## dropout原理

直觉-1：每次迭代之后，神经网络都会变得更小，采用一个较小的神经网络好像和使用正则化的效果是一样的。

直觉-2：神经元不能依赖任何特征，因为每个特征都可能被随机清除，所以不能把所有赌注放在一个节点上，不能给一个输入太多权重，这样改单元通过这种方式积极传播开

dropout的功能类似于L2正则化。

## 实现方法

比如我们有一个神经网络，要在第3层进行dropout。

定义keep_prob 为保留单元的概率

$d^{[3]} = np.random.rand(a^{[3]}.shape[0], a^{[3]}.shape[1]) < keep_prob$  这一步是生成一个bool矩阵，其中1的概率为keep_prob。

$a^{[3]} = np.multiply(a^{[3]}, d^{[3]})$   

$a^{[3]} /= keep\_prob$  这里除以keep_prob是为了让期望保持不变，因为直接去除几个结果期望肯定会改变。

> 设置 keep _prob==1则保留这一层的结果，不进行dropout。

另外dropout只在训练时执行，测试阶段则不进行dropout，因为在测试阶段我们不希望输出结果是随机的。

## keras中使用dropout

使用dropout正则化随机禁用隐藏层中神经元的某些部分。 在Keras库中，你可以在任何隐藏层之后添加dropout，并且可以指定rate，该rate决定了**上一层**中禁用的神经元的百分比。

https://machinelearningmastery.com/dropout-regularization-deep-learning-models-keras/