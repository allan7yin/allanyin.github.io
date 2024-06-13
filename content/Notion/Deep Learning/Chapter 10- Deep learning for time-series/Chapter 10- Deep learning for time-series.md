Time-series data can be any data obtained via measurements at regular intervals, such as:

- daily price of stock
- electricity consumption in a city
- weekly sales of a store

Unlike data we’ve worked with so far (e.g bank records), working with time-series data involves understanding the **dynamics of a system — its periodic cycles,** how it trends over time, etc. By far, the most common time-series related task is **forecasting:** predicting what will happen next in a series. Forecast electricity consumption a few hours in advance so you can anticipate demand; forecast revenue a few months in advance so you can plan your budget; forecast the weather a few days in advance so you can plan your schedule. Forecasting is what this chapter focuses on. But there’s actually a wide range of other things you can do with time-series:

- **Classification:** Assign one or more categorical labels to a time-series
- **Event detection:** Identify occurrence of specific events within a continuous data stream, a particularly useful application is **hot-word detection** (_think things like “Hey Siri”_)
- **Anomaly detection:** Detect anything unusual happening within a continuous data-stream  
      
    

When working with time-series data, we will encounter a wide range of domain-specific data-representation techniques. To understand this well, we should be familiar with **Fourier transform** … but I’m not, so here are notes for it:

---

### Fourier Transform

> [!info] But what is the Fourier Transform? A visual introduction.  
> An animated introduction to the Fourier Transform.  
> [https://www.youtube.com/watch?v=spUNpyF58BY&ab_channel=3Blue1Brown](https://www.youtube.com/watch?v=spUNpyF58BY&ab_channel=3Blue1Brown)  

The above is a very good video explaining the fundamentals of the **Fourier Transform**. The main idea consists of expressing a series of values in terms of a superposition of waves of different frequencies. So one sound you hear (where sound is actually pressure recorded over time, so time-series data), is actually the superposition of multiple sinusoidal waves at different frequencies. These sinusoidal waves are often referred to as "harmonics" or "frequency components.” This idea is _**highly valuable when it comes to pre-processing data and performing feature engineering**_, as a way to prepare data before training a model with it.

> While this is **very important**, this chapter will focus on building the model to be trained on this processed data. Of course, to compete in **kaggle competitions**, becoming familiar with different pre-processing techniques is vital. What we will focus on instead, is RNN and how to apply them to time-series forecasting.

---

  

We’ll start off with a simple example **→ temperature-forecasting example**

Throughout this chapter, all of our code examples will target a single problem: predicting the temperature 24 hours in the future, given a time-series of hourly measurements of quantities such as atmospheric pressure and humidity, recorded over the recent past by a set of sensors on the roof of a building. **As you will see, it’s a fairly challenging problem!**