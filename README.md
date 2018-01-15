###### 依赖工具
1. 自动化构建工具[Bazel](https://www.bazel.build/)
2. 脚本语言调用c/cpp库工具[Swig](http://www.swig.org/)
3. 矩阵处理工具[Eigen](http://eigen.tuxfamily.org)
4. GPU加速计算库Nvidia-cuBLAS
5. 结构化数据存储格式[protobuf](https://github.com/google/protobuf)

###### 目录结构
core/
 - api_def
 - common_runtime
 - debug                        
 - distributed_runtime     
 - example                         
 - framework                      
 - graph - DAG图相关
 - grappler  
 - kernels - 核心Op，如【matmul, conv2d, argmax, batch_norm】等
 - lib - 基础公共库
    - gtl - google template library, 包含array_slice/ iterator_range/ inlined_vector/ map_util/ stl_util等基础库
 - ops - 均为.cc文件，为基本op操作，如array_ops/ math_ops/ image_ops/ function_ops/ random_ops/ io_ops/ control_flow_ops/ data_flow_ops；以及定义了op梯度计算方式：array_grad/ math_grad/ functional_grad/ nn_grad/ random_grad
 - platform - 平台相关文件，如设备内存分配
 - profiler
 - protobuf - 均为.proto文件，用于数据传输时的结构序列化
 - public - 公共头文件，用于外部接口调用的API定义，主要是session.h/ tensor_c_api.h
 - user_ops - 用户自定义op           
 - util

stream_executor/
> StreamExecutor is a unified wrapper around the CUDA and OpenCL host-side programming models (runtimes)
 - cuda
 - host
 - lib
 - platform