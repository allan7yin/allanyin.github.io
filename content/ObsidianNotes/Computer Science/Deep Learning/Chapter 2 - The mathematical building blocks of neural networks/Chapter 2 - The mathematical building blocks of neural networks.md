Deep learning is built on top of the idea of neural networks. What are neural networks? Imagine these as layers of tensor operations. Often, each layer of a neural network is simply an affine function on the input tensor following by an activation function.

- **Activation Function:** This is a mathematical operation applied to the output of each neuron in a neural network, as it introduces non-linearity to the network. This is a **very important** thing to do. They decide if a neuron should be activated (some element in a tensor) — output a non-zero value, or not. **Why do we need this?** Activation functions allow the network to learn from and model complex patterns in the data. Layers of only affine functions results in an overall affine mapping. So, this means many affine layers, is the same as just one single layer, which is just a basic **linear model**. With activation functions, these affine layers can be made to implement very complex, non-linear geometric transformations.

The next important thing to know about is how the **“training”** of a neural network works. A NN learns by tweaking the model parameters, a.k.a, the values of elements of weights of the layer (weights are tensors). How do we do this? We do this through examining the **loss function**. Say we pass an input tensor through our NN. Then, we compare this to our expected output, and compute a loss value. We want to move our weights in a way that minimizes this loss value.

  

**→ SGD**

SGD stands for stochastic gradient descent. Mathematically, we calculate the gradient of the loss function, which gives us a vector of partial derivatives in the direction of steepest ascent. So, we want to move model weights in the direction opposite of the gradient. While given a loss function, it is mathematically possible to find the gradient. Can we find the absolute minimum with derivatives instantly? If we have maybe 2 or 3 parameters, sure. But, most models have thousands or millions of parameters, and computing the derivative for this would be intractable. So, we find the SGD and move the parameters until we have a small enough loss value.

![[Screenshot_2024-01-08_at_10.06.57_PM.png]]

Consider the above diagram. We have the the loss function plotted. To move our loss towards minimum, we use SGD. There are 3 main types:

- **True SGD:** In which we take each sample in the data set, and adjust the weights after a pass through
- **Mini-Batch SGD (Most Common One):** In which draw a batch of data, with each sample being randomly drawn (hence the “stochastic” part), the dataset is divided into smaller batches, and the model parameters are updated based on the average gradient computed over a mini-batch of data.
- **Batch gradient descent:** This is the same as above, except we do this with the **entire** dataset instead of smaller batches

  
This idea is formalized in the  
**back-propagation algorithm.** The idea is, to find the gradient of the loss function, which is basically a function measuring difference from expected output to the output of the NN, we can apply **chain rule from calculus:**

![[Screenshot_2024-01-09_at_10.28.11_AM.png]]

![[Screenshot_2024-01-09_at_10.28.01_AM.png]]

This idea is **excellently** explained by **3Blue1Brown:**

![[Screenshot_2024-01-28_at_3.46.58_PM.png]]

> [!info] Backpropagation calculus | Chapter 4, Deep learning  
> Help fund future projects: https://www.  
> [https://www.youtube.com/watch?v=tIeHLnjs5U8&list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi&index=4&ab_channel=3Blue1Brown](https://www.youtube.com/watch?v=tIeHLnjs5U8&list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi&index=4&ab_channel=3Blue1Brown)  

Luckily, we don’t need to implement this in modern day ML. Modern day NN frameworks such as TensorFlow, support **automatic differentiation**. This is implemented with computation graphs like the ones above. We use a `Gradient Tape` in TensorFlow:

```Python
x = tf.Variable(tf.random.uniform((2, 2))) 
with tf.GradientTape() as tape:
	y=2*x+3
grad_of_y_wrt_x = tape.gradient(y, x)
```

So now, we have a good idea of what is happening inside this black box of a NN, and the diagram from earlier is now clear:

![[Screenshot_2024-01-09_at_10.52.23_AM.png]]

This entire process is shown with just a few lines of keras:

```Python
# Load Data
(train_images, train_labels), (test_images, test_labels) = mnist.load_data() train_images = train_images.reshape((60000, 28 * 28))
train_images = train_images.astype("float32") / 255
test_images = test_images.reshape((10000, 28 * 28))
test_images = test_images.astype("float32") / 255

# Define Model
model = keras.Sequential([
    layers.Dense(512, activation="relu"),
    layers.Dense(10, activation="softmax")
])

# Compile Model -> define optimizer, loss function
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

# Training Loop
model.fit(train_images, train_labels, epochs=5, batch_size=128)
```

After these 5 epochs, the model will have performed 2,345 gradient updates (469 per epoch).

  

We know that a layer in a NN applies an affine transformation on the input and then performs an activation function. In Keras, the `Dense` layer is:

```Python
output = activation(dot(W, input) + b)
```

Let’s look closely at how this layer is implemented in TensorFlow. We’ll make a class that creates 2 TensorFlow variables `W` and `b`, and exposes a `__call()__` method that applies the preceding transformation.

```Python
import tensorflow as tf

class NaiveDense:
		def __init__(self, input_size, output_size, activation):
				self.activation = activation
		
				w_shape = (input_size, output_size)
				w_initial_value = tf.random.uniform(w_shape, minval=0, maxval=1e-1)
				self.W = tf.Variable(w_initial_value)
		
				b_shape = (output_size,
				b_initial_value = tf.zeros(b_shape)
				self.b = tf.Variable(b_initial_value)
		
		def __call__(self, inputs)::
				return self.activation(tf.matmul(inputs, self.W) + self.b)
		
		@property
		def weights(self):
				return [self.W, self.b]
```

---

### Summary of Chapter 2:

- _Tensors_ form the foundation of modern machine learning systems. They come in various flavours of `dtype`, `rank`, and `shape`.
- Deep learning models consist of chains of simple tensor operations, parameterized by _weights_, which are themselves tensors. The weights of a model are where its “knowledge” is stored.
- _Learning_ means finding a set of values for the model’s weights that minimizes a _loss function_ for a given set of training data samples and their corresponding targets.
- Learning happens by drawing random batches of data samples and their targets, and computing the gradient of the model parameters with respect to the loss on the batch. The model parameters are then moved a bit (the magnitude of the move is defined by the learning rate) in the opposite direction from the gradient. This is called _mini-batch stochastic gradient descent_.
- The entire learning process is made possible by the fact that all tensor operations in neural networks are differentiable, and thus it’s possible to apply the chain rule of derivation to find the gradient function mapping the current parameters and current batch of data to a gradient value. This is called _back-propagation_.