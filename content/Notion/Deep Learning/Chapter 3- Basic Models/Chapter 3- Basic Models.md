### Linear Classifier - Pure tensorflow + Keras intro

Now, we will start by building a Linear Classifier as a quick demonstration of `TensorFlow` capabilities.

**Quick Note**: Remember how "learning works". Once we get the loss function, we calculate it's gradient. Recall the gradient is a vector built from the partial derivatives of the multi-variable function. That is the direction of the greatest ascent. As a we are trying to minimize loss, we move in the opposite direction, the negative of it. This is why later on, you will see us do `W.assign_sub(grad_loss_wrt_W * learning_rate)` and `b.assign_sub(grad_loss_wrt_b * learning_rate)`.

**=> Here is what np.vstack does:**

```Python
array1 = np.array([[1, 2, 3],
                   [4, 5, 6]])

array2 = np.array([[7, 8, 9],
                   [10, 11, 12]])

result = np.vstack((array1, array2))

# this results in

[[ 1  2  3]
 [ 4  5  6]
 [ 7  8  9]
 [10 11 12]]
```

A linear classifier is an affine transformation `(prediction=W•input+b)` trained to minimize the square of the difference between predictions and the targets. The expression `prediction = W•input + b` represents the forward pass or the output computation of a linear layer in a neural network or linear regression model.

The full code for this can be found on my GitHub. This note will go over the ideas/knowledge. We create this with pure tensorflow, then after, show how we use Keras to reduce many steps in our code.

First, we can make some mock data:

```Python
num_samples_per_class = 1000

negative_samples = np.random.multivariate_normal(
    mean=[0, 3],
    cov=[[1, 0.5],[0.5, 1]],
    size=num_samples_per_class)

positive_samples = np.random.multivariate_normal(
    mean=[3, 0],
    cov=[[1, 0.5],[0.5, 1]],
    size=num_samples_per_class)

inputs = np.vstack((negative_samples, positive_samples)).astype(np.float32)

targets = np.vstack((np.zeros((num_samples_per_class, 1), dtype="float32"), np.ones((num_samples_per_class, 1), dtype="float32")))
```

This creates 2 "classes" of 2D points. We then make a vertical stack with them, forming all of our input data, `inputs`. We also need `targets` which will be the corresponding expected value for each of the inputs.

Now, let's look at how we can define the forward pass:

```Python
input_dim = 2
output_dim = 1
W = tf.Variable(initial_value=tf.random.uniform(shape=(input_dim, output_dim)))
b = tf.Variable(initial_value=tf.zeros(shape=(output_dim,)))

def model(inputs):
  return tf.matmul(inputs, W) + b
```

Now, we know that the `inputs` tensor is a `2000 x 2` tensor. So, as we want the resultant tensor of this layer to be a `2000 x 1` tensor, we need `W` to be a `2 x 1` tensor. `b` is simply a scalar (broadcasted to element-wise).

So, this represents a single pass through this layer. Now, lets look at the loss function and training step:

```Python
def square_loss(targets, predictions):
  per_sample_losses = tf.square(targets - predictions)
  return tf.reduce_mean(per_sample_losses)

learning_rate = 0.1

def training_step(inputs, targets):
  with tf.GradientTape() as tape:
    predictions = model(inputs)
    loss = square_loss(targets, predictions)
  grad_loss_wrt_W, grad_loss_wrt_b = tape.gradient(loss, [W, b])
  W.assign_sub(grad_loss_wrt_W * learning_rate)
  b.assign_sub(grad_loss_wrt_b * learning_rate)

  return loss
```

