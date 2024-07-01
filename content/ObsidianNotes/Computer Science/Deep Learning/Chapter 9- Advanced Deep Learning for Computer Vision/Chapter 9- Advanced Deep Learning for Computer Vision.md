In this chapter, we will look at:

- Different branches of computer vision:
    - Image classification
    - Image segmentation
    - Object detection
- **Modern convnet architectures:** residual connections, batch normalization, depth-wise separable convolutions

We’ve seen the introduction to **convolutional networks** in the previous chapter. But, theres a **LOT MORE** to computer vision that just simple classification! This chapter will dive deeper into more diverse applications of computer vision and more advanced practices.

---

**→ We won’t go into object detection, as thats a bit too specific for now. If you want to learn more about it later on, here’s the keras docs for it:**

> [!info] Keras documentation: Object Detection with RetinaNet  
> Keras documentation  
> [https://keras.io/examples/vision/retinanet/](https://keras.io/examples/vision/retinanet/)  

### Image Segmentation Example

Image segmentation with deep learning is about using a model to assign a class to each pixel in an image, thus _segmenting_ the image into different zones (such as “background” and “foreground”, or “road”, “car”, and “sidewalk”) → important in things like **video editing tools, autonomous driving, medical imaging, etc.** There are 2 main types of image segmentation:

- _**Semantic Segmentation:**_ each pixel is independently classified into a semantic category, e.g “cat”. If there are > 2 cats in the image, they both mapped to the same generic “cat” family.
- _**Instance Segmentation:**_ not only seeks to classify images pixels by category, but also to parse out individual object instances → in image with > 2 cats, they would be treated as “cat1” and “cat2”

Let’s first take a look at **semantic segmentation**. We’ll work with the Oxford-IIIT Pets dataset (www.robots.ox.ac.uk/~vgg/data/pets/), which contains 7,390 pictures of various breeds of cats and dogs, together with foreground-background **segmentation masks** for each picture.

> ==A== ==**_segmentation mask_**== ==is the image-segmentation equivalent of a label: 1. it’s an image the same size as the input  
> image, with a single color channel where each integer value corresponds to the class of the corresponding pixel in the input image.  
> ==

![[Screenshot_2024-02-04_at_7.13.29_PM.png]]

In our case, the pixels of our segmentation masks can take on one of three integer values:

- 1 (foreground)
- 2 (background)
- 3 (contour)

Our `colab` notebook will have specific details on processing the images and how to visually display their **segmentation masks**. Here is how we define the model:

```Python
from tensorflow import keras
from keras import layers

def build_model(img_size, num_classes):
  #  If img_size is (200, 200), adding (3,) to it results in a new tuple (200, 200, 3), since each pixel has (R,G,B) depth
  inputs = keras.Input(shape=img_size + (3,)) 
  # re-scale: normalize the values to [0,1]
  x = layers.Rescaling(1./255)(inputs)

  # Conv2D block
  x = layers.Conv2D(64, 3, strides=2, activation="relu", padding="same")(x)
  x = layers.Conv2D(64, 3, activation="relu", padding="same")(x)
  x = layers.Conv2D(128, 3, strides=2, activation="relu", padding="same")(x)
  x = layers.Conv2D(128, 3, activation="relu", padding="same")(x)
  x = layers.Conv2D(256, 3, strides=2, padding="same", activation="relu")(x)
  x = layers.Conv2D(256, 3, activation="relu", padding="same")(x)

  # Conv2DTranspose block
  x = layers.Conv2DTranspose(256, 3, activation="relu", padding="same")(x)
  x = layers.Conv2DTranspose(256, 3, activation="relu", padding="same", strides=2)(x)
  x = layers.Conv2DTranspose(128, 3, activation="relu", padding="same")(x)
  x = layers.Conv2DTranspose(128, 3, activation="relu", padding="same", strides=2)(x)
  x = layers.Conv2DTranspose(64, 3, activation="relu", padding="same")(x)
  x = layers.Conv2DTranspose(64, 3, activation="relu", padding="same", strides=2)(x)

  outputs = layers.Conv2D(num_classes, 3, activation="softmax", padding="same")(x)
  model = keras.Model(inputs, outputs) 
  return model

model = build_model(img_size=img_size, num_classes=3) 
model.summary()
```

Now, there’s something new here: `layers.Conv2DTranspose`. Let’s dive deeper into the design of this model:

**→ First Half**

- The first half of this model resembles the image classification models we’ve seen before: a stack of `Conv2D` layers, with gradually increasing filter sizes. These layers de-sample **(de-sample refers to the reduction spatial dimensions through convolutions)** the input images three times, by a factor of 2 each time (stride is 2) → so, we end with an activation size of (25,25,256)
- The purpose of this first half is to encode the images into smaller feature maps, where each spatial location (or pixel) contains information about a large spatial chunk of the original image.
    - A difference between the first half of this model, and what we have seen before, is how downsampling. Before, we used a `MaxPooling2D` layer to downsample feature maps
    - Here, we are not. Instead, we add `strides` to every other layer (not setting stride gives it a default value of `1`). Why is this?
        - In **image segmentation**, we care a lot about _spatial location_ of information in the image. This is because we need to produce per-pixel target masks as output.
        - When you do 2 × 2 max pooling, you are _**completely destroying location information**_ within each pooling window: you return one scalar value per window, with zero knowledge of which of the four locations in the windows the value came from.
        - So while max pooling layers performs well in classification tasks, they would hurt us quite a bit for a segmentation task. **So**, we used **strided convolutions > 1** to downsample feature maps while retaining location information.

**→ Second Half**

The second half of the model is a stack of `Conv2DTranspose` layers. The following video is very helpful in learning about this process:

> [!info] Transpose Convolutions  
>  
> [https://www.youtube.com/watch?v=qb4nRoEAASA&ab_channel=ВалентинРоманов](https://www.youtube.com/watch?v=qb4nRoEAASA&ab_channel=ВалентинРоманов)  

The idea is mostly the same as what convolution looks like in the forward direction. Normally, when we run a convolution on an image with a filter (say of size 3), the feature map extracted from that image has smaller dimensions. The transpose convolution (just as it name implies), does the opposite. Given an feature map and a filter, the filter increases the size of the feature map. consider the visual below:

![[Screenshot_2024-02-05_at_6.37.25_PM.png]]

![[Screenshot_2024-02-05_at_6.37.41_PM.png]]

  

Why do we need this? In image segmentation, we want the output shape to be the same as the input image. After layers of convolutions, the images are no longer the same size. So, we need to apply the **inverse** of the previous convolutions, where we are _**up-scaling the feature maps**_. Think if this similar to the inverse of a convolution.

---

> **One thing** the book did not touch on is **padding**. What is it? Often times, when we perform a convolution, the output size is reduced. Without padding, the pixels at the borders of the input are used less in the convolution operation. **Padding** helps mitigate this issue. There are 2 common types of padding:
> 
> - **Valid (no padding):** no padding is added, and convolution is only applied to pixels where the filter and input overlap completely.
> - **Same(Zero padding)****:** padding is added in such a way that the output size is the same as the input-size (assuming a stride of 1).
>     - If the filter-size is an odd number, the same amount of padding is added to both sides.
>     - If even, more is added right/bottom.

---

Now that we know what transpose convolutions are, we can compile and fit our model:

```Python
model.compile(optimizer="rmsprop", loss="sparse_categorical_crossentropy")
callbacks = [
	keras.callbacks.ModelCheckpoint("oxford_segmentation.keras",
																	save_best_only=True)
]

history = model.fit(train_input_imgs, train_targets, epochs=50,
										callbacks=callbacks,
										batch_size=64,
										validation_data=(val_input_imgs, val_targets))

# displaying training and validation loss
epochs = range(1, len(history.history["loss"]) + 1)
loss = history.history["loss"]
val_loss = history.history["val_loss"]
plt.figure()
plt.plot(epochs, loss, "bo", label="Training loss")
plt.plot(epochs, val_loss, "b", label="Validation loss")
plt.title("Training and validation loss")
plt.legend()
```

![[Screenshot_2024-02-05_at_7.05.38_PM.png]]

From this, we notice that we begin to overfit around the 25th epoch. Let’s reload our best performing model, and show how to use it predict a segmentation task:

```Python
from tensorflow.keras.utils import array_to_img

model = keras.models.load_model("oxford_segmentation.keras")

i=4
test_image = val_input_imgs[i] 
plt.axis("off") 
plt.imshow(array_to_img(test_image))

mask = model.predict(np.expand_dims(test_image, 0))[0]

def display_mask(pred):
	mask = np.argmax(pred, axis=-1) mask *= 127
	plt.axis("off") 
	plt.imshow(mask)
display_mask(mask)
```

![[Screenshot_2024-02-05_at_9.48.24_PM.png]]

While not perfect (there is some small geometric shape to the right), it still works well. We’ve now looked at the basics of image classification and image segmentation, and we can already accomplish a lot with what we already know. However, the convnets experienced ML engineers develop in real-world scenarios are not as simple. We’ll now dive into more **modern convnet architecture patterns** to learn more about thought processes that go into enabling us to make quick and accurate decisions about how to put together state-of-the-art models.

---

### Convnet Architecture Patterns

A model’s “architecture” is the sum of the choices that went into creating it: which layers to use, how to configure them, and in what arrangement to connect them. For instance, using convolution layers means that you know in advance that the relevant patterns present in your input images are **translation-invariant (below for separate note):**

[[Equivariance + Invariance]]

Model architecture is often the difference between success and failure. If you make inappropriate architecture choices, your model may be stuck with suboptimal metrics, and no amount of training data will save it. Inversely, a good model architecture will accelerate learning and will enable your model to make efficient use of the training data available, reducing the need for large datasets.

> A good model architecture is one that _reduces the size of the search space_ or otherwise _makes it easier to converge to a good point of the search space_. **Just like feature engineering and data curation, model architecture is all about** **_making the problem simpler_** **for** **gradient descent** **to solve.**

_In the following sections, we’ll review a few essential convnet architecture best practices:_

- Residual connections
- Batch normalization
- Separable convolutions

---

### Modularity, Hierarchy, Reuse — Formula

In all aspects of complex systems, there is a common solution on how to make them simpler:

1. **Modularity:** structure the problem solution into modules
2. **Hierarchy:** organize these modules into hierarchy
3. **Reuse:** reuse the same modules in multiple places as appropriate

This is at the heart of any organization, whether it be in the _**design of the cathedral, the human body, or the Keras codebase**_. This is something your familiar with in software design. You have modular design, where e separate things into different parts like **controllers, services, etc.** Also, those modules have a certain hierarchy (incoming requests processed by controller, which calls some services, which then calls libraries, etc.). Last but not least, they are re-usable. **Write once use anywhere**.  
  

Likewise, deep learning model architecture is primarily about making clever use of modularity, hierarchy, and reuse. You’ll notice that all popular convnets architectures are not only structured into layers, they’re structured into related groups of layers (called “blocks” or “modules”). Consider the VGG16 architecture we worked with earlier:

![[Screenshot_2024-02-06_at_1.51.38_PM.png]]

In the VGG16 design, we had several “blocks” of repeated “conv, conv, max pooling” layers. Furthermore, most **convnets feature pyramid-like structures**. That is to say, _the number of filters grows with layer depth_, while the size of the feature maps shrinks accordingly. This **pyramid** structure is visible in the above. Deeper hierarchies are intrinsically good because the encourage more feature reuse, and therefore, abstraction. In general, a deep stack of **narrow layers** perform better than a **shallow** **stack of large layers.** However, theres a catch → there is a limit to how deep you can stack layers, due to the problem of _vanishing gradients_. This leads us to our first essential model architecture pattern: **residual connections.**

---

### Residual Connections

Consider the game of **Telephone**. You’re probably familiar with how this game works, and how the message being transferred easily falls apart, especially as more and more people are involved in the message transfer process. As it happens, backpopogation in a sequential deep learning model is pretty similar to the game of telephone. Consider this video:

> [!info] Vanishing/Exploding Gradients (C2W1L10)  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=qhXZsFVxGKo&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=qhXZsFVxGKo&ab_channel=DeepLearningAI)  

Consider the following note (for a **dense sequential model**, but the point stands):

![[JPEG_image-4830-B06C-00-0.jpeg]]

So, as the network increases in layers, we are at increasing risk of the gradient being **exponentially large** or exponentially small. There is no solution for this, in fact, for a long time, this was a huge problem in training neural networks. A partial solution, which does not completely solve the problem, but helps a lot, is making careful choices on how we initialize weights.

> [!info] Weight Initialization in a Deep Network (C2W1L11)  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=s2coXdufOzE&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=s2coXdufOzE&ab_channel=DeepLearningAI)  

