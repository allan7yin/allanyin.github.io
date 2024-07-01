### Leveraging a Pre-trained Model - `Transfer Learning`

> [!info] Transfer Learning (C3W2L07)  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=yofjFQddwHE&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=yofjFQddwHE&ab_channel=DeepLearningAI)  

A common and highly effective approach to deep learning on small images datasets is to use a pre-trained model. A _pre-trained model_ is a model that was previously trained on a large dataset, typically on a large-scale image-classification task. If the original datasets was large enough, and general enough, the spatial hierarchy learned by the pre-trained model can effectively act as a generic model.

- This is the case even when the new problems involve completely different than those of the original task
- For instance, you might train a model on ImageNet (where classes are mostly animals and everyday objects) and then repurpose this trained model for something as remote as identifying furniture items in images

> **→ When does transfer-learning make sense?**
> 
> - Transfer learning makes sense when we have a lot of data for the problem you are **transferring from,** and relatively little data for the problem you are **transferring to**
>     - e.g Say you gave **1 million** samples for image recognition task → a lot of data to learn low-level features in earlier layers
>     - Now, say we want model to recognize x-rays, and we only have **100** examples → very little data
>     - So, a lot of knowledge learned from image recognition can be transferred over
> - So, why does transfer learning work? Remember that in **convnets**, as the model deepens, the features captured become more abstract. Going back to the cat/dog example, in the later layers, the model may begin to recognize ears, or a whiskers → **in the earlier stages, the features extracted are curves and edges**
> - The idea is, these simple patters such as edges, can be re-used to train on additional images. So, if we have small amounts of data, it may make sense to simply only train the final layers of the model on the new data, and to leave the rest as is (pre-trained).

  

For this dog vs cat problem, we’ll use **VGG16 ArchitectureVGG16,** which is old (2014), but is easy to understand given the current content. This may be your first encounter with one of these cutesy model names — **VGG, ResNet, Inception, Xception**, and so on; you’ll get used to them because they will come up frequently if you keep doing deep learning for computer vision. There are two ways to use a pre-trained model: **_feature extraction_** and **_fine-tuning_**. We’ll cover both of them. Let’s start with feature extraction.

---

**→ Feature Extraction**

Feature extraction consists of using the representations learned by a previously trained model to extract interesting features from new samples. These features are then run through a classifier, which is trained from scratch.

- **Convnets we’ve seen consist of 2 parts:**
    - _**Convolutional Base:**_ Series of pooling and convolutional layers
    - Densely connected classifier

In the case of convnets, feature extraction consists of taking the convolutional base of a previously trained network, running the new data through it, and training a new classifier on top of the output.

![[Screenshot_2024-02-03_at_11.13.11_AM.png]]

We cannot re-use the densely connected portion. These are less generic, and have learned which combinations of features correspond to what object for the past problem. These relations may not hold for the new problem.

  

Thats not to say we should always be re-using the entire convolutional base either. Note that the level of generality (and therefore reusability) of the representations extracted by specific convolution layers depends on the depth of the layer in the model. Layers that come earlier in the model extract local, highly generic feature maps (such as visual edges, colours, and textures), whereas layers that are higher up extract more-abstract concepts (such as “cat ear” or “dog eye”). So if your new dataset differs a lot from the dataset on which the original model was trained, you may be better off using only the first few layers of the model to do feature extraction, rather than using the entire convolutional base.

  

Let’s put this into practice by using the **convolutional base** of the **VGG16 network** trained on ImageNet, to extract interesting features from cat and dog images, and then train a dog-versus-cats classifier on top of these features. This is already available on **Keras,** and many more.

```Python
conv_base = keras.applications.vgg16.VGG16(
    weights="imagenet",
    include_top=False,
    input_shape=(180, 180, 3))
```

We pass three arguments to the constructor:

- `weights` specifies the weight checkpoint from which to initialize the model.
- `include_top` refers to including (or not) the densely connected classifier on top of the network. By default, this densely connected classifier corresponds to the 1,000 classes from ImageNet. Because we intend to use our own densely connected classifier (with only two classes: cat and dog), we don’t need to include it.
- `input_shape` is the shape of the image tensors that we’ll feed to the network. This argument is purely optional: if we don’t pass it, the network will be able to process inputs of any size. Here we pass it so that we can visualize (in the following summary) how the size of the feature maps shrinks with each new convolution and pooling layer.

  

Here is what the model looks like:

```Python
Model: "vgg16"
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 input_1 (InputLayer)        [(None, 180, 180, 3)]     0         
                                                                 
 block1_conv1 (Conv2D)       (None, 180, 180, 64)      1792      
                                                                 
 block1_conv2 (Conv2D)       (None, 180, 180, 64)      36928     
                                                                 
 block1_pool (MaxPooling2D)  (None, 90, 90, 64)        0         
                                                                 
 block2_conv1 (Conv2D)       (None, 90, 90, 128)       73856     
                                                                 
 block2_conv2 (Conv2D)       (None, 90, 90, 128)       147584    
                                                                 
 block2_pool (MaxPooling2D)  (None, 45, 45, 128)       0         
                                                                 
 block3_conv1 (Conv2D)       (None, 45, 45, 256)       295168    
                                                                 
 block3_conv2 (Conv2D)       (None, 45, 45, 256)       590080    
                                                                 
 block3_conv3 (Conv2D)       (None, 45, 45, 256)       590080    
                                                                 
 block3_pool (MaxPooling2D)  (None, 22, 22, 256)       0         
                                                                 
 block4_conv1 (Conv2D)       (None, 22, 22, 512)       1180160   
                                                                 
 block4_conv2 (Conv2D)       (None, 22, 22, 512)       2359808   
                                                                 
 block4_conv3 (Conv2D)       (None, 22, 22, 512)       2359808   
                                                                 
 block4_pool (MaxPooling2D)  (None, 11, 11, 512)       0         
                                                                 
 block5_conv1 (Conv2D)       (None, 11, 11, 512)       2359808   
                                                                 
 block5_conv2 (Conv2D)       (None, 11, 11, 512)       2359808   
                                                                 
 block5_conv3 (Conv2D)       (None, 11, 11, 512)       2359808   
                                                                 
 block5_pool (MaxPooling2D)  (None, 5, 5, 512)         0         
                                                                 
=================================================================
Total params: 14714688 (56.13 MB)
Trainable params: 14714688 (56.13 MB)
Non-trainable params: 0 (0.00 Byte)
_________________________________________________________________
```

Notice that the final feature map has shape `(5,5,512)` . This is the feature map on top of which we’ll stick a densely connected classifier. At this point, there are 2 ways that we can proceed with this:

1. Run the **convolutional base** over our dataset, record its output to a **NumPy Array**, and then use this data as input to a standalone, densely connected classifier, similar to the ones seen in [[Chapter 4- Getting started with NN- Classification and Regression]]**.**
    1. This solution is fast and cheap to run
    2. The convolutional base needs to run only once, and this is the most expensive part of the pipeline
    3. This technique won’t allow us to use **data augmentation**
2. Extend the model we have `conv_base` by adding `Dense` layers on top, and run the whole thing from end to end on the input data
    1. This allows us to use data augmentation
    2. This technique is far more expensive that the first option

  

We’ll learn how to use both of these. Let’s first try the first one:

---

**→ Fast Feature Extraction Without Data Augmentation**

We’ll start by extracting features as NumPy arrays by calling the `predict()` method of the `conv_base` model on our training, validation, and test data sets:

```Python
import numpy as np

def get_features_and_labels(dataset): 
	all_features = []
	all_labels = []
	for images, labels in dataset:
		preprocessed_images = keras.applications.vgg16.preprocess_input(images) 
		features = conv_base.predict(preprocessed_images) 
		all_features.append(features)
		all_labels.append(labels)
	return np.concatenate(all_features), np.concatenate(all_labels)

train_features, train_labels = get_features_and_labels(train_dataset) 
val_features, val_labels = get_features_and_labels(validation_dataset) 
test_features, test_labels = get_features_and_labels(test_dataset)
```

So, what is this doing? `predict()` only expects images, not labels, but our current data set yields batches that contain images and their labels. Moreover, the `VGG16` model expects inputs that are pre-processed with the function `keras.applications.vgg16.preprocess_input` , which scales pixels to an appropriate range. Let’s see the shape of the extracted features:

```Python
>>> train_features.shape
(2000, 5, 5, 512)
```

We finish this model with our own densely connected layers, regularized with dropout:

```Python
inputs = keras.Input(shape=(5, 5, 512))
x = layers.Flatten()(inputs)
x = layers.Dense(256)(x)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs, outputs)

model.compile(loss="binary_crossentropy",
                     optimizer="rmsprop",
                     metrics=["accuracy"])
callbacks = [
    keras.callbacks.ModelCheckpoint(
       filepath="feature_extraction.keras",
       save_best_only=True,
       monitor="val_loss")
]
history = model.fit(
    train_features, train_labels,
    epochs=20,
    validation_data=(val_features, val_labels),
    callbacks=callbacks)
```

```Python
.
.
.
Epoch 20/20
63/63 [==============================] - 0s 6ms/step - loss: 0.0519 - accuracy: 0.9985 - val_loss: 6.4517 - val_accuracy: 0.9690
```

```Python
import matplotlib.pyplot as plt

acc = history.history["accuracy"]
val_acc = history.history["val_accuracy"]
loss = history.history["loss"]
val_loss = history.history["val_loss"]
epochs = range(1, len(acc) + 1)
plt.plot(epochs, acc, "bo", label="Training accuracy") 
plt.plot(epochs, val_acc, "b", label="Validation accuracy") 
plt.title("Training and validation accuracy")
plt.legend()
plt.figure()
plt.plot(epochs, loss, "bo", label="Training loss")
plt.plot(epochs, val_loss, "b", label="Validation loss")
plt.title("Training and validation loss")
plt.legend()
plt.show()
```

We reach a validation accuracy of about **97%** — much better than we achieved in the previous section with the small model trained from scratch. This is a bit of an unfair comparison, however, because ImageNet contains many dog and cat instances, which means that our pre-trained model already has the exact knowledge required for the task at hand. This won’t always be the case when you use pre-trained features.

![[Screenshot_2024-02-03_at_8.43.05_PM.png]]

You’ll notice that we over-fit almost instantly, even with a fairly large dropout. This is because this method does not use data-augmentation, which is meant to avoid it. We’ll now look at it.

---

### **Feature Extraction Together with Data Augmentation**

This will be slower and more expensive than the first technique mentioned, but, it allows us to use data-augmentation during training: creating a model that chains the `conv_base` with a new dense classifier, and training it end to end on the inputs.

  

Something important to do, is to _**freeze the convolutional base**_ so that it’s weights can’t move. Otherwise, if we do not freeze them, the representations previously learned by the convolutional base will be modified during training. This means that weights in our final new dense layers, will propagate through the rest of the model, destroying the representations previously learned.

```Python
conv_base  = keras.applications.vgg16.VGG16(
    weights="imagenet",
    include_top=False)
conv_base.trainable = False
```

So now, our new model consist of:

1. Data Augmentation layer
2. Frozen pre-trained convnet model
3. Dense classifier

```Python
data_augmentation = keras.Sequential(
    [
        layers.RandomFlip("horizontal"),
        layers.RandomRotation(0.1),
        layers.RandomZoom(0.2),
] )
inputs = keras.Input(shape=(180, 180, 3))
x = data_augmentation(inputs)
x = keras.applications.vgg16.preprocess_input(x)
x = conv_base(x)
x = layers.Flatten()(x)
x = layers.Dense(256)(x)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(1, activation="sigmoid")(x)
model = keras.Model(inputs, outputs)
model.compile(loss="binary_crossentropy",
              optimizer="rmsprop",
              metrics=["accuracy"])
```

This results in the following validation accuracy and test accuracy:

![[Screenshot_2024-02-04_at_11.52.31_AM.png]]

```Python
Epoch 50/50
63/63 [==============================] - 9s 139ms/step - loss: 0.3046 - accuracy: 0.9930 - val_loss: 2.7341 - val_accuracy: 0.9780

63/63 [==============================] - 6s 87ms/step - loss: 2.2369 - accuracy: 0.9730
Test accuracy: 0.973
```

We end with a test accuracy of `97.3%` . This is really a slight modest improvement over the first technique we used, which is disappointing given the strong results on the validation data.

---

### Fine-tuning a pre-trained model

![[Screenshot_2024-02-04_at_12.46.20_PM.png]]

  

- Another widely used technique for model reuse, complementary to feature extraction, is fine-tuning. **Fine-tuning** consists of unfreezing a few of the top layers of the convolutional base of a frozen model for feature extraction, and jointly training both the newly added part of the model (in this case, the fully connected classifier) and these top layers.
    - Slightly adjusts the more abstract representations of the model being reused in order to make them more relevant for the problem at hand
- Earlier, we saw that, we need to freeze the convolutional base during training, otherwise, back-propagation would destroy the previously trained results.
- For the same reason, it’s only possible to fine-tune the top layers of the convolutional base once the classifier on top has already been trained
    - Classifier is the ==**green**== section to the left
    - top layers of the convolutional base are ==**yellow**==
- Thus, here are the steps for fine-tuning a network:
    
    - Add our custom network on top of an already-trained base network
    - Freeze the base network
    - Train the part we added
    - Unfreeze some layers in the base network
    - Jointly train both these layers and the part we added
    
      
    

We’ve already done the first 3 → Let’s finish the last 2

- We’ll fine-tune the last 3 convolutional layers (so `conv block 5`)

So, if we’re going to be first training the entire model with the convolutional base frozen, why don’t we then just unfreeze everything, and fine-tune the entire convolutional base. **_Technically, we could, but you need to consider:_**

- Earlier layers are more generic features → more useful to fine-tune higher layers as these encode more specialized features
- Risks over-fitting → convolutional base has 15 million parameters, so its risky to attempt to train it on your small dataset

```Python
conv_base.trainable = True
for layer in conv_base.layers[:-4]:
    layer.trainable = False
```

Now we can begin fine-tuning the model. We’ll do this with the `RMSprop` optimizer, using a very low learning rate. The reason for using a low learning rate is that we want to limit the magnitude of the modifications we make to the representations of the three layers we’re fine-tuning. Updates that are too large may harm these representations.

```Python
model.compile(loss="binary_crossentropy",
              optimizer=keras.optimizers.RMSprop(learning_rate=1e-5),
              metrics=["accuracy"])
callbacks = [
    keras.callbacks.ModelCheckpoint(
        filepath="fine_tuning.keras",
        save_best_only=True,
        monitor="val_loss")
]
history = model.fit(
    train_dataset,
    epochs=30,
    validation_data=validation_dataset,
    callbacks=callbacks)
```

Here, we get a test accuracy of `98.5%`. This would one of the top submissions for this original competition, although an unfair comparison, since in the original competition, pre-trained models were not already trained on images such as cats and dogs (VGG16 is).

---
### Summary

- Convnets are the best type of machine learning models for computer vision tasks. It’s possible to train one from scratch even on a very small dataset, with decent results.
- Convnets work by learning a hierarchy of modular patterns and concepts to represent the visual world.
- On a small dataset, overfitting will be the main issue. Data augmentation is a powerful way to fight overfitting when you’re working with image data.
- It’s easy to reuse an existing convnet on a new dataset via feature extraction. This is a valuable technique for working with small image datasets.
- As a complement to feature extraction, you can use fine-tuning, which adapts to a new problem some of the representations previously learned by an existing model. This pushes performance a bit further.

---

**→ Interesting**

There is also multi-task learning. **Transfer learning** is a sequential process, when you learn from task A and transfer that task B. In multi-task learning, you simultaneously try to have one NN do several things at the same time, and each of these tasks helps, hopefully, all of the other tasks.

> [!info] Multitask Learning (C3W2L08)  
> Take the Deep Learning Specialization: http://bit.  
> [https://www.youtube.com/watch?v=UdXfsAr4Gjw&ab_channel=DeepLearningAI](https://www.youtube.com/watch?v=UdXfsAr4Gjw&ab_channel=DeepLearningAI)