---
title: "NLP-CBOW"
description: 
date: 2022-10-14T16:58:12+08:00
hidden: false
comments: true
draft: false

---
**CBOW**

Import Library
{{< highlight python >}}
from tensorflow import keras
import tensorflow as tf
##these two can find in mofan python##
from utils import process_w2v_data
from visual import show_w2v_word_embedding
##these two can find in mofan python##
{{< /highlight >}}


Define CBOW
{{< highlight python >}}
class CBOW(keras.Model):
    def __init__(self,v_dim,emb_dim):
        super().__init__()
        self.v_dim = v_dim
        self.embeddings = keras.layers.Embedding(
            input_dim = v_dim, output_dim = emb_dim,
            #input words num as v_dim, output embedding dimension as emb_dim
            embeddings_initializer = keras.initializers.RandomNormal(0.,0.1)
            #random num from normal distribution, mean=0, std=0.1
        )
        self.nce_w = self.add_weight(
            name = 'nce_w',shape=[v_dim,emb_dim],
			#shape=[in,out]
            initializer = keras.initializers.TruncatedNormal(0.,0.1)
			#Truncated Normal Distribution
        )
        self.nce_b = self.add_weight(
			#easier than tf.Variable
            name = 'nce_b',shape=(emb_dim,),
			#only 1 dimension
            initializer = keras.initializers.Constant(0.1)
        )
        self.opt = keras.optimizers.Adam(0.01)

    def call(self,x,training=None,mask=None):
        o = self.embeddings(x) 
		#x.shape=[n,skip_window*2], o.shape=[n, skip_window*2, emb_dim]
		#[dim3,dim2,dim1]
        o = tf.reduce_mean(o,axis=1) 
		#mean of dimension 1, o.shape=[n, emb_dim]
        return o

    def loss(self,x,y,training=None):
        embedded = self.call(x,training)
        return tf.reduce_mean(
            tf.nn.nce_loss(
            weights = self.nce_w, biases = self.nce_b, labels = tf.expand_dims(y,axis=1),
            inputs = embedded, num_sampled=5, num_classes=self.v_dim)
        )

    def step(self,x,y):
        with tf.GradientTape() as tape:
            loss = self.loss(x,y,True)
            grads = tape.gradient(loss,self.trainable_variables)
        self.opt.apply_gradients(zip(grads, self.trainable_variables))
        return loss.numpy()
{{< /highlight >}}

Train Function
{{< highlight python >}}
def train(model,data):
    for t in range(2500): 
		#2500 steps
        bx,by = data.sample(8)
		#chose 8 of the data
        loss = model.step(bx,by)
        if t%200 == 0:
            print("step: {} | loss: {}".format(t, loss))
{{< /highlight >}}


Run
{{< highlight python >}}
if __name__=="__main__":
    d = process_w2v_data(corpus,skip_window=2,method="cbow")
	#use the former and the later 2 words to predict
    m = CBOW(d.num_word, 2)
	#you can chage emb_dim to any num you want
    train(m,d)
    show_w2v_word_embedding(m, d, "./cbow.png")
{{< /highlight >}}


**Refrence**
https://mofanpy.com/tutorials/machine-learning/nlp/requirements

https://developer.aliyun.com/article/837200

https://blog.csdn.net/qq_35632833/article/details/103649578

https://blog.csdn.net/my_name_is_learn/article/details/109516945
