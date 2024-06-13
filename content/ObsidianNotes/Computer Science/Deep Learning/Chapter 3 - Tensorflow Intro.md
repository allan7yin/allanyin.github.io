### Introduction

This starts in chapter 3 of deep learning, where we begin to look at `Tensorflow` and `Keras`. So, what have we learned so far/know? We've seen the fundamental ways in which we train a neural network:

- First, low-level tensor manipulation - the infrastructure that underlies all modern day machine learning. This translates to `Tensorflow` APIs:
    - _Tensors_, including special tensors that store the network's state (variables)
    - _Tensor operations_, such as addition, `relu`, `matmul`
    - _Back propagation_, a way to compute the gradient of mathematical expressions (handled in `Tensorflow` via the `GradientType` object
- Second, high-level deep learning concepts. This translates to `Keras` APIs:
    - _Layers_, which are combined into a model
    - A _loss function_, which defines the feedback signal used for learning
    - An _optimizer_, which determines how learning proceeds
    - _Metrics_ to evaluate model performance, such as accuracy
    - A _training loop_ that performs mini-batch stochastic gradient descent

---

### Constant tensors and variables

```Python
import tensorflow as tf

# all-ones tensor
x = tf.ones(shape = (2,1)) # all-ones tensor (2x1 matrix in this case)
print(x)

# random tensors
x = tf.random.normal(shape=(3, 1), mean=0., stddev=1.)
```

`TensorFlow` tensors can be thought of as multidimensional arrays. So, can we go in and manually adjust the values inside of a `TensorFlow` tensor? **no**. Unlike something like `NumPy`, tensors in `TensorFlow` are constants.

```Python
import numpy as np
x = np.ones(shape=(2, 2))
x[0, 0] = 0. # this is fine, "." after 0 indicates we are working with floating point

# now, what if we try this with tensorflow array, we will get an error
x = tf.ones(shape=(2, 2))
x[0, 0] = 0.
```

To train a model, we'll need to update its state, which is a set of tensors. So, these need to be mutable, hence, we use variables. To create a variable, you need to provide some initial value, such as a random tensor.

```Python
v = tf.Variable(initial_value=tf.random.normal(shape=(3, 1)))

# is
array([[-0.75133973],
       [-0.4872893 ],
       [ 1.6626885 ]], dtype=float32)

v.assign(tf.ones((3, 1)))
# is
array([[1.],
       [1.],
       [1.]], dtype=float32)

v[0, 0].assign(3.)
# is
array([[3.],
       [1.],
       [1.]], dtype=float32)
```

Also, `TensorFlow` is capable of performing math operations, like `NumPy`:

```Python
a = tf.ones((2, 2))
b = tf.square(a) # square of a tensor, element-wise
c = tf.sqrt(a) # square root of a tenso, element-wise
d=b+c # element-wise addition
e = tf.matmul(a, b) # tensor products
e *= d # element-wise multiplication
```

But, what makes `TensorFlow` different from `NumPy`? But here’s something NumPy can’t do: retrieve the gradient of any differentiable expression with respect to any of its inputs.

```Python
input_var = tf.Variable(initial_value=3.) with tf.GradientTape() as tape:
   result = tf.square(input_var)
gradient = tape.gradient(result, input_var)

## here is an easier example to understand
import tensorflow as tf

x = tf.Variable(3.0)
with tf.GradientTape() as tape:
    y = tf.square(x)
dy_dx = tape.gradient(y, x) # derivative of y = x^2, x = 3, we get 6
```

It is actually possible for these inputs to be any arbitrary tensor. However, only trainable variables are tracked by default. With a constant tensor, you would have to manually mark it as being tracked by calling `tape.watch()` on it.

```Python
input_const = tf.constant(3.) with tf.GradientTape() as tape:
   tape.watch(input_const)
   result = tf.square(input_const)
gradient = tape.gradient(result, input_const)
```

The gradient tape is a powerful utility, even capable of computing _second-order gradients_:

```Python
time = tf.Variable(0.)
with tf.GradientTape() as outer_tape:
with tf.GradientTape() as inner_tape: position = 4.9 * time ** 2
    speed = inner_tape.gradient(position, time)
acceleration = outer_tape.gradient(speed, time)
```