Source code: \tensorflow\core\common_runtime\direct_session.cc

Rendezvous定义在core/framework/rendezvous.h中
1. A Rendezvous is an abstraction for passing a Tensor from a producer to a comsumer where the consumer may safly request the Tensor before or after it has been produces. A producer never blocks when using a Rendezvous. A consumer has the choice of making a blocking call or providing a callback: in either case, the consumer receives the Tensor as soon as it is available. （简言之：Nonblocking send, blocking receive）
2. A Rendezvous key encodes a single <producer, consumer> pair. It is an error to call Send() or Recv*() more than once with the same key。
3. 在消息通信机制中，消息传递涉及到信箱容量问题。一个极端的情况是信箱容量为0，那么，当send在receive之前执行的话，则发送进程被阻塞，直到receive做完。执行receive时信件可以从发送者直接拷贝到接收者，不用任何中间缓冲。类似的，如果receive先被执行，接受者将被阻塞直到send发生。上述策略称为回合（rendezvous）原则。
4. tensorflow的消息传递属于【发送不阻塞，接收阻塞】，实现场景有以下两种：
	- LocalRendezvous (本地消息传递)
	- RpcRemoteRendezvous (分布式消息传递)
	- IntraProcessRendezvous (rendezvous_mgr.h)，这是一种特殊的通信形式，用于本地不同设备间通信。
5. IntraProcessRendezvous: Buffering of Tensor values is delegated to a "local" Rendezvous obtained from NewLocalRendezvous(). This class just adds functionality to coordinate multiple process-local devices.
6. Rendezvous使用示例：
	```c++
    Rendezvous* rendezvous = NewNaiveRendezvous();
    TF_CHECK_OK(rendezvous->Send("input", some_input_tensor));
    TF_CHECK_OK(executor->Run({ExecutorOpts, rendezvous, nullptr}));
    TF_CHECK_OK(rendezvous->Recv("input", &output_tensor));
    ```
7. 在Op Kernels中，有SendOp和RecvOp两个类（kernels/sendrecv_ops.h），与Rendezvous结合使用。
8. Each note: port specified in inputs is replaced with a feed node, which will pick up the provided input tensor from specially-initialized entries in a Rendezvous object used for the Run call.