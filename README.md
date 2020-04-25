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
Our raw data is the weekly yield of US tresury bonds with different maturities. 
![bond yeild](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/bond%20yield.png)
The data presents a significant size gap. The longer the term, the greater the yield, so we standardized the data.
By dividing the indicator of whether the economy is depressed, we draw a boxplot of different interest spreads. It can be seen from the figure that the spread of the two sets of data with and without depression shows a large gap, even no overlapping. This is an intuitive result that shows that the interest rate differential will indeed affect the economic situation.
![description](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/description.png)
## Feature engineering
#### Term spread Generating
Bsed on the data of US treasury bonds with maturity of 3 months, 1 year, 10 years and 20 years from 1962.1.6-2007.12.29, we combine the short-term and long-term separately to get 4 sets of term spreads: 10 years-3 months, 10 years-1 year, 20 years-3 months, 20 years-1 year.
#### Lag Term Generating
Unlike the common classification problems, in our case with inversion and recession, each sample data is not independent. The historical data is required to better reflect the current interest rate structure.
For the 4 sets of interest term spreads, we generate lag items for each sample from T-1 to T-10 to provide historical data up to 10 weeks ago. In this way, the impact of historical data is included in each independent sample. In the following analysis , two groups of lagged data—T to T-5 and T to T-10—are respectively generated to compare the influence of different lag terms.
![Lag](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/Lag.png)
#### Duration of inversion
We also counted the duration of each inversion and included this feature in each sample to reflect which stage the current sample is of the entire inverted curve. For this feature, 0 represents that the sample currently has s positive term spread, and a positive integer indicates that how many weeks the inversion has last since the first one appeared.We counted the duration for each term spread, so we have 4 new features corresponding to 4 term preads.
## Input description
Feature name | Explaination
------------ | -------------
x_0.25, x_1, x_10, x_20 | Original yield with maturity of 3 months, 1 year, 10 years and 20 years
10y-3m, 20y-1y, 10y-1y, 20y-3m | Interest term spread without lagging
x_10_0_n | The lag term of 10y-3m, n can be 1,2...10, meaning T-n term 
x_10_1_n | The lag term of 10y-1y, n can be 1,2...10, meaning T-n term
x_20_0_n | The lag term of 20y-3m, n can be 1,2...10, meaning T-n term
x_20_1_n | The lag term of 20y-1y, n can be 1,2...10, meaning T-n term
time10_0 | Duration weeks of inversion of 10y-3m
time10_1 | Duration weeks of inversion of 10y-1y
time20_0 | Duration weeks of inversion of 20y-3m
time20_1 | Duration weeks of inversion of 20y-1y

## Output description
Data processing of output is a challenging problem for us. We mainly face two problems.
1. Recession is relatively rare in the entire data set and will cause extreme imbalance in the data set;
2. How to properly process the output to reasonably reflect the relationship between the recession and each sample. Specifically, we notice that there is a certain time interval between the occurrence of inversion and the beginning of recession, that is, the current inversion has a lagging effect on the prediction of recession. How should we reasonably reflect this relationship?

Faced with these problems, we have devised the following solutions.

According to NBER's definition of recession:
>The NBER does not define a recession in terms of two consecutive quarters of decline in real GDP. Rather, a recession is a significant decline in economic activity spread across the economy, lasting more than a few months, normally visible in real GDP, real income , employment, industrial production, and wholesale-retail sales.

In simple terms, if certain economic indicators (such as GDP) have declined significantly for several consecutive months, it is defined as a depression. Then, by screening the two criteria—the change rate of the economic index and the duration of the rise or fall—a complete economic cycle can be divided into several intervals and marked with integers such as -2, -1, 1, 2, etc to reflect which stage of the entire economic cycle the current interest rate structure corresponds to. However, it still cannot effectively solve the problem of data imbalance. According to these two criteria, the number of samples corresponding to the rising or flat period will be significantly higher than the number of samples during the depression period. An alternative method is to mark the entire economic cycle equally in time, but this will result in the inability to effectively distinguish between depression and non-depression periods.

Since there is a certain time interval between the occurrence of inversion and the beginning of recession, that is, the current inversion has a lagging effect on the prediction of recession, we have reasons to believe that the interest rate for a period before recession will have a different pattern from the non-depression period. Therefore, we adopt the following scheme. We first collect the start time of the total 7 recessions in the sample covering period showing below.(https://www.nber.org/cycles/)
> December 1969(IV)<br>
November 1973(IV)<br>
January 1980(I)<br>
July 1981(III)<br>
July 1990(III)<br>
March 2001(I)<br>
December 2007 (IV)

We then count the interval between the start time of the inversion and the start time of the nearest recession afterward.The average value of the 7 intervals is about 48 weeks, so we believe the interest rates pattern within 48 weeks before a recession have the predictive power, and we round it up to 52 weeks which is integer year. Next, we mark the samples within 52 weeks before the start of each recession as 1, other data as 0, and deal with the problem of data imbalance through Up-Sampling.
![Output](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/Output.PNG)

## Data preprocessing
#### Dropping NaN
In the data, the data of 20-year bond yield from 1987 to 1989 are missing. However, the 3-month bond yield, 1-year bond yield and 10-year bond yield data are complete during this period. By calculating the spreads of these data, we found that there is no interest rate inversion during this time, so this part of the data is not the main target of our research, and it will not significantly affect our model results. In the independent variables of the model, we express the interest rate structure at a specific time by the duration of the recession and the time lag. Such input can ensure that the model does not apply each individual interest rate separately, but fully considers the overall effect of changes in interest rates. So, this part of the null value will not affect our overall estimate of the interest rate structure. Considering the above issues, we deleted this part of the incomplete data.
#### Standardization
Through descriptive analysis results, we found that different yield values are different in size. In order to eliminate the model result error caused by the size of the data itself, we standardized the data.
#### Up-sampling 
Considering that economic recession is an unconventional situation, we need to consider the amount of data between the recession and the normal situation. Through statistics, we have a total of 2234 samples, while the number of samples related recession is only 340. In order to eliminate the impact of data imbalance on the model results, we upscaled the recession samples, the data with label=1 in the training set.
## Training models and predicting
#### Choosing ML models and lags
We tried three models(LR,SVM and Tree), and use CV accuracy(F1 score method), F1 score(on single test data) and confusion matrix to assess model performance.

* [Scenario 1: lag up to T-5](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/univariate_T5.ipynb)  

  * Input:   
    * 4 types of yeild with different maturities  
    * 4 sets of term spreads: 10 years-3 months, 10 years-1 year, 20 years-3 months, 20 years-1 year.  
    * corresponding lag term for each spread form T-1 to T-5   
    * duration of inversion for each spread  

  * Output:
  
![T5](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/T5.png)

* [Scenario 2: lag up to T-10](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/univariate_T10.ipynb)    

  * Input:  
    * 4 types of yeild with different maturities   
    * 4 sets of term spreads: 10 years-3 months, 10 years-1 year, 20 years-3 months, 20 years-1 year  
    * corresponding lag term for each spread form T-1 to T-10  
    * duration of inversion for each spread  

  * Output:
  
![T10](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/T10.png)

* When we compare the results between the three models under 2 scenarios,  CV accuracy and F1 score of Tree model are always better than the others. The tree model works best.

* When we compare the results between the 2 scenarios.The results of scenario 2 is always better than scenario 1, which means T-10(the spread lags up to 10 weeks) performs better.
![10vs5](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/10vs5.PNG)


#### Different term spread
We also compare the results between different term spread，i.e., we take only one set of term spread as inuput at a time. Specifically, we first take [10 years-3 months](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/10_0_T10.ipynb) term spread and its corresponding lag term and duration of inversion as input to get one result, then [10 years-1 year](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/10_1_T10.ipynb), [20 years-3 months](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/20_0_T10.ipynb), and [20 years-1 year](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/code/20_1_T10.ipynb).
![5Scen](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/5Scen.PNG)


#### Multi-dimensional Output 
We divided the 52-week sample before the recession into 4 parts to predict a more accurate time. Mark the sample as 0 within 52 to 39 weeks before the start of each recession, as 1 within 39 to 26 weeks, as 2 within 26 to 13 weeks, and as 3 within 13 weeks.
Output of 2 scenarios (T-5, T-10):
![multi_cm](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/multi_cm.PNG)
The results of the multi-dimensional output is not good. It shows that the inverted curve can only predict the recession in about a year, but it cannot accurately predict the accurate time.
## Tree model outcome analysis
This is how tree works in scenario 2.In the picture, X[0]~X[3] mean duration of inversion, X[4]~X[7] mean yield spread without lag, X[8]~X[11] mean original bond yield with different maturities, X[12]~X[51] mean yield spread with max lag 10.In the decision structure, yield spread is wildly used, but <0 is not the criteria, usually the decision boundary is between 0 and 1. This means spread can explain recession but it doesn’t have to be inversion, as long as the yield difference between long-term bond and short-term bond is smaller than 1, it can be a sign of recession.
![allT10](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/allT10.png "scenario 2")

This is how tree works when we only take 10 years-1 year as input. In the picture, X[0] means duration of inversion, X[1] means yield spread without lag, X[2] and X[3] mean original bond yield with different maturities. Similarly, yield spread is wildly used in the decision structure, and the decision boundary for the node is usually between 0~1. But this time, we also find another interesting outcome, the original yield of 10 years and 1 year bond are often used (11 times out of 41 branching nodes). It shows that the original yield of these two bonds also have predictive power.
![10-1](https://github.com/knowsnothing753/PHBS_MLF_2019/blob/master/figure/10-1.png "scenario 2")

## Conclusion
* The Tree model works better than LR and SVM. 
* The model with up to T-10 term spread lag is better than T-5. 
* In th tree model, yield spread is widely used as branching criteria but the boundary is usually betweent 0 and 1 instead of 0 as we expect.
* The 10 years-1 year pair works better than other spread term, and besideds the spread the original yield of these two bond are often used as a branching criteria in tree model.
* The inversion is usually accompanied by the recession one year later. 
* The results of the multi-dimensional output shows that the inverted curve can only predict the recession in about a year, but it cannot accurately predict the specific quarter.
## Improvement in the future
* We will continue to improve the results of multi-dimensional output, try to build a model that can accurately predict the occurrence time of the recession.
* We will try to explain the eocnomic reason of why the tree model works so, i.e., using spread and some bond yield as branching criteria to predict recession .
## Reference
Yield curve inversion: a recession indicator. from https://github.com/PHBS/MLF/wiki/Yield-curve-inversion:-a-recession-indicator
