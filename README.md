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
For now, our raw data has the daily and monthly yield of US tresury bonds with different maturities. 
![bond yeild](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/bond%20yield.png)
The data presents a significant size gap. The longer the term, the greater the yield, so we standardized the data.
By dividing the indicator of whether the economy is depressed, we draw a boxplot of different interest spreads. It can be seen from the figure that the spread of the two sets of data with and without depression shows a large gap, even no overlapping. This is an intuitive result that shows that the interest rate differential will indeed affect the economic situation.
![description](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/description.png)
## Feature Engineering
#### Interest rate spread Generating
We use the US Treasury bond rate with periods of 3 months, 1 year, 10 years and 20 years of 1962.1.6-2007.12.29.  
We combine the short-term and long-term separately to get 4 sets of interest spreads: 3 months-10 years, 3 months-20 years, 1 year-10 years, 1 year-20 years.
#### Lag Term Generating
Unlike the common classification problems, in our case with inversion and recession, each sample data is not independent. The support of historical data is required to better reflect the current interest rate structure.
For each original interest rate and 4 sets of interest spreads, we generate lag items from T to T-10 for each weekly sample to provide historical data. In this way, the impact of historical data is included in each independent sample. In the following analysis and model, two groups of data from T to T-5 and T to T-10 are respectively collected to compare the influence of historical data length on the model.
#### Duration of inversion
We counted the duration of each inversion and included this feature in each sample to reflect which stage the current sample is in the entire inverted curve. In this feature, 0 represents that it is currently in the positive spread range, and a positive integer indicates that how many weeks it has entered the negative spread period.
## Input description
Feature | Explaination
------------ | -------------
LABEL | Output, indicating if the current interest structure shows certain characteristic associated with recession (introduced in detail below)
13weeks, 26weeks, 39weeks, 52weeks | Split the *label* output into	quaterly data using dummy variable
time0_10, time1_20, time1_10, time0_20 | Duration of inversion within each interest spread. *0* indicating 3 months, 1 indicating 1 year, 10 indicating 10 year, 20 indicating 20 year. For example, time0_10 stands for spread between 10-year interest rate and 3-month interst-rate  
10y-3m,  20y-1y,  10y-1y,  20y-3m | Original interest spread 
x_0.25, x_1, x_10, x_20 | Original interest of 3-month, 1-year, 10-year and 20-year respectively.
x_10_0_n, x_20_1_n,	x_10_1_n,	x_20_0_n | The lag term where n indicating T-N

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
Through descriptive analysis results, we found that different yield values are different in size. In order to eliminate the model result error caused by the size of the data itself, we standardized the data.
#### up-sampling 
Considering that economic recession is an unconventional situation, we need to consider the amount of data between the recession and the normal situation. Through statistics, we have a total of 2234 samples, while the recession is only 340. In order to eliminate the impact of data imbalance on the model results, we upscaled the recession samples, the data with label=1 in the training set.
## Training Model and Predicting
#### Choosing ML Models and Lags
We choose three models(LR,SVM and Tree). And we use CV accuracy, F1 score and confusion matrix to assess model performance.

Scenario 1: 
Input: 4 sets of interest spreads: 3 months-10 years, 3 months-20 years, 1 year-10 years, 1 year-20 years.And US Treasury bond rate, duration of inversion and lag term generating from T to T-5 related to these 4 sets of interest spreads.
Output:
![T5_4](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/T5_4.PNG)

Scenario 2: 
Input: 4 sets of interest spreads: 3 months-10 years, 3 months-20 years, 1 year-10 years, 1 year-20 years.And US Treasury bond rate, duration of inversion and lag term generating from T to T-10 related to these 4 sets of interest spreads.
Output:
![T10_4](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/T10_4.PNG)

When we compare the results between the three models under 2 scenarios,  CV accuracy and F1 score of Tree model are always better than the others. The tree model works best.
![10vs5](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/10vs5.PNG)
When we compare the results between the 2 scenarios.The results of scenario2 is always better than scenario1, which means T-10(the spread lags 10 weeks) performs better.

#### Different Spread
We also compare the results between different spread. Besides taking all 4 sets of interest spreads as input, we also take each of 4 sets of interest spreads as inuput individually.
![5Scen](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/5Scen.PNG)

#### Multi-dimensional Output 
We divided the 52-week sample before the recession into 4 parts to predict a more accurate time. Mark the sample as 0 within 52 to 39 weeks before the start of each recession, as 1 within 39 to 26 weeks, as 2 within 26 to 13 weeks, and as 3 within 13 weeks.
Output of 2 scenarios (T-5, T-10):
![multi_cm](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/data/multi_cm.PNG)
The results of the multi-dimensional output is not good. It shows that the inverted curve can only predict the recession in about a year, but it cannot accurately predict the accurate time.
## Conclusion
After processing the data, we used the LR, Tree, and SVC models to predict the future economic situation. The Tree model works best. Through the model results, we found that the effect of T-10 is better than T-5. So, we chose T-10, which means the spread lags 10 weeks.We found that a inverted yield is indeed a good indicator for predicting recession. The invertion is basically accompanied by the recession one year later. However, the results of the multi-dimensional output shows that the inverted curve can only predict the recession in about a year, but it cannot accurately predict the accurate time.
## Improvement in the future
* We will continue to improve the results of multi-dimensional output, clarify the problems of the current model, and try to build a model that can accurately predict the occurrence time of the recession.
* Adjust the hyperparameters of the model through Grid Research to improve the prediction accuracy.
## Reference
