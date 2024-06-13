This document will go over some of the important introductory R commands we will use in this course. In particular, these will be functions related to the various distributions you have already seen.

  

### Importing the Data

```R
mydata <- read.csv("stat231f23dataset12345678.csv")
```

_In_ **_R_** _we use_ `_<-_` _to create objects. Here I am creating the object_ `_mydata_` _to store my dataset_

  

### Inspecting the Data

One of the first things we like to do is to check the dimensions of our dataset:

```R
dim(mydata)
## [1] 894 13
```

So, the above data has 894 rows and 13 columns. To check the column names, we simply do:

```R
colnames(mydata)
## [1] "city" "date" "day.of.week" "time" "time.hour"
## [6] "lat" "lng" "subject.age" "subject.race" "subject.sex"
## [11] "violation" "vehicle.make" "outcome"
```

We can access specific entries using square bracket notation:

```R
mydata[1,3]

## [1] "Tuesday"
```

This is first column and third row. We cam also specify ranges of values:

```R
mydata[1:5, 11:12]
## violation vehicle.make
## 1 DISPLAY ST REG-FRONT/REAR other
## 2 IVC - NO TURN SIGNAL LIGHTS/ 1ST & 2ND toyota
## 3 OBSTRUCTION DRIVER'S VIEW/TINTED WINDOWS chevrolet
## 4 HEADLIGHT TWO REQUIRED-MOTOR VEHICLE other
## 5 STOP AT STOP SIGN other
```

In this case, we are getting the first 5 rows, and columns 11 to 12. We can also extract select columns by their names:

```R
mydata$violation[1:5]
## [1] "DISPLAY ST REG-FRONT/REAR"
## [2] "IVC - NO TURN SIGNAL LIGHTS/ 1ST & 2ND"
## [3] "OBSTRUCTION DRIVER'S VIEW/TINTED WINDOWS"
## [4] "HEADLIGHT TWO REQUIRED-MOTOR VEHICLE"
## [5] "STOP AT STOP SIGN"
```

Again, the `[1:5` selects rows 1 to 5. Something similar:

```R
mydata$vehicle.make[1:5]
## [1] "other" "toyota" "chevrolet" "other" "other"
```

  

### Checking for Missing Values

Very important: upon importing your dataset you should check for missing values (usually indicated by `NA`). The only missing values should occur in the variates `vehicle.make`, and `outcome`, as these are not available for San Francisco, and Chicago, respectively. One useful function is is `.na()`. Consider the following code examples:  
  

```R
mydata$vehicle.make[5]
## [1] "other"

is.na(mydata$vehicle.make[5])
## [1] FALSE

mydata$vehicle.make[500]
## [1] NA

is.na(mydata$vehicle.make[500])
## [1] TRUE
```

  

**⇒ Quick Trick**

We can quickly check whether there are any missing values in the sample by combining `is.na()`. with `sum()`:

```R
sum(is.na(mydata[,1:11]))
## [1] 0
```

As we know, `sum()` returns the sum of a list of numbers, treating the value `FALSE` as `0`, and `TRUE` as `1`.

  

### Probability Calculations

In the following, we will show some general commands, which aren’t specific to the assignment questions.

  

### Gaussian

We can generate a random observation from the `G(0,1)` distribution:

```R
set.seed(1)
rnorm(1,0,1)
## [1] -0.6264538
```

We use `set.seed(1)` so that every time we run a sequence of random number generations, it will be replicable and the values will be the same. The `rnorm()` command generates random numbers from a **Gaussian distribution**. The first number indicates how many observations we want, the second number indicates the **mean** of the distribution, and the third number indicates the **standard deviation** of the distribution.

  

Instead, we could generate 5 observations from a `G(1, 2)` distribution as follows:

```R
set.seed(1)
rnorm(5, 1, 2)
## [1] -0.2529076 1.3672866 -0.6712572 4.1905616 1.6590155
```

In the above, we have always specified the mean and distribution. If we do not include these, **R** will assume we are operating on a `G(0,1)` distribution:

```R
set.seed(1)
rnorm(5)
## [1] -0.6264538 0.1836433 -0.8356286 1.5952808 0.3295078
```

  

We can also conduct probability calculations using the **Gaussian distribution**, we can use `pnorm()`:

```R
pnorm(5,1,2)
## [1] 0.9772499
```

This gives us `P(Y ≤ 5)` where `Y ∼ G(1, 2)`. What about probabilities bounded from above and below? We can use some subtraction:

```R
pnorm(3, 0, 5) - pnorm(2, 0, 5)
## [1] 0.07032514
```

This gives `P(2 ≤ Y ≤ 3)` where `Y ∼ G(0, 5)`.

  

We can generate quantiles using `qnorm()`:

```R
qnorm(0.975, 1, 2)
## [1] 4.919928
```

This returns the value a such that `P(Y ≤ a) = 0.975` where `Y ∼ G(1, 2)`.

  

Finally, we can use the `dnorm()` function to evaluate the value of the probability density function of the Gaussian distribution for specific values:

```R
dnorm(5, 1, 2)

## [1] 0.02699548
```

  

These functions are also available for the other types of distributions. No need to go over them here, but they all follow the same similar syntax structure `(r,o,q,d)-norm`. Reference tjose when needed:

![[RTutorial0.pdf]]