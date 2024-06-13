Consider the XOR problem on some example data:

![[Screenshot_2024-06-11_at_7.00.22_PM.png | 500]]

What do we see? This is no separating hyperplane for this dataset. So, we cannot fit a hyperplane to this data. We can consider 2 solutions:

1. Fix input representation but use a richer model (e.g ellipsoid)
2. Fix hyperplane as classifier but use a richer input representation

These 2 are the same, through **a reproducing kernel.** What a neural network does is it learns the feature map simultaneously with the hyperplane (hence you visually get the idea of the entire Hilbert space rotating and shifting). Now, recall that our perceptron model took on the following form:

![[Screenshot_2024-06-12_at_2.22.13_PM.png | 500]]

Where we take the inner product of our data point $x \in \R^d$﻿ and the weight vector $w$﻿, and pass the result to the sign function, which maps this result to one of the output options (predicting a value, or labeling it as + or - for binary classification, etc.) Now, we can add more layers to the perceptron, forming a multi-layer perceptron:

![[Screenshot_2024-06-12_at_2.33.10_PM.png | 500]]

- 1st linear transformation: $z = Ux + c, U \in \R^{2x2}, c \in \R^2$﻿
- Element-wise nonlinear activation function (to help learn non-linear relationships in the data); **makes all the difference!**
- 2nd linear transformation: $\hat y = \langle h, w \rangle + b$﻿

Now, you can probably see the inklings of a neural network. We have weight matrix $U$﻿ and we multiply it by input vector $x$﻿, add the bias, and this is the output of the first layer. Then, we take this output, apply some non-linear activation function, and then feed it to a final output layer, where the linear mapping between this result and prediction, is learned. The above case is when there is only value to be predicted, so in the basic perceptron binary classification we saw, this is $+1$﻿ or $-1$﻿. Of course, we’re not always going to be doing classification tasks, and so, a more general diagram of a standard dense, fully-connected neural network:

![[Screenshot_2024-06-12_at_2.53.07_PM.png | 500]]

==A couple things to note about our neural network:==

- We can have an arbitrary number of layers, and continue to increase our depth to create **deep neural networks**. However, keep in mind, the deeper the model, the more powerful it becomes, and the easier it is for the model to overfit on the training data, a critical issue with deep neural networks. As usual, we almost always will end off our NN with some dense linear layer, to map the results of the model with prediction labels.

Let’s see an example of iterating through a MLP (Multi-layer Perceptron) model, over test data set (since we haven’t yet seen how to do backtracking)
![[Screenshot_2024-06-12_at_3.37.51_PM.png | 600]]

Now the above is very simple. We have $l$﻿ layers in our model, hence, we iterate from $k = 1$﻿ until $k = l$﻿. So, each layer has weight matrix $W_k$﻿, and then we go through the same motions: (1) Multiply weight by the output $h$﻿ of the previous layer (which in the first iteration, is just the input data), and add on the bias term. Then, we pass this through a non-linear element-wise activation function, and assign this to the output of this layer.

- At the very end of this loop, we take the output of these layer, and run it through one final dense linear layer, before passing it to final activation function **Softmax**, which is commonly used as the last activation function of an NN to normalize the output of a network to a probability distribution over predicted output classes.

So, thats clear, here’s just a quick look at what some example activation functions look like:
![[Screenshot_2024-06-12_at_4.04.27_PM.png | 500]]
![[Screenshot_2024-06-12_at_4.04.53_PM.png | 500]]
Now, let’s take a look at what happens during training for NN/MLP, as there is where the _**magic happens!**_