The above video is a good watch to learn more about this issue in FNN’s. **What about CNN’s?**

  

For CNN’s, we force each function in the chain to be non-destructive — to retain a noiseless version of the information contained in the previous input. The easiest way to implement this is to use a _residual connection._ This is a very simple operation:

![[Screenshot_2024-02-06_at_7.26.16_PM.png]]

We add the input of a layer or block of layers, to its output (as the above shows). The residual connection acts as an **information shortcut** around destructive or noisy blocks (such as blocks that contain `relu` activations or dropout layers), enabling error gradient information from early layers to propagate noiselessly through a deep network. In practice, this might look something like:

```Python
# first some input tensor 
x ... 

# save a pointer to the original input, this is the residual 
residual = x

# the below computation block can potentally be destructive or noisy, and thats fine 
x = block(x)

# add the original input to the layer's output
x = add([x, residual])
```

Adding the input and output tensors implies that they must have the same shape. We know that this won’t be the case if the block contains convolutional layers with an increased number of filters, or a max-pooling layer. Consider the following code:

```Python
from tensorflow import keras
from tensorflow.keras import layers

inputs = keras.Input(shape=(32, 32, 3))
x = layers.Conv2D(32, 3, activation="relu")(inputs)
residual = x
x = layers.Conv2D(64, 3, activation="relu", padding="same")(x)
residual = layers.Conv2D(64, 1)(residual)
x = layers.add([x, residual])
```

