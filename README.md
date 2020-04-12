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

#### Duration of inversion

## Input description
## Output description

## Preprocessing
#### Drop NaN
#### Standardization
#### up-sampling 

## Training Model and Predicting
#### Choosing ML Models and Lags

#### Different Spread
#### Comparision


## Conclusion

## Improvement in the future

## Reference