**-> Training**
For the NN, we need a loss function. We define this function $l$﻿ and we use it to measure the difference between prediction $\hat p$﻿ and truth $y$﻿. This can be through things like **squared loss** or through something like **log loss.** So, how does the model learn? The core concept is in backpopogation and gradient descent. 
$$
\min_{w} \frac{1}{n} \sum_{i=1}^{n} [\ell \circ f](x_i, y_i; w) 
$$
$$
w \leftarrow w - \eta \cdot \frac{1}{n} \sum_{i=1}^{n} \nabla [\ell \circ f](x_i, y_i; w) 
$$
This is the loss function we are interested in. In the training phase of a Multi-Layer Perceptron (MLP) or any neural network, the goal is to find the set of weights that minimizes the average loss over all training data, hence, we are looking for mine $w$ that is average over ALL data points, where we are taking loss function $l$ of the model function $f$. 

Then, to learn, we need to minimize average loss, optimization algorithms such as Gradient Descent are used. The basic idea is to iteratively adjust the weights www in the direction that reduces the loss. This is done by computing the gradient of the loss function with respect to the weights and updating the weights as follows:

$$
w_{t+1} \leftarrow w_t - \eta \nabla g(w_t)
$$

There are 2 things we should mention here: (1) Learning rate and (2) Batch size.
### Learning Rate
where $\eta$ is the learning rate, which controls the size of the weight updates, the step size if you will, and $\nabla_w L$ is the gradient of the loss with respect to the weights. This step size is very important. Why?
- Consider if the learning rate is **too high**, then, we run the risk of missing local/global minima of the loss function (imagine this graphed in higher-dimension)
- If the learning rate is **too low**, then, we will take too long to find minima, or become stuck

### Batch Size
Each iteration will required a full pass over **the entire training set**! So, to adjust the weights **ONCE**, we need to iterate over the **ENTIRE** dataset. This, while theoretically would provide the best learning, is incredibly expensive. A middle-ground for this is train in batches, so we run the batch of data, take average loss, and update the weights from that. We call this **mini-batch** , where we take some random set of data $B \subseteq  \{1,...,n \}$ suffices:
$$
w \leftarrow w - \eta \cdot \frac{1}{|B|} \sum_{i \in B} \nabla[\ell \circ f](x_i, y_i; w)
$$
Moving forward, for the sake of simplicity, we'll define

$$
g(w_t) = \frac{1}{|B|} \sum_{i \in B} \nabla[\ell \circ f](x_i, y_i; w)
$$
The average gradient over a mini-batch. 
### Momentum 
**-> Gradient Descent Basics**
Momentum is an optimization technique used to accelerate the convergence of gradient descent by adding a fraction of the previous update to the current update. It is designed to help gradient descent algorithms navigate the optimization landscape more effectively, especially in scenarios where the gradients oscillate or progress slowly.

**-> Standard Momentum**
The standard one: $w_{t+1} \leftarrow w_t - \eta \nabla g(w_t)$. Now, how would this change with momentum. Consider:

$$
\begin{align*} w_{t+1} &= \underbrace{w_t - \eta_t \nabla g(w_t)}_{\text{gradient step}} + \underbrace{\beta_t (w_t - w_{t-1})}_{\text{momentum}} = \underbrace{(1 + \beta_t) w_t - \beta_t w_{t-1}}_{\text{extrapolation}} - \eta_t \nabla g(w_t) \end{align*}
$$
Now, we look for a better way to represent momentum, we define variable $\tilde v$ represent a form of momentum or velocity in the optimization process. In the above, $\beta_t$ to be the momentum coefficient for the momentum term. Notice that momentum incorporates the previous weight as well. We define intermediate $\tilde v$:
$$
\tilde v_{t+1} = \beta \tilde v_t + \eta_t\nabla g(w_t), \text{where we get } \tilde v_{t+1} = w_t - w_{t+1}
$$