The below visual demonstrates what is going on here:

![[JPEG_image-4F0F-B677-2C-0.jpeg]]

The point is, at the end of all this, the residual has been fitted to match the normal training output, with padding (this maintains the knowledge), and then adding the 2 tensors. What about if we had `MaxPooling` layers?

```Python
#### Case whe target block includes a max pooling layer 
inputs = keras.Input(shape=(32, 32, 3))
x = layers.Conv2D(32, 3, activation="relu")(inputs)
residual = x
x = layers.Conv2D(64, 3, activation="relu", padding="same")(x)
x = layers.MaxPooling2D(2, padding="same")(x)
residual = layers.Conv2D(64, 1, strides=2)(residual)
x = layers.add([x, residual])
```

To make these ideas more concrete, here is an example of a simple convnet structured into a series of blocks, each made of two convolutions layers and one optimal max pooling layer, with a residual connected around each block:

```Python
inputs = keras.Input(shape=(32, 32, 3))
x = layers.Rescaling(1./255)(inputs)
	
def residual_block(x, filters, pooling=False): 
	residual = x
	x = layers.Conv2D(filters, 3, activation="relu", padding="same")(x) 
	x = layers.Conv2D(filters, 3, activation="relu", padding="same")(x)
	if pooling:
		x = layers.MaxPooling2D(2, padding="same")(x)
		residual = layers.Conv2D(filters, 1, strides=2)(residual)
	elif filters != residual.shape[-1]:
		residual = layers.Conv2D(filters, 1)(residual)
	x = layers.add([x, residual])
	return x

x = residual_block(x, filters=32, pooling=True)
x = residual_block(x, filters=64, pooling=True)
x = residual_block(x, filters=128, pooling=False)

x = layers.GlobalAveragePooling2D()(x)
outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs=inputs, outputs=outputs)
model.summary()
```

