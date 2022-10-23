---
title: "Tensorflow"
description: 
date: 2022-10-14T16:58:12+08:00
hidden: false
comments: true
draft: true

---
{{< highlight python >}}
import tensorflow as tf

def add_layer(inputs, in_size, out_size,activation_function=None):
    Weights = tf.Variable(tf.random.normal([in_size,out_size]))#insize rows, outsize columns
    biases = tf.Variable(tf.zeros([1,out_size])+0.1)#recommand not full zero
    Wx_plus_b = tf.matmul(inputs,Weights)+biases
    if activation_function is None:
        outputs = Wx_plus_b
    else:
        outputs = activation_function(Wx_plus_b)
    return outputs
{{< /highlight >}}



**Refrence**
https://mofanpy.com/tutorials/machine-learning/tensorflow/intro-NN

https://blog.csdn.net/m0_37605642/article/details/98854753

https://blog.csdn.net/QAQIknow/article/details/118858870

https://github.com/keras-team/keras#release-and-compatibility

https://la60312.medium.com/setup-cuda-11-with-rtx3080-for-deep-learning-554aa4ff1e76