Lets break this down. `square_loss` is the loss function. This is a pretty standard way of defining a loss function. We perform element-wise squaring of the different between predict and target tensor. This way, errors become "amplified", and we remove negatives (as we don't really care about negatives, only that it has a loss value). We then get the mean over all values in the tensor and return it.

The function below is a `training_step`. What this does is it finds the gradient of the model function, and moves the values of `W` and `b` in the opposite direction of the gradient with respect to those variables with the learning rate.

To see the results/effectiveness of this model, we can:

```Python
for step in range(40):
  loss = training_step(inputs, targets)

predictions = model(inputs)
plt.scatter(inputs[:, 0], inputs[:, 1], c=predictions[:, 0] > 0.5)
plt.show()
```

Here, we have run the training for 40 epochs. After that, we see what the layer is able to output given the input, and display it.

---

## Keras

So, we've seen a nice example of how we can use Tensorflow to implement a simple linear classifier. This brings us into using Keras. So, what is Keras? Keras is library that is built on top of Tensorflow. It makes a lot of the processes of writing machine learning code simpler by building abstractions on top of the Tensorflow framework. The fundamental building block of Keras, are `Layers`, which is a class defined by Keras. All layers we create in our neural network will inherit from this class. These layers often have a state: weights, which one or many tensors learned with stochastic gradient descent, which together, contain the network's knowledge.

![[Screenshot_2024-01-09_at_12.35.01_PM.png]]

Different types of layers are appropriate for different tensor formats and different types of data processing. For instance, simple vector data, stored in rank-2 tensors of shape (samples, features), is often processed by densely connected layers, also called fully connected or dense layers (the Dense class in Keras). **Sequence data**, stored in rank-3 tensors of shape (samples, timestamps, features), is **typically processed by recurrent layers, such as an LSTM layer**, or **1D convolution layers (Conv1D)**. Image data, stored in rank-4 tensors, is usually processed by **2D convolution layers (Conv2D)**.

  

Recall the layer we made in pure tensorflow. Now, here is what it would look like in Keras:

```Python
from tensorflow import keras

class SimpleDense(keras.layers.Layer):                                      # all layers inherit from the Layer base class 
  def __init__(self, units, activation=None): super().__init__()
    self.units = units
    self.activation = activation

  def build(self, input_shape):
    input_dim = input_shape[-1]
    self.W = self.add_weight(shape=(input_dim, self.units),
                             initializer="random_normal")
    self.b = self.add_weight(shape=(self.units,),
                               initializer="zeros")
  def call(self, inputs):
    y = tf.matmul(inputs, self.W) + self.b if self.activation is not None:
    y = self.activation(y) return y

  # once this layer has been defined, we can use it like a function. This is because the Layer parent class
  # overrides the __call__() method, which then needs the call() method, which we defined in our layer

my_dense = SimpleDense(units=32, activation=tf.nn.relu)
input_tensor = tf.ones(shape=(2, 784))
output_tensor = my_dense(input_tensor)
print(output_tensor.shape)
# >> (2,32)
```

This introduces an important feature of Keras, automatic shape inference.

---

Consider the following code:

```Python
from tensorflow.keras import layers
layer = layers.Dense(32, activation="relu")
```

This layer will return a tensor where the first dimension has been transformed to be 32. It can only be connected to a downstream layer that expects 32-dimensional vectors as its input.  
When using Keras, you don’t have to worry about size compatibility most of the time, because the layers you add to your models are dynamically built to match the shape of the incoming layer.  

In chapter 2, we saw an example dense layer:

```Python
model = NaiveSequential([
    NaiveDense(input_size=784, output_size=32, activation="relu"), # output size is also the number of neurons in the layer 
    NaiveDense(input_size=32, output_size=64, activation="relu"),
    NaiveDense(input_size=64, output_size=32, activation="relu"),
    NaiveDense(input_size=32, output_size=10, activation="softmax")
])
```

This is less than ideal as for every dense layer, it needs to the know the shape of the layer before it.

In `SimpleDense`, we no longer create weights in the constructor like in the `Naive- Dense` example; instead, we create them in a dedicated state-creation method, `build()`, which receives as an argument the first input shape seen by the layer. The `build()` method is called automatically the first time the layer is called (via its `__call__()` method). In fact, that’s why we defined the computation in a separate `call()` method rather than in the `__call__()` method directly.

With automatic shape inference, our previous example becomes simple and neat:

```Python
model = keras.Sequential([
    SimpleDense(32, activation="relu"),
    SimpleDense(64, activation="relu"),
    SimpleDense(32, activation="relu"),
    SimpleDense(10, activation="softmax")
])
```

When solving real-world problems with deep-learning, the way in which we build our models is important. So far, in the above, we've seen simple sequential models, where we are just stacking layers on top of each other.

To learn from data, you have to make assumptions about it. These assumptions define what can be learned. As such, the structure of your hypothesis space—the architecture of your model—is extremely important. It encodes the assumptions you make about your problem, the prior knowledge that the model starts with.

Picking the right network architecture is more an art than a science, and although there are some best practices and principles you can rely on, only practice can help you become a proper neural-network architect. We want to see explicit principles for building neural networks and develop intuition as to what works or doesn’t work for specific problems.

---

Now, let's return to looking at Keras functionality. Once the model's architecture has been defined, there are 3 things left that we need:

1. _Loss function_: We aim to minimize this value through training
2. _Optimizer_: Determines how the network will be updated based on the loss function value. It implements a specific variant of the stochastic gradient descent (SGD)
3. _Metrics_: The measures of success we want to monitor during training and validation, such as classification accuracy.

Once these have been picked or defined, we can use the built in `compile()` and `fit()` methods to start training the model. We could also write our own custom training loops, which we will explore later on.

```Python
model = keras.Sequential([keras.layers.Dense(1)])
model.compile(optimizer="rmsprop",
              loss="mean_squared_error",
              metrics=["accuracy"])
```

The above declares a linear classifier in the first step. Inside of the `compile()` method, we specify the `optimizer`, `loss function`, and `metrics`.

We can use strings. Personally, I would use the actual objects from Keras, such as `keras.optimizers.RMSprop()`.

```Python
model.compile(optimizer=keras.optimizers.RMSprop(),
              loss=keras.losses.MeanSquaredError(),
              metrics=[keras.metrics.BinaryAccuracy()])
```

This is the way you will have to do it if you want to make custom losses or metrics or if we want to specify the learning rate to the optimizer.

```Python
model.compile(optimizer=keras.optimizers.RMSprop(learning_rate=1e-4), loss=my_custom_loss,
              metrics=[my_custom_metric_1, my_custom_metric_2])
```

There are many optimizers and loss functions to choose from. We'll see later on that for certain problems, there are patterns as to what loss functions and what optimizers to choose. Fortunately, when it comes to common problems such as classification, regression, and sequence prediction, there are simple guidelines you can follow to choose the correct loss. For instance, you’ll use binary crossentropy for a two-class classification problem, categorical crossentropy for a many-class classification problem, and so on. Only when you’re working on truly new research problems will you have to develop your own loss functions. In the next few chapters, we’ll detail explicitly which loss functions to choose for a wide range of common tasks.

So, what does `fit()` method do? The `fit()` method implements the training loop itself. It's key arguments include:

1. _data_: consists of inputs and targets
2. _epochs_
3. _batch-size_

```Python
 history = model.fit(
            inputs,
            targets,
            epochs=5,
            batch_size=128
)
```

```Python
>>> history.history
    {"binary_accuracy": [0.855, 0.9565, 0.9555, 0.95, 0.951],
     "loss": [0.6573270302042366,
             0.07434618508815766,
             0.07687718723714351,
             0.07412414988875389,
             0.07617757616937161]}
```

Now, whenever we are building a model like this, we often reserve a part of the data for validation. When training, we run the risk of over-training, where the model has essentially "memorized" the training set. What we care about is the model's effectiveness on new data. The following illustrates this:

```Python
model = keras.Sequential([keras.layers.Dense(1)]) model.compile(optimizer=keras.optimizers.RMSprop(learning_rate=0.1),
loss=keras.losses.MeanSquaredError(),
              metrics=[keras.metrics.BinaryAccuracy()])

indices_permutation = np.random.permutation(len(inputs))
shuffled_inputs = inputs[indices_permutation]
shuffled_targets = targets[indices_permutation]

num_validation_samples = int(0.3 * len(inputs))
val_inputs = shuffled_inputs[:num_validation_samples]
val_targets = shuffled_targets[:num_validation_samples]
training_inputs = shuffled_inputs[num_validation_samples:]
training_targets = shuffled_targets[num_validation_samples:]

model.fit(
  training_inputs,
  training_targets,
  epochs=5,
  batch_size=16,
  validation_data=(val_inputs, val_targets)
)
```

The value of the loss on the validation data is called the “validation loss,” to distin- guish it from the “training loss.” Note that it’s essential to keep the training data and validation data strictly separate: the purpose of validation is to monitor whether what the model is learning is actually useful on new data. If any of the validation data has been seen by the model during training, your validation loss and metrics will be flawed.

Note that if you want to compute the validation loss and metrics after the training is complete, you can call the evaluate() method:

```Plain
loss_and_metrics = model.evaluate(val_inputs, val_targets, batch_size=128)
```

Now, we don't want to usee the `call()` method for validation data, called an `inference`. A better way to do inference is to use the `predict()` method. It will iterate over the data in small batches and return a NumPy array of predictions. And unlike `__call__()`, it can also process TensorFlow Dataset objects.

```Plain
predictions = model.predict(new_inputs, batch_size=128)
```