For this model, if we were to run `model.summary()` , we would get:

```Python
Model: "model"
__________________________________________________________________________________________________
 Layer (type)                Output Shape                 Param #   Connected to                  
==================================================================================================
 input_1 (InputLayer)        [(None, 32, 32, 3)]          0         []                            
                                                                                                  
 rescaling (Rescaling)       (None, 32, 32, 3)            0         ['input_1[0][0]']             
                                                                                                  
 conv2d (Conv2D)             (None, 32, 32, 32)           896       ['rescaling[0][0]']           
                                                                                                  
 conv2d_1 (Conv2D)           (None, 32, 32, 32)           9248      ['conv2d[0][0]']              
                                                                                                  
 max_pooling2d (MaxPooling2  (None, 16, 16, 32)           0         ['conv2d_1[0][0]']            
 D)                                                                                               
                                                                                                  
 conv2d_2 (Conv2D)           (None, 16, 16, 32)           128       ['rescaling[0][0]']           
                                                                                                  
 add (Add)                   (None, 16, 16, 32)           0         ['max_pooling2d[0][0]',       
                                                                     'conv2d_2[0][0]']            
                                                                                                  
 conv2d_3 (Conv2D)           (None, 16, 16, 64)           18496     ['add[0][0]']                 
                                                                                                  
 conv2d_4 (Conv2D)           (None, 16, 16, 64)           36928     ['conv2d_3[0][0]']            
                                                                                                  
 max_pooling2d_1 (MaxPoolin  (None, 8, 8, 64)             0         ['conv2d_4[0][0]']            
 g2D)                                                                                             
                                                                                                  
 conv2d_5 (Conv2D)           (None, 8, 8, 64)             2112      ['add[0][0]']                 
                                                                                                  
 add_1 (Add)                 (None, 8, 8, 64)             0         ['max_pooling2d_1[0][0]',     
                                                                     'conv2d_5[0][0]']            
                                                                                                  
 conv2d_6 (Conv2D)           (None, 8, 8, 128)            73856     ['add_1[0][0]']               
                                                                                                  
 conv2d_7 (Conv2D)           (None, 8, 8, 128)            147584    ['conv2d_6[0][0]']            
                                                                                                  
 conv2d_8 (Conv2D)           (None, 8, 8, 128)            8320      ['add_1[0][0]']               
                                                                                                  
 add_2 (Add)                 (None, 8, 8, 128)            0         ['conv2d_7[0][0]',            
                                                                     'conv2d_8[0][0]']            
                                                                                                  
 global_average_pooling2d (  (None, 128)                  0         ['add_2[0][0]']               
 GlobalAveragePooling2D)                                                                          
                                                                                                  
 dense (Dense)               (None, 1)                    129       ['global_average_pooling2d[0][0]']                          
                                                                                                  
==================================================================================================
Total params: 297697 (1.14 MB)
Trainable params: 297697 (1.14 MB)
Non-trainable params: 0 (0.00 Byte)
__________________________________________________________________________________________________
```

