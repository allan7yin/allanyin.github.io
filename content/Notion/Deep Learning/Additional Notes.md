## How. Much. Data?

> Generally speaking, the rule of thumb regarding machine learning is that you need **at least ten times as many rows (data points) as there are features (columns) in your dataset**. This means that if your dataset has 10 columns (i.e., features), you should have at least 100 rows for optimal results.

### Stochastic Gradient Descent

This is a type of optimizer we can use in Deep Learning. What does it mean? We understand the fundamental idea of using gradient descent in machine learning (opposite direction to the gradient is the where we want to go to minimize loss function).

Normal gradient descent is known as `Batch Gradient Descent`. So, how does it differ from `Stochastic Gradient Descent`? Here's how:

- Batch Size:
    - SGD: Uses a single or a small batch of training examples to compute the gradients and update the parameters in each iteration.
    - Normal Gradient Descent: Utilizes the entire dataset to compute the gradients and update the parameters in each iteration.
- Gradient Computation:
    - SGD: Computes gradients for each mini-batch or single example and updates the parameters based on these gradients.
    - Normal Gradient Descent: Computes gradients for the entire dataset collectively and updates the parameters based on the average of these gradients.
- Parameter Update:
    - SGD: Updates the parameters after each mini-batch or example, leading to more frequent updates and potential oscillations in the parameter space.
    - Normal Gradient Descent: Updates the parameters once after computing gradients for the entire dataset, resulting in more stable updates and a smoother convergence path.
- Computational Efficiency:
    - SGD: Scales well with large datasets since it uses only a small portion of the data in each iteration, making it computationally efficient.
    - Normal Gradient Descent: Requires computing gradients for the entire dataset in each iteration, making it less efficient, particularly for large datasets.
- Convergence Behavior:
    - SGD: Due to the noise introduced by mini-batches or individual examples, SGD can converge faster, especially in noisy or ill-conditioned optimization landscapes. However, the convergence path may exhibit more oscillations.
    - Normal Gradient Descent: The use of the full dataset provides a more accurate estimate of the gradients, leading to smoother convergence with fewer oscillations. However, it can be slower for large datasets and may get stuck in shallow local optima.

In stochastic gradient descent (SGD), the loss function is typically computed using a single or a small batch of training examples, rather than the entire dataset. This is what distinguishes SGD from batch gradient descent, where the loss function is computed using the entire dataset. This is important as we're calculating the gradient from this loss function.

---

### RMSprop

`RMSprop` (Root Mean Square Propagation) is an optimization algorithm and an extension of the basic stochastic gradient descent (SGD) algorithm that addresses some of its limitations.

The key idea behind RMSprop is to adapt the learning rate for each parameter by considering the historical gradients (as opposed to directly setting it). It achieves this by calculating an exponentially weighted moving average of the squared gradients for each parameter. Before moving on, we should know what an "exponentially weighted moving average" entails:

- An exponentially weighted average is a way to calculate the weighted average of a sequence of values, where more recent values have a higher influence on the average than older values. It assigns exponentially decreasing weights to the values based on their recency.

Here is how we could work with it:

- Initialize the average:
    - Start with an initial value for the average, often set to the first value in the sequence.
- Iterate through the values:
    - For each new value in the sequence, calculate the exponentially weighted average by combining it with the previous average using a weighting factor.
    - The weighting factor, often denoted as decay_rate or alpha, determines the influence of the new value relative to the previous average. It is usually a value between 0 and 1, with values closer to 1 giving more weight to recent values.
- Update the average:
    - Multiply the previous average by (1 - decay_rate) to decrease its weight.
    - Add the new value multiplied by decay_rate to increase its weight.
    - This weighted combination forms the new exponentially weighted average.

---

### One-hot encoding

This is a common way to converting categorical data into tensors. It transforms categorical variables into binary vectors, where each category is represented by a binary value (0 or 1). The name "one-hot" comes from the fact that only one element in each binary vector is "hot" (1), while all others are "cold" (0).