### 1.1 Artificial Intelligence, Machine Learning, Deep Learning

These 3 terms are thrown around a lot. They represent a hierarchy:

![[Screenshot_2023-04-27_at_11.17.48_AM.png]]

**⇒ Artificial Intelligence**

This is the most broad terminology, and encompasses the idea of the trying to imitate human intellectual capabilities in a computer. Throughout history, this term has been applied to many different methods. In the very beginning, AI was done so by designing and implementing a strict set of instructions for a program to follow (think of things like computer in chess games, etc). However, this sort of approach is highly impractical, and perhaps impossible, to accomplish more complex tasks, where there are not strict pre-defined rules (such as speech recognition, or natural language processing). Machine learning and deep learning are again, methods of implementing artificial intelligence.

  

**⇒ Machine Learning**

Machine learning algorithms are typically based on statistical models and are designed to make predictions or decisions based on patterns found in the data. Machine learning is another subset of AI, and is largely the most successful one so far. The idea of machine learning is as follows:

- Traditional AI has always been, feed in the set of rules and data, and expect the output
- Machine learning changes this, and feeds the machine the input data and the expected output. It is then the job of the machine, to “learn” what set of rules would result in the output from the input. It is trained on that data.
- It’s presented with many examples relevant to a task, and it finds statistical structure in these examples that eventually allows the system to come up with rules for automating the task.

![[Screenshot_2023-04-27_at_11.25.03_AM.png]]

For machine learning, we need 3 things: **(1)** Set of inputs **(2)** Examples of expected output **(3)** Ways to measure whether the algorithm is performing well. The fundamental purpose of machine learning models are to transform the data set into meaningful outputs, a process that is “learned” from exposure to known examples of input and output. In other words, want to learn _useful_ representations of the data.

- What does this mean? Representations are different ways to look at data.
    - A colour image can be encoded via RGB, or something like HSV (hue-saturation-value). Now, if we wanted to find all red pixels, the RGB representation would be simpler to use. On the other hand, if we wanted to make the image less saturated, HSV is better.

Machine learning algorithms are not creative in finding these transformations, they’re merely searching through a predefined set of operations, called a _hypothesis space_.

  

**⇒ Deep Learning**

Deep learning is a specific subfield of machine learning: a new take on learning representations from data that puts an emphasis on learning successive layers of increasingly meaningful representations. The “deep” in “deep learning” isn’t a reference to any kind of deeper understanding achieved by the approach; rather, it stands for this idea of successive layers of representations.

- Number of layers is called the **depth** of the model
- Modern deep learning involves tens or even hundreds of layers

  

These layered representations are learned via models called **neural networks**, structured in literal layers above one another. Deep learning is a mathematical framework for learning representations from data.