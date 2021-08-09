---
title: 毕设笔记
date: 2021-04-21 22:13:59
tags:
- 数字图像处理
categories:
- 毕设
---

## 论文

我的毕设论文题目是：*科学文献中曲线信息自动提取算法*，简单来说就是写一个算法能够从统计图中提取出曲线的信息(包括说明文字)，以及曲线每个点的对应的值。

### 基本思路

#### 目标提取

  1. 读取一张图片

![](https://raw.githubusercontent.com/baichen99/pics/master/img/test_img.png)

  2. canny边缘检测+霍夫直线检测检测出水平线和竖直线。

     <img src="https://raw.githubusercontent.com/baichen99/pics/master/img/lines.png" style="zoom:33%;" />

  3. 提取出最长的两根，根据这两条线求外接矩形得把图像分为**象限内**和**象限外**两部分

     <img src="https://raw.githubusercontent.com/baichen99/pics/master/img/getAxisCoords.png" style="zoom:33%;" />

  4. 然后使用MSER算法和NMS算法，分别检测X轴，Y轴外侧的文字。

     [MSER NMS](https://zhuanlan.zhihu.com/p/66789730)

![](https://raw.githubusercontent.com/baichen99/pics/master/img/xyboxes.png)

    1. 可以看到X轴下方有文字和数字两类，数字总是在靠近坐标轴一侧。因此我们从上往下扫描，碰到的第一行检测框即为数字，剩下的为描述文字。根据此方法过滤出数字和描述文字。

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/withdrawNums1.png"/>

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/withdrawNums2.png"  />

#### 字符识别算法

CRNN+CTC

[参考资料](https://zhuanlan.zhihu.com/p/43534801)

#### 确定曲线上一点的坐标值

这个方法基于文字检测和文字识别算法。

![](https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-26%20%E4%B8%8B%E5%8D%885.01.49.png)

工作流程如上图：

1. 通过数字识别算法识别出每个方框内的数值nums，并获取检测框中间点的坐标pixels
2. 剔除误差较大的点，并进行线性回归
3. 预测任意一点坐标

因为识别结果可能不准确，这样会造成较大误差，所以要先对数据进行清洗。

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-26%20%E4%B8%8B%E5%8D%885.00.10.png" style="zoom:50%;" />

[随机抽样一致RANSAC: Random Sample Consensus](https://zhuanlan.zhihu.com/p/36301702)

[线性回归及RANSAC异常值清除算法案例](https://blog.csdn.net/mago2015/article/details/84295425)



#### 曲线提取

1. 去噪
2. otsu二值化
3. 确定曲线位置getAxisCoords
4. 霍夫直线检测去除直线
5. 形态学变换 得到细线
6. 三次样条插值



## 形态学操作

https://blog.csdn.net/zizi7/article/details/50907949























































## RNN

### 基本概念

![](https://raw.githubusercontent.com/baichen99/pics/master/img/rnn-net.png)

$a^{\langle t \rangle} = f(w_a[a^{t-1}, x^{t}])$   

$y^{\langle t \rangle} = g(w_ya^{t}+b_y)$

> $w_a, w_y$实现了权值共享，另外有的图中$a^{\langle t \rangle}$也写作$s_{t}$，表示时间步t的单元状态。


定义某个时间步t的损失函数（交叉熵损失函数）：

$L^{\langle t \rangle}(\hat{y}^{\langle t \rangle}, y^{\langle t \rangle}) = -y^{\langle t \rangle}log\hat{y}^{\langle t \rangle} - (1-y^{\langle t \rangle})log(1-y^{\langle t \rangle})$

$y的结果是0或1，\hat y是预测值范围在0到1$

由此可以定义整个网络的损失函数

$L(\hat{y}, y) = \sum_{t=1}^{T_y} L^{\langle t \rangle}(\hat{y}^{\langle t \rangle}, y^{\langle t \rangle})$

最后通过反向传播算法求权重。

### 不同类型的RNN

上面的例子$T_x = T_y$，也就是输入x的长度和y相同，我们称之为多对多结构。但实际情况中有不一定是x，y有相同长度。比如有多对一结构：$T_y = 1$
<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-18%20%E4%B8%8B%E5%8D%884.50.13.png" style="zoom:50%;" />
一对多结构，比如音频序列生成：$T_x = \varnothing$

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-18%20%E4%B8%8B%E5%8D%884.49.31.png" style="zoom:50%;" />

多对多结构：比如机器翻译，输入输出的长度都不等

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-18%20%E4%B8%8B%E5%8D%884.51.00.png" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-18%20%E4%B8%8B%E5%8D%884.52.09.png" style="zoom:50%;" />



### RNN单元

![](https://raw.githubusercontent.com/baichen99/pics/master/img/v2-de5ad673d74a07ea40a24f916956e675_b.webp.gif)

### GRU(简化版)

由于存在梯度消失，前面的状态很难影响到后面的预测。举个例子：
`the cat, which already ate..., was ...` 

在这个例子中cat是单数，所以后面用was，如果cat是负数则用were。但梯度消失会导致无法根据cat的形式决定be动词形式。

在GRU中 $c^{\langle t \rangle} = a^{\langle t \rangle}$，然后用$\tilde c^{\langle t \rangle} = tanh(w_c[c^{\langle t-1 \rangle}, x^{\langle t \rangle}])$替代老的记忆单元。

$\Gamma_u$决定更新与否。$\Gamma_u = sigmoid(w_u[c^{\langle t-1 \rangle}, x^{<t}]+b_u),  \Gamma_u\in(0, 1)$   

$c^{\langle t \rangle} = \Gamma_u * {\tilde c}^{\langle t \rangle} + (1-\Gamma_u)*c^{\langle t-1 \rangle}$

> 因为对于大部分输入sigmoid的输出要么非常接近0，要么非常接近1。为了方便理解直接把它的输出看错0或1两个离散值。那么当$\Gamma_u=1时，c^{\langle t \rangle}$就要更新。=0时保持原来的值不变。

实例说明：

对于`the cat, which already ate..., was ...`这个例子设cat对应的t=2， was对应的t=10。

那么假设$c^{2}=1, \Gamma_u=1$ 中间的$\Gamma_u$都设置为0，那么就有$c^{2}=c^{3}=c^{4} ... =c^{10}=1$，这样就能让$c^{10}$保持和$c^{2}$相同都为1。这样就能根据cat的形式决定be动词的形式了。



### LSTM

有三个门：$\Gamma_u$更新门，$\Gamma_f$遗忘门，$\Gamma_o$输出门。

$\tilde c^{\langle t \rangle} = tanh(w_c[a^{\langle t-1 \rangle}, x^{\langle t \rangle}] + b_c)$

$\Gamma_u = sigmod(w_u[a^{\langle t \rangle}, x^{\langle t \rangle}] + b_u)$

$\Gamma_o = sigmod(w_o[a^{\langle t \rangle}, x^{\langle t \rangle}] + b_o)$

$c^{\langle t \rangle} = \Gamma_u * \tilde c^{\langle t \rangle} + \Gamma_f * c^{\langle t \rangle}$

$a^{\langle t \rangle} = \Gamma_o * tanh(c^{\langle t \rangle})$

