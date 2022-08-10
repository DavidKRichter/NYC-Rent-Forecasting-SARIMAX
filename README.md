# Using SARIMAX Modeling to Predict the New York Rental Market

2022 is a hard year to be a renter in New York City. Rental prices have reached record highs and news sources including the [Times](https://www.nytimes.com/2022/07/11/business/economy/rent-inflation-interest-rates.html) report that potential homeowners priced out of a similarly hot housing market may be driving up prices even further. A more recent Times [story](https://www.nytimes.com/2022/08/01/nyregion/why-its-so-hard-to-find-an-affordable-apartment-in-new-york.html) blames New York's restrictive zoning regulations for the city's failure to keep up with increasing demand. So how can NYC Renters save money in this hot rental market?


![Rental Index Chart](../../blob/main/images/rental_index.png)

![Rental Increase Chart](../../blob/main/images/yoy_increases.png)

## Business Problem

Under these circumstances, New York renters can benefit from a reliable forecast of the direction the housing market will take over the coming months. If rents are expected to shoot up even more, then the best strategy would be to sign a new lease sooner rather than later. Renters would also be advised to try to lock in a longer term lease, even if it means paying a higher monthly rent.

On the other hand, if rents are expected to fall or stabilize, renters should avoid longer term leases at high prices points and, if possible, wait until prices fall before looking for a new apartment.

## Data and Modeling

For this study I built a SARIMAX model to predict rents based on StreetEasy's NYC Rental Index, a metric that uses a repeat-price method to measure monthly change in median rent and that includes 185 index values from January 2007 through May 2022. Because data was only available for this relatively short date range, we're not able to be as confident in our predictions as we would be if were able to validate our model over a longer time frame.

SARIMAX modeling is an extension of SARIMA modeling, in which values of a time series are regressed against past values and error terms for that same series. With SARIMAX modeling, the time series is also regressed against additional exogenous variables.

## Choosing Exogenous Variables

Since I wanted to be able to predict rental values in advance, I needed to choose exogenous variables that were correlated with rental prices at a minimum six month lag. 

I began by identifying a number of variables that I thought showed potential as leading indicators for changes in rental prices. The variables fell into four basic categories:

1. Measures of Home Prices
   - StreetEasy Sale Price Index NYC
   - StreetEasy Sale Price Index Manhattan
   - StreetEasy Sale Price Index Brooklyn
   - StreetEasy Sale Price Index Queens
2. Measures of the Price of Credit
   - Federal Funds Rate
   - 30 Year Mortgage Rate
3. Measures of Inflation
   - PPI (Producer Price Index) for Building and Construction Materials
   - CPI (Consumer Price Index) for Urban Consumers
4. Measures of Demand
   - Total Personal Income in New York State
   - Total Wages and Salaries in New York State

## Exogenous Variables as Leading Indicators

While it seems likely that all of these factors have played some role in driving current rental prices, we can only rely on the predictive power of our exogenous variables if our model make good predictions for earlier time periods as well. I therefore divided the data set into five three year periods and for each period, calculated the correlations of the target variable with each of the exogenous variables for a series of lags. My goal was to identify the exogenous variables that were consistent leading indicators for the target variable at a certain number of lags.

Because the 2013-2019 period showed stable growth in both rental prices and most of the exogenous variables, correlations were often consistently high for all lags throughout this period. My choice of leading indicators was therefore heavily dependent on correlations for the other three periods.

Ultimately I only chose fives lagged variables as leading indicators for modeling:

1. Total Wages and Salaries NYS 6 Month Lag
2. Employees in Construction NYS 6 Month Lag
3. Manhattan Sale Prices 6 Month Lag
4. Cost of Building Materials 12 Month Lag
5. Employees in Construction NYS 24 Month Lag

The first four of these variables showed positive correlations at the given lag for all five periods. Employees in Construction showed a strong negative correlation--but only for three of the five periods. I was therefore uncertain whether it would serve as a good predictor, but wanted to test its performance since I was intrigued by the possibility of establishing a negative relationship between the size of the construction industry and rental prices.

Since I wanted to avoid multicollinearity in the predictive variables I was using for modeling, I selected only combinations of these variables for which the maximum VIF was below 10.

## SARIMA Modeling

For model validation, I trained models on the 2007-2019 period and tested them based on their dynamic predictions for the 2019-2022 period. 

As a baseline for my SARIMAX modeling, I looked at the performance of SARIMA models on the data using two metrics: AIC and RMSE for test data. AIC measures how well the model describes the data it has been trained on, penalizing models for adding terms that don't contribute significantly to the model's performance. RMSE tell us how the model is performing on test data. 

The table below compares AIC and RMSE for a baseline model, the SARIMA model with the lowest AIC and the SARIMA model whose dynamic predictions yielded the smallest RMSE based on our set of test data.

|                      | AIC  | RMSE Test Data |
|----------------------|------|----------------|
| Baseline AR(1) Model | 1132 | 226            |
| Best SARIMA by AIC   | 643  | 236            |
| Best SARIMA by RMSE  | 1055 | 176            |

## SARIMAX Modeling

Ideally, the best SARIMAX model would have modeled the 2007-2019 data better than the best SARIMA model, as measured by AIC score. However, because the 2007-2019 period was fairly uneventful, the best SARIMAX model by AIC actually underperformed the best SARIMA model by that metric--with an AIC of 644, one point higher than the best SARIMA model. Furthermore, its RMSE for the test data was 226, which means that it also was worse than SARIMA at making predictions.

I next looked at the best SARIMAX models by RMSE. The top 2 used Building Materials and Total Wages and Salaries as exogenous regressors and their dyanmic predictions yielded RMSEs of 112 and 116. However, these had AIC scores of 1096 and 1055, 400 points higher than our best SARIMA models. Furthermore, dynamic predictions for three periods of training data showed much larger error than the error for the training data, with true values falling outside the predictions' 95% confidence intervals. While adding additional MA terms would have both widened the confidence interval and improved the model, they would have done little to reduce the model's awkward fit on the training data.

## Choosing a SARIMAX Model

As the table below shows, the best SARIMAX model by AIC had a high level of error, while the best SARIMAX model in terms of error had a high RMSE. The model I ultimately chose had the lowest AIC score out of the 20 SARIMAX models with the lowest RMSE for the testing data. 

|                      | AIC  | RMSE Test Data |
|----------------------|------|----------------|
| Best SARIMAX by AIC  | 644  | 226            |
| Best SARIMAX by RMSE | 1096 | 112            |
| Best SARIMAX Overall | 835  | 125            |

This model included three exogenous regressors: 
- Building Materials 12 Month Lag
- Employees in Construction 6 Month Lag
- Manhattan Sale Prices 6 Month lag. 

The first version of this model had an error of 128, which was only slightly above that of the lowest RMSE models, while its AIC was 855. Error was reduced to 125 and AIC to 835 by adding AR terms through lag 2 and MA terms through lag 7. Below we can see the error of the model's predictions for four periods, the last of which is the testing period:

|           | RMSE Dynamic Predictions |
|-----------|--------------------------|
| 2010-2013 | 154                      |
| 2013-2016 | 42                       |
| 2016-2019 | 47                       |
| 2019-2022 | 125                      |

Below we see these dynamic predictions plotted, along with true values for the rental index:


![Model Performance on Past Data](../../blob/main/images/BestSARIMAXModel.png)

Even though the model was selected primarily for its ability to predict rental prices for the 2019-2022 period, it also performs well on earlier periods, including both the stable 2013-2019 period and the more anomalous recovery period of 2010-2013. While this model's AIC was 200 points above that of the best SARIMA model, it was the only SARIMAX model to both minimize AIC score while significantly improving upon the predictions of the most accurate SARIMA model.

### Model Coefficients and Interpretation

While the Manhattan Sale Price coefficient for this model was not significant, Building Materials 12 Month Lag and Employees in Construction 6 Month Lag had coefficients of 6.5523 and 16.1230 respectively, with p-values below 0.0005. Below we see how each of these coefficients contributes to the overall predictions made by our model. 


![Model Coefficients](../../blob/main/images/exog_contributions_best.png)

This means that in our model, variation in the cost of Building Materials accounts for over $660 in Rental Price variation over the 2007-2022 period, while variation in the number of Employees in Construction accounts for $330 of variation in Rental Prices.

This shows up most dramatically at the beginning of COVID when a drop in Employees in Construction precedes a drop in rental prices and in late 2021, when the rise in the cost of construction materials a year earlier corresponds to a rise in rental prices. 

While there is an intuitive relationship between high costs of building material and high housing costs, the fact that the number of employees in construction is a leading indicator of rental price rises is slightly harder to explain. In the long run, we would expect that more employees in construction would stabilize rental prices, but in the short run, it may be that increases in the number of employees in construction are anticipating high future demand, though not enough to prevent prices from rising. It therefore makes sense to think of high costs of construction materials as a possible cause of high housing costs, but while growth in the construct sector seems to simply be a leading indicator of high costs.

### Using our Model for Forecasting

To use our model for forecasting through June 2023, we had to extrapolate values for our exogenous variables at the end of our data set. Because the cost of Building Materials is lagged by 12 Months, we only had to extrapolate a single value, whereas for Manhattan Sale Prices and Employees in Construction we had to extrapolate values for around a half a year. Values for Manhattan Sale Prices have very little effect on the model, so the only space for significant error comes from Employees in Construction, which we assumed to be rising at a monthly rate equal to the average monthly rate over the past year. 

![Retrained Model Performance on Past Data](../../blob/main/images/FinalModel.png)

Prior to forecasting, the SARIMAX model was retrained on the full 2007-2022 data set. Coefficients for our two main variables rose (7.2239 for Building Materials and 21.0244 for Employees in Construction), but remained within the 95% confidence interval that was calculated based on data for the 2007-2019 period. This should reassure us, since it means that the model's coefficients haven't been drastically shifted due to the greater volatility of the 2019-2022 period.

![Retrained Model Coefficients](../../blob/main/images/exog_contributions.png)

Forecasting through June 2023, our retrained model predicts that median rents will drop to almost $3100 in September of 2022 and stabilize over the winter before rising above $3400 by the middle of 2023. The worst case scenario (the upper bound of the model's 95% confidence interval) sees an only slight slowing in rental increases, with rents rising above $3700 by mid-2023, while the best case scenario (the lower bound of the model's 95% confidence interval) sees median rents dropping below $3000 and then stabilizing between $3000 and $3100 by mid-next year.

![Model Forecast](../../blob/main/images/forecast.png)

### Recommendations and Caveats

Assuming the mean prediction to be the correct one, we would advise renters to wait until the fall or winter to sign a new lease. While our model suggests a strong likelihood that rents could rise slightly during this period rather than fall, we're unlikely to see rises comparable to those we've seen in the past six months.

That said, we need to account for some of the limitations of our SARIMAX model. 

First, our model is based on a limited data set, including only 185 months of Rental Data. This implies a real possibility that the correlation between Building Materials and rents and between Employees in Construction and rents may be purely coincidental. Our choice to adopt this model is therefore based not only on the model's ability to fit the data but also on the plausibility that these variables are in fact related to rental prices. 

Second, because the target data has been smoothed, our model doesn't fully account for seasonal fluctuations in rental prices. However, because rents are typically lower during winter months, the prospect of seasonal fluctuations should if anything strengthen our recommendation to wait until fall or winter before signing a new lease.

Third, the fact that we selected a model based on its full 2007-2022 track record means that the model necessarily fails to register the effects of variables whose influence on the rental market is less consistent and more dependent on specific conjunctural factors. For example, we were unable to incorporate interest rate data into our model, even though shifts in US monetary policy and the bond market are likely have economic effects that extend to the rental market. 

Despite these caveats, the good performance of our model over a variety of time periods gives us reason to believe the cost of building materials may be a consistent long-term driver of rental prices. While increases over the past year have likely not had their full effect on the market, the overall slowing of PPI growth should make renters and policy makers optimistic that rental price increases will also slow over the coming months.

### Sources

[StreetEasy Price Indices](https://streeteasy.com/blog/data-dashboard)

Freddie Mac, 30-Year Fixed Rate Mortgage Average in the United States [MORTGAGE30US], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/MORTGAGE30US, July 18, 2022.

Board of Governors of the Federal Reserve System (US), Federal Funds Effective Rate [FEDFUNDS], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/FEDFUNDS, July 21, 2022.

U.S. Bureau of Economic Analysis and Federal Reserve Bank of St. Louis, Total Personal Income in New York [NYOTOT], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/NYOTOT, July 25, 2022.

U.S. Bureau of Economic Analysis and Federal Reserve Bank of St. Louis, Total Wages and Salaries in New York [NYWTOT], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/NYWTOT, July 20, 2022.

U.S. Bureau of Labor Statistics and Federal Reserve Bank of St. Louis, All Employees: Construction: Residential Building Construction in New York [SMU36000002023610001SA], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/SMU36000002023610001SA, July 22, 2022.

U.S. Bureau of Labor Statistics, Producer Price Index by Industry: Building Material and Supplies Dealers [PCU44414441], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/PCU44414441, July 19, 2022.

U.S. Bureau of Labor Statistics, Consumer Price Index for All Urban Consumers: All Items in U.S. City Average [CPIAUCSL], retrieved from FRED, Federal Reserve Bank of St. Louis; https://fred.stlouisfed.org/series/CPIAUCSL, July 25, 2022.

### Links to presentation, notebook, and data

View this project's [Google Slides](../../blob/main/presentation.pdf) presentation.

View the Jupyter [notebook](../../tree/main/index.ipynb) for this project.

View [CSV files](../../tree/main/data).

### Repository Structure

├── data : data used for modeling\n
├── images : images used in README\n
├── README.md : project information and repository structure\n
├── index.ipynb : jupyter notebook used for modeling\n
├── presentation.pdf : presentation for stakeholders\n
└── requirements.txt: requirements file
