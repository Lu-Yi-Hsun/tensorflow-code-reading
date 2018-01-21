##### MNIST 数据集
MNIST数据集是NIST数据集的子集，包含以下4个文件。
 - train-images-idx3-ubyte.gz：训练集图片文件
 - train-labels-idx1-ubyte.gz：训练集标记文件
 - t10k-images-idx3-ubyte.gz：测试集图片文件
 - t10k-labels-idx1-ubyte.gz：测试集标记文件

##### 训练方法
目前已经有各种方法被应用在MNIST这个训练集上，相应的代码位于/tensorflow/examples/tutorials/mnist/下。
 - mnist_softmax.py： MNIST 采用 Softmax 回归训练
 - fully_connected_feed.py： MNIST 采用 Feed 数据格式训练
 - mnist_with_summaries.py： MNIST 使用卷积神经格络（CNN），并且训练过程可视化
 - mnist_softmax_xla.py.py： MNIST 使用 XLA 框架

 ##### Softmax回归训练
 1. 加载数据；
 2. 构建回归模型，计算采用softmax函数拟合后的预测值，并且定义损失函数和优化器；
 	> # 定义损失函数和优化器
	> y_ = tf.placeholder(tf.float32, [None, 10]) # 输入的真实值的占位符
	> # 我们用 tf.nn.softmax_cross_entropy_with_logits 来计算预测值 y 与真实值 y_的差值，并取均值
	> cross_entropy = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(y, y_))
	> # 采用 SGD 作为优化器
	> train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
3. 训练模型。训练1000次并且每次循环中都随机抓取训练数据中的100个数据点来替换之前的占位符。这种训练方式称为随机训练，使用SGD方法进行梯度下降与每次使用所有训练数据BGD相比既能学习到数据集的总体特征，又能加速训练过程。
	> for_ in range(1000):
	> batch_xs, batch_ys = mnist.train.next_batch(100)
	> sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
4. 评估模型。tf.argmax(y,1)返回的是模型对任一输入 x 预测到的标记值。
	> Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
	> Extracting /tmp/tensorflow/mnist/input_data\train-images-idx3-ubyte.gz
	> Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
	> Extracting /tmp/tensorflow/mnist/input_data\train-labels-idx1-ubyte.gz
	> Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
	> Extracting /tmp/tensorflow/mnist/input_data\t10k-images-idx3-ubyte.gz
	> Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
	> Extracting /tmp/tensorflow/mnist/input_data\t10k-labels-idx1-ubyte.gz
	> 0.9203

 ##### CNN训练
这里主要总结下MNIST上CNN训练中构建模型步骤。构建一个CNN模型，需要以下几步。
- 定义输入数据并预处理数据。读取数据MNIST，并分别得到训练集的图片和标记的矩阵，以及测试集的图片和标记的矩阵。
- 初始化权重与定义网络结构。我们将要构建一个拥有 3 个卷积层和 3 个池化层，随后接 1 个全连接层和 1 个输出层的卷积神经网络。
	- 初始化权重格法如下，我们设置卷积核的大小为 3×3：
	> w = init_weights([3, 3, 1, 32]) # patch 大小为 3×3，输入维度为 1，输出维度为 32
	> w2 = init_weights([3, 3, 32, 64]) # patch 大小为 3×3，输入维度为 32，输出维度为 64
	> w3 = init_weights([3, 3, 64, 128]) # patch 大小为 3×3， 输入维度为 64，输出维度为 128
	> w4 = init_weights([128 * 4 * 4, 625]) # 全连接层， 输入维度为 128 × 4 × 4,是上一层的输出数据又三维的转变成一维， 输出维度为 625
	> w_o = init_weights([625, 10]) # 输出层，输入维度为 625, 输出维度为 10，代表 10 类(labels)
	- 定义一个网络模型，主要结构如上所述。在卷积层/池化层、全连接层、输出层后都会接一个dropout,它表示一层中有多少比例的神经元被留下。
	- 定义损失函数。这里我们仍然采用 tf.nn.softmax_cross_entropy_with_logits 来比较预测值和真实值的差异，并做均值处理；定义训练的操作（train_op），采用实现 RMSProp 算法的优化器 tf.train.RMSPropOptimizer，学习率为 0.001，衰减值为 0.9，使损失最小；定义预测的操作（predict_op）。
对于卷积神经网络，当训练 100 轮后精确率已经接近99.22%。

 ##### RNN训练