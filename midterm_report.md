
# Flight Delay Due to Weather Trend Analysis
#### Jackson Montijo (jhm343), Alexander Schmack (as2968), Kais Baillargeon (kpb52)

## Our Project
We set out to understand how daily local weather trends at an airport pair impact the expected delay of a given flight between those two airports. As 5% of all flights were delayed by weather in 2018, we saw signifigant value in decreasing the lost time of our employees due to flight delays. In this midterm report, we discuss the work completed so far and the results of our preliminary analysis.

## Our Dataset
As discussed in our original project proposal, we used two government data sets as the basis of our analysis.

**On-Time : Reporting Carrier On-Time Performance (1987-present)**
The original dataset was extremely large, with data going back to 1987, so we decided to only use data from the year 2018 - the most recent available. We then generated uniform, random sample of 20% of each month's data to produce a manageable yet well distributed dataset for analysis. We felt this was important as we assume that there will be seasonal component to our model. Finally, we filtered the entries so that only the airports of Atlanta, Los Angeles, Chicago, Dallas, Denver, New York (JFK), San Fransisco, Seattle, Las Vegas, and Orlando were included - the top 10 busiest US airports by passenger traffic.

**Daily Summaries Station Details**
We took the daily summaries for weather stations located at our target airports for each day in 2018 and joined them to the dataset above. For each flight, we found the summary on its date of departure for both the origin and destination airports. Data for origin airports was suffixed with an "x" and for destination airports with a "y".

**Feature Engineering**
With the datasets now joined, we employed several feature engineering techniques to prepare it for use in our model. First, columns unrelated to the analysis or providing only duplicate information in different formats were removed. Finally, we created one-hot vectors for nominal data originally found in the On-time reporting dataset. These were flight operator, origin airport, and destination airport. The final dataset used in our model has **103,391 examples and 67 features.**

### Missing or Corrupted Data
None of the data in our dataset is missing or corrupted. The data was collected by two government agencies with strict data governance policies and contained no optional variables across each entry. We were also careful to only use data where every variable was measured, avoiding the removal or addition of data points with time. 

### Dataset Statistics
When analyzing the general statistics of our dataset, we decided to look at 3 key features. Delay, carrier, and weather type. Respectively, these high level analyses give us a sense of how delays in our dataset compared with the known statistics we began the project with, how carriers affect the chance of delay, and how frequently certain weather events affected delayed flights.

**Delays**
37,783 examples were delayed on arrival, which means approximately 36.5% of flights were delayed in 2018. This is higher than the 24% reported by the U.S. Government Accountability Office (U.S. GAO) in 2007 that we referenced when justifying the value of our exploration in the initial project proposal. Of this 36.5%, approximately 2.4% of delayed flights were due to weather. Therefore, less than 1% of total flights were delayed due to weather, which is much lower than the 5% reported by the U.S. GAO.
To analyze the frequency of delay times, we removed all on-time flights to produce the distribution of delays across the whole dataset. There were some outliers delayed by over 1000 minutes, however, these were far higher than the average delay of just **33.18 minutes**. After applying the empirical distribution function to the dataset, we saw that the vast majority of delays were under 200 minutes long. However, to get a better sense of how delays varied around the mean, we decided to create a histogram using only delays between 0 and 60 minutes. The interesting spike in delays around 15 minutes can be seen below.
![](https://lh3.googleusercontent.com/56E_ayf0IlPrt0Drf73P57CqJpPQfq2A6U0w3Cu1mJ4IVCFJURGibbpRJERqW6ulX0yPyV9IkEAz)

**Carrier Performance**
![](https://lh3.googleusercontent.com/sBtCioAgSyUOM373WJ8wHpN306gNDxYQYR2618yTVghtRm5GDWaBRgHx34fFN7qL_301lWtE_NED)
Skywest Airlines had the higehst percentage of flights delayed at **1.7%** (51 out of 3006) while Frontier Airlines had the lowest percentage at **0.23%** (7 out of 3066).

**Weather**
![](https://lh3.googleusercontent.com/nhQ5MR6ueRbDwaPYdkuh4FSzn50ew3enPPmmN4MCiSN_2C24mLX_WhulWQhpXgsD8aYU_GytkDqR)
At origin airports, the most common weather event affecting delayed flights was Fog (WT01), followed by thunder (WT03), and finally smoke (WT08).
![](https://lh3.googleusercontent.com/IZjGVIuKbl8790AoNi_fYPL8C4rgl2-R45hoPqS67qF9atEooECeswHQ_PkfjHnJnwhoK93vANgT)
At destination airports, smoke was more likely to affect delayed flights than thunder. This differs from at origin airports, however, fog was still the most common by a margin of over 150 flights.

## Our Model

**Methodology**
First we must create a model to classify our data into two groups, one for weather delay and one for no weather delay. A later extension of this would be to do a multi-class classification: Severe Weather Delay, NAS Weather Delay, and No Weather Delay. Once we have classified our data, we must then use a regression model to predict the expected time of the delay.

**Building the Y Vector**
The data we have gives us time of delay due to "Severe Weather" and time of delay due to "National Air System". Of these NAS delays, we know 55% of them are due to weather. To figure out, which 55% of these delays are the ones due to weather, we must run an unsupervised clustering method on the weather features of the data. Once we have our clusters, we select the clusters with the highest fraction of points that have a NAS delay. These clusters are likely to have the NAS delay caused by weather. We then select clusters in descending order until we have 55% of the data points selected. These are the NAS delays that we accept as caused by weather.

### Avoiding Over and Under Fitting
We have devised two solutions to ensuring that our model will generalize well. Firstly, we have broken the model into two parts: first a classification problem, and then a regression problem. By breaking the problem into two parts we help resolve the issue of zero-inflated data set. Secondly, we can throw out some examples of no weather delay in the classification model, to ensure that we do not overfit to the data. Lastly, we sampled our data randomly from each month of the year 2018, to ensure we have a sample with weather representative of all parts of the year. Because there is such a high volume of flights daily, we have a very large amount of data available to us, so expanding training and test set sizes is always feasible, providing another way to ensure the model generalizes well.

### Testing our Model
We created a train-test split using 20% of the data from each month to train and 5% of the data from each month to test. This results in an equivalent 80/20 split. As described above, we sampled from every month as we expect there to be a seasonal component to our model. Once sampled, we'll be training our model using the train split and testing using the test split.

### Preliminary Analysis
Preliminary analysis using our train set, a y vector, and least squares regression revealed the following. Smoke or haze at destination airport has the highest impact on a flight being delayed, followed by icy conditions at origin airports, and finally Atlanta as a destination airport. We are not treating these results as definitive, as we still have plans to optimize our algorithm.

## Plans for Model Development
We plan on extending our analysis from its current form as a single model for predicting whether a flight was delayed due to weather and not other factors by using a second model to be applied after the first that predict what weather conditions are most likely to have caused the delay. From there, we'll be able to understand how simple weather forecasts containing attributes such as temperature, weather conditions, and date - such as those found in your phones weather app - along with flight inaformation can be used to predict whether flights from the major airports we are analyzing will be delayed or not.