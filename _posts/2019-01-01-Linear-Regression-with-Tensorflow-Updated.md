Yaay!!! Welcome to the new year 2019, this is going to be my first post in the year, I am glad about it as I get to start the year on a very high vibe. 

To the matter on ground, Data Science and Machine learning has come a long way and still very much evolving fast, one of the tool that has helped in great advancement in Machine learning especially is Tensorflow, I last posted about starting out with Tensorflow with the most basic of example - building a linear regression model, however a lot has change. Some of things that have changed include the following:

* Tensorflow now uses tf.keras as it's base model definition
* Tensorflow uses `eager execution` as it's default approach to running your models, rather than the initial `graph and session` approach
* A lot of clean-ups are also on the way in `tf.slim` api of Tensorflow, basically they seems to have been merged to the core api.
* Tensorflow now feels more python-like than the initial C-like presentation it forces developer into.

However, with this comes a new way to approach working with Tensorflow, I made some examples on how to quickly get started with Tensorflow in the `Eager Execution` mode. This mode is actually painless compared to what we had in Tensorflow even exactly 1 year ago.

It allows to build an imperative, define-by-run approach to doing machine learning, and it has auto differentiation that allows you to automatically calculate the gradient of your forward pass operations.

**Basic Operations With Tensorflo**

Let's go into how to do some basic operations in Tensorflow. It is important to remember that Tensorflow is a library that allows you to do `numerical computation`, so many times, people have had to forget this notion and only see Tensorflow as a Deep Neural Network Library, this conciousness need to be allowed to settle in for anybody that wants to use Tensorflow to maximize data. I believe this is the reason why every Tensorflow basic tutorial starts with a Linear Regression, so that the `numerical computation` foundation of Tensorflow is not forgotten.

To begin tensorflow in eager execution, it's imperative that we import tensorflow and establish that we want to use eager execution at the top of our application. Once enabled once, it's enabled for the run-time of the current application.

```python
import tensorflow as tf
tf.enable_eager_execution()
```

a. Do basic Operations like sum and multiplication

```python
#Add and multiply 2 Scalas
print(tf.add(1, 2))
print(tf.multiply(2, 3))

#Add and multiply 2 matrixes.
print(tf.add([[1, 2], 
              [2, 3]], [[5, 4],
                        [6, 7]]))
print(tf.multiply([[1, 2], 
                   [2, 3]], [[5, 4],
                            [6, 7]]))
```

b. Do Square and get mean of Scala and Matrixes


```python
#Square a scalar and Vector
print(tf.square(10))
print(tf.square([[2, 3], [5,4]]))

#Get the mean of 1-D and 2-D array of Number
print(tf.reduce_mean([1, 2, 3, 4, 5, 6, 7, 7,8 , 6, 6, 3, 2, 2]))

print(tf.reduce_mean([[10, 10, 3], 
                      [40, 5, 6], 
                      [5, 6, 6]], axis=0, keepdims=True))
```

c. Get the minimum value in a 1-D and 2-D Array


```python
#Get the minimum value in a 1-D Array
print(tf.reduce_min([1, 2, 3, 4, 5, 6, 7, 7,8 , 6, 6, 3, 2, 2]))


#Get the minimum value in a 2-D Tensor along a particular axis
print(tf.reduce_min([[10, 10, 3], 
                     [40, 5, 6], 
                     [5, 6, 6]], axis=1, keepdims=True))

```

d. Some few other operations
```python
#Generating Random Numbers 
x = tf.random_uniform([4, 4])
print(x)


#Creating Tensor Constants in Eager Execution
y = tf.constant([[10, 10, 3], 
                     [40, 5, 6], 
                     [5, 6, 6]])

print(y)

#Calculate Sigmoid of an Array of Numbers
sig = tf.sigmoid(x)
print(sig)
```

