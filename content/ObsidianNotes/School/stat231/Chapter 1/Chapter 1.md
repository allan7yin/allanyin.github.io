## Introduction to Statistical Science

Statistical Science is concerned with all aspects of empirical studies.,

- **Empirical Study:** one in which we learn by observation or experimentation.

A key feature of an empirical study is that it involves **uncertainty**. Let’s look at some important terminology:

- **unit:** an individual person, place, or thing about which we can take some measurement(s)
- **population:** a collection of units, e.g.
    - All the donuts sold in the Math CnD yesterday
    - All current UW undergraduate students
    - All persons currently aged 90 or over living in Ontario

  

**As with all statistical terminology, it is essential to be precise!**

  

- **process:** a collection of units, but those units are ‘produced’ over time.

  

Both **populations** and **processes** are collections of units. A key feature of processes is that they usually occur over time whereas populations are static (defined at one moment in time), like an instance of a process. e.g.

- **Population**: All current UW undergraduate students.
- **Process**: All UW undergraduate students for the **next ten years**.

  

To see these more in action, consider the following example:

> _In Winter 2021 STAT 231 had seven assignments, of which the best five would count towards the final grade. At the end of term I downloaded data on how many assignments each student completed, as well as the time they submitted Assignment 6 according to_ _**Crowdmark**__. I was interested in learning about how students in STAT 231 approach assignments to help understand student behaviour in STAT 231 in Winter 2021 and in future terms._

  

**⇒ Question: What is the population or process? What are the units?**

We are interested in learning about the process of STAT 231 students in Winter 2021 and in future terms. So:

- **units we are interested in:** STAT 231 students (in Winter 2021 and future terms)
- The **units** in the study are STAT 231 students in Winter 2021.
- **Important**: be very careful when describing the units of a study.

  

We now introduce the concept of variates:

- **variate:** characteristics of the units which are usually represented by letters such as x, y, z. Variates come in many different forms can be categorized:
    - Continuous
    - Discrete
    - Categorical
    - Ordinal
    - Complex

  

For now, we want to emphasize **continuous** and **discrete** variates.

- **continuous: v**ariates are those that can be measured - at least in theory - to an infinite degree of accuracy. For example:
    - height
    - weight
    - lifetime of an electrical component
    - the time a patient takes to recover from a disease after receiving treatment
    - the volume of tea I drink in a day.
- **discrete:** those that can only take a finite or countably infinite number of values. For example:
    - the number of car accidents on a certain stretch of highway in a year
    - how many cups of tea I drink in a day.
- **categorical:** variates are those where units fall into a (non-numeric) category, such as hair colour or university program
- **ordinal: v**ariates are those where an ordering is implied, but not necessarily through a numeric measure. Examples include ‘strongly disagree, disagree, neutral, agree, strongly agree’ in surveys.
- **complex:** variates are more unusual, and include open-ended responses to a survey question, or an image. Analyzing complex variates often requires processing to ‘convert’ them into one of the other types (e.g., using text analysis to decide if a Tweet is positive, negative, or neutral).

  

Here is an interesting example to consider:

![[Screenshot_2023-09-24_at_6.04.35_PM.png]]

  

Here, we know that the **size** is a **ordinal** variate, and the **volume** is a **continuous** variate. Here, the **continuous** variate has been converted into a **ordinal** variate. Now, notice that each level of change in ordinal variates is not constant.

  

When you are making decisions about the type of a variate, carefully think about what it is we are trying to measure. Variates like time, or distance, can theoretically be measured to an infinite degree of accuracy, and so are usually treated as continuous. In contrast, the number of assignments a student completes can only take on one of eight values.

- If we decide a variate is discrete, we will usually use a discrete probability distribution (e.g., Poisson) to model it.
- If we decide a variate is continuous, we will usually use a continuous probability distribution (e.g., Gaussian) to model it.

  

Consider carefully the type of a variate. This is often not a very straight-forward process, and there are certainly exceptions given the specific problem. For example:

- **suppose** a survey asks first year students for their age. Age is typically measured in years, rather some continuous measure of time. In that case, if the only responses were 18 or 19, would it still be appropriate to use a **continuous probability distribution** to describe this variate?

  

**⇒ Again, we now introduce some new terminology**

- **attribute:** of a population or process is **a function of a variate** which is defined **for all units** in the population or process.

  

There are 3 types of **empirical** study:

1. Sample surveys
2. Observational Studies
3. Experimental Studies

  

Let’s look at each one:

**⇒ Sample Surveys**

Where information is obtained about a finite population by selecting a ‘representative’ sample of units from the population and determining the variates of interest for each unit in the sample.

- A poll to predict who will win an election is a common example of a survey sample. A polling company will select a number of people at random and ask them questions about their voting preferences.

  

**⇒ Observational and Experimental**

- An observational study is where information about a population or process is collected without any attempt to change one or more variates for the sampled units.
    - A random sample of players is selected on 1 September 2023, and all in-game activity is logged for one week.
- In contrast to observational studies, an experimental study is one in which the experimenter intervenes and changes or sets the values of one or more variates for the units in the study.
    - A random sample of players is selected on 1 September 2023. Half the players are offered a BrixBux promotion, and all in-game activity is logged for one week.

  

The next important section wte will look at is data summaries. We are looking for ways to generate useful representations of our data. We will consider both _numerical_ and _graphical_ summaries.

  

