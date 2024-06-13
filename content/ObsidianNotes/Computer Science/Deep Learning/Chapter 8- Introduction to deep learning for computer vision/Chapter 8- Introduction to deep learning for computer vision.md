Computer vision is the earliest and biggest success story of deep learning. Every day, you’re interacting with deep vision models — via Google Photos, Google image search, YouTube, video filters in camera apps, OCR software, and many more. These models are also at the heart of cutting-edge research in autonomous driving, robotics, AI-assisted medical diagnosis, autonomous retail checkout systems, and even autonomous farming.

> This chapter introduces convolutional neural networks, also known as **_convnets_**, the type of deep learning model that is now used almost universally in computer vision applications. You’ll learn to apply convnets to image-classification problems — in particular those involving small training datasets, which are the most common use case if you aren’t a large tech company.

Before jumping into code, what does a **convnet** aim to achieve? Convolutional Neural Networks (ConvNets or CNNs) are designed to automatically and adaptively learn hierarchical representations of data. In the context of image processing, the idea is that, as you progress through the layers of a ConvNet, the network learns to detect increasingly complex and abstract features.

- Perhaps the first layers learn about edges, textures, etc.
- Later layers learn complex and hierarchical features → combinations of edges, parts of objects, etc.
- In the last layers, you often find fully connected dense layers → these combine information learned from earlier layers to make predictions or classifications. The features at this stage are highly abstract (this image has ear shaped edges → is a cat).

  

The following listing shows what a basic convnet looks like. It’s a stack of **`Conv2D`** and **`MaxPooling2D`** layers. So, let’s see a quick example of **convnets** for MNIST recognition:

```Python
from tensorflow import keras
from tensorflow.keras import layers

inputs = keras.Input(shape=(28, 28, 1))
x = layers.Conv2D(filters=32, kernel_size=3, activation="relu")(inputs) 
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=64, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=128, kernel_size=3, activation="relu")(x)
x = layers.Flatten()(x)
outputs = layers.Dense(10, activation="softmax")(x)

model = keras.Model(inputs=inputs, outputs=outputs)
```

![[Screenshot_2024-01-27_at_12.26.05_PM.png]]

Recall that our simple `Sequential` model reached a `97.8%` accuracy on test data. With a convnet, the above reaches `99.1%` accuracy on test data … **significantly better.** But, why does this work so well? To answer this, we dive more into what the **“convolutional”** part means and what the **`Conv2D`** and **`MaxPooling2D`** layers are doing.

---

## The Convolution Operation

The convolution operation is something that has its roots in mathematics:

![[Screenshot_2024-01-27_at_3.06.52_PM.png]]

This then extends to image processing:

![[Screenshot_2024-01-27_at_3.07.56_PM.png]]

![[Screenshot_2024-01-27_at_3.11.37_PM.png]]

In a **filter** in convnets, is some matrix (in image recognition, this is a 3x3 matrix, also more commonly called **kernel**) that performs dot product with every 3x3 sub-matrix in the input tensor. So, how does this apply to **convolutional neural networks?** The model is essentially trying to find what the kernels should be, based on the data. To refresh, refer to the videos again:

> [!info] But what is a convolution?  
> Discrete convolutions, from probability to image processing and FFTs.  
> [https://www.youtube.com/watch?v=KuXjwB4LzSA&t=779s&ab_channel=3Blue1Brown](https://www.youtube.com/watch?v=KuXjwB4LzSA&t=779s&ab_channel=3Blue1Brown)  

> [!info] Convolutional Neural Networks (CNNs) explained  
> CNNs for deep learning  
> [https://www.youtube.com/watch?v=YRhxdVk_sIs&ab_channel=deeplizard](https://www.youtube.com/watch?v=YRhxdVk_sIs&ab_channel=deeplizard)  

The fundamental difference between a densely connected layer and a convolution layer is this: `**Dense**` layers can learn global patterns (things found across all samples in the dataset) in their input feature space:

- **e.g.** for a MNIST digit, patterns involving all pixels
    
    Whereas convolution layers learn local patterns — in the case of images, patterns found in small 2D windows of the inputs. In the above example, these small 2D windows were all 3x3.
    

![[Screenshot_2024-01-27_at_11.28.55_AM.png]]

Recall that each sample in MNIST is a 128x128 grey-scale picture, so a tensor with shape `(128,128,1)`. In the above model, the output tensor of the last layer had shape `(3,3,128)` — a 3x3 feature map of 128 channels. We’ll see more later on, on how the `Conv2D` layer does this, why this is the output, and how we interpret these.

  

So, this idea that convnets are capable of learning local patterns (feature maps), gives rise to 2 interesting properties:

1. _**The patterns learnt are translation-invariant**_. After learning a pattern in the lower-right, it can recognize it anywhere. On the contrary, consider a densely connected model for this — it would have to learn the pattern anew if it appeared at a new location. This makes convnets efficient when processing images (as real world images are fundamentally translation-invariant) → can use fewer training samples to learn representations that have generalization power.
2. _**They can learn spatial hierar**_c_**hies of patterns.**_ A first convolution layer will learn small local patterns such as edges, a second convolution layer will learn larger patterns made of the features of the first layers, and so on (shown below). This allows convnets to efficiently learn increasingly complex and abstract visual concepts: _The visual world is fundamentally spatially hierarchical._

![[Screenshot_2024-01-27_at_12.58.19_PM.png]]

Convolutions operate over rank-3 tensors called _feature maps_, with two spatial axes (_height_ and _width_) as well as a _depth_ axis (also called the _channels_ axis). The convolution operation extracts patches from its input feature map and applies the same transformation to all of these patches, producing an _output feature map_. This output feature map is still a rank-3 tensor: it has a width and a height. Now, we will go over how convolutions work in a **convnet.**

---

Consider the following MNIST example (visuals from the below source):

> [!info] Convolutional Neural Networks from Scratch | In Depth  
> Visualizing and understanding the mathematics behind convolutional neural networks, layer by layer.  
> [https://www.youtube.com/watch?v=jDe5BAsT2-Y&t=37s&ab_channel=far1din](https://www.youtube.com/watch?v=jDe5BAsT2-Y&t=37s&ab_channel=far1din)  

![[Screenshot_2024-01-29_at_8.07.45_PM.png]]

We start off with a single MNIST digit, which is actually a (28x28x1) tensor, where the last dimension is 1 since the image is grey-scale, so values can be represented from `0` to `255`. The first layer is our first convolutional layer. Let’s take a close look:

![[Screenshot_2024-01-29_at_8.34.39_PM.png]]

The yellow square at the bottom is the kernel (or filter). What happens in a convolutional layer is:

- For each corresponding matrix in the input layer (feature map, and in the above, consider the yellow square in the input layer), we take the dot-product between that matrix and the kernel. Once this is done, we shift the sub-square in the input layer one unit to the right. This movement is decided by the **stride value**.
- Once this is done, we will have a output feature map with shape (24x24x2). Why the 2? We have 2 kernels, so we run the above process for both kernels, each one creating a (24,24,1) output. We stack these on-top of each other to create the output tensor, and the feature map for the next layer.
- While this is for matrices, you can imagine that if the input feature map had last dimension > 2, we would be doing tensor dot-product.

Now, let’s ignore what **max-pooling layer** is doing for now. Given a (24x24x2) feature map, what would the next convolutional layer do?

![[Screenshot_2024-01-29_at_8.42.43_PM.png]]

Second layer for convnet

![[Screenshot_2024-01-29_at_8.42.25_PM.png]]

Same thing as left, just separated tensor for clarity

Now the next later has 4 filters, with kernel size being 3 once again. Imagine the pairs of matrices above as stacked on top of each other, forming shape (3,3,2) (rank-3 tensor). So, how does the **convolution** look like? Consider the first matrix pair (the first filter):

- Look at the following. Each filter has 2 matrices, and each one convolves in a different feature map dimension (as input one is 24,24,2). The result of each individual convolution is summed together to create an output:
    
    ![[Screenshot_2024-01-29_at_9.08.53_PM.png]]
    
    This layer will output 4 feature maps, which will be stacked together as input for the next layer (after Max-pooling)
    
    - In actuality, this is imagine this as tensor dot-product between the (24,24,2) feature map and the (3,3,2) filter, where the `2` is automatically decided by the input → need to match last dimension for tensor dot-product
- As you’ve probably noticed, the number of feature maps outputted from a layer is dependent on the number of filters being used. The above is 4 layers.

The above should provide a visual + intuitive explanation of what a convolution looks like in a **convnet**. Here is how the textbook visualizes this:

![[Screenshot_2024-01-29_at_9.16.25_PM.png]]

This is exactly what has been shown above, except as 3D tensors. First, we have out feature map, which in the above, is a (5,5,2) tensor → this is like each of the feature matrices in the images above. Now, we have three 3x3 input patches .. a.k.a, we have 3 filters, each one with kernel size 3, and of course, depth of 2 to match the input feature map. Then, consider the following for each filter:

- Look at the left-most input patch → this is a (3,3,2) tensor. This then multiplies (tensor dot-product) with every such shape in the input feature map (showed in the slightly greyed out image above). This operation results in a (3,3,1) shape.
- This is repeated with the other filters, hence why the output feature map is a (3,3,3) tensor.
---
### Max-Pooling Operation

After convolution layer, we often apply whats known as a **max-pooling operation**. So, what is this? Essentially, a max-pooling operation aims to reduce the dimensions of the feature map, while capturing the most important features in the feature map. Consider the following image:

![[Screenshot_2024-01-29_at_10.17.06_PM.png]]

Now, the red square is referred to as a **“pooling window”**. Just like how a filter operates, this moves over every thing in the input feature map. For that respective area in the feature map (as seen in image), the max of values in the area is taken, and added to the output feature map. Essentially, this aims to aggressively downsample feature maps. But why do we need to even downsample? Why not just keep fairly large feature maps all the way up? Consider what out initial MNIST model would look like without these max-pooling layers:

```Python
inputs = keras.Input(shape=(28, 28, 1))
x = layers.Conv2D(filters=32, kernel_size=3, activation="relu")(inputs)
x = layers.Conv2D(filters=64, kernel_size=3, activation="relu")(x)
x = layers.Conv2D(filters=128, kernel_size=3, activation="relu")(x)
x = layers.Flatten()(x)
outputs = layers.Dense(10, activation="softmax")(x)
model_no_max_pool = keras.Model(inputs=inputs, outputs=outputs)
```

Here is summary of the model:

```Python
>>> model_no_max_pool.summary()
Model: "model_1" 
_________________________________________________________________ 
Layer (type)             Output Shape                  Param # 
=================================================================
input_2 (InputLayer)  [(None, 28, 28, 1)]                 0
_________________________________________________________________ 
conv2d_3 (Conv2D)     (None, 26, 26, 32)                 320 
_________________________________________________________________ 
conv2d_4 (Conv2D)     (None, 24, 24, 64)                18496 
_________________________________________________________________
conv2d_5 (Conv2D)     (None, 22, 22, 128)               73856 
_________________________________________________________________ 
flatten_1 (Flatten)     (None, 61952)                     0 
_________________________________________________________________ 
dense_1 (Dense)           (None, 10)                    619530 
================================================================= 
Total params: 712,202
Trainable params: 712,202
Non-trainable params: 0 
_________________________________________________________________
```

What’s wrong with this setup? Two things:

- It isn’t conducive to learning a spatial hierarchy of features
    - By selecting the maximum value within each pooling window, the network retains the most important information and discards less relevant details. This enables the network to focus on the most discriminative features while reducing the spatial dimensions.
- Max-pooling reduces the spatial dimensions of the input feature maps, leading to a decrease in the number of parameters and computations in the subsequent layers. This reduction in computational complexity can make the network more efficient in terms of memory usage and training time.
- Also helps prevent over-fitting and improves generalization

---

### Training a convnet from scratch on a small dataset

Having to train an image-classification model using very little data is a common situation, which you’ll likely encounter in practice if you ever do computer vision in a professional context. As a practical example, we will focus on classifying **cats** and **dogs**.

- This dataset will contain 5,000 pictures of cats and dogs (2,500 for each). We will use 2000 pictures for training, 1000 for validation, and 2,000 for testing (very small training set).
    - We’ll see that a naive **convnet** reaches around 70%, which is our baseline, at which point the main issue becomes over-fitting
    - We will learn about _**data augmentation**_ → a powerful technique for mitigating over-fitting in computer vision

In the next section, we will look at 2 more essential techniques for applying deep learning models to small datasets: **feature extraction with a** _**pre-trained model**_ as well as _**fine-tuning a pre-trained model.**_

Together, these 3 strategies — training a small model from scratch, doing feature extraction using a pre-trained model, and fine-tuning a pre-trained model — will constitute our **toolbox** for tackling the problem of performing image classification with small datasets.

> deep learning models are by nature highly repurposable: you can take, say, an image-classification or speech-to-text model trained on a large-scale data- set and reuse it on a significantly different problem with only minor changes. Specifically, in the case of computer vision, many pre-trained models (usually trained on the ImageNet dataset) are now publicly available for download and can be used to boot-strap powerful vision models out of very little data. This is one of the greatest strengths of deep learning: feature reuse.

Now, let’s download the dataset → found on Kaggle:

> [!info] Dogs vs. Cats  
> Create an algorithm to distinguish dogs from cats  
> [https://www.kaggle.com/c/dogs-vs-cats/data](https://www.kaggle.com/c/dogs-vs-cats/data)  

==**The code for this can be found in my notebook to see the entirety**==

> [!info] Google Colaboratory  
>  
> [https://colab.research.google.com/drive/132hBqQzvf8hvYdtZts1J00GO5GGF218V#scrollTo=LYegk9Pw-7ht](https://colab.research.google.com/drive/132hBqQzvf8hvYdtZts1J00GO5GGF218V#scrollTo=LYegk9Pw-7ht)  

Now, let’s begin building the model. Let's now build the model. To start, we'll build a naive model: some **stacked convolutional layers** with **max-pooling** layers in-between. We'll use the same one as saw in the beginning for MNIST, but since images are now larger, we need to make our model accordingly larger.

- We’ll add 2 more **convolutional layers** → this will augment the capacity of the model and further reduce the size of the feature maps so that they aren’t overly large when we reach the `Flatten` layer.
- This is a binary classification task, so we will end with a single `**Dense**` layer with one neuron, and a `sigmoid` activation function.

```Python
from tensorflow import keras
from keras import layers

inputs = keras.Input(shape=(180, 180, 3))
x = layers.Rescaling(1./255)(inputs)
x = layers.Conv2D(filters=32, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=64, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=128, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=256, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=256, kernel_size=3, activation="relu")(x)
x = layers.Flatten()(x)

outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs=inputs, outputs=outputs)
```

This model looks like this:

```Python
Model: "model"
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 input_1 (InputLayer)        [(None, 180, 180, 3)]     0         
                                                                 
 rescaling (Rescaling)       (None, 180, 180, 3)       0         
                                                                 
 conv2d (Conv2D)             (None, 178, 178, 32)      896       
                                                                 
 max_pooling2d (MaxPooling2  (None, 89, 89, 32)        0         
 D)                                                              
                                                                 
 conv2d_1 (Conv2D)           (None, 87, 87, 64)        18496     
                                                                 
 max_pooling2d_1 (MaxPoolin  (None, 43, 43, 64)        0         
 g2D)                                                            
                                                                 
 conv2d_2 (Conv2D)           (None, 41, 41, 128)       73856     
                                                                 
 max_pooling2d_2 (MaxPoolin  (None, 20, 20, 128)       0         
 g2D)                                                            
                                                                 
 conv2d_3 (Conv2D)           (None, 18, 18, 256)       295168    
                                                                 
 max_pooling2d_3 (MaxPoolin  (None, 9, 9, 256)         0         
 g2D)                                                            
                                                                 
 conv2d_4 (Conv2D)           (None, 7, 7, 256)         590080    
                                                                 
 flatten (Flatten)           (None, 12544)             0         
                                                                 
 dense (Dense)               (None, 1)                 12545     
                                                                 
=================================================================
Total params: 991041 (3.78 MB)
Trainable params: 991041 (3.78 MB)
Non-trainable params: 0 (0.00 Byte)
_________________________________________________________________
```

**→ Data Pre-processing**

So, we now have a basic model built out. The next step is processing our images so that they can be used as training data for the NN. We have already split the data set into training, validation, and test (not shown here, but in the notebook). The steps for getting the images into the model are essentially:

1. Read the picture files
2. Decode the JPEG content to RGB grids of pixels
3. Convert these into floating-point tensors
4. Re-size them ti a shared size (we’ll use 180x180)
5. Pack them into batches (we’ll use 180x180)
6. Pack them into batches (we’ll use batches of 32 images)

Luckily, Keras has utilities to take care of these steps automatically through the utility function `image_dataset_from_directory()`, which let’s you quickly set up a data pipeline that can automatically turn image files into batches of pre-processed tensors:

```Python
from tensorflow.keras.utils import image_dataset_from_directory

train_dataset = image_dataset_from_directory(
    new_base_dir / "train",
    image_size=(180, 180),
    batch_size=32)
validation_dataset = image_dataset_from_directory(
    new_base_dir / "validation",
    image_size=(180, 180),
    batch_size=32)
test_dataset = image_dataset_from_directory(
    new_base_dir / "test",
    image_size=(180, 180),
    batch_size=32)

# the above returned values are tf.data.Dataset objects 
```

---

### Understanding TensorFlow `Dataset` object

Tensorflow makes available the `tf.data` API to create efficient input pipelines for machine learning models. Its core class is `tf.fata.Dataset`. A `Dataset` is an iterator: you can use it in a `for` loop. It will typically return batches of input data and labels. You can pass a `Dataset` object directly to the `fit()` method of a `Keras` model.


The Dataset class handles many key features that would otherwise be cumbersome to implement yourself—in particular, asynchronous data prefetching (preprocessing the next batch of data while the previous one is being handled by the model, which keeps execution flowing without interruptions). You can look this more later.

---

Going back to the dogs vs cats example, let’s see the shape of these `Dataset` objects:

```Python
for data_batch, labels_batch in train_dataset:
	print("data batch shape:", data_batch.shape)
	print("labels batch shape:", labels_batch.shape)
	break

-> data batch shape: (32, 180, 180, 3) 
-> labels batch shape: (32,)
```

Now, let’s begin training the model. We’ll pass the above `dataset` object `train_dataset` to the `.fit()` as well as the `validation_dataset`. Let’s use a callback to save the model every epoch with the arguments `save_best_only=True` and `monitor=val_loss`→ these metics tell the callback to only save a new file (overwriting the previous one) when the current value of the `val_loss` metric is lower than at any previous time during training.

- so, saved model will always contain the state of the model corresponding to its best performing epoch on validation data
    - With this, we won’t have to re-train a new model for a lower number of epochs if we start overfitting: we can just reload our saved file

Let’s see this part in action:

```Python
callbacks = [
	keras.callbacks.ModelCheckpoint(
		filepath="convnet_from_scratch.keras"
		save_best_only=True,
		monitor="val_loss")
]

history = model.fit(
    train_dataset,
    epochs=30,
    validation_data=validation_dataset,
    callbacks=callbacks)
```

After training our model, we obtain the following results:

![[Screenshot_2024-01-31_at_9.49.23_PM.png]]

Quite apparently, we are overfitting very early into the training. We have saved the model locally at its best performing state throughout this, and we can load it back in:

```Python
test_model = keras.models.load_model("convnet_from_scratch.keras")
test_loss, test_acc = test_model.evaluate(test_dataset)
print(f"Test accuracy: {test_acc:.3f}")

->
63/63 [==============================] - 2s 31ms/step - loss: 0.5863 - accuracy: 0.7465
Test accuracy: 0.747
```

This model, at its best, performed at a test accuracy of `74%`, let’s improve this.

  

**→ Data Augmentation**

With any model like this that is trained on a relatively small dataset (2000 images), our main concern is over-fitting. To combat this, we’ve explored various regularization techniques such as `L1` and `L2` as well as `Dropout`. We’ll now take a look at `Data Augmentation`, a regularization technique used almost _**universally**_ when processing images with deep learning models.

- Data augmentation takes the approach of generating more training data from existing samples via a number of random transformations that yield believable-looking images.
- The goal of this is so that your model will never see the exact same picture twice. This helps expose the model to more aspects of the data, so it can generalize better.
- We do this in `Keras` by adding a number of `data augmentation` layers at the start of the model

Consider:

```Python
data_augmentation = keras.Sequential(
	[
		layers.RandomFlip("horizontal"),
		layers.RandomRotation(0.1),
		layers.RandomZoom(0.2),
	] 
)
```

The above 3 are just a few of the available layers, let’s quickly go over them:

- `RandomFlip("horizontal")` — Applies horizontal flipping to a random 50% of the images that go through it
- `RandomRotation(0.1)` — Rotates the input images by a random value in the range **[-10%, +10%]** (these are fractions of a full circle — in degrees, the range would be **[-36 degrees, + 36 degrees]**)
- `RandomZoom(0.2)` — Zooms in or out of the image by a random factor in the range **[-20%, +20%]**
- Also consider things like changing `r,g,b` colours for pixels, maintaining the entity of the image

_You can see these augmentations in colab (link above)_

![[Screenshot_2024-01-31_at_10.59.04_PM.png]]

![[Screenshot_2024-01-31_at_10.59.46_PM.png]]

Each one of these augmentations is a new image for the model to train on. If we use this configuration, the model will never see the same input twice. But, the inputs it sees are still heavily correlated, as we are simply remixing existing information, not producing new information. As such, **this may not be enough to completely avoid over-fitting**. To further reduce over-fitting, we can add `Dropout` layers to the model, in particular, right before the densely connected classifier.

> **Note:** Random image augmentation layers, just like `Dropout`, are inactive during inference (whenever we call `predict()` and `evaluate()`).

Now, let’s see the new `convnet`:

```Python
inputs = keras.Input(shape=(180, 180, 3))

x = data_augmentation(inputs)
x = layers.Rescaling(1./255)(x)
x = layers.Conv2D(filters=32, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=64, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=128, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=256, kernel_size=3, activation="relu")(x)
x = layers.MaxPooling2D(pool_size=2)(x)
x = layers.Conv2D(filters=256, kernel_size=3, activation="relu")(x)
x = layers.Flatten()(x)
x = layers.Dropout(0.5)(x)

outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs=inputs, outputs=outputs)
model.compile(loss="binary_crossentropy",
              optimizer="rmsprop",
              metrics=["accuracy"])
```

Plotting the training out, we get:  
  

![[Screenshot_2024-02-02_at_1.40.14_PM.png]]

In this, we can see that the model begins to overfit around the 60-70 epoch mark

The accuracy for this model ends up consistently in the 80-85% range — a big improvement over our first try. We can again check this model’s performance by loading the best working model and evaluating it on test data:

```Python
test_model = keras.models.load_model(
            "convnet_from_scratch_with_augmentation.keras")
test_loss, test_acc = test_model.evaluate(test_dataset) 
print(f"Test accuracy: {test_acc:.3f}")
```

We could further tune the models parameters (such as the number of filters per convolution layer, or number of layers in the model), and we would likely be able to improve up to 90% accuracy. But, it would prove difficult to go any higher just by training our own **convnet** from scratch, since we have so little data to work with. To improve accuracy for such a problem, we will have to use **pre-trained models,** which is the main focus of the next section.