We say *intermediate velocity* because it represents a temporary calculation that combines the effects of the current gradient and momentum before making the final update to the weights. Notice that this equation incorporates the previous velocity. Then, we can interpret the momentum as gradient averaging:
$$
\begin{align*} v_{t+1} &= \frac{\alpha_{t-1}}{\alpha_t} \beta_t v_t + \frac{\eta_t}{\alpha_t} \nabla g(w_t), \text{where } \alpha_t v_{t+1} = w_t - w_{t+1} \\ &= \underbrace{(1 - \gamma_t) v_t + \gamma_t \nabla g(w_t)}_{\text{gradient averaging}},\underbrace{ w_{t+1} = w_t - \alpha_t v_{t+1}}_{\text{gradient update}}, \\ \text{where } \alpha_t &= \alpha_{t-1} \beta_t + \eta_t \text{ and } \gamma_t := \frac{\eta_t}{\alpha_t} \in [0, 1]. \end{align*}
$$
Now, a couple things to note about this:
- This is essentially a weighted average, note how we are multiplying first term by $\frac{\alpha_{t-1}}{\alpha_t}$ and the second term by $\frac{1}{\alpha_t}$, and then adding together (comparing this to intermediate velocity).
- Then, after grouping like terms, the gradient averaging becomes clear:
	- Now that we have $v_{t+1} = (1 - \gamma_t) v_t + \gamma_t \nabla g(w_t)$, then we can rearrange to obtain: $\gamma_t \nabla g(w_t) = v_{t+1} - (1 - \gamma_t) v_t$. So, substituting this back into our gradient step, we can see how the new gradient update was derived: $w_{t+1} = w_t - \alpha_t v_{t+1}$.

So, what does this mean? We interpret this as the velocity is a weighted average of old velocity + some multiple of the gradient, where we usually choose $\gamma \lt 1$. For example, say we make $\gamma = 0.9$, this would mean we define the current velocity to be 90% of old velocity + 10% of the new gradient. 

![[Screenshot 2024-06-12 at 10.30.17 PM.png | 300]]

Look at this diagram, this illustrates why we need momentum. Without momentum, gradient updates would take the black line. With momentum, give more weight to the trajectory of the direction. 

**-> Nesterov Momentum**
Another interesting concept is Nesterov momentum, which is sometimes also called Nesterov Accelerated Gradient. This is very similar to the first velocity we derived above:
$$
z_{t+1} = w_{t+1} + \beta_t(w_{t+1} - w_t)
$$
$$
\begin{align*} v_{t+1} &= \frac{\alpha_{t-1}}{\alpha_t} \beta_t v_t + \frac{\eta_t}{\alpha_t} \nabla g(z_t), \text{where } \alpha_t v_{t+1} = w_t - w_{t+1} \\ &= \underbrace{(1 - \gamma_t) v_t + \gamma_t \nabla g(z_t)}_{\text{gradient averaging}},\underbrace{ w_{t+1} = w_t - \alpha_t v_{t+1}}_{\text{gradient update}}, \\ \text{where } \alpha_t &= \alpha_{t-1} \beta_t + \eta_t,
z_t = w_t - \beta_{t-1} \alpha_{t-1} v_t = w_t - \alpha_t (1 - \gamma_t) v_t,\text{ and } \gamma_t := \frac{\eta_t}{\alpha_t} \in [0, 1]. \end{align*}
$$
So, structurally, this is very similar to the above derivation for velocity. The more important one is the first momentum definition. 


**-> Optimizers**
What are optimizers? SGD is one of them, and these are the methods in which the model "learns", or updates its weights. They define specific strategies for how the model's parameters (weights) should be adjusted during training to minimize the loss function. Each optimizer implements a different algorithmic approach to weight updates, aiming to improve convergence speed and overall model performance. Here a couple of other commonly used ones, with each being favoured for different tasks like classification or regression. Let's briefly look at 3 notable ones: **Adagrad, RMSprop, and Adam**:

First off, all 3 follow very similar skeleton:
$$
w_{t+1} = w_{t} - \frac{\eta_t}{\sqrt{s_t + \epsilon}} \odot v_t
$$
**-> Adagrad** (Adaptive Gradient Algorithm):
$$
\begin{align*}
s_t &= s_{t-1} + \nabla g(\mathbf{w}_t) \odot \nabla g(\mathbf{w}_t) \\ v_t &= \nabla g(\mathbf{w}_t)
\end{align*}
$$