Types of numerical measures:  
• Measures of location (sample mean, median, and mode)  
• Measures of variability or dispersion (sample variance, sample standard deviation, range, and interquartile range (IQR))  
• Measures of shape (sample skewness and sample kurtosis)  

  

**⇒ Quick Recap: Expected Value**

**Expected Value for Discrete Random Variable** with range A and probability function f(x)

$E[X] = \sum_{x∈A} xf(x)$

**Expected Value for Continuous Random Variable** with range (-∞, ∞) and probability density function g(y)

$E[Y] = \int\limits_{-\infty}^{\infty} yg(y) dy$

  

**⇒ Measures of Central Tendency**

Consider some data like $y_1, y_2, y_3, y_4, ..., y_n$﻿

- **Sample Mean:** **$\overline{y} = \frac{y_1 + y_2 + y_3 + ... y_n}{n} = \frac{1}{n} \sum_{i=1}^{n} y_i$**﻿
    - Note that we often like to place a horizontal bar on top of values that are “sample” something
- **Sample Median:** **$P(Y\le M) = \int\limits_{-\infty}^{\infty} g(y)dy = 0.5$**﻿
- This is for continuous random variable
- **Sample Median:** **$P(X \le M) = P(X \ge M) = 0.5$**﻿
    - This is for discrete random variable

  

Now, consider some data like $y_{(1)}, y_{(2)}, y_{(3)}, y_{(4)}, ..., y_{(n)}$﻿, is in order/sorted (**note we like to add () around the term number to indicate this is ordered**. For an odd number of observations:

- **sample median** = $\hat{m} = y_{\frac{n+1}{2}}$﻿, which is informally just the middle value. Of course, for an even number of observations, we just do: $\hat{m} = \frac{1}{2}[y_{\frac{n}{2}} + y_{\frac{n}{2} + 1}]$﻿, so averager of the 2 middle terms
- The **sample mode** is the most common value in a set of data.

  

### Measures of Variability and Dispersion

**Sample Variance** is defined as:

$s^2 = \frac{1}{n-1} \sum_{i=1}^{n}(y_i - \overline{y})^2 = \frac{1}{n-1}[\sum_{i=1}^{n} (y_i)^2 - n\overline{y}^2]$

and **sample standard deviation** is just **s**, the square root of the variance. The following are things we expect from a **Gaussian Distribution:**

- Approximately 68% of the sample should lie in the interval $[\overline{y} - s, \overline{y} + s]$﻿
- Approximately 95% of the sample should lie in the interval $[\overline{y} - 2s, \overline{y} + 2s]$﻿

  

**⇒ Range** is defined as: $max(y_1, ..., y_n) - min(y_1, ..., y_n) = y_{(n)} - y_{(1)}$﻿

- The range is a very crude measure of the spread of the data. It’s also very susceptible to outliers!
- For a metric like this, we like to say it is not **robust**

  

**⇒ Quantiles** are an important concept for summarizing data. The **pth** quantile (also called the 100pth percentile) is the value such that a fraction p of the data fall at or below this value.

![[Screenshot_2023-09-25_at_11.08.54_AM.png]]

> You will not be asked to compute quantiles by hand, other than the 50th percentiles (the median) (thank god)

  

The 25th, 50th and 75th percentiles are known as quartiles. It is common for data to be divided into quartiles, as they divide the data into quarters.

- q(0.25), or the 25th percentile is known as the lower or first quartile.
- q(0.75), or the 75th percentile is known as the upper or third quartile.

  

**IQR (Interquartile range)** is defined as: $IQR = q(0.75) - q(0.25)$﻿. IQR is often used as a measure of spead instead of the crude range, as it is more **robust** and less affected by extreme outliers.

![[Screenshot_2023-09-25_at_11.12.13_AM.png]]

We need to be careful when interpreting IQR. Consider the following 2 charts:

![[Screenshot_2023-09-25_at_11.12.52_AM.png]]

![[Screenshot_2023-09-25_at_11.13.46_AM.png]]

Notice that while their upper and lower quartile is the same, the way the range is defined is different. One is inclusive to both boundary values while the other is inclusive to only the left, and exclusive to the right.

  

**⇒ We finish off numerical summaries with measures of shape and consider some graphical summaries**

Skewness and kurtosis are ways of quantifying the shape of a graph of a dataset. In this course we most often look at the values of sample skewness and sample kurtosis for a given data set to see if the values are close to the values for Gaussian (Normal) data. For sake of time, will simply post screenshots for their formulas.

![[Screenshot_2023-09-25_at_11.24.49_AM.png]]

Now, what can we infer about skewness from a graph? Consider:

![[Screenshot_2023-09-25_at_11.26.16_AM.png]]

![[Screenshot_2023-09-25_at_11.26.22_AM.png]]

![[Screenshot_2023-09-25_at_11.26.35_AM.png]]

![[Screenshot_2023-09-25_at_11.26.57_AM.png]]

Kurtosis measures the degree to which data in a distribution are concentrated in the tails and the overall "tailedness" of the distribution. Consider the following:

![[Screenshot_2023-09-25_at_11.31.23_AM.png]]

![[Screenshot_2023-09-25_at_11.31.52_AM.png]]

![[Screenshot_2023-09-25_at_11.32.14_AM.png]]