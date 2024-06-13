### Working with Keras: A deep dive

In this chapter, we look more closely at the capabilities of keras. So far, we have only used the `Sequential` model, where it is a very simplistic layer-by-layer structure. This kind of design is condusive to:

- one input and one output
- relatively simple machine learning problems

But, they are far from enough from being able to handle other types of problems we want to solve with machine learning. Hence, we introduce the `Functional` api. Consider the following `Sequential` model:

```Python
from tensorflow import keras
from tensorflow.keras import layers
model = keras.Sequential([
    layers.Dense(64, activation="relu"),
    layers.Dense(10, activation="softmax")
])
```

We could also use the `.add()` method as well:

```Python
model = keras.Sequential()
model.add(layers.Dense(64, activation="relu"))
model.add(layers.Dense(10, activation="softmax"))
```

What if we want to see the shape of our layers? Recall that the weights are only created after the model is built. We can do this with `.build()` method, and then view the layers with `.summary()`. When you call `model.build()`, Keras initializes the model's weights and biases and creates the necessary layers based on the provided input shape. It determines the shape of the output based on the layers and connections defined in the model. The input shape provided should match the shape of the input data that will be fed into the model during training or inference.:

```Python
import tensorflow as tf
from tensorflow.keras import layers

# Define your model
model = tf.keras.Sequential()
model.add(layers.Dense(64, activation='relu', input_shape=(10,)))
model.add(layers.Dense(1, activation='sigmoid'))

# Build the model
model.build(input_shape=(None, 3)) #  tensor of shape (None, 3) typically represents a multi-dimensional array with an unspecified number of rows, but with a fixed number of columns equal to 3.

# basicaly, shape (None, 3) means we have any number of rows, but there is always 3 columns

# Print the model summary
model.summary()
```

What if we don't call `.build()` explicitly? If you don't call model.build() explicitly, Keras will automatically build the model when you fit the model with data or make predictions. If we wanted to, we could even provide names to our layers, like so:

```Python
model = keras.Sequential(name="my_example_model")
model.add(layers.Dense(64, activation="relu", name="my_first_layer"))
model.add(layers.Dense(10, activation="softmax", name="my_last_layer"))
model.build((None, 3))
```

So, we will see the following printed:

```Python
Model: "my_example_model"
_________________________________________________________________
  Layer (type)                Output Shape             Param #
=================================================================
my_first_layer (Dense)         (None, 64)                256
_________________________________________________________________
my_last_layer (Dense)          (None, 10)                650
=================================================================
Total params: 906
Trainable params: 906
Non-trainable params: 0
_________________________________________________________________
```

> the number of parameters for a layer in a neural network is the total number of entries in the layer's weight matrix, including both weights and biases.

> In a dense (fully connected) layer with 32 neurons, each connected to an input tensor of shape (10, 20), the weight matrix would have dimensions (20, 32), resulting in **20 × 32 = 640 weights**.  
> Additionally, you have one bias term for each of the 32 neurons, adding another 32 parameters. So, in total, you have  
> **640+32=672 parameters** in this layer.

When building a Sequential model incrementally, it’s useful to be able to print a summary of what the current model looks like after you add each layer. Now, we jump to the crux of this section.

---

### The Functional API

![[Screenshot_2024-01-21_at_12.38.45_PM.png]]

The Sequential model is easy to use, but its applicability is extremely limited: it can only express models with a single input and a single output, applying one layer after the other in a sequential fashion. In practice, it’s pretty common to encounter models with multiple inputs (say, an image and its metadata), multiple outputs (different things you want to predict about the data), or a nonlinear topology.  
In such cases, you’d build your model using the Functional API. This is what most Keras models you’ll encounter in the wild use. It’s fun and powerful—it feels like playing with LEGO bricks.  

Consider the following example:

```Python
inputs = keras.Input(shape=(3,), name="my_input")
# shape = (3,0) is the same as (None, 3)

# (None, 3) means any tensor can come in, as long as last parameter is 3
# so, (7,3), (1, 5 , 3), (1, 1, 1, 1, 1, 3), are ALL valid
features = layers.Dense(64, activation="relu")(inputs)
outputs = layers.Dense(10, activation="softmax")(features)
model = keras.Model(inputs=inputs, outputs=outputs)
```

