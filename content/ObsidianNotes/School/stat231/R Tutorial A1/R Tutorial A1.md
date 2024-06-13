This tutorial will cover some of the **R** code needed to conduct the analyses in Assignment 1.

  

This tutorial will analyze a dataset consisting of tweets downloaded from Twitter (now known as X1). The dataset consists of tweets (posts) sent by five different users on the website, with each row corresponding to one tweet. There are numerous variates containing information about the tweets, which will be described as we introduce them in the tutorial. More information is provided in the file Twitter Variates, found within the R Tutorial 1 folder on LEARN.

  

### 1. Importing the Data

As mentioned in the first **R Tutorial**, we can use:

```R
tweets <- read.csv("tweets.csv")
```

  

This is a fairly large dataset, containing 28 columns. This can make it difficult to quickly visualize it. Instead, we will focus on the variates we will analyze in this tutorial, namely:

- `day.of.week`: The day of week the tweet was published.
- `time.of.day`: The time of day a tweet was published, expressed in seconds after midnight.
- `media.binary`: A binary variate which takes the value 1 if the tweet contains at least one media item, and 0 otherwise.
- `urls.binary`: A binary variate which takes the value 1 if the tweet contains at least one web link (url), and 0 otherwise.
- `likes`: The number of likes a tweet received.
- `retweets`: The number of retweets a tweet received.

  

For example, say I wanted to find which days the first 5 tweets were published:

```R
tweets$day.of.week[1:5]
## [1] "Friday" "Sunday" "Wednesday" "Tuesday" "Friday"
```

  

### 2. Frequency Tables

First, let’s look at the distribution of the day of the week tweets were published. We’ll use the `table()` command:

```R
table(tweets$day.of.week)
##
## Friday Monday Saturday Sunday Thursday Tuesday Wednesday
## 168     123     101      80      129    134      163
```

Something small to note is that **R** has by default, ordered the columns in alphabetical order, which may not always be something we want. To allow more flexible orders, we can define and pass an ordering to the R function:

```R
tweets$day.of.week <- factor(tweets$day.of.week, levels = c("Monday", "Tuesday",
"Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
table(tweets$day.of.week)

##
## Monday Tuesday Wednesday Thursday Friday Saturday Sunday
## 123 134 163 129 168 101 80
```

  

### 3. Bar Plots

Next, let’s make a bar plot of our data. We use the barplot() command:

```R
barplot(table(tweets$day.of.week))
```

![[Screenshot_2023-09-18_at_9.30.35_PM.png]]

Now, quite obviously, this bar plot doesn't look the best. We can add additional options to `barplot` to make the graph look better:

```R
barplot(
  table(tweets$day.of.week),             # Data to create the bar plot
  xlab = "Day of Week",                  # Label for the x-axis
  ylab = "Frequency",                    # Label for the y-axis
  las = 1,                               # Label axis style (0 for horizontal, 1 for vertical)
  main = "Bar plot of day.of.week",      # Title of the plot
  col = c("red", "orange", "yellow",     # Colors for the bars
          "forestgreen", "dodgerblue3", "darkorchid2", "darkorchid4"),
  ylim = c(0, 200),                      # Y-axis limits (0 to 200)
  density = 25,                          # Bar density (spacing between bars)
  angle = c(30, 60, 90, 120, 150, 180, 210), # Bar angle (rotation)
  cex.names = 0.75                       # Character expansion factor for axis labels
)
```

![[Screenshot_2023-09-18_at_9.36.11_PM.png]]

Now, our bar graph looks a lot better. But, thats an extra thing and not particularly useful or necessary. The `density` and `angle` correspond to the coloured lines inside of the actual bar.

  

### 4. Summarizing Across a Binary Variate

Now, suppose we want to compare the distribution of tweet publication day with the use of media items. We can use the `table()` command on two variates as follows:

```R
table(tweets$day.of.week, tweets$media.binary)
##
## 0 1
## Monday 46 77
## Tuesday 56 78
## Wednesday 80 83
## Thursday 53 76
## Friday 73 95
## Saturday 49 52
## Sunday 39 41
```

Again, for this, we can construct a bar graph:  
  

```R
barplot(table(tweets$media.binary, tweets$day.of.week), beside = T, xlab = "Day of Week",
ylab = "Frequency", las = 1, main = "Bar plot of day.of.week and media.binary",
col = c("forestgreen", "dodgerblue3"), ylim = c(0, 100), density = 50, angle = c(45,
135))

legend("topright", legend = c("No media", "Media"), fill = c("forestgreen", "dodgerblue3"),
density = 50, angle = c(45, 135))
```

![[Screenshot_2023-09-18_at_9.41.30_PM.png]]

  

### 5. Calculating with Tables

Our summary table and barplot allows us to get a sense of how many tweets with and without media are published on each day. A more sophisticated analysis might be to calculate proportions or percentages. For example, what proportion of tweets published on a Monday contained media? What percentage of tweets containing no media were published on Wednesdays?

  

An easy way to calculate these is with the `prop.table()` function:

```R
mytable <- table(tweets$day.of.week, tweets$media.binary)
prop.table(mytable, 1)
##
## 0 1
## Monday 0.3739837 0.6260163
## Tuesday 0.4179104 0.5820896
## Wednesday 0.4907975 0.5092025
## Thursday 0.4108527 0.5891473
## Friday 0.4345238 0.5654762
## Saturday 0.4851485 0.5148515
## Sunday 0.4875000 0.5125000
```

So, how does the above work? Line `mytable <- table(tweets$day.of.week, tweets$media.binary)` creates a new table, where it is of the form in the above printed text. For each day of the week row, the columns `0` and `1` represents the `medya.binary` variable (as it has value 0 or 1). Each row will show how many of each there are per day. Passing it to `prop.table(mytable,1)` just converts the table to proportions (probabilities) within each row. Specifically, it calculates the proportions of each unique combination of values in the table relative to the total count within each row.

```R
prop.table(mytable, 2)
##
## 0 1
## Monday 0.11616162 0.15338645
## Tuesday 0.14141414 0.15537849
## Wednesday 0.20202020 0.16533865
## Thursday 0.13383838 0.15139442
## Friday 0.18434343 0.18924303
## Saturday 0.12373737 0.10358566
## Sunday 0.09848485 0.08167331
```

Passing `2` instead just calculates the proportions in terms of the column.

  

### 6. Numerical Summaries for Continuous Variates

The previous commands concerned categorical variates. Let’s explore a continuous variate: `time.of.day`. For starters, there are numerous built in commands for numerical summaries in **R.** Consider the following:

```R
mean(tweets$time.of.day)
## [1] 46214.19

sd(tweets$time.of.day)
## [1] 17540.82
```

But, the [`time.of.day`](http://time.of.day) variate is measured in seconds after midnight, which isn't the best for understanding or analysis. Sometimes, it might make sense for us to create a new variate from an existing one, so that we convert this data into **hours after midnight:**

```R
tweets$time.of.day.hour <- tweets$time.of.day/3600 \#60*60 = 3600 seconds in an hour
mean(tweets$time.of.day.hour)
## [1] 12.83727

sd(tweets$time.of.day.hour)
## [1] 4.872449
```

We can see that the sample mean tweet publication time is 12.8 hours after midnight, or about 12:50pm.

  

Let’s save these, rounding to one decimal point, in case we will need them later one:

```R
time.mean <- mean(tweets$time.of.day.hour)
round(time.mean, 1)
## [1] 12.8

time.sd <- sd(tweets$time.of.day.hour)
round(time.sd, 1)
## [1] 4.9

summary(tweets$time.of.day.hour)
## Min. 1st Qu. Median Mean 3rd Qu. Max.
## 0.2311 9.2162 12.7607 12.8373 16.0792 23.8278
```

`summary` is a nice command to use to see the shape of the data. It gives you the 5 points of data:

- Min
- 1st Quartile
- Median + Mean
- 3rd Quartile
- Max

  

Sometimes we may need to construct our own functions. For example, for skewness and kurtosis we can define new functions to calculate these, and then use them just like how we used `mean()` and `sd()`:

```R
skewness <- function(x) {
(sum((x - mean(x))ˆ3)/length(x))/(sum((x - mean(x))ˆ2)/length(x))ˆ(3/2)
}

kurtosis <- function(x) {
(sum((x - mean(x))ˆ4)/length(x))/(sum((x - mean(x))ˆ2)/length(x))ˆ2
}

skewness(tweets$time.of.day.hour)
## [1] -0.05922016

kurtosis(tweets$time.of.day.hour)
## [1] 2.742395
```

**⇒ Note:** length() returns the number of values in an R object.

  

### 7. Relative Frequency Histogram

We’ll now generate a relative frequency histogram of tweet publication time using `truehist()`. This function is not included by default, so we tell R we want to load the ‘MASS’ package, which contains `truehist()`.

```R
library(MASS)
truehist(tweets$time.of.day.hour, xlab = "Publication time (hours after midnight)",
ylab = "Relative Frequency", main = "Relative frequency histogram of publication time",
xlim = c(0, 25), ylim = c(0, 0.1), las = 1, col = "dodgerblue3", density = 25,
angle = 45)
```

![[Screenshot_2023-09-18_at_10.35.22_PM.png]]

  

**⇒ Adding a Gaussian PDF**

We now want to superimpose a theoretical pdf of the Gaussian distribution with mean and standard deviation equal to the sample mean and sample standard deviation for our dataset. We do this with the following commands:

```R
truehist(tweets$time.of.day.hour, xlab = "Publication time (hours after midnight)",
ylab = "Relative Frequency", main = "Relative frequency histogram of publication time",
xlim = c(0, 25), ylim = c(0, 0.1), las = 1, col = "dodgerblue3", density = 25,
angle = 45)
curve(dnorm(x, time.mean, time.sd), col = "red", add = TRUE, lwd = 1.5)
```

![[Screenshot_2023-09-18_at_10.37.10_PM.png]]

**⇒ Choosing the Number of Bins**

When using `truehist()` we may sometimes want to use a different number of bins (that is, rectangles in our histogram). We can control this with the `nbins` option:

```R
truehist(tweets$time.of.day.hour, xlab = "Publication time (hours after midnight)",
ylab = "Relative Frequency", main = "Relative frequency histogram of publication time",
xlim = c(0, 25), ylim = c(0, 0.2), las = 1, col = "dodgerblue3", density = 25,
angle = 45, nbins = 100)
curve(dnorm(x, time.mean, time.sd), col = "red", add = TRUE, lwd = 1.5)
7
```

![[Screenshot_2023-09-18_at_10.44.06_PM.png]]

Here we have requested 100 bins. Note however, that this is not an appropriate choice for this graphical summary.

  

**⇒ PROTIP:** If we choose too many bins then the shape of our histogram will be largely influenced by individual values in the dataset (rather than showing an overall pattern or shape). This can make it difficult to use for understanding the distribution of our data.

  

Let’s try 5 bins and see what the data looks like:

```R
truehist(tweets$time.of.day.hour, xlab = "Publication time (hours after midnight)",
ylab = "Relative Frequency", main = "Relative frequency histogram of publication time",
xlim = c(0, 25), ylim = c(0, 0.075), las = 1, col = "dodgerblue3",
density = 25, angle = 45, nbins = 5)
curve(dnorm(x, time.mean, time.sd), col = "red", add = TRUE, lwd = 1.5)
```

![[Screenshot_2023-09-18_at_11.20.27_PM.png]]

While having too many bins can be a problem, so can having too few bins. Here, we have lost most of the information about the overall shape of the distribution of our sample. This plot is thus of limited utility in assessing the fit of a Gaussian model. The default number of bins, as seen in our first plots, works well.

> **PROTIP:** _Sometimes R’s default choice for the number of bins will be suitable, but not always. Some trial and error may be necessary to find an appropriate number of bins for this graphical summary. Note that there is rarely a single ‘best’ number of bins to use. In your assignment, you should choose a number of bins that allows you to use your plot effectively as a tool for assessing the fit of a Gaussian model. As long as your plot can be used for this purpose, then you can be confident you have chosen an appropriate number of bins._

  

### 8. Empirical Cumulative Distribution Function Plot

Another useful graphical summary is the empirical cumulative distribution function (e.c.d.f.). R has a helpfully named `ecdf()` function we can use for this. Let’s plot the e.c.d.f. and add the Gaussian cumulative distribution function at the same time.

```R
plot(ecdf(tweets$time.of.day.hour), xlab = "Publication time (hours after midnight)",
main = "e.c.d.f. of publication time", las = 1, lwd = 2, pch = NA)
curve(pnorm(x, time.mean, time.sd), col = "red", add = TRUE, lwd = 1.5, lty = 2)
```

![[Screenshot_2023-09-19_at_10.32.15_AM.png]]

Some quick notes about the syntax of the above:

- The `lty` option specifies the type of line that is drawn (it’s short for ‘line type’). Setting this to 2 specifies a dashed line. Try other numbers and see what happens!
- The `pch` option is used to specify the symbols used to plot points (it’s short for ‘plot character’). Usually we specify a number for this to choose a particular symbol, but here I have set it to ‘NA’ which tells R not to plot symbols at all. Try changing `pch = NA` to `pch = 1` and see what happens!

  

**⇒ Quantiles**

We can use an e.c.d.f. to estimate quantiles of our sample, but we can also use **R** to compute these directly using the `quantile()` function:

```R
quantile(tweets$time.of.day.hour, prob = 0.5)
## 50%
## 12.76069
```

Which tells us the 0.5th quantile (that is, the sample median) is 60.8.

  

### 9. Scatterplots

Now, we want to look at how we can create scatter plots in **R**. We’ll finish off by comparing two other variates: likes and retweets. Likes and retweets on Twitter may be thought of (though are not always intended) as ‘endorsements’, with a retweet often being seen as a stronger endorsement than a like. We may therefore expect these variates to be correlated. We can summarize these variates with a scatterplot using the `plot()` command:

```R
plot(tweets$likes, tweets$retweets, xlab = "Likes", ylab = "Retweets",
main = "Likes and Retweets", pch = 1, cex = 0.5, col = "navy",
las = 1, cex.axis = 0.5)
```

![[Screenshot_2023-09-19_at_11.04.19_AM.png]]

> **Syntax Note:** To generate a scatterplot we specify two variates as the first two arguments of the function. The first argument is plotted on the x-axis, while the second argument is plotted on the y-axis.  
>   
> 
> **Syntax Note:** `pch = 1` specifies a plot symbol of a circle, while `cex = 0.5` and `cex.axis = 0.5` control the size of the plot symbol and axis labels, respectively. Try changing these numbers!

  

For the dataset for this particular example, a scatter plot may not be completely appropriate, as there are many points focused into one area. While you are not required to do this for Assignment 1, you may (if you wish) experiment with a couple of techniques to help visualize data such as these.

  

One option is to exclude the particularly extreme points from our plot. For example, in my plot there are two isolated tweets with over 40,000 retweets. We can exclude these in a few ways, one is to create a new subset of our data using the `subset()` command:

```R
subtweets <- subset(tweets, subset = (retweets < 40000))
```

Plotting this subset gives us:

```R
plot(subtweets$likes, subtweets$retweets, xlab = "Likes", ylab = "Retweets",
main = "Likes and Retweets (Subset)", pch = 1, cex = 0.5,
col = "navy", las = 1, cex.axis = 0.5)
```

![[Screenshot_2023-09-19_at_11.07.30_AM.png]]

> **Important:** If you exclude points from a plot this must be noted when reporting the results!

  

Another technique we can use to improve a plot is to transform the data, such as by taking logs. Let’s try that on the original dataset (i.e., without excluding any tweets):

```R
tweets$likes.log <- log(tweets$likes + 1) # log-transform the likes variate
tweets$retweets.log <- log(tweets$retweets + 1) # log-transform the retweets variate
plot(tweets$likes.log, tweets$retweets.log, xlab = "likes.log",
	ylab = "retweets.log", main = "Likes and Retweets (log-transformed)",
	pch = 1, cex = 0.5, col = "navy", las = 1)
```

![[Screenshot_2023-09-19_at_11.08.32_AM.png]]

This makes the plot a lot easier to interpret. It also highlights some important features of the data. Most notably, we see ‘lines’ (or rows) of points, due to the fact that likes and retweets are discrete, rather than continuous variates.

  

Finally, we can calcuate the sample correlation between vairates using `cor()`: