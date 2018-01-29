##### Shell脚本
1. [8262](https://github.com/tensorflow/tensorflow/pull/8262):set -e表示遇到错误理解终止脚本，set +e表示遇到错误报错但是继续执行脚本；
2. [8331](https://github.com/tensorflow/tensorflow/pull/8331):sed命令用法；
```
# 将字符串中所有":"替换为" "，如果没有g标记则只替换字符串中第一个":"
for g in $(echo $GOPATH | sed "s/:/ /g"); do
	TF_DIR="${g}/src/github.com/tensorflow/tensorflow"
	PROTOC="${TF_DIR}/bazel-out/host/bin/external/protobuf/protoc"
 	if [ -x "${PROTOC}" ]; then
		break
	fi
done
```

##### Tensorflow
1. [8546](https://github.com/tensorflow/tensorflow/pull/8546): [Create a custom C++ op](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/docs_src/extend/adding_an_op.md)(tensorflow/docs_src/extend/adding_an_op.md)；
2. [8638](https://github.com/tensorflow/tensorflow/pull/8638): Support pandas Series and DataFrame as input for tf.constant(tensorflow/python/framework/tensor_util.py)；
3. [8662](https://github.com/tensorflow/tensorflow/pull/8662):Support self_adjoint_eig to complex64 and complex128(tensorflow/core/kernels/self_adjoint_eig_v2_op.cc)；
4. [8901](https://github.com/tensorflow/tensorflow/pull/8901):Add gzip and zlib support for FixedLengthRecordReader(tensorflow/core/kernels/fixed_length_record_reader_op.cc)；
5. [9000](https://github.com/tensorflow/tensorflow/pull/9000):Allow the output of `tf.argmax` as index type(tensorflow/python/ops/array_ops.py)；
```
# This following issue will raise a TypeError:
a = tf.constant([1, 2, 3], dtype=tf.float32)
b = tf.argmax(a)
tf.Session().run(a[b])

TypeError: Input 'strides' of 'StridedSlice' Op has type int32 that does not match type int64 of argument 'begin'.
```
6. [9514](https://github.com/tensorflow/tensorflow/pull/9514):Dynamic ksize and strides with MaxPool and AvgPool(tensorflow/core/kernels/maxpooling_op.cc)；