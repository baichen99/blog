---
title: crnn-ctc
date: 2021-04-23 08:22:07
tags:
- crnn
- cnn
- ctc
categories:
- 神经网络
---

## CTC

[CTC算法详解](https://xiaodu.io/ctc-explained/)

CTC（Connectionist Temporal Classification）算法，它可以让RNN直接对序列数据进行学习，而无需事先标注好训练数据中输入序列和输出序列的映射关系，打破了RNN应用于语音识别、手写字识别等领域的数据依赖约束，使得RNN模型在序列学习任务中取得更好的应用效果。

[端到端模型（end-to-end learning）](https://blog.csdn.net/qq_38410428/article/details/91381151)

在序列学习任务中，RNN模型对训练样本一般有这样的依赖条件：输入序列和输出序列之间的映射关系已经事先标注好了。由于输入序列和输出序列是一一对应的，所以RNN模型的训练和预测都是端到端的，即可以根据输出序列和标注样本间的差异来直接定义RNN模型的Loss函数，传统的RNN训练和预测方式可直接适用。

但有些数据，比如音频数据，天然难以分割，在大规模训练下这种数据要求是完全不切实际的。而如果输入序列和输出序列之间映射关系没有提前标注好，那传统的RNN训练方式就不能直接适用了，无法直接对音频数据和图像数据进行训练。

![](https://raw.githubusercontent.com/baichen99/pics/master/img/r2.png.webp)

Connectionist Temporal Classification（CTC）[1]是Alex Graves等人在ICML 2006上提出的一种端到端的RNN训练方法，它可以让RNN直接对序列数据进行学习，而无需事先标注好训练数据中输入序列和输入序列的映射关系，使得RNN模型在语音识别等序列学习任务中取得更好的效果。

[ctc原理](https://zhuanlan.zhihu.com/p/42719047)



## 代码

[OCR model for reading Captchas](https://keras.io/examples/vision/captcha_ocr/#model)