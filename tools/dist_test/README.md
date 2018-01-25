### 分布式Tensorflow
##### 分布式架构
分布式是指训练在多个工作节点（worker）上。部署有以下两种方法。
1. 设置 CUDA_VISIBLE_DEVICES 环境变量，限制各个工作节点只可见一个 GPU，启
动进程时添加环境变量即可。
```
CUDA_VISIBLE_DEVICES='' python ./distributed_supervisor.py --ps_hosts=127.0.0.1:2222,127.0.0.1:2223 --worker_hosts=127.0.0.1:2224,127.0.0.1:2225 --job_name=ps --task_index=0
CUDA_VISIBLE_DEVICES='0' python ./distributed_supervisor.py --ps_hosts=127.0.0.1:2222,127.0.0.1:2223 --worker_hosts=127.0.0.1:2224,127.0.0.1:2225 --job_name=worker --task_index=0
CUDA_VISIBLE_DEVICES='1' python ./distributed_supervisor.py --ps_hosts=127.0.0.1:2222,127.0.0.1:2223 --worker_hosts=127.0.0.1:2224,127.0.0.1:2225 --job_name=worker --task_index=1
```
2. 使用 tf.device()指定使用特定的 GPU。

分布式架构主要有客户端（client）和服务端（server）组成，服务端又包括主节点（master）和工作节点（worker）两者组成。
 - 客户端用于建立Tensorflow计算图，并建立与集群进行交互的会话层。因此，代码中只要包含Session()就是客户端；
 - 服务端是一个运行了tf.train.Server实例的进程，并有主节点服务（master service）和工作节点服务（worker service）之分；
 - 主节点服务实现了tensorflow::Session接口，通过RPC来远程连接工作节点。在Tensorflow服务端中，一般是task_index为0的job；
 - 工作节点服务时限了worker_service.proto接口，使用本地设备对部分图进行计算；

 在运行Tensorflow分布式时，首先需要创建一个集群cluster对象，集群式作业job的集合，作业是任务task的集合。一般把作业划分为参数作业parameter job和工作节点作业worker job。参数作业运行的服务器称为参数服务器parameter server，负责管理参数的存储和更新；工作节点作业负责管理无状态且主要从事计算的任务，如运行操作。

##### 分布式模式
1. 数据并行。数据并行的原理很简单，其中CPU主要负责梯度平均和参数更新，而GPUs主要是负责训练模型副本（model replica）。这里成为模型副本是因为它们都是基于训练样例的子集训练得到的，模型之间具有一定的独立性。
	- 在GPUx上分别定义模型网络结构；
	- 对于单个GPU，分别从数据管道读取不同的数据块，然后进行前向传播，计算出损失，再计算当前变量的梯度；
	- 把所有GPU输出的梯度数据转移到CPU上，先进行梯度求平均操作，然后进行模型变量的更新；
	- 重复上述1~3步直至收敛；
2. 同步更新和异步更新。分布式随机梯度下降的参数更新可以分为同步和异步两种方式。除了参数更新的机制，还有一个重要的问题：训练时数据是如何分发到工作节点上的呢？
	> 数据并行的目的主要是提高SGD的效率。例如，假如每次SGD的mini-batch大小是1000个样本，那么如果切成10份，每份100个，然后将模型复制10份，就可以在10个模型上同时计算。但是，因为10个模型的计算速度可能是不一致的，有的快有的慢，那么在 CPU 更新变量的时候，是应该等待这一mini-batch全部计算完成，然后求和取平均来更新呢，还是让一部分先计算完的就先更新，后计算完的将前面的覆盖呢？这就引本了同步更新和异步更新的问题。
	- 图内复制（in-graph replication）。所有操作都在同一个图中，用一个客户端来生成图，然后把所以莋分配到所有参数服务器和工作节点上；
	- 图间复制（between-graph replication）。每个工作节点创建一个图，训练的参数保存在参数服务器，数据不用分发，各个节点独立计算，计算完成后把要更新的参数告诉参数服务器，参数服务器来更新参数；
3. 模型并行。可以对模型进行切分，让模型的不同部分执行在不同的设备上，这样一个批次样本可以在不同的设备上同时执行。

##### 分布式API
 - tf.train.ClusterSpec({"ps": ps_hosts, "worker": worker_hosts})。创建TensorFlow集群描述信息。
 - tf.train.Server(cluster, job_name, task_index)。创建一个服务（主节点服务或者工作节点服务），用于运行相应作业上的计算任务，运行的任务在task_index指定的机器上启动。当集群规模比较大时，就需要使用本动化的管理节点、监控节点的工具，如集群管理工具Kubernetes。
 - tf.device(device_name_or_function)。设定在指定的设备上执行张量运算，指定代码运行在 CPU 或 GPU 上。

##### 分布式训练代码框架
 - 第1步：命令行参数解析，获取集群的信息ps_hosts和worker_hosts，以及当前节点的角色信息job_name和task_index；
 - 第2步：创建当前任务节点的服务器；
 - 第3步：如果当前节点是参数服务器，则调用server.join()无休止等待；如果是工作节点，则执行第4步；
 - 第4步：构建要训练的模型，构建计算图；
 - 第5步：创建tf.train.Supervisor来管理模型的训练过程，创建一个supervisor来监督训练过程；

### 参考文献
1. Martín Abadi .etc TensorFlow: Large-Scale Machine Learning on Heterogeneous Distributed Systems
2. [distributed_tensorflow](https://github.com/tobegit3hub/tensorflow_examples/tree/master/distributed_tensorflow)
3. Mu Li .etc: Parameter Server for Distributed Machine Learning
4. Jianmin Chen .etc: Revisiting Distributed Synchronous SGD
5. tensorflow/tools/dist_test/python/mnist_replica.py
