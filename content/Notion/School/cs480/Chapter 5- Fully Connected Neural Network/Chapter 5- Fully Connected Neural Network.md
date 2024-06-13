Consider the XOR problem on some example data:

![[Screenshot_2024-06-11_at_7.00.22_PM.png]]

What do we see? This is no separating hyperplane for this dataset. So, we cannot fit a hyperplane to this data. We can consider 2 solutions:

1. Fix input representation but use a richer model (e.g ellipsoid)
2. Fix hyperplane as classifier but use a richer input representation

  

These 2 are the same, through **a reproducing kernel.** What a neural network does is it learns the feature map simultaneously with the hyperplane (hence you visually get the idea of the entire Hilbert space rotating and shifting). Now, recall that our perceptron model took on the following form:

![[Screenshot_2024-06-12_at_2.22.13_PM.png]]

Where we take the inner product of our data point $x \in \R^d$﻿ and the weight vector $w$﻿, and pass the result to the sign function, which maps this result to one of the output options (predicting a value, or labeling it as + or - for binary classification, etc.) Now, we can add more layers to the perceptron, forming a multi-layer perceptron:

![[Screenshot_2024-06-12_at_2.33.10_PM.png]]

- 1st linear transformation: $z = Ux + c, U \in \R^{2x2}, c \in \R^2$﻿
- Element-wise nonlinear activation function (to help learn non-linear relationships in the data); **makes all the difference!**
- 2nd linear transformation: $\hat y = \langle h, w \rangle + b$﻿

Now, you can probably see the inklings of a neural network. We have weight matrix $U$﻿ and we multiply it by input vector $x$﻿, add the bias, and this is the output of the first layer. Then, we take this output, apply some non-linear activation function, and then feed it to a final output layer, where the linear mapping between this result and prediction, is learned. The above case is when there is only value to be predicted, so in the basic perceptron binary classification we saw, this is $+1$﻿ or $-1$﻿. Of course, we’re not always going to be doing classification tasks, and so, a more general diagram of a standard dense, fully-connected neural network is:

![[Screenshot_2024-06-12_at_2.53.07_PM.png]]

A couple things to note about our neural network:

- We can have an arbitrary number of layers, and continue to increase our depth to create **deep neural networks**. However, keep in mind, the deeper the model, the more powerful it becomes, and the easier it is for the model to overfit on the training data, a critical issue with deep neural networks. As usual, we almost always will end off our NN with some dense linear layer, to map the results of the model with prediction labels.

  

Let’s see an example of iterating through a MLP (Multi-layer Perceptron) model, over test data set (since we haven’t yet seen how to do backtracking)

![[Screenshot_2024-06-12_at_3.37.51_PM.png]]

Now the above is very simple. We have $l$﻿ layers in our model, hence, we iterate from $k = 1$﻿ until $k = l$﻿. So, each layer has weight matrix $W_k$﻿, and then we go through the same motions: (1) Multiply weight by the output $h$﻿ of the previous layer (which in the first iteration, is just the input data), and add on the bias term. Then, we pass this through a non-linear element-wise activation function, and assign this to the output of this layer.

- At the very end of this loop, we take the output of these layer, and run it through one final dense linear layer, before passing it to final activation function **Softmax**, which is commonly used as the last activation function of an NN to normalize the output of a network to a probability distribution over predicted output classes.

  

So, thats clear, here’s just a quick look at what some example activation functions look like:

![[Screenshot_2024-06-12_at_4.04.27_PM.png]]

![[Screenshot_2024-06-12_at_4.04.53_PM.png]]

Now, let’s take a look at what happens during training for NN/MLP, as there is where the _**magic happens!**_

  

**→ Training**

For the NN, we need a loss function. We define this function $l$﻿ and we use it to measure the difference between prediction $\hat p$﻿ and truth $y$﻿. This can be through things like **squared loss** or through something like **log loss.** So, how does the model learn? The core concept is in backpopogation and gradient descent.

  

  

  

  

Let’s see an algorithm for the feed-forward of **multi-layer perceptron** for running the model on test data:

[](https://www.notion.soundefined)