> In Keras, keras.Input() is a function that is used to create an input tensor or input layer for a neural network model. It returns a **symbolic tensor** object that represents the input to the model.
> 
> - **What is a symbolic tensor?**
>     - In the above, a symbolic tensor essentially defines the structure that input tensors will take.
> - However, these symbolic tensors are used as if they were layers in the context of the Keras functional API.  
>       
>     
> 
> When you connect these input tensors to other layers (such as `**layers.Concatenate**` and `**layers.Dense**`), Keras internally creates the necessary connections and treats the input tensors as if they were layers in the context of defining the model architecture.

The next layer, `features = layers.Dense(64, activation = "relu") (inputs)` is the next later that comes after the input layer. So, as no data has been passed yet, it has shape `(None, 64)`, as the input layer currently has a shape of `(None, 3)`. We instantiate the model, finally, with the `.Model` method.

Ok, so we've just used the functional API to create another sequential model. _**Sure**_. But now, we move on to looking at how to create models that take in multiple inputs, and have multiple outputs. Consider the following code snippet:

```Python
vocabulary_size = 10000
num_tags = 100
num_departments = 4

title = keras.Input(shape=(vocabulary_size,), name="title")
text_body = keras.Input(shape=(vocabulary_size,), name="text_body")
tags = keras.Input(shape=(num_tags,), name="tags")

# above are the 3 model inputs. We create them each as a input layer
features = layers.Concatenate()([title, text_body, tags])
features = layers.Dense(64, activation="relu")(features)
# we then combine input features into a single tensor, features, by concatenating them

# our model has 2 ouputs
priority = layers.Dense(1, activation="sigmoid", name="priority")(features)
department = layers.Dense(num_departments, activation="softmax", name="department")(features)

# now, declare the model
model = keras.Model(inputs=[title, text_body, tags],
                       outputs=[priority, department])
```

> The above is no longer a **sequential model**. It is no longer a “one layer follows another” format. We now have 3 layers instead of one single input layer.

The Functional API is a simple, LEGO-like, yet very flexible way to define arbitrary graphs of layers like these. So, what exactly constitutes combining different input layers?

When you combine two input layers using operations like concatenation or merging, you are essentially combining the tensors that flow through those input layers. The tensors are concatenated or merged along a specific axis, allowing the model to process and learn from the combined information. So, combining input layers in Keras is a way to merge the tensors that correspond to those input layers, allowing the model to leverage the combined information during training and inference.

Going back to the model, how can we train it? The same way as how we would do so with `Sequential` models, with `.fit()`. We provide code that continues on from the code snippet above:

```Python
import numpy as np
num_samples = 1280

# create some dummy input data
title_data = np.random.randint(0, 2, size=(num_samples, vocabulary_size))
text_body_data = np.random.randint(0, 2, size=(num_samples, vocabulary_size))
tags_data = np.random.randint(0, 2, size=(num_samples, num_tags))

# create corresponding dummy targets
priority_data = np.random.random(size=(num_samples, 1))
department_data = np.random.randint(0, 2, size=(num_samples, num_departments))

model.compile(optimizer="rmsprop",loss=["mean_squared_error", "categorical_crossentropy"], metrics=[["mean_absolute_error"], ["accuracy"]])

model.fit([title_data, text_body_data, tags_data], [priority_data, department_data], epochs=1)

model.evaluate([title_data, text_body_data, tags_data],[priority_data, department_data]) priority_preds, department_preds = model.predict([title_data, text_body_data, tags_data])
```

> Notice that in the above, we can pass multiple inputs and outputs to the `model.fit()` method. Also, notice that in the top, we compiled our model with **multiple loss functions** and **metrics** which is common. But in such setups, **it is common to still use only one optimizer for the entire model**. By specifying multiple loss functions, you indicate that the model is expected to optimize and minimize each individual loss function simultaneously during training. The model's overall objective is to minimize the combined impact of both losses on the final model performance.

---

### The power of the functional API: Access to Layer Connectivity

A **Functional** model is an explicit graph data structure. Let’s visualize the connectivity of the model we just defined by using `plot_model()` function:

```Python
keras.utils.plot_model(model, "ticket_classifier.png")
```

![[Untitled 6.png|Untitled 6.png]]

To aid in debugging, and to see the shapes of each of the layers as well, we can add `show_shapes=True` when plotting the model:

```Python
keras.utils.plot_model(model, "ticket_classifier_with_shape_info.png", show_shapes=True)
```

![[Untitled 1 3.png|Untitled 1 3.png]]

One nice thing about this design, is that we can think of models as LEGO bricks. What this means is that we can add reuse models to create other models. Or, we can add new layers to existing models to create new models. This is called **feature extraction**, and it means creating models that reuse intermediate features from another model. Let's consider the example code from above. Say we wanted to add another output the above model. We do not need to create a new model, instead, we only need to do this:

```Python
features = model.layers[4].output
difficulty = layers.Dense(3, activation="softmax", name="difficulty")(features)

new_model = keras.Model(
inputs=[title, text_body, tags], outputs=[priority, department, difficulty])

keras.utils.plot_model(
    new_model, "updated_ticket_classifier.png", show_shapes=True)
```

Above, we access the 5th layer (count: title, text_body, tags, concatenate, dense_10), as we want to "connect" it to the new output layer we will be making.

![[Untitled 2 4.png|Untitled 2 4.png]]

---

### Subclassing the Model Class

The last model-building pattern you should know is the most advanced one: `Model` subclassing. Overview:

- In the `__init__()` method, define the layers the model will use
- In the `call()` method, define the forward pass of the model, reusing the layers previously created
- Instantiate your subclass, call it on data to create its weights

A nice way to become more familiar with this, is to take a look at the customer management code example we gave above:

```Python
title = keras.Input(shape=(vocabulary_size,), name="title")
text_body = keras.Input(shape=(vocabulary_size,), name="text_body")
tags = keras.Input(shape=(num_tags,), name="tags")

# above are the 3 model inputs. We create them each as a input layer
features = layers.Concatenate()([title, text_body, tags])
features = layers.Dense(64, activation="relu")(features)
# we then combine input features into a single tensor, features, by concatenating them

# our model has 2 ouputs
priority = layers.Dense(1, activation="sigmoid", name="priority")(features)
department = layers.Dense(num_departments, activation="softmax", name="department")(features)

# now, declare the model
model = keras.Model(inputs=[title, text_body, tags],
                       outputs=[priority, department])
```

Now, the below is how we could implement this by making our own model:

```Python
class CustomerTicketModel(keras.Model):
def __init__(self, num_departments):
  super().__init__()
  self.concat_layer = layers.Concatenate()
  self.mixing_layer = layers.Dense(64, activation="relu")
  self.priority_scorer = layers.Dense(1, activation="sigmoid")
  self.department_classifier = layers.Dense(
              num_departments, activation="softmax")

def call(self, inputs):
  title = inputs["title"]
  text_body = inputs["text_body"]
  tags = inputs["tags"]

  features = self.concat_layer([title, text_body, tags])
  features = self.mixing_layer(features)

  priority = self.priority_scorer(features)
  department = self.department_classifier(features)

  return priority, department
```

So, lets go over whats going on in the code above. `__init__()` is the constructor for the class, and in it, we declare/define all the layers we will use in the model. ==**Specifically, we do not define connections between layers yet.**== The relations between layers is defined in the `call()` method. We also pass the inputs to it like that. To create an instance of the `CustomerTicketModel`:

```Python
model = CustomerTicketModel(num_departments=4)
priority, department = model(
    {"title": title_data, "text_body": text_body_data, "tags": tags_data})

# the object we pass to the model must be the same as `inputs` from the call method
```

So far, everything looks very similar to Layer subclassing, a workflow you encountered in chapter 3. What, then, is the difference between a `Layer` subclass and a `Model` subclass? It’s simple: a “layer” is a building block you use to create models, and a “model” is the top-level object that you will actually `train`, `export` for `inference`, etc. In short, a Model has `fit()`, `evaluate()`, and `predict()` methods. Layers don’t. Other than that, the two classes are virtually identical.

To compile the class:

```Python
model.compile(optimizer="rmsprop", loss=["mean_squared_error", "categorical_crossentropy"], metrics=[["mean_absolute_error"], ["accuracy"]])

model.fit({"title": title_data, "text_body": text_body_data,
"tags": tags_data}, [priority_data, department_data], epochs=1)

model.evaluate({"title": title_data, "text_body": text_body_data,
"tags": tags_data}, [priority_data, department_data])

priority_preds, department_preds = model.predict({"title": title_data,
                                                  "text_body": text_body_data, "tags": tags_data})

# The structure of the input data must match exactly what is expected by the call() method — here, a dict with keys title, text_body, and tags.
```

When you call `model.predict()` with input data, it performs the forward pass computations on the input data through the layers of the model and generates the corresponding output(s). The returned result represents the predicted output(s) of the model for the provided input data.

One thing to notice from the above code, is the changes in what we provide to the model before `.fit()` or `.evaluate`. Before, we would simply give the training data to the model, which was when we were working with `Sequential` models, and there was only one input to the model. But now, we are creating a model that has more than one input and more than one output. So, we pass each one separately so that we can create a layer for each respective data set.

---

A crucial part of the keras api, is the ability for us to interlock different types of models. Wether we are using the `Sequential`, `Functional`, or custom model, all models in keras can smoothly interoperate with each other. For instance, consider the following code:

```Python
class Classifier(keras.Model):
  def __init__(self, num_classes=2):
    super().__init__()
    if num_classes == 2:
      num_units = 1
      activation = "sigmoid"
    else:
      num_units = num_classes
      activation = "softmax"
    self.dense = layers.Dense(num_units, activation=activation)

  def call(self, inputs):
    return self.dense(inputs)

inputs = keras.Input(shape=(3,))
features = layers.Dense(64, activation="relu")(inputs)
outputs = Classifier(num_classes=10)(features)
model = keras.Model(inputs=inputs, outputs=outputs)

```

In Keras, you can combine models by creating a new model that incorporates the layers of multiple existing models. This process is often referred to as model composition or model merging. There are several ways to combine models in Keras, depending on your specific requirements. Let's specifically look at this for `Functional` api:

- The Functional API allows you to create a model by specifying the input and output tensors of each model and connecting them together.
- You can use the output tensors of one model as inputs to another model by treating the model as a callable function.
    - We see this in the above code. The output of `features` layer is used as input for the `Classifier` model we have instantiated

If you can use the Functional API—that is, if your model can be expressed as a directed acyclic graph of layers—I recommend using it over model subclassing. Otherwise, we will need to sub-class the `Model` class. In general, using Functional models that include subclassed layers provides the best of both worlds: high development flexibility while retaining the advantages of the Functional API.

---

### Using built-in training and evaluation loops

So far, we've seen how we work with certain workflows in progressive difficulty. Now, we look at training. So far, all we've really done is use the `fit()` method. We can make more advanced training architecture by writing training algorithms from scratch. Consider the following code:

```Python
from tensorflow.keras.datasets import mnist
from tensorflow.keras import layers

import keras

def get_mnist_model():
  inputs = keras.Input(shape=(28 * 28,))
  features = layers.Dense(512, activation="relu")(inputs)
  features = layers.Dropout(0.5)(features)
  outputs = layers.Dense(10, activation="softmax")(features)

  model = keras.Model(inputs, outputs)
  return model

(images, labels), (test_images, test_labels) = mnist.load_data()
images = images.reshape((60000, 28 * 28)).astype("float32") / 255
test_images = test_images.reshape((10000, 28 * 28)).astype("float32") / 255
train_images, val_images = images[10000:], images[:10000]
train_labels, val_labels = labels[10000:], labels[:10000]

model = get_mnist_model()

model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

model.fit(train_images, train_labels, epochs=3, validation_data=(val_images, val_labels))
test_metrics = model.evaluate(test_images, test_labels)
predictions = model.predict(test_images)
```

There are a couple of ways we can customize this simple workflow:

- Provide our own custom metrics
- Pass _callbacks_ to the `fit()` method to schedule actions to be taken at specific points during training

We will take a look at these:

### Writing our own metrics

Metrics are key to measuring the performance of your model — in particular, to measuring the difference between its performance on the training data and its performance on the test data. Commonly used metrics for `classification` and `regression` are already part of the built-in `keras.metrics` module, and most of the time that’s what you will use. But if you’re doing anything out of the ordinary, you will need to be able to write your own metrics.

Just as how we made a custom `Layer` by sub-classing it, and we made custom models by subclassing `Model`, we make custom metrics by sub-classing `keras.metrics.Metric` class. Consider the following example:

```Python
import tensorflow as tf

class RootMeanSquaredError(keras.metrics.Metric):
  def __init__(self, name="rmse", **kwargs):
    super().__init__(name=name, **kwargs)
    self.mse_sum = self.add_weight(name="mse_sum", initializer="zeros")
    self.total_samples = self.add_weight(name="total_samples", initializer="zeros", dtype="int32")


  def update_state(self, y_true, y_pred, sample_weight=None):
    y_true = tf.one_hot(y_true, depth=tf.shape(y_pred)[1])
    mse = tf.reduce_sum(tf.square(y_true - y_pred))
    self.mse_sum.assign_add(mse)
    num_samples = tf.shape(y_pred)[0]
    self.total_samples.assign_add(num_samples)
```

So, what does a metric even need to remember? A metric needs to remember specific information in order to compute its value accurately over the course of training or evaluation. The exact information it needs to remember depends on the specific metric being used. Here are a few examples:

- `Accuracy`: To calculate accuracy, the metric needs to remember the count of correct predictions and the total count of predictions made. It accumulates the correct and total counts across batches or samples.
- `Mean Squared Error (MSE)`: MSE requires the metric to remember the sum of squared differences between the predicted values and the true values. It accumulates the squared differences across batches or samples.

This sort of infomation is calcualted at the end of every batch (or epoch, depends on the model is defined).

You use the `result()` method to return the current value of the metric:

```Python
def result(self):
  return tf.sqrt(self.mse_sum / tf.cast(self.total_samples, tf.float32))
```

Meanwhile, you also need to expose a way to reset the metric state without having to reinstantiate it—this enables the same metric objects to be used across different epochs of training or across both training and evaluation. You do this with the `reset_state()` method:

```Python
def reset_state(self):
  self.mse_sum.assign(0.)
  self.total_samples.assign(0)
```

To use a custom metric, it is the same as using built-in ones:

```Python
model = get_mnist_model()
model.compile(optimizer="rmsprop",
             loss="sparse_categorical_crossentropy",
              metrics=["accuracy", RootMeanSquaredError()])
model.fit(train_images, train_labels, epochs=3,
         validation_data=(val_images, val_labels))
test_metrics = model.evaluate(test_images, test_labels)
```

### Using Callbacks

Right now, we know that we want to overtrain in order to find the best place to stop training. But, if we just run `.fit()` for random or increasing number of epochs, its like launching a paper airplane: we never know how it is going to go, until it is over.

To have more control, and more data about the training process, we will use callbacks. A callback is an object (a class instance implementing specific methods) that is passed to the model in the call to `fit()` and that is called by the model at various points during training. It has access to all the available data about the state of the model and its performance, and it can take action: interrupt training, save a model, load a different weight set, or otherwise alter the state of the model.

The `keras.callbacks` module includes a number of built-in callbacks. We will takea look at 2 specific ones and how to use them:

### The Earlystopping and Modelcheckpoint Callbacks

The `EarlyStopping` callback interrupts training once a target metric being `monitored` has stopped improving for a fixed number of epochs. For instance, this callback allows you to interrupt training as soon as you start overfitting, thus avoiding having to retrain your model for a smaller number of epochs. This callback is typically used in combination with `ModelCheckpoint`, which lets you continually save the model during training (and, optionally, save only the current best model so far: the version of the model that achieved the best performance at the end of an epoch). Here is an example of how callbacks are used:

```Python
callbacks_list = [
    keras.callbacks.EarlyStopping(
        monitor="val_accuracy",
        patience=2,
    ),
    keras.callbacks.ModelCheckpoint(
      filepath="checkpoint_path.keras",
      monitor="val_loss",
      save_best_only=True,
    )
]

model = get_mnist_model()
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

model.fit(train_images, train_labels,
          epochs=10,
          callbacks=callbacks_list,
          validation_data=(val_images, val_labels))
```

So, let's digest the above code.

- First, we've created a list of the callbacks we will want to use.
- `EarlyStopping` interrupts training when improvement stops
    - Here, we are monitoring the model's validation accuracy
    - By setting `patience=2`, we interrupt training when accuracy has stopped improving for 2 epochs
- `ModelCheckpoint` saves the model's weight to a file after every epoch.
    - The file location is denoted through `filepath`
    - `monitor="val_loss"` and `save_best_only=True` mean we won't overwrite the model unless `val_loss` has improved, which allows you to keep the best model seen during training.
- In the compilation, we use metric of `accuracy` as for the callbacks, we have been monitoring the model's accuracy.
- Note that because the callback will monitor validation loss and validation accuracy, you need to pass validation_data to the call to `fit()`.

Note that you can always save models manually after training as well—just call `model.save('my_checkpoint_path')`. To reload the model you’ve saved, just use `model = keras.models.load_model("checkpoint_path.keras")`

### Writing our own callbacks

If you need to take a specific action during training that isn’t covered by one of the built-in callbacks, you can write your own callback. Callbacks are implemented by sub-classing the `keras.callbacks.Callback` class. Then, we implement any of the following methods, which are called at various points in training:

```Python
on_epoch_begin(epoch, logs)
on_epoch_end(epoch, logs)
on_batch_begin(batch, logs)
on_batch_end(batch, logs)
on_train_begin(logs)
on_train_end(logs)

# the names are self-explanatory
```

These methods are all called with a `logs` argument, which is a dictionary containing information about the previous batch, epoch, or training run—training and validation metrics, and so on. The `on_epoch_*` and `on_batch_*` methods also take the `epoch` or `batch` index as their first argument (an integer).

Here is an example:

```Python
from matplotlib import pyplot as plt

class LossHistory(keras.callbacks.Callback):
  def on_train_begin(self, logs):
    self.per_batch_losses = []

  def on_batch_end(self, batch, logs):
    self.per_batch_losses.append(logs.get("loss"))

  def on_epoch_end(self, epoch, logs):
    plt.clf()
    plt.plot(range(len(self.per_batch_losses)), self.per_batch_losses,
             label="Training loss for each batch")
    plt.xlabel(f"Batch (epoch {epoch})")
    plt.ylabel("Loss")
    plt.legend()
    plt.savefig(f"plot_at_epoch_{epoch}")
    self.per_batch_losses = []
```

And here is how we would use our custom callback:

```Python
model = get_mnist_model()
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])
model.fit(train_images, train_labels,
          epochs=10,
          callbacks=[LossHistory()],
          validation_data=(val_images, val_labels))
```

### Monitoring and visualization with TensorBoard

To do good research or develop good models, you need rich, frequent feedback about what’s going on inside your models during your experiments.

![[Untitled 3 3.png|Untitled 3 3.png]]

TensorBoard ([www.tensorflow.org/tensorboard](http://www.tensorflow.org/tensorboard)) is a browser-based application that you can run locally. It’s the best way to monitor everything that goes on inside your model during training. With TensorBoard, you can

- Visually monitor metrics during training
- Visualize your model architecture
- Visualize histograms of activations and gradients  Explore embeddings in 3D

How to use this? Easiest way to use TensorBoard with a `Keras` model is to incorporate it as a callback in the `fit()` method:

```Python

model = get_mnist_model()
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])

tensorboard = keras.callbacks.TensorBoard(
    log_dir="/full_path_to_your_log_dir",
)

model.fit(train_images, train_labels,
          epochs=10,
          validation_data=(val_images, val_labels),
          callbacks=[tensorboard])
```

Once the model starts running, it will write logs at the target location. To learn more about how to use this, look online on how to start a local TensorBoard server.

### Writing your own training and evaluation loops

The `fit()` method will meet our needs most times. However, it isn’t meant to support everything a deep learning researcher may want to do. After all, the built-in `fit()` workflow is solely focused on `supervised learning`: a setup where there are known targets (also called labels or annotations) associated with your input data, and where you compute your loss as a function of these targets and the model’s predictions.

However, not every form of machine learning falls into this category. Others include:

- `generative learning`
- `self-supervised learning`
- `reinforcement learning`

When the built-in `fit()` method is not enough, we will need to write our own custom training logic. As a reminder, the contents of a typical training loop look like this:

1. Run the forward pass (compute the model’s output) inside a gradient tape to obtain a loss value for the current batch of data.
2. Retrieve the gradients of the loss with regard to the model’s weights.
3. Update the model’s weights so as to lower the loss value on the current batch  
    of data.  
    

  

**→ Training vs Inference**

Some Keras layers, such as the `Dropout` layer, have different behaviours during training and during inference (when you use them to generate predictions). Such layers expose a training Boolean argument in their `call()` method. Calling `dropout(inputs, training=True)` will drop some activation entries, while calling `dropout(inputs, training=False)` does nothing. By extension, `Functional` and `Sequential` models also expose this training argument in their `call()` methods. **Remember to pass** **`training=True`** **when you call a Keras model during the forward pass!** Our forward pass thus becomes `predictions = model(inputs, training=True)`.

  

Recall that the first time we learned about the training loop, we did so via writing our own training loop with the gradient tape:

```Python
input_dim = 2
output_dim = 1
W = tf.Variable(initial_value=tf.random.uniform(shape=(input_dim, output_dim)))
b = tf.Variable(initial_value=tf.zeros(shape=(output_dim,)))

def model(inputs):
  return tf.matmul(inputs, W) + b
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

```Python
def train_step(inputs, targets): 
	with tf.GradientTape() as tape:
	  predictions = model(inputs, training=True)         # training = True is !!
	  loss = loss_fn(targets, predictions)
		gradients = tape.gradients(loss, model.trainable_weights)
    optimizer.apply_gradients(zip(model.trainable_weights, gradients))
```

  

### A complete training and evaluation loop

Let’s combine the forward pass, backward pass, and metrics tracking into a `fit()`-like training step function that takes a batch of data and targets and returns the logs that would get displayed by the `fit()` progress bar.

```Python
model = get_mnist_model()

loss_fn = keras.losses.SparseCategoricalCrossentropy()
optimizer = keras.optimizers.RMSprop()
metrics = [keras.metrics.SparseCategoricalAccuracy()]
loss_tracking_metric = keras.metrics.Mean()

def train_step(inputs, targets): 
	with tf.GradientTape() as tape:
		predictions = model(inputs, training=True)
		loss = loss_fn(targets, predictions)
	gradients = tape.gradient(loss, model.trainable_weights) 
	optimizer.apply_gradients(zip(gradients,modell.trainable_weights))

	logs = {}
	for metric in metrics:
		metric.update_state(targets, predictions) 
		logs[metric.name] = metric.result()

	loss_tracking_metric.update_state(loss)
	logs["loss"] = loss_tracking_metric.result()
	return logs
```

We will need to reset the state of our metrics at the start of each epoch and before running evaluation. Here’s a utility function to do it.

```Python
def reset_metrics():
	for metric in metrics:
	  metric.reset_state()
  loss_tracking_metric.reset_state()
```

Now that we have defined the training step function, we write the training loop itself:

```Python
training_dataset = tf.data.Dataset.from_tensor_slices(
    (train_images, train_labels))
training_dataset = training_dataset.batch(32)
epochs = 3

for epoch in range(epochs):
	reset_metrics()
	for inputs_batch, targets_batch in training_dataset:
		logs = train_step(inputs_batch, targets_batch)
	print(f"Results at the end of epoch {epoch}")
	for key, value in logs.items():
		print(f"...{key}: {value:.4f}")
```

Now finally, we write the evaluation loop. It is actually just a subset of the training loop, but we omit code that updates the model weights, and set `training=False` when computing the forward pass.

```Python
def test_step(inputs, targets):
	predictions = model(inputs, training=False)
	loss = loss_fn(targets, predictions)

	logs = {}
	for metric in metrics:
    metric.update_state(targets, predictions)
	  logs["val_" + metric.name] = metric.result()

		loss_tracking_metric.update_state(loss)
		logs["val_loss"] = loss_tracking_metric.result() return logs

val_dataset = tf.data.Dataset.from_tensor_slices((val_images, val_labels))
val_dataset = val_dataset.batch(32)
reset_metrics()

# run every test data sample through the model 
for inputs_batch, targets_batch in val_dataset:
	logs = test_step(inputs_batch, targets_batch)
	print("Evaluation results:")
for key, value in logs.items():
	print(f"...{key}: {value:.4f}")
```

So, we’ve now reimplemented the fundamental features of `fit()` and `evaluate()`. In reality, those 2 built-in **keras functions** support many more features and several key performance optimizations. Let’s take a look at one of these optimizations: TensorFlow function compilation.

  

→ TensorFlow function compilation

The above functions we have written are significantly than `fit()` and `evaluate()` from keras. We make it faster with `tf.function`. In the way we have written code above, each line is executed line by line (eagerly). While this is type of execution makes it easier to debug our code, it is far from optimal from a performance standpoint.

  

It’s more performant to _compile_ your TensorFlow code into a _computation graph_ that  
can be globally optimized in a way that code interpreted line by line cannot. The syntax to do this is very simple: just add a  
`@tf.function` to any function you want to compile before executing, as shown in the following listing:

```Python
@tf.function
def test_step(inputs, targets):
	predictions = model(inputs, training=False)
	loss = loss_fn(targets, predictions)

	logs = {}
	for metric in metrics:
    metric.update_state(targets, predictions)
	  logs["val_" + metric.name] = metric.result()

		loss_tracking_metric.update_state(loss)
		logs["val_loss"] = loss_tracking_metric.result() return logs

val_dataset = tf.data.Dataset.from_tensor_slices((val_images, val_labels))
val_dataset = val_dataset.batch(32)
reset_metrics()
```

On the Colab CPU, we go from taking 1.80 s to run the evaluation loop to only 0.8 s. Much faster! So:

- When debugging code → run eagerly without the `@tf.function` decorator
- Once working and we want it to run faster → use `@tf.function` decorator

---

Now, we’ve seen 2 methods of writing our training logic. We can either subclass the `Model` class and write our own training loop (at the cost of writing more code and missing out on many convenient features of `fit()` such as callbacks, or use the built-in training loop, which may not suit our needs enough. There is a middle-ground: _**override the**_ `_**train_step()**_` _**method of the**_ `_**Model**_` _**class.**_ When we do this, we can enjoy the benefits of `fit()` while it runs our own learning algorithm.

```Python
class CustomModel(keras.Model): 
	def train_step(self, data):
		inputs, targets = data
		with tf.GradientTape() as tape:
			predictions = self(inputs, training=True)
			loss = loss_fn(targets, predictions)
	
		gradients = tape.gradient(loss, self.trainable_weights)
		self.optimizer.apply_gradients(zip(gradients, self.trainable_weights))
	
		loss_tracker.update_state(loss) 
		return {"loss": loss_tracker.result()}
	
		@property
		def metrics(self): 
			return [loss_tracker]

inputs = keras.Input(shape=(28 * 28,))
features = layers.Dense(512, activation="relu")(inputs)
features = layers.Dropout(0.5)(features)
outputs = layers.Dense(10, activation="softmax")(features)
model = CustomModel(inputs, outputs)
model.compile(optimizer=keras.optimizers.RMSprop())
model.fit(train_images, train_labels, epochs=3)
```

![[Screenshot_2024-01-22_at_8.22.52_PM.png]]

---

## WHOA!

That was a **BIG** chapter, and a lot to digest I’m sure. The next chapter we will look at is NLP (not the next chapter number-wise, but I’m interested). Before moving on, let’s try a submission for

> [!info] Binary Classification with a Bank Churn Dataset  
> Playground Series - Season 4, Episode 1  
> [https://www.kaggle.com/competitions/playground-series-s4e1/overview](https://www.kaggle.com/competitions/playground-series-s4e1/overview)  

to practice writing deep learning models that work (so far, only done samples from the textbook).