With residual connections, you can build networks of arbitrary depth, without having to worry about vanishing gradients. Before moving onto the next section, here are 2 nice videos to watch:

> [!info] C4W2L03 Resnets  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=ZILIbUvp5lk&list=RDCMUCcIXc5mJsHVYTZR1maL5l9w&start_radio=1&t=38s&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=ZILIbUvp5lk&list=RDCMUCcIXc5mJsHVYTZR1maL5l9w&start_radio=1&t=38s&ab_channel=DeepLearningAI)  

> [!info] C4W2L04 Why ResNets Work  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=RYth6EbBUqM&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=RYth6EbBUqM&ab_channel=DeepLearningAI)  

**You should look to incorporate these into your projects when possible**. Now, let’s move onto the next essential convnet architecture pattern: _batch-normalization._

---

### Batch Normalization

_Normalization_ is a broad category of methods that seek to make different samples seen by a machine learning model more similar to each other, which helps the model learn and generalize well to new data. The most common form of data normalization is one you’ve already seen several times in this book: **centring the data on zero by subtracting the mean from the data and then dividing by the standard deviation**. In effect, this makes the assumption that the data follows a normal (or Gaussian) distribution.

Now previously, we’ve looked at instances where we’ve normalized data **before** feeding it into a model. However, data normalization may be of interest after every transformation operated by the network: **even if the data entering a** `**Dense**` **or** `**Conv2D**` **network has a 0 mean and unit variance**, there’s no reason to expect that this will be the case for the data coming out. In this case, _could normalizing intermediate activations help?_ This is what **batch normalization** is doing.

- This is done through `BatchNormalization` layer in Keras
- Adaptively normalizes data even as the mean and variance change over time during training
- During inference (when a big enough batch may not be available), it uses an exponential moving average of the batch-wise mean and variance of the data seen during training
- The comment to the right is actually true for a lot of deep learning — many things have hypothesis why they are true, but no certitudes. **You may sometimes feel like in this field, and in this book, its always** _“how to do things”_ **but never quite satisfactorily explains** _why_ **it works:** its because we know the how but we don’t know the why

  

In practice, the main effect of batch normalization appears to be that it helps with gradient propagation — much like residual connections — and thus allows for deeper networks.

> **Fun fact:** Batch normalization is used very liberally in many of the advanced convent architectures such as ResNet50, EfficientNet, an Exception

```Python
x = ...
x = layers.Conv2D(32, 3, use_bias=False)(x)
x = layers.BatchNormalization()(x)
```

> **NOTE** Both `Dense` and `Conv2D` involve a **_bias vector_**, a learned variable whose purpose is to make the layer **_affine_** rather than purely linear. For instance, `Conv2D` returns, schematically, `y = conv(x, kernel) + bias`, and `Dense` returns `y = dot(x, kernel) + bias`. Because the normalization step will take care of centring the layer’s output on zero, the bias vector is no longer needed when using `BatchNormalization`, and the layer can be created without it via the option `use_bias=False`. This makes the layer slightly **leaner**.

> **Note:** _Francis Chollet_ recommends (in his opinion) to place the previous layer’s activation **after** the batch normalization layer. Although, this is opinion various among many deep learning researchers:

```Python
### How not to use
x = layers.Conv2D(32, 3, activation="relu")(x)
x = layers.BatchNormalization()(x)

### How TO USE 
x = layers.Conv2D(32, 3, use_bias=False)(x) # note the lack of activation here 
x = layers.BatchNormalization()(x)
x = layers.Activation("relu")(x)
```

The intuitive reason for this approach is that batch normalization will centre your inputs on zero, while your $relu$﻿ activation uses zero as a pivot for keeping or dropping activated channels: doing normalization before the activation maximizes the utilization of the $relu$﻿.

> **On batch normalization and fine-tuning**
> 
> Batch normalization has many quirks. One of the main ones relates to fine-tuning:
> 
> - when fine-tuning a model that includes BatchNormalization layers, I recommend leaving these layers frozen (set their trainable attribute to False).
> - Otherwise they will keep updating their internal mean and variance, which can interfere with the very small updates applied to the surrounding Conv2D layers.

Now, let’s take a look at the last architecture pattern in our series: **depth-wise separable convolutions.**

---

### Depth-wise separable convolutions

What if I told you that there’s a layer you can use as a drop-in replacement for `Conv2D` that will make your model smaller _(fewer trainable weight parameters)_ and leaner(fewer floating-point operations) and cause it to perform a few percentage points better on its task? That is precisely what the _depth-wise separable convolution_ layer does (`SeparableConv2D` in Keras). What is this?

> [!info] Groups, Depthwise, and Depthwise-Separable Convolution (Neural Networks)  
> Patreon: https://www.  
> [https://www.youtube.com/watch?v=vVaRhZXovbw&ab_channel=AnimatedAI](https://www.youtube.com/watch?v=vVaRhZXovbw&ab_channel=AnimatedAI)  

The above is an amazing video on the this. Depth-wise separable convolutions have the same expressive power as normal convolutions, but they require a fraction of the memory and number of computations. So, how do they work? Here is a representation of how a regular convolution would work:

![[Screenshot_2024-02-08_at_9.22.43_AM.png]]

Now, the problem encountered with normal convolution, is it requires a substantial amount of computations and memory for trainable weights.

- **Doubling** the number of features in the input doubles the depth of filter
- **Doubling** the number of features in the output doubles the number of filters
- So simply doubling the number of features, requires **4x** more computation → this is far from ideal

The idea of depth-wise separable convolutions aims to reduce this, while maintaining the same expressive power.

![[Screenshot_2024-02-08_at_9.26.07_AM.png]]

So, what if we split the input into 2 parts? Now, the depth of each filter is cut in half. Half of the filters will convolve with the first half of the input, and the other half of the filters will convolve with the second half of the input. So now, we still have 8 filters, but much less computations and memory required. Why stop here? Why not split the input into every single channel?

![[Screenshot_2024-02-08_at_9.29.43_AM.png]]

So now, we have the above. Theres **one problem** → notice that each feature map in the output only depends on the first filter and the first input feature map. This will continue to propagate throughout the entire model, causing us to loss a lot of expressive power. The solution is, after such a convolution, we perform a convolution with a **1x1 kernel → point-wise convolution.**

![[Screenshot_2024-02-08_at_9.34.43_AM.png]]

![[Screenshot_2024-02-08_at_9.34.26_AM.png]]

So, two in the left are the same, and the right diagram demonstrates how they have the same expressive power. In this, the **depth-wise separable 3x3 convolution** needs only around **11% (9x faster)** of the **standard 3x3 convolution**. When it comes to larger-scale models, depthwise separable convolutions are the basis of the Xception architecture, a high-performing convnet that comes packaged with Keras. You can read more about the theoretical grounding for depth-wise separable convolutions and **Xception** in the paper _“Xception: Deep Learning with Depth- wise Separable Convolutions.”3:_

> [!info]  
>  
> [https://openaccess.thecvf.com/content_cvpr_2017/papers/Chollet_Xception_Deep_Learning_CVPR_2017_paper.pdf](https://openaccess.thecvf.com/content_cvpr_2017/papers/Chollet_Xception_Deep_Learning_CVPR_2017_paper.pdf)  

**→ It’s important to remember:** we shouldn’t just always use these over standard `Conv2D` layers. While they may be faster, it won’t always be a significant difference, only for large models. Standard Conv2D layers typically offer better representational capacity and are better suited for capturing complex spatial patterns and relationships in the data, whereas `SeparableConv2D` may sacrifice some representational capacity for efficiency

- `SeparableConv2D` layers are commonly used in tasks where computational efficiency is critical, such as mobile and embedded vision applications, where memory and processing power are limited.
- Standard `Conv2D` layers are often preferred in tasks where model accuracy and performance are of primary importance, such as high-resolution image classification or object detection tasks.

==When using== ==**separable convolutional layers**====, we are making one important assumption: “feature channels are largely independent”. If this does not hold, we should not consider using them.==

---
### Putting it Together: A mini Xception-like model

As a reminder, here are the convent architecture principles you’ve learned so far:

- Your model should be organized into repeated **_blocks of layers_**, usually made of multiple convolution layers and max pooling layer
- The number of filters in your layers should increase as the size of the spatial feature maps decreases
- Deep and narrow is better than broad and shallow
- Introducing residual connections around blocks of layers helps you train deeper networks
- Can be beneficial to introduce batch normalization layers after your convolution layers
- Cab be beneficial to replace `Conv2D` layers with `SeparableConv2D` layers, which are more parameter-efficient

With these ideas, we’ll build a simple model that resembles a smaller version of the Xception model, and we’ll apply it to the dogs vs. cats task from the last chapter. We’ll use the data from the previous example as well.

> [!info] Google Colaboratory  
>  
> [https://colab.research.google.com/drive/1W6z5_EH2vMECRnfvx3qFdbl7pxFCf_SI#scrollTo=0Gf9WRiqRVKh](https://colab.research.google.com/drive/1W6z5_EH2vMECRnfvx3qFdbl7pxFCf_SI#scrollTo=0Gf9WRiqRVKh)  

Here is the model:

```Python
from keras import layers

data_augmentation = keras.Sequential(
	[
		layers.RandomFlip("horizontal"),
		layers.RandomRotation(0.1),
		layers.RandomZoom(0.2),
	] 
)

inputs = keras.Input(shape=(180, 180, 3))
x = data_augmentation(inputs)

x = layers.Rescaling(1./255)(x)
x = layers.Conv2D(filters=32, kernel_size=5, use_bias=False)(x)

for size in [32, 64, 128, 256, 512]: 
  residual = x

  x = layers.BatchNormalization()(x)
  x = layers.Activation("relu")(x)
  x = layers.SeparableConv2D(size, 3, padding="same", use_bias=False)(x)
  x = layers.BatchNormalization()(x)
  x = layers.Activation("relu")(x)
  x = layers.SeparableConv2D(size, 3, padding="same", use_bias=False)(x)
  x = layers.MaxPooling2D(3, strides=2, padding="same")(x)
  residual = layers.Conv2D(
      size, 1, strides=2, padding="same", use_bias=False)(residual)
  x = layers.add([x, residual])

x = layers.GlobalAveragePooling2D()(x)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs=inputs, outputs=outputs)
```

![[Screenshot_2024-02-08_at_8.42.52_PM.png]]

You’ll find that our new model achieves a test accuracy of 90.8%, compared to 83.5% for the naive model in the last chapter. At this point, if you want to further improve performance, you should start systematically tuning the hyperparameters of your architecture—a topic we’ll cover in detail in chapter 13. We haven’t gone through this step here, so the configuration of the pre- ceding model is purely based on the best practices we discussed, plus, when it comes to gauging model size, a small amount of intuition.