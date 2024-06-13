### Fundamentals of Machine Learning

This chapter focuses on the essence of machine learning, and key concepts related to:

- the tension between generalization and optimization
- evaluation methods for machine learning models
- best practices to improve model fitting
- best practices to achieve better generalization

**Bias:**

- Bias refers to the error introduced by approximating a real-world problem, which may be complex, by a too-simplistic model.
- High bias can lead to underfitting, where the model is too simple and unable to capture the underlying patterns in the data.

### Generalization

Machine learning models are capable of learning because of generalization. This is important as we've witnessed the central problem of machine learning: _over-fitting_. Generalization is an important topic in machine learning. As humans, we exhibit a remarkable ability to interpolate: too see data, and infer others. In real-world datasets, it’s fairly common for some inputs to be invalid. Perhaps a MNIST digit could be an all-black image, for instance, or something like figure 5.2.

![[Untitled 3.png|Untitled 3.png]]

- What are these? Who knows. What's worse is that sometimes, we will have perfectly fine data, that is mislabeled. If a model were to go out of its way to incorporate these outliers, its generalization performance would degrade.
- A remarkable fact about deep learning models is that they can be trained to fit anything, as long as they have enough representational power. Don’t believe me? Try shuffling the MNIST labels and train a model on that. Even though there is no relationship whatsoever between the inputs and the shuffled labels, the training loss goes down just fine, even with a relatively small model. Naturally, the validation loss does not improve at all over time, since there is no possibility of generalization in this setting.
- As it turns out, the nature of generalization in deep learning has rather little to do with deep learning models themselves, and much to do with the structure of information in the real world. Let’s take a look at what’s really going on here.

  

The input to an MNIST classifier (before preprocessing) is a 28 × 28 array of integers between 0 and 255. The total number of possible input values is thus 256 to the power of 784 — much greater than the number of atoms in the universe. However, very few of these inputs would look like valid MNIST samples: actual handwritten digits only occupy a tiny subspace of the parent space of all possible 28 × 28 uint8 arrays. What’s more, this subspace isn’t just a set of points sprinkled at random in the parent space: it is highly structured. First, the subspace of valid handwritten digits is continuous: if you take a sample and modify it a little, it will still be recognizable as the same handwritten digit. Further, all samples in the valid subspace are connected by smooth paths that run through the subspace. This means that if you take two random MNIST digits A and B, there exists a sequence of “intermediate” images that morph A into B, such that two consecutive digits are very close to each other (see figure 5.7). Perhaps there will be a few ambiguous shapes close to the boundary between two classes, but even these shapes would still look very digit-like.

  

In technical terms, you would say that handwritten digits form a manifold within the space of possible 28 × 28 uint8 arrays. That’s a big word, but the concept is pretty intuitive. A “manifold” is a lower-dimensional subspace of some parent space that is locally similar to a linear (Euclidian) space. For instance, a smooth curve in the plane is a 1D manifold within a 2D space, because for every point of the curve, you can draw a tangent (the curve can be approximated by a line at every point). A smooth surface within a 3D space is a 2D manifold. And so on. This applies to not only MNIST data. This is why we have the _manifold hypothesis_, which hypothesizes that this manifold idea, applies to all real-world data. This so far has been the case. **The manifold hypothesis implies that:**

- Machine learning models only have to fit relatively simple, low-dimensional, highly structured subspaces within their potential input space (latent mani- folds).
- Within one of these manifolds, it’s always possible to interpolate between two inputs, that is to say, morph one into another via a continuous path along which all points fall on the manifold.

---

As we can see, there are many things to consider, and we've seen sort of how we need to prepare data and how a model would preserve generalization. While deep learning is well-suited to manifold learning, the power to generalize is more a consequence of the natural structure of your data than a consequence of any property of your model. You’ll only be able to generalize if your data forms a manifold where points can be interpolated.

As we know, we can imagine deep learning as "curve-fitting" to the manifold. For a model to perform well, it needs a dense sampling. This means that the training data should densely cover the entirety of the input data manifold (training data should not be all concentrated in one area, needs to be spread out to allow for interpolation).

A denser coverage of the input data manifold will yield a model that generalizes better. You should never expect a deep learning model to perform anything more than crude interpolation between its training samples, and thus you should do everything you can to make interpolation as easy as possible.

---

### Evaluating Machine Learning Models

Another important thing we need to mention is evaluating machine learning models. We will formally show the evaluation methods (although we've already seen them in past chapters).

  

Evaluating a model always boils down to splitting the available data into three sets: training, validation, and test. You train on the training data and evaluate your model on the validation data. Once your model is ready for prime time, you test it one final time on the test data, which is meant to be as similar as possible to production data. Then you can deploy the model in production.

  

Why do we need to separate validation and training data? Part of developing a model is configuration: for example, choosing the number of layers or the size of the layers (called the hyperparameters of the model, to distinguish them from the parameters, which are the network’s weights). You do this tuning by using as a feedback signal the performance of the model on the validation data. In essence, this tuning is a form of learning: a search for a good configuration in some parameter space. As a result, tuning the configuration of the model based on its performance on the validation set can quickly result in over-fitting to the validation set, even though your model is never directly trained on it.

Splitting your data into training, validation, and test sets may seem straightforward, but there are a few advanced ways to do it that can come in handy when little data is available. Let’s review three classic evaluation recipes: simple holdout validation, K-fold validation, and iterated K-fold validation with shuffling.

---

### Simple Holdout Validation

- The most fundamental one. Set apart some fraction of your data as your test set. Train on the remaining data, and evaluate on the test set. As you saw in the previous sections, in order to prevent information leaks, you shouldn’t tune your model based on the test set, and therefore you should also reserve a validation set.
- This is the simplest evaluation protocol, and it suffers from one flaw: if little data is available, then your validation and test sets may contain too few samples to be statistically representative of the data at hand.

### K-Fold Cross-Validation

With this approach, you split your data into K partitions of equal size. For each partition i, train a model on the remaining K - 1 partitions, and evaluate it on partition i. Your final score is then the averages of the K scores obtained. This method is helpful when the performance of your model shows significant variance based on your train- test split. Like holdout validation, this method doesn’t exempt you from using a distinct validation set for model calibration.

![[Untitled 1 2.png|Untitled 1 2.png]]

### Iterated K-Fold Validation with Shuffling

This one is for situations in which you have relatively little data available and you need to evaluate your model as precisely as possible. I’ve found it to be extremely helpful in **Kaggle** competitions. It consists of applying K-fold validation multiple times, shuffling the data every time before splitting it K ways. The final score is the average of the scores obtained at each run of K-fold validation. **Note that you end up training and evaluating P * K models (where P is the number of iterations you use), which can be very expensive.**

### **Things to keep in mind about model evaluation**

- **Data representativeness:** You want both your training set and test set to be representative of the data at hand. We shouldn't just take the first, say, 80% as training and the remaining as test, as we don't know the order or relative order of types in the data. **For this reason, you usually should randomly shuffle your data before splitting it into training and test sets.**
- **The arrow of time:** If you’re trying to predict the future given the past, you should not randomly shuffle your data before splitting it, because doing so will create a temporal leak: your model will effectively be trained on data from the future. In such situations, you should always make sure all data in your test set is posterior to the data in the training set.
- **Redundancy in your data:** If some data points in your data appear twice (fairly common with real-world data), then shuffling the data and splitting it into a training set and a validation set will result in redundancy between the training and validation sets. In effect, you’ll be testing on part of your training data, which is the worst thing you can do! **Make sure your training set and validation set are disjoint.**

---

### Improving Model Fit

To achieve the perfect fit, you must first overfit. Since you don’t know in advance where the boundary lies, you must cross it to find it. Thus, your initial goal as you start working on a problem is to achieve a model that shows some generalization power and that is able to overfit. Once you have such a model, you’ll focus on refining generalization by fighting overfitting. There are 3 main problems we'll encounter:

1. Training doesn’t get started: your training loss doesn’t go down over time.
2. Training gets started just fine, but your model doesn’t meaningfully generalize:  
    you can’t beat the common-sense baseline you set.  
    
3. Training and validation loss both go down over time, and you can beat your  
    baseline, but you don’t seem to be able to overfit, which indicates you’re still under-fitting.  
    

Let's see ways to address this issues:

### Tuning Gradient Descent Parameters

Sometimes, training doesn’t get started or your training loss doesn’t go down over time. This is a situation we an **ALWAYS** fix. When this happens, it’s always a problem with the configuration of the gradient descent process: your choice of optimizer, the distribution of initial values in the weights of your model, your learning rate, or your batch size. All these parameters are interdependent, and as such it is usually sufficient to tune the learning rate and the batch size while keeping the rest of the parameters constant.

### Leveraging better architecture priors

You have a model that fits, but for some reason your validation metrics aren’t improving at all. They remain no better than what a random classifier would achieve: your model trains but doesn’t generalize. What’s going on? **This is perhaps the worst machine learning situation you can find yourself in.** **It indicates that something is fundamentally wrong with your approach, and it may not be easy to tell what.** It could the model being used is not the best, data is not sufficient enough to predict targets, or other architecture decisions. In general, you should always make sure to read up on architecture best practices for the kind of task you’re attacking—chances are you’re not the first person to attempt it.

---

Now, we examine some additional information that is crucial to know during the process of configuring our model.

### Increasing Model Capacity

If you manage to get to a model that fits, where validation metrics are going down, and that seems to achieve at least some level of generalization power, congratulations: you’re almost there. Next, you need to get your model to start overfitting. Consider the following small model - a simple logistic regression trained on MNIST pixels:

```Python
model = keras.Sequential([layers.Dense(10, activation="softmax")])
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])
history_small_model = model.fit(
    train_images, train_labels,
    epochs=20,
    batch_size=128,
    validation_split=0.2)

# lets plot out the loss to see the learning
import matplotlib.pyplot as plt
val_loss = history_small_model.history["val_loss"] epochs = range(1, 21)
plt.plot(epochs, val_loss, "b--",
         label="Validation loss")
plt.title("Effect of insufficient model capacity on validation loss")
plt.xlabel("Epochs")
plt.ylabel("Loss")
plt.legend()
```

Here is what we would see:

![[Untitled 2 3.png|Untitled 2 3.png]]

  

Validation metrics seem to stall, or to improve very slowly, instead of peaking and reversing course. The validation loss goes to 0.26 and just stays there. You can fit, but you can’t clearly overfit, even after many iterations over the training data. You’re likely to encounter similar curves often in your career. **Remember that it should always be possible to overfit****.** Here are potential reasons for this:

- **_representational power of model_****:** the model is not powerful enough, one with more capacity, that is, one that can store more information. How can we do this?
    - adding more layers (layers with more parameters)
    - using kinds of layer that are more appropriate for the problem at hand

Let’s try training a bigger model, one with two intermediate layers with 96 units each:

```Python
model = keras.Sequential([
    layers.Dense(96, activation="relu"),
    layers.Dense(96, activation="relu"),
    layers.Dense(10, activation="softmax"),
])
model.compile(optimizer="rmsprop",
              loss="sparse_categorical_crossentropy",
              metrics=["accuracy"])
history_large_model = model.fit(
    train_images, train_labels,
    epochs=20,
    batch_size=128,
    validation_split=0.2)
```

Here is what we now see:

![[Untitled 3 2.png|Untitled 3 2.png]]

Clearly, we can see when this model will overtrain (around 8-9 epochs).

---

**Once your model has shown itself to have some generalization power and to be able to overfit, it’s time to switch your focus to maximizing generalization.**

### Improving Generalization

### 1. Dataset Curation

Deep learning is curve fitting, not magic. As such, it is essential that you make sure that you’re working with an appropriate dataset. Spending more effort and money on data collection almost always yields a much greater return on investment than spending the same on developing a better model.

- Make sure you have enough data. Remember that you need a dense sampling of the input-cross-output space. More data will yield a better model. Sometimes, problems that seem impossible at first become solvable with a larger dataset.
- Minimize labeling errors—visualize your inputs to check for anomalies, and proofread your labels.
- Clean your data and deal with missing values (we’ll cover this in the next chapter).
- If you have many features and you aren’t sure which ones are actually useful, do  
    feature selection.  
    

A particularly important way to improve the generalization potential of your data is feature engineering. For most machine learning problems, feature engineering is a key ingredient for success. Let’s take a look.

  

### 2. Feature Engineering

Feature engineering is the process of using your own knowledge about the data and about the machine learning algorithm at hand (in this case, a neural network) to make the algorithm work better by applying hardcoded (non-learned) transformations to the data before it goes into the model. In many cases, it isn’t reasonable to expect a machine learning model to be able to learn from completely arbitrary data. The data needs to be presented to the model in a way that will make the model’s job easier.

### Using Early Stopping

- In deep learning, we always use models that are vastly over parameterized: they have way more degrees of freedom than the minimum necessary to fit to the latent manifold of the data. This over parameterization is not an issue, because you never fully fit a deep learning model. Such a fit wouldn’t generalize at all. You will always interrupt training long before you’ve reached the minimum possible training loss.
- In the examples in the previous chapter, we would start by training our models for longer than needed to figure out the number of epochs that yielded the best validation metrics, and then we would retrain a new model for exactly that number of epochs. This is pretty standard, but it requires you to do redundant work, which can sometimes be expensive.
- In Keras, it’s typical to do this with an EarlyStopping callback, which will interrupt training as soon as validation metrics have stopped improving, while remembering the best known model state. You’ll learn to use callbacks in chapter 7.

---

### Regularizing your model

Regularization is a technique used in deep learning to prevent overfitting and improve generalization ability of a model. Regularization helps to address this issue by adding constraints or penalties to the model's learning process. Here are some methods of doing so:

- **Reducing the network's size**
    - A model that is too small will not overfit. The simplest way to mitigate overfitting is to reduce the size of the model (the number of learnable parameters in the model, determined by the number of layers and the number of units per layer)
    - If the model has limited memorization resources, it won’t be able to simply memorize its training data; thus, in order to minimize its loss, it will have to resort to learning compressed representations that have predictive power regarding the targets - precisely the type of representations we’re interested in
    - At the same time, keep in mind that you should use models that have enough parameters that they don’t under-fit: your model shouldn’t be starved for memorization resources. There is a compromise to be found between too much capacity and not enough capacity.
    - **There is no magical formula that just optimizes this for you. Kind of trial and error**

The general workflow for finding an appropriate model size is to start with relatively few layers and parameters, and increase the size of the layers or add new layers until you see diminishing returns with regard to validation loss.

---

### Adding Weight Regularization

- ==we prefer simpler models over more complex ones==
- ==hence, we want to minimize the number of neurons needed, whilst still being able to over-fit==

A simple model in this context is a model where the distribution of parameter values has less entropy (or a model with fewer parameters, as you saw in the previous section). Thus, a common way to mitigate overfitting is to put constraints on the complexity of a model by forcing its weights to take only small values, which makes the distribution of weight values more regular. This is called weight regularization, and it’s done by adding to the loss function of the model, a cost associated with having large weights. This cost comes in two flavours:

1. **L1 regularization (Lasso Regression)** —The cost added is proportional to the absolute value of the weight coefficients (the L1 norm of the weights).
2. **L2 regularization (Ridge Regression)** —The cost added is proportional to the square of the value of the weight coefficients (the L2 norm of the weights). L2 regularization is also called weight decay in the context of neural networks. Don’t let the different name confuse you: weight decay is mathematically the same as L2 regularization.

→ L1 regularization adds a penalty term to the loss function that is proportional to the absolute value of the weights. The L1 regularization term has the effect of shrinking less important weights towards zero, effectively selecting a subset of the most relevant features.

→ L2 regularization adds a penalty term to the loss function that is proportional to the square of the weights. It encourages smaller weights without enforcing sparsity as strongly as L1 regularization. The L2 regularization term penalizes large weights more than small weights, leading to a more balanced reduction in all weights. This helps prevent individual weights from dominating the model's learning process.

The regularization parameter (λ) controls the strength of the regularization effect. A larger value of λ will result in more weight decay and stronger regularization.

  

**→ How do we decide on what λ parameter to use?**

Won’t look into this as not mentioned.

To add model weight regularization in Keras, we do:

```Python
from tensorflow.keras import regularizers 

model = keras.Sequential([
    layers.Dense(16,
                 kernel_regularizer=regularizers.l2(0.002),
                 activation="relu"),
    layers.Dense(16,
                 kernel_regularizer=regularizers.l2(0.002),
                 activation="relu"),
    layers.Dense(1, activation="sigmoid")
])
model.compile(optimizer="rmsprop",
              loss="binary_crossentropy",
              metrics=["accuracy"])

history_l2_reg = model.fit(
    train_data, train_labels,
    epochs=20, batch_size=512, validation_split=0.4)
```

Here, we have selected to use the L2 weight regulator, with regularization parameter of 0.002. When to use L1 and when to use L2? This question is fairly dependent on the data being processed, but here is one potential indicator:

- L1 regularization encourages sparsity in the model, meaning it promotes solutions with fewer non-zero weights. If you expect that only a small number of features are relevant, or if you want a model that emphasizes a smaller set of influential features, L1 regularization is a suitable choice.
- L2 regularization does not lead to sparse solutions like L1 regularization. If you believe that all features are potentially relevant to the problem and want the model to make use of all available information, L2 regularization is a suitable choice.

Do more research for the specific problem you are tackling. **Note that weight regularization is more typically used for smaller deep learning models**. ==**Large deep learning models tend to be so over-parameterized that imposing constraints on weight values hasn’t much impact on model capacity and generalization. In these cases, a different regularization technique is preferred: dropout.**==

### Dropout

Dropout regularization is a technique used in deep learning to prevent overfitting and improve the generalization ability of neural networks. It randomly drops out (i.e., temporarily removes) a certain percentage of neurons during the training process. This helps to create a more robust and less sensitive model by reducing interdependencies between neurons.

Here's an overview of how dropout regularization works:

- During training
    - For each training example and for each layer of the neural network, dropout is applied.
    - A dropout rate is defined, typically ranging from 0.1 to 0.5. It represents the probability that a neuron will be temporarily dropped out.
    - Neurons are randomly selected to be dropped out with the defined probability. The selected neurons are effectively deactivated for that particular forward and backward pass.
    - The forward pass and backward pass are then performed on the modified network, treating the dropped out neurons as inactive. This leads to updating only a fraction of the model's weights in each training iteration.
- During testing/inference
    - During testing or inference, dropout is not applied. The entire network is used for forward pass computations.
    - However, to maintain the expected contribution of neurons, the weights of the remaining neurons are scaled by the dropout rate. This ensures that the overall effect of the network remains consistent with the training phase.

> **Important  
>   
> **It is important to recognize we are **NOT** making the neurons themselves “0” — thats doesn’t really make sense. Rather, we are making some of the weights (which are elements of the layer’s weight tensor) 0. **Note: In Feed-Forward models, the each layer has one weight tensor, and it is 2D, so a matrix always.** When we are making some of the weights 0, this means, in feed-forward models, we are making some elements in the weight matrix = 0.

![[JPEG_image-4B13-9C60-0C-0.jpeg]]

In a neural network, we can think of each weight as a line connecting 2 neurons. This makes sense since. Consider an arbitrary layer with a weight matrix, with dimensions 7x7. Then, we know that since the first dimension is 7, the tensor coming out of the previous layer has shape (a,b,c,…7), where the are 7 attributes for each piece of data. This also means that the previous layer has 7 neurons, and the current layer has 7 neurons. So, every element in the 7x7 weight matrix corresponds to one weight from previous neuron to current neuron.

---

At test time, no units are dropped out; instead, the layer’s output values are scaled down by a factor equal to the dropout rate, to balance for the fact that more units are active than at training time.

We will look at this in raw code:

```Python
layer_output *= np.random.randint(0, high=2, size=layer_output.shape)
```

Above, the first line of code creates a vector of random integers. Then, we make each value 0, with probability 1/2 (as high=1, means we have 2 poss options, 0 or 1).

At test time:

```Python
layer_output *= 0.5
```

Since at training time we are dropping 50% of the units in the output, we need to scale appropriately during test time. So, we scale by `0.5` as we previously dropped half the units.

Note that this process can be implemented by doing both operations at training time and leaving the output unchanged at test time, which is often the way it’s implemented in practice (see figure 5.20):

```Python
layer_output *= np.random.randint(0, high=2, size=layer_output.shape)
layer_output /= 0.5
```

Here is a quick visual demonstration:

![[Untitled 4.png]]

Here is how we could do this in Keras. We will show this for the IMDB example:

```Python
model = keras.Sequential([
    layers.Dense(16, activation="relu"),
    layers.Dropout(0.5),
    layers.Dense(16, activation="relu"),
    layers.Dropout(0.5),
    layers.Dense(1, activation="sigmoid")
])
model.compile(optimizer="rmsprop",
              loss="binary_crossentropy",
              metrics=["accuracy"])
history_dropout = model.fit(
    train_data, train_labels,
    epochs=20, batch_size=512, validation_split=0.4)
```

**We do not need to actually perform the scaling, we Keras is doing this for us.** We need only to fit normally.

![[Untitled 5.png]]

---

### Chapter Summary

To recap, these are the most common ways to maximize generalization and prevent  
overfitting in neural networks:  

- **Get more training data, or better training data**
- **Develop better features**
- **Reduce the capacity of the model**
- **Add weight regularization (for smaller models)**
- **Add dropout**