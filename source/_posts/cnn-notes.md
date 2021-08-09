---
title: cnn笔记
date: 2021-04-20 16:11:54
tags:
- cnn
- 卷积神经网络
categories:
- 神经网络
---

## 基本概念

假设输入图像为(39x39x3)，卷积层的参数如下：

- $n_H = n_W = 39$
- 信道数量（number of channels）：$n_c = 3$
- 卷积核大小：$f = 3$
- 步长stride：$s = 1$
- padding：$p = 0$
- 过滤器数量：$10 filters$

得到结果为(37x37x10)。37通过公式：$\frac{n+2p-f}{s} + 1$计算得来。

> 对于3D卷积，卷积核大小为$f\times f \times n_c$，图像通道数=filter通道数。卷积后$n_c = 1$

### 池化层

Max Pooling：在卷积核范围内找到最大值作为输出。

eg: $\begin{pmatrix} 1&3&2&1\\2&9&1&1\\1&3&2&3\\5&6&1&2 \end{pmatrix}$在经过$f=s=2$的max pooling池化层之后为$\begin{pmatrix}9&2\\6&3\end{pmatrix}$

池化层输出大小也可以用上面的公式计算。也就是$(\lfloor \frac{n_H-f}{s}+1 \rfloor , \lfloor \frac{n_W-f}{s}+1 \rfloor , n_c)$

### 权值共享

[权值共享理解](https://www.zhihu.com/question/47158818)

- 减少参数
- 由于**图片的底层特征是与特征在图片中的位置无关的**，所以可用权值共享。底层特征比如边缘，要检测边缘肯定是对全局进行检测，与位置没有关系。高级特征一般是与位置有关的，比如一张人脸图片，眼睛和嘴位置不同，那么处理到高层，不同位置就需要用不同的神经网络权重，这时候卷积层就不能胜任了，就需要用局部全连接层和全连接层。

## 经典网络

### LeNet-5

|         层         | 输入大小 |  f   | filter个数 | 输出大小 |
| :----------------: | :------: | :--: | :--------: | :------: |
|      卷积层C1      | 32x32x1  |  5   |     6      | 28x28x6  |
| 池化层S2(avg pool) | 28x28x6  |  2   |     6      | 14x14x6  |
|      卷积层C3      | 14x14x6  |  5   |     16     | 10x10x16 |
| 池化层S4(avg pool) | 5x5x6  |  2   |     6      | 5x5x16  |

全连接层FC5, 120个神经元。

全连接层FC6, 84个神经元。

最后一层softmax层，输出10种结果(0-9)。

**总结：**

1. 卷积层后接一个池化层。
2. 层数越深$n_H, n_W$减少, $n_c$增加

### AlexNet

![](https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%886.46.36.png)

### VGG64

![](https://raw.githubusercontent.com/baichen99/pics/master/img/%E6%88%AA%E5%B1%8F2021-04-20%20%E4%B8%8B%E5%8D%886.57.47.png)

## 使用keras编写CNN

[keras-手把手入门1-手写数字识别-深度学习实战](http://nooverfit.com/wp/keras-手把手入门1-手写数字识别-深度学习实战/)

[深度学习中的batch的大小对学习效果有何影响？](https://www.zhihu.com/question/32673260)

```python
import keras
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras import backend as K

# batch_size 太小会导致训练慢，过拟合等问题，太大会导致欠拟合。所以要适当选择
batch_size = 128
# 0-9手写数字一个有10个类别
num_classes = 10
# 12次完整迭代，差不多够了
epochs = 12
# 输入的图片是28*28像素的灰度图
img_rows, img_cols = 28, 28

# 训练集，测试集收集非常方便
(x_train, y_train), (x_test, y_test) = mnist.load_data()
print(x_train.shape, y_train.shape)

# (60000, 28, 28) -> (60000, 28, 28, 1)
(x_train, y_train), (x_test, y_test) = mnist.load_data()
# https://stackoverflow.com/questions/49057149/expected-conv2d-1-input-to-have-shape-28-28-1-but-got-array-with-shape-1-2
if K.image_data_format() == 'channels_first':
    x_train = x_train.reshape(x_train.shape[0], 1, img_rows, img_cols)
    x_test = x_test.reshape(x_test.shape[0], 1, img_rows, img_cols)
    input_shape = (1, img_rows, img_cols)
else:
    x_train = x_train.reshape(x_train.shape[0], img_rows, img_cols, 1)
    x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
    input_shape = (img_rows, img_cols, 1)
input_shape = (img_rows, img_cols, 1)

x_train = x_train.astype('float32')
x_test = x_test.astype('float32')
# 归一化
x_train /= 255
x_test /= 255
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')


# 转换成one-hot编码
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)


# 牛逼的Sequential类可以让我们灵活地插入不同的神经网络层
model = Sequential()
# 加上一个2D卷积层， 32个filters，激活函数选用relu，
# 卷积核的窗口选用3*3像素窗口
model.add(Conv2D(32,
                 kernel_size=3,
                 activation='relu',
                 input_shape=(28, 28, 1)))
# 64个通道的卷积层
model.add(Conv2D(64,
                 kernel_size=3,
                 activation='relu'))
# 池化层是2*2像素的
model.add(MaxPooling2D(pool_size=(2, 2)))
# 对于池化层的输出，采用0.35概率的Dropout
model.add(Dropout(0.35))
# 展平所有像素，比如[28*28] -> [784]
model.add(Flatten())
# 对所有像素使用全连接层，输出为128，激活函数选用relu
model.add(Dense(128, activation='relu'))
# 对输入采用0.5概率的Dropout
model.add(Dropout(0.5))
# 对刚才Dropout的输出采用softmax激活函数，得到最后结果0-9
model.add(Dense(num_classes, activation='softmax'))
# 模型我们使用交叉熵损失函数，最优化方法选用Adadelta
model.compile(loss=keras.metrics.categorical_crossentropy,
              optimizer=keras.optimizers.Adadelta(),
              metrics=['accuracy'])
model.summary()


# 令人兴奋的训练过程
model.fit(x_train, y_train, batch_size=batch_size, epochs=epochs,
          verbose=1, validation_data=(x_test, y_test))

score = model.evaluate(x_test, y_test, verbose=0)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```



