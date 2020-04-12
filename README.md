# Yield-curve Inversion and Recession
## Team Member
Student Number | Github ID
------------ | -------------
1901212658 | [knowsnothing753](https://github.com/knowsnothing753)
1901212611 | [jinzhaol](https://github.com/jinzhaol)
1901212529 | [Oliver-Qu](https://github.com/Oliver-Qu)
1901212556 | [hanchengzheng](https://github.com/hanchengzheng)
## Goal of the project
An inverted yield curve often signals an impending recession in the past few decades in the U.S. Therefore, we speculate that an inverted yield curve may be a key indicator for predicting recession. We tried to find the relationship between the term structure of interest rates and recession. We'd like to use the yield of the US tresury bonds to predict whether there will be a recession in the future and hope to construct a model that can estimate the exact time of the recession based on yield curve. 
## Data description
For now, our raw data has the daily and monthly yield of US tresury bonds with different maturities. For each sample, if there is a recession within a year we will classify it as 1, if not, 0. Moreover, We will do some feature engineering in the future, for example, considering the duration of each inversion. We will put more information after we finish the data constructing.

## Feature Engineering
#### Interest rate spread Generating
We use the weekly frequency of 1962.1.6-2007.12.29, the interest rate period is 3 months, 1 year, 10 years, 20 years of the US Treasury bond rate, to match the several depressions provided on NBER.
We combine the short-term and long-term separately to get 4 sets of interest spreads: 3 months-10 years, 3 months-20 years, 1 year-10 years, 1 year-20 years.
#### Lag Term Generating
Unlike the common classification problems, in our case with inversion and recession, each sample data is not independent. The support of historical data is required to better reflect the current interest rate structure.
For each original interest rate and 4 sets of interest spreads, we generate lag items from T to T-10 for each weekly sample to provide historical data. In this way, the impact of historical data is included in each independent sample. In the following analysis and model, two groups of data from T to T-5 and T to T-10 are respectively collected to compare the influence of historical data length on the model.
#### Duration of inversion
We counted the duration of each inversion and included this feature in each sample to reflect which stage the current sample is in the entire inverted curve. In this feature, 0 represents that it is currently in the positive spread range, and a positive integer indicates that how many weeks it has entered the negative spread period.
## Input description
## Output description
Data processing of output is a challenging problem we face. We mainly face two problems.
1. Recession is relatively rare in the entire data set and will cause extreme imbalance in the data set;
2. How to properly process the output to reasonably reflect the relationship between the recession and each sample. Specifically, we notice that there is a certain time interval between the occurrence of inversion and the beginning of recession, that is, the current inversion has a lagging effect on the prediction of recession. How should we reasonably reflect this relationship?

Faced with these problems, we have devised the following solutions.

According to NBER's definition of recession:
>The NBER does not define a recession in terms of two consecutive quarters of decline in real GDP. Rather, a recession is a significant decline in economic activity spread across the economy, lasting more than a few months, normally visible in real GDP, real income , employment, industrial production, and wholesale-retail sales.

In simple terms, if certain economic indicators (such as GDP) have declined significantly for several consecutive months, it is defined as a depression. Then, by screening the two criteria of the change rate of the economic index and the duration of the rise or fall, a complete economic cycle can be divided into several intervals and marked with integers such as 1, 2, 3, etc in order to reflect which stage of the entire economic cycle the current interest rate structure corresponds to. However, it still cannot effectively solve the problem of data imbalance. According to these two criteria, the number of samples corresponding to the rising or flat period will be significantly higher than the number of samples during the depression period. An alternative method is to mark the entire economic cycle equally in time, but this will result in the inability to effectively distinguish between depression and non-depression periods.

Since there is a certain time interval between the occurrence of inversion and the beginning of recession, that is, the current inversion has a lagging effect on the prediction of recession, we have reason to believe that the interest rate for a period before recession will be different from the non-depression period. Structural features. Therefore, we adopt the following scheme. We counted the interval between the start time of each recession and the start time of the latest inversion. There were 7 recessions in the sample statistical period. The average value of the 7 time intervals is about 48. We round it up to take a complete integer year and 52 Week is the time interval for the final division output. Mark the sample within 52 weeks before the start of each recession as 1, other data as 0, and deal with the problem of data imbalance through Up-Sampling. (In order to fully reflect this information, we set a feature on the input to record the duration of inversion)
## Preprocessing
#### Drop NaN
In the data, the US 20-year bond yield from 1987 to 1989 has data missing. Considering the issue of statistical caliber of data, we believe that it is unreasonable to use other methods of data to integrate. However, the 3-month bond yield, 1-year bond yield and 10-year bond yield data are complete during this period. By calculating the spreads of these data, we found that there is no interest rate inversion during this time, so this part of the data is not the main target of our research, and it will not significantly affect our model results. In the independent variables of the model, we express the interest rate structure at a specific time by the duration of the recession and the time lag. Such input can ensure that the model does not apply each individual interest rate separately, but fully considers the overall effect of changes in interest rates. So, this part of the null value will not affect our overall estimate of the interest rate structure. Considering the above issues, we deleted this part of the incomplete data.
#### Standardization
#### up-sampling 

## Training Model and Predicting
#### Choosing ML Models and Lags

#### Different Spread
#### Comparision


## Conclusion

## Improvement in the future

## Reference
