---
title: keras or tensorflow 显存占用问题
date: 2017/08/11 22:18
comments: true
categories: 
- 深度学习
tags: 
- keras
- tensorflow
---

# 事件起因
实验室服务器上是四块GTX1080ti,但是在运行tensorflow或keras程序时，发现四块卡的显存全部被占用了。
就是下面这种情况:
![案发现场.png](http://upload-images.jianshu.io/upload_images/1575688-63872cc6cb7b8c18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

正常应该是这样的

![没有运行.png](http://upload-images.jianshu.io/upload_images/1575688-54c06288ad2a3f3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这让我很费解啊。
# 原因分析
出现这种情况的原因是，tensorflow初始化时会默认占满全部显卡和全部剩余显存，这种情况肯定是不行的，你一个人用了全部的，其他人还用不用了？
对于这种情况[tensorflow官网](https://www.tensorflow.org/tutorials/using_gpu#allowing-gpu-memory-growth)给出的说法是
> By default, TensorFlow maps nearly all of the GPU memory of all GPUs (subject to [CUDA_VISIBLE_DEVICES
](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars)) visible to the process. This is done to more efficiently use the relatively precious GPU memory resources on the devices by reducing [memory fragmentation](https://en.wikipedia.org/wiki/Fragmentation_(computing)).

说白了就是，我们为了防止碎片化和为了更好的利用内存，默认占用全部显存。

但是我们不想用这么多也是有办法的

# 解决办法
## tensorflow
1. 使用tensorflow时可以使用如下代码来选着使用某块显卡
 ~~~ python
 # Creates a graph.
with tf.device('/gpu:0'):
   # write your code here
~~~
2. 限制显存使用，官网同样提供了两种解决方案

a. 
~~~python
config = tf.ConfigProto()
config.gpu_options.allow_growth = True
session = tf.Session(config=config, ...)
~~~ 

在这种方案下，显存占用会随着epoch的增长而增长，也就是运行后面的eopch时，会去申请新的显存，前面已经完成的epoch所占用的显存并不会释放，原因也是为了防止碎片化。

b. 
~~~python
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.3
session = tf.Session(config=config, ...)
~~~
这种方法就比较给力了，告诉tensorflow，我这块显卡只给你30%的显存，其余的你给我放着不动。
## keras
由于keras是使用的tensorflow后端，所以需要加上额外的语句。
~~~python
import os
os.environ["CUDA_VISIBLE_DEVICES"] = "1"
from keras.backend.tensorflow_backend import set_session
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.3
set_session(tf.Session(config=config)) # 此处不同
~~~
上面的语句中设定使用那一块显卡和tensorflow有些不同(我没试验过keras是不是可以用tensorflow指定gpu的语句)，需要使用CUDA_VISIBLE_DEVICES这个值来设定，这个值就是让某几块(使用','分隔)显卡可以被cuda看见，那么程序也就只能调用那几块显卡了。

**需要注意的是，虽然代码或配置层面设置了对显存占用百分比阈值，但在实际运行中如果达到了这个阈值，程序有需要的话还是会突破这个阈值。换而言之如果跑在一个大数据集上还是会用到更多的显存。以上的显存限制仅仅为了在跑小数据集时避免对显存的浪费而已。**

下面是只设置1,2卡可见的情况
~~~python
os.environ["CUDA_VISIBLE_DEVICES"] = "1,2"
~~~

![多卡.png](http://upload-images.jianshu.io/upload_images/1575688-da0ecba0f0fb6b96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面是正常状态
![正常状态.png](http://upload-images.jianshu.io/upload_images/1575688-00198dcb5409cf01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)