### 如何高效学习Tensorflow源代码
根据想要学习Tensorflow的目的不同，可以简单分为以下2类：

##### 算法研究者
1. 了解自己感兴趣基本领域，如图像分类、物体检测、语音识别等，了解其所用的技术，如CNN、RNN知道实现的基本原理；

2. 尝试运行[models](https://github.com/tensorflow/models)对应的基本模型。如果研究计算机视觉，可以多关注compression(图像压缩)、im2txt（图像描述）、inception(评估ImageNet)、resnet(残差网络)、slim(图像分类)和street(路标识别或验证码识别)；如果研究自然语言处理，可以看lm_1b(语言模型)、namignizer(起名字)、swivel(转换词向量)、syntaxnet(分词与语法分析)、texsum(文本摘要)、word2vec(词转换为向量)。
	- mnist

3. 结合项目，找到相关的论文用Tensorflow实现论文中的算法；

4. 熟悉Tensorflow代码结构

##### 系统研究者
1. 首先需要弄清平台的很多名称、概念、定义，譬如Tensor、DAG、Operator、Variable、Device、Optimizer等；

2. 以MNIST的例子程序作为TF的入门，用一个简单的Softmax实现了手写体数字识别的神经网络，只有一层参数。熟悉以后可以继续看用卷积神经网络实现的MNIST，相对于单层Softmax有更高的准确率。

3. 了解Tensorflow的分布式模型，熟悉分布式版本的MNIST；

4. 熟悉[Tensorflow源码结构](code_structure.md)，根据需要了解其源代码实现；