# Using SARIMAX modeling to predict the New York Rental Market

2022 is a hard time to be a renter in New York City. Rental prices have reached record highs and news outlets including the [New York Times](https://www.nytimes.com/2022/07/11/business/economy/rent-inflation-interest-rates.html) report that potential homeowners priced out of a hot housing market may be driving demand for rentals up even further. A more recent Times [story](https://www.nytimes.com/2022/08/01/nyregion/why-its-so-hard-to-find-an-affordable-apartment-in-new-york.html) blames New York's restrictive zoning regulations for the city's failure to keep up with demand increasing demand.

## Business Problem

Under these circumstances, renters can benefit from a reliable forecast of the direction the housing market will take over the coming months. If rents are expected to shoot up even more, then the best strategy for renters would be to lock in a new lease as soon as possible. They would also be advised to try to lock in a longer term lease if it at all possible, even if it means paying a slightly higher monthly rent.

On the other hand, if rents are expected to fall or stabilize, renters should probably avoid longer term leases and, if possible, wait until prices bottom out before looking for a new apartment.

## Data and Modeling

For this study I built a SARIMAX model to predict rents based on StreetEasy's NYC Rental Index, a metric that uses a repeat-price method to measure monthly change in median rent and that includes values from January 2007 through May 2022.

SARIMAX modeling is an extension of SARIMA modeling, in which values of in a time series are regressed against past values and error terms for that same time series. With SARIMAX modeling, the time series is also regressed against additional exogenous variables.

## Choosing Exogenous Variables

Since I wanted to be able to predict rental values in advance, I needed to choose exogenous variables that were correlated with rental prices at a minimum six month lag. 

I began by identifying a number of variables that I thought showed potential as leading indicators for changes in rental prices. The variables I initially chose were:

1. Sale Price Indices for NYC, Manhattan, Brooklyn and Queens
2. Federal Funds Funds Rate
3. Average 30 Year Mortgage Rate
4. PPI (Producer Price Index) for Building and Construction Materials
5. Employees in Construction in New York State
6. CPI (Consumer Price Index) for Urban Consumers
7. Total Personal Income in New York State
8. Total Wages and Salaries in New York State

These metrics fall into four basic categories
1. Sale Prices Indices measure the price of a comparable good
2. Fed Funds Rate and Mortgage Rate measure the price of credit
3. PPI for Building Materials and Employees in Construction both affect the supply of housing
4. Total Income and Total Wages and Salaries both affect demand for housing

Categories three and four also correspond to different ways of explaining the housing crisis New York is currently facing. Category three explains it in terms of barriers to the construction of affordable housing, while category four explains it in terms of excess demand--which means either more people trying to more to New York or inflation driven by higher incomes. 

## Exogenous Variables as Leading Indicators

While it's likely that all of these factors have played some role in driving **current** rental prices, we can only rely on the predictive power of our exogenous variables if they make good predictions for earlier time periods as well. I therefore divided the data set into five three year periods and calculated the correlations of the target variable with each of the exogenous variables for a series of lags, the goal being to identify the exogenous variables that were consistent leading indicators for the target variable at a certain number of lags.

Because the 2013-2019 period showed stable growth in both rental prices and most of the exogenous variables, correlations were often consistently high for all lags throughout this period. My choice of leading indicators was therefore heavily dependent on correlations for the other three periods.

Ultimately I only chose fives lagged variables as leading indicators for modeling:

1. Total Wages and Salaries 6 Month Lag
2. Employees in Construction 6 Month Lag
3. Manhattan Sale Prices 6 Month Lag
4. Cost of Building Materials 12 Month Lag
5. Employees in Construction 24 Month Lag

The first four of these variables showed positive correlations at the given lag for all five periods. Employees in Construction showed a strong negative correlation--but only for three of the five periods. I was therefore uncertain whether it would serve as a good predictor, but wanted to include it since I was intrigued by the possibility of establishing a negative relationship between the size of the construction industry and rental prices.

Since, I wanted to avoid multicollinearity in my predictive variable I was using for modeling, I selected only combinations of these variables for which the maximum VIF was below 10.

## SARIMA Modeling

For model validation, I trained models on the 2007-2019 period and testeed them based on their dynamic predictions for the 2019-2022 period. 

As a baseline for my SARIMAX modeling, I looked at the performance of SARIMA models on the data using two metrics: AIC and RMSE for test data. AIC measures how well the model describes the data it has been trained on, penalizing models for adding terms that don't contribute significantly to the model's performance. RMSE tell us how the model is performing on test data. 

The best SARIMA model in terms of AIC had an AIC score of 643, while the best SARIMA model in terms of error had an RMSE of 176. 

## SARIMAX Modeling

Ideally, the best SARIMAX model would have modeled the 2007-2019 data better than the best SARIMA model, as measured by AIC score. However, because the 2007-2019 period was fairly uneventful, the best SARIMAX model by AIC actually underperformed the best SARIMA model by that metric--with an AIC of 644, one point higher than the best SARIMA model. Furthermore, it's RMSE for the test data was 226, which means that it also was worse than SARIMA at making predictions.

I next looked at the best SARIMAX models by RMSE. The top 2 used Building Materials and Total Wages and Salaries as regressors. However, these had AIC scores of 1096 and 1055. Furthermore, dynamic predictions for three periods of training data showed much larger error than the error for the training data, with true values falling outside the predictions' 95% confidence intervals. While adding additional MA terms would have widened the confidence interval and improved the model, they would have done little to reduce the overall error.

The model I ultimately chose included three exogenous regressors: Building Materials 12 Month Lag, Employees in Construction 6 Month Lag, and Manhattan Sale Prices 6 Month lag. The first version of this model had an error of 128, which only slightly above that of the lowest error models, while it's AIC was 855. Error could be reduced to 135 and AIC to 835 by adding AR terms through lag 2 and MA terms through lag 7. This also reduced the error of the 2010-2013 dynamic prediction from 322 to 154. This is ideal because it means that the means that even though the model was selected primarily for its ability to predict rental prices for the 2019-2022 period, it also performs well on earlier periods, including both the stable 2013-2019 period and the more anomalous recovery period of 2010-2013. While this model's AIC wasn't quite as low as that of the best SARIMA model it did satisfy the condition of minimizing AIC score while significantly improving upon the performance of the most accurate SARIMA model.

### Model coefficients and interpretation

While the Manhattan Sale Price coefficient was not significant, Building Materials 12 Month Lag and Employees in Construction 6 Month Lag had coefficients of 6.5523 and 16.1230 respectively, with p-values below 0.0005.

This means that in our model, Building Materials costs account for $520 in total Rental Prices variation over the 2007-2022 period, while Employees in Construction account for a total $333 variation in Rental Prices.

This shows up most dramatically at the beginning of COVID when a drop in Employees in Construction precedes a drop in rental prices and in late 2021, when the rise in the cost of construction materials a year earlier corresponds to a rise in rental prices. 

While there is an intuitive relationship between high costs of building material and high housing costs. The fact that the number of employees in construction is a leading indicator of rental price rises is slightly harder to explain. In the long run, we would expect that more employees in construction would stabilize rental prices, but it seems like in the short run, the number of employees in construction is anticipating high demand, though not early enough to prevent prices from rising. It therefore makes sense to think of high construction costs as a cause of high housing costs, but a growth in the construction sector as simply a leading indicator.

### Using our Model for Forecasting