***Full notebook can be found [here](https://colab.research.google.com/github/adekunleba/tensorflow_tutorial/blob/master/IntroductiontoTensorflowOperations.ipynb)***

Let's take the operations a little further and try to build a simple linear regression model with this. I love to think about machine learning projects as a scientist trying to make some data useful, either so that they can build a model that will essentially abstract some thought portion from human, or in all build a product from data that will eventually make life painless for human in terms of helping to solve quite challenging scenerios, for example, building a self driving car could be seen as applying machine learning algorithm to environment data including images and weather data to help the car make it's next decision.

So, we can generate a toy data using tensorflow random number generation techniques that gives us what can be used for training a linear regression model.

Highlights of the code below is to :
* Generate random numbers with tf.random_normal as your x
* Compute a random y for each elements in X 
* Convert the generated data to Tensorflow Dataset.

*Tensorflow Dataset allows you to build a robust data pipeline and ensuring you keep track of the X and y accordingly in your pipeline. tf.data.Dataset represents a sequence of elements where each of the element is a Tensor Object (internal data representation for tensorflow) with a single trianing sample usually having the X and Y. You can think of X as anything from Columns of each entity of your data to images. Another good advantage of tensorflow Dataset is how you are able to batch easily.*

```python

def make_synthetic_data(w, b, noise_level, batch_size, num_batches):
  def batch(_):
    x = tf.random_normal(shape=[batch_size, tf.shape(w)[0]])
    y = tf.matmul(x, w) + b + noise_level + tf.random_normal([])
    return x, y
  
  
  #Use Cpu to do this
  with tf.device("/device:CPU:0"):
    return tf.data.Dataset.range(num_batches).map(batch)

```

With the data generation function, we can go ahead to generate our data

```python
true_w = [[-2.0], [4.0], [1.0]]
true_b = [0.5]
noise_level = 0.01

# Training constants.
batch_size = 64
learning_rate = 0.1 

data = make_synthetic_data(true_w, true_b, noise_level, batch_size, 20)
```

The next procedure is usually to explore your data, however, in our case we are working with a toy example, we can skip that part and move on to the meat of what we are actually here for - how will tensorflow help us build a model against this data.

The approach is highlighted thus:
* Make a model - we use `tf.keras.layers.Dense`. The question that came to mind with this is how does Dense layer represents regression? Dense layer data is usually a flattened array which is synonymous to your normal `X` data with many columns on which you want to try and get your `weight` and `bias`. Furthermore, the loss function used, helps the model to fully understand what we are trying to do.
* Write the loss function, for our sake, it's going to be a Mean Square Error, which is one of the `loss functions` used in training Regression models.
* Write the auto-differentiation algorithm
* Write the optimizer to use to train the model.
* Train your model

```python
layers = tf.keras.layers
model = layers.Dense(1)

mse = lambda xs, ys: tf.reduce_mean(tf.square(tf.subtract(model(xs), ys)))
tfe = tf.contrib.eager
loss_and_grads = tfe.implicit_value_and_gradients(f=mse)

#Build Optimizer
optimizer = tf.train.GradientDescentOptimizer(learning_rate=learning_rate)

#Train the model
for i, (xs, ys) in enumerate(tfe.Iterator(data)):
  loss, grads = loss_and_grads(xs, ys)
  print("Iteration %d: loss = %s" % (i, loss.numpy()))
  
  optimizer.apply_gradients(grads)
```

This approach is the basic minimum in building a machine learning model with Tensorflow, in most case, you will just have to structure your code to be more robust on the above process when dealing with bigger projects such as Image Recognition, Text analysis and other machine learning projects. there is a sample in the accompanying notebook on how to bring the process above together in a python scripts of functions and class.

Some other things you will want to add in-between the codes is using `Tensorboard` to monitor the training procedure and `model checkpointing` to ensure models are saved for use later. 

***The full code for training a Linear regression model can be found [here](https://colab.research.google.com/github/adekunleba/tensorflow_tutorial/blob/master/Linear_Regression_with_Tensorflow.ipynb)***