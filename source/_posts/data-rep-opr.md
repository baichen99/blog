---
title: 考研笔记：数据的表示与运算
date: 2021-06-26 14:48:12
tags:
- 二进制
- 补码
- 计算机组成原理
categories:
- 计算机组成原理
- 考研
---

## 各种码的转换

原->补：符号为不变，右边开始数，第一个1后面的数字都变号

补->原：同上

移->补：符号位取反

[x]补->[-x]补:同第一





## 补码范围

因为原码中有+0，-0之分，所以其取值范围为$[-(2^n-1), 2^n-1]$

对于补码，认为其只有+0，因此-0(10000)空余出来，规定其表示最小负数$-2^n$

对于定点数，原码最小值为1.11111，转为补码为1.000001， 但这不是最小负数，最小负数是人为规定的1.00000



## 浮点数的范围

### IEEE 754 float

以IEEE 754为例，其格式如下：

|      | 数符 | 阶码 |   尾数   |
| :--: | :--: | :--: | :------: |
| 位数 |  1   |  8   | 23(补码) |

最大正数：

- 阶码：8位能表示的范围是0-255，去掉最高和最低位范围是1-254，减去偏置`2^{8-1}-1=127`范围是-126-127。
- 尾数：注意隐含位1，其他为全为1，根据n项和求和公式可得尾数最大为$1+\frac{1}{2} + ... + (\frac{1}{2})^{23} = \frac{1\times (1-2^{-24}) }{1- 1/2} = 2 - 2^{-23}$

所以最大正数为$2^{127} \times (2-2^{-23})$

最小正数：

- 阶码最小为-126
- 尾数最小为1.0

所以最小正数为$1.0 \times 2^{-126}$

### 题目中给定格式

如：7位阶码，1位数符，8位尾数，阶码用移码，尾数用补码，求表示范围：

分析：

- 7位无符号数，其表示范围为$0-127$，减去偏置$2^{6} = 64$，范围是-64~63
- 尾数为8位补码，首先在定点数补码中，将`1.0000`规定位最小负数代表-1，最大正数为8位全1，即$1-2^{-8}$

因此表示范围为$[-2^{63}, (1-2^{-8})\times 2^{63}]$



注意：

- IEEE 754中有隐含位，其他情况不考虑
- IEEE 754中移码偏置为$2^{r-1}-1$，而其他情况偏置为$2^{r-1}$

- 规格化后的浮点数范围看王道书P65