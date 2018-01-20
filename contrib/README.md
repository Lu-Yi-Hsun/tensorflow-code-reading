### contrib目录
这里包含的是非官方封装成的高级API库。这些库很有可能在完善之后被官方迁移到更核心的目录中，现在有一部分package在/tensorflow/models中有了更完整的体现。常用的包如下：
 - framework: 很多函数（如get_variables, get_global_step）都在这里定义；
 - layers: 
 	- initializers.py 中主要是做变量初始化的函数。 
 	- layers.py 中有关于层操作和权重偏置变量的函数。 
 	- optimizers.py 中包含损失函数和 global_step 张量的优化器操作。
	- regularizers.py 中包含带有权重的正则化函数。 
	- summaries.py 中包含将摘要操作添加到 tf.GraphKeys.SUMMARIES 集合中的函数。
 - learn： 使用TF进行深度学习的高级API。包括训练模型和评估模型、读取批处理数据和队列功能；
 - rnn: 这个包提供了额外的 RNN Cell， 也就是对 RNN隐藏层的各种改进， 如 LSTMBlockCell、GRUBlockCell、 FusedRNNCell、 GridLSTMCell、 AttentionCellWrapper 等；
 - seq2seq: 这个包提供了建立神经网络 seq2seq 层和损失函数的操作；
 - slim: TensorFlow-Slim是一个用于定义、训练和评估复杂模型的轻量级库。TF-Slim 现在已经被逐渐迁移到 /tensorflow/models/tree/master/slim 中，这里包含了几种广泛使用的卷积神经网络图像分类模型的代码，可以从头训练模型或者预训练模型开始微调；