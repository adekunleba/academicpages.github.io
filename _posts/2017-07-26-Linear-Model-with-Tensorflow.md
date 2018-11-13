Linear Model is a foundational model when it comes to Machine Learning, this simple article is to explore building a simple Linear model with Tensorflow. The basic idea is to lay a foundation of a model that is very important in understanding deep neural network.
Deep Neural Network (DNN) is intuitively getting a good representation of your input data that a model can use to predict rightly the information contained in the data as would a human. Okay, that sounds a little complex, DNN takes your data, which may be image, text and even stock market data, and passes it through layers of neurons - here neurons means just function, such as the Linear function we are trying to model here - and uses this new representations to predict perfectly the information contained in the input data.
### Why Tensorflow?
Tensorflow is one of the many state of the art Deep Learning software Library that makes learning a DNN fast and scalable. It's well optimized and can abstract a lot of codes that would have gone into training a neural network by coding all the mathematics, note that Tensorflow itself is a library for numerical computation. It is built to be scalabel and distributed and can work perfectly in production.

### Linear Regresssion
Linear Regression is the mapping of a function <a href="https://www.codecogs.com/eqnedit.php?latex=$$f(x)$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$f(x)$$" title="$$f(x)$$" /></a> to <a href="https://www.codecogs.com/eqnedit.php?latex=$$R^n$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$R^n$$" title="$$R^n$$" /></a> with <a href="https://www.codecogs.com/eqnedit.php?latex=$$f(x,&space;W,&space;b)$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$f(x,&space;W,&space;b)$$" title="$$f(x, W, b)$$" /></a> where <a href="https://www.codecogs.com/eqnedit.php?latex=$$f(x)&space;=&space;Wx&space;&plus;&space;b$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$f(x)&space;=&space;Wx&space;&plus;&space;b$$" title="$$f(x) = Wx + b$$" /></a> predicts a continous value <a href="https://www.codecogs.com/eqnedit.php?latex=$$R^n$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$R^n$$" title="$$R^n$$" /></a>. <a href="https://www.codecogs.com/eqnedit.php?latex=$$W$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$W$$" title="$$W$$" /></a> is the weight that gives a representation of the data <a href="https://www.codecogs.com/eqnedit.php?latex=$$x$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$x$$" title="$$x$$" /></a> for the predicted value <a href="https://www.codecogs.com/eqnedit.php?latex=$$R^n$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$R^n$$" title="$$R^n$$" /></a> and <a href="https://www.codecogs.com/eqnedit.php?latex=$$b$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$b$$" title="$$b$$" /></a> is the bias that helps to generalize the Weight <a href="https://www.codecogs.com/eqnedit.php?latex=$$W$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$W$$" title="$$W$$" /></a> on the data.

<a href="https://www.codecogs.com/eqnedit.php?latex=$$f(x)&space;=&space;Wx&space;&plus;&space;b&space;$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$f(x)&space;=&space;Wx&space;&plus;&space;b&space;$$" title="$$f(x) = Wx + b $$" /></a>

To minize the error on the function, one will apply gradient descent.

The idea is compute the error from the prediction and minimize the error with gradient descent

<a href="https://www.codecogs.com/eqnedit.php?latex=$$&space;Error&space;=&space;\sum_{i=0}^m&space;\frac{1}{m}(f(x_i)&space;-&space;y_i)^2&space;$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$&space;Error&space;=&space;\sum_{i=0}^m&space;\frac{1}{m}(f(x_i)&space;-&space;y_i)^2&space;$$" title="$$ Error = \sum_{i=0}^m \frac{1}{m}(f(x_i) - y_i)^2 $$" /></a>

For every iteration over data sample (Batch Gradient Descent)
Gradient Descent intuitively decrease the prediction error and optimize the weight to generalize on all samples by multiplying the prediction erorr (<a href="https://www.codecogs.com/eqnedit.php?latex=$$f(x)-y$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$f(x)-y$$" title="$$f(x)-y$$" /></a>) with the data sample (of that case), scaled by an alpha over all the training cases and finally deducting this new value from weight of previous iteration

Putting it into equation

<a href="https://www.codecogs.com/eqnedit.php?latex=$$&space;\nabla{W}&space;=&space;W_j&space;-&space;\alpha&space;\frac{1}{m}&space;\sum_{i=0}^m(f(x_i)&space;-&space;y_i)x_i&space;$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$&space;\nabla{W}&space;=&space;W_j&space;-&space;\alpha&space;\frac{1}{m}&space;\sum_{i=0}^m(f(x_i)&space;-&space;y_i)x_i&space;$$" title="$$ \nabla{W} = W_j - \alpha \frac{1}{m} \sum_{i=0}^m(f(x_i) - y_i)x_i $$" /></a>

### Coding it up:
**1. Import Required Packages and data**
```python
import os
os.environ['TF_CPP_MIN_LOG_LEVEL']='2'

import numpy as np
import matplotlib.pyplot as plt
import tensorflow as tf
import xlrd
import pandas as pd

import python_utils

DATA_FILE = 'data/fire_theft.xls'
```
**2. Read in Data using xlrd and specify learning rate**
```python
book = xlrd.open_workbook(DATA_FILE, encoding_override='utf-8')
sheet = book.sheet_by_index(0)
data = np.asarray([sheet.row_values(i) for i in range(1, sheet.nrows)])
n_samples = sheet.nrows - 1

train_X = data[:, 0].astype(np.float32)
train_Y= data[:, 1].astype(np.float32)
print(train_X.dtype)

n_sample = data.shape[0]
learning_rate = 0.001
```
**3. Define Tenforflow objects that will compute the model**
 * For input, weigth, bias and target
 
```python
 X_n = tf.placeholder(np.float32) #specifing shape=[n_samples] worked just fine
 Y_n = tf.placeholder(np.float32)

 W = tf.Variable(0., name="W")
 b = tf.Variable(1., name="b")
```

 * Computing  and minimize the error by computing gradient from scratch and try to descent it to the minimum error
 
 ```python
 prediction_n = X_n * W + b
 error = prediction_n - Y_n
 mse = tf.reduce_sum(tf.square(error))

 gradient = 1/n_sample * (tf.reduce_sum(tf.multiply(X_n, error)))
 training_op = tf.assign(W, W - learning_rate * gradient)
 ```
**4. Train the model**
```python
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for i in range(10):
        mse_val, _ = sess.run([mse, training_op], feed_dict={X_n:train_X, Y_n:train_Y})
        print(mse_val)
    best_theta = W.eval()
    print("Best Theta is: ", W.eval())
```
Okay so we are done training a Linear Model with Tensorflow. However, we can save ourselves from the headeache of the gradient descent Mathematics, this is where the power of Tensorflow really comes to play.
**5. Replace Step 3 with Tenforflow optimizer**
```python
prediction_n = tf.add(tf.multiply(X_n, W), b)
error = prediction_n - Y_n
mse = tf.reduce_mean(tf.square(error))
optimizer = tf.train.GradientDescentOptimizer(learning_rate=learning_rate).minimize(mse)
```
**6. Retrain the model with Tensorflow Optimizer**
```python
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for i in range(10):
        mse_val, _ = sess.run([mse, optimizer], feed_dict={X_n:train_X, Y_n:train_Y})
        print(mse_val)
```
## And We are Truly Done!!!

### Key Tensorflow Issues to note:
* ```tf.matmul()``` will only multiply a 2-D tensor hence for dot product multiplication between two 1-D Tensor, the available approach is to use ```tf.reduce_sum(tf.multiply(Vec_1, Vec_2))```
* ```tf.train.*Optimizers*()``` are well optimized to take your loss function and train it quite well. In first example, even while training on the examples one by one, the optimizer is able to yet optimize the loss function since it knows what to combine to give the necessary output
* You can evaluate any of the Variables while the training is going on or is concluded consider the _Best_theta_ part

Once you define the *Error* you can then call optimization functions to minize the error, Different optimization techniques are in Tensorflow that you can make use of, and they make the work a little easier
Examples of optimizer that exists
* AdaDeltaOptimizer        
* AdagradDAOptimizer
* AdagradOptimizer
* AdamOptimizer
* FtrlOptimizer
* GradientDescentOptimizer
* MomentumOptimizer
* ProximalGradientDescentOptimizer
* ProximalAdagradOptimizer
* RMSPropOptimizer
* SyncReplicasOptimizer

Understanding the process of building a linear model is important in overall Deep Learning Networks this is because, Linear units - that contains basic afine transformation: **<a href="https://www.codecogs.com/eqnedit.php?latex=$$y&space;=&space;Wx&space;&plus;&space;b$$" target="_blank"><img src="https://latex.codecogs.com/gif.latex?$$y&space;=&space;Wx&space;&plus;&space;b$$" title="$$y = Wx + b$$" /></a>** are core to Deep Learning. Infacts, it is the basic building block for first ever Neural Network which is Perceptron.
