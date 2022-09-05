# Using SARIMAX Modeling to Predict the New York Rental Market

2022 is a hard year to be a renter in New York City. Rental prices have reached record highs and news sources including the [Times](https://www.nytimes.com/2022/07/11/business/economy/rent-inflation-interest-rates.html) report that potential homeowners priced out of a similarly hot housing market may be driving up prices even further. A more recent Times [story](https://www.nytimes.com/2022/08/01/nyregion/why-its-so-hard-to-find-an-affordable-apartment-in-new-york.html) blames New York's restrictive zoning regulations for the city's failure to keep up with increasing demand. So how can NYC Renters save money in this hot rental market?


![Rental Index Chart](../../blob/main/images/rental_index.png)

![Rental Increase Chart](../../blob/main/images/yoy_increases.png)

## Business Problem

Under these circumstances, New York renters can benefit from a reliable forecast of the direction the housing market will take over the coming months. If rents are expected to shoot up even more, then the best strategy would be to sign a new lease sooner rather than later. Renters would also be advised to try to lock in a longer term lease, even if it means paying a higher monthly rent.

On the other hand, if rents are expected to fall or stabilize, renters should avoid longer term leases at high prices points and, if possible, wait until prices fall before looking for a new apartment.

## Data and Modeling

For this study I built a SARIMAX model to predict rents based on StreetEasy's NYC Rental Index, a metric that uses a repeat-price method to measure monthly change in median rent and that includes 185 index values from January 2007 through May 2022. Because data was only available for this relatively short date range, our model will necessarily have greater error than it would if it were based on a larger amount of amount.

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

## Use of First Differencing

Because the Rental Index is not a stationary series, I knew that a SARIMA model fit to the data would have to employ a single degree of differencing. In other words, the model needs to predict changes in rental price rather than prices, since these values are distributed around the same constant for the entirety of the series. Exogenous variables used to predict rental prices therefore need to be correlated with the first difference of the Rental Price, rather than the rental price itself.

## Exogenous Variables as Leading Indicators

While it seems likely that all of these factors have played some role in driving current rental prices, we can only rely on the predictive power of our exogenous variables if they are consistently correlated with the target during multiple periods. I therefore divided the data set into five three year periods and for each period, calculated the correlations of the differenced target variable with the first difference of each of the exogenous variables for a series of lags. My goal was to identify the exogenous variables that were consistent leading indicators for the target variable at a certain number of lags.

In order to rank the most consistently correlated variables, I calculated bootstrapped confidence intervals based on the each set of five correlations and selected the exogenous variables with the highest 2.5% confidence interval and the lowest 97.5% confidence interval.

Based on these criteria, I chose the following exogenous variables to include in SARIMAX modeling:

1. Manhattan Sale Price Index 16 Month Lag 1 Difference
2. Manhattan Sale Price Index 9 Month Lag 1 Difference
3. Federal Funds Rate 4 Month Lag 1 Difference
4. Wages and Salaries 6 Month Lag 1 Difference
5. Building Materials 14 Month Lag 1 Difference

## SARIMA Modeling

In selecting exogenous variables for modeling, I used correlations from all periods in the data set, from 2007 to 2022. This was necessary because without including all periods, it would have been unlikely that I'd be able to identify leading indicators for the dramatic rise and fall in rental prices during the COVID-19 period. However, this all foreclosed the possibility of including a holdout testing set.

In the absence of a testing set, SARIMAX models cannot be validated based on error, since error doesn't tell us whether or not our model is overfitting on the data. We therefore have to instead use an information criterion (either AIC or BIC) in order to measure the degree to which our model is being improved through the addition of new regressors.

As a baseline for my SARIMAX modeling, I first determined the best SARIMA model by AIC by testing out combinations of endogenous regressors and adding additional terms to decrease the correlation of residuals. By fitting a SARIMA model before adding exogenous regressors, I wanted to ensure that these exogenous regressors were actually improving the model rather than simply being fit to approximate it. In addition to recording the model's AIC based on its month-ahead predictions, I also calculated AIC values for 6 month and 12 month dynamic predictions. Since the purpose of this study was to predict rental values in advance, it was necessary to compare not just how well the model made step-ahead predictions, but also how will it made predictions over longer time periods.

Below we see AIC scores for our SARIMA model for Step-ahead predictions, 6 month dynamic forecasts and 12 month dynamic forecasts, as well as the model's heteroskedasticity, skew, and kurtosis values.

|              | (1, 1, 4)x(0, 1, 1, 12) SARIMA Model |
|--------------|--------------------------------------|
| 1 Month AIC  | 895.884                              |
| 6 Month AIC  | 1365.436                             |
| 12 Month AIC | 1595.835                             |
| Heterosk.    | 1.40.                                |
| Skew         | -0.30.                               |
| Kurtosis.    | 4.96.                                |

Looking at a plot showing the model's 12 month predictions and confidence intervals we can see that this model failed to predict the most recent rises in rent with true values falling outside the model's blue-shaded prediction intervals.

## SARIMAX Modeling

To choose the optimal SARIMAX model I added every possible combination of exogenous variables to the SARIMA model above and printed out AIC, 6 month AIC and 12 month AIC fore each model. While none of the SARIMAX models improved on the one-step-ahead or 6 month AIC score, two of them improved on the model's 12 month AIC score, in the best cased by 25 points. This model used three regressors:

1. Manhattan Sale Price Index 9 Month Lag 1 Difference
2. Federal Funds Rate 4 Month Lag 1 Difference
3. Building Materials 14 Month Lag 1 Difference

After adding an additional MA term to this model, its 12 month AIC was only 15 points better than that of the best SARIMA model, while one-step and 6 month ahead AIC were significantly worse. However, as we can see below it also had lower measures of heteroskedasticity, skew and kurtosis.


|              | (1, 1, 4)x(0, 1, 1, 12) SARIMA Model | (1, 1, 5)x(0, 1, 1, 12) + Exogs |
|--------------|--------------------------------------|---------------------------------|
| 1 Month AIC  | 895.884                              | 961.145                         |
| 6 Month AIC  | 1365.436                             | 1388.149                        |
| 12 Month AIC | 1595.835                             | 1580.282.                       |
| Heterosk.    | 1.40.                                | 1.07.                           |
| Skew         | -0.30.                               | 0.03                            |
| Kurtosis.    | 4.96.                                | 3.59                            |

Below we see these dynamic predictions plotted, along with true values for the rental index. While this model still shows significant error in its dynamic predictions, the most rent rise in rent did fall within the model's prediction intervals, a reflection of the model's lower AIC for 12 month dynamic predictions.


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

Second, because the target data has been smoothed, our model doesn't fully account for seasonal fluctuations in rental prices. However, because rents are typically around $80 lower during winter months as compared with summer months (as shown in the chart below), the prospect of seasonal fluctuations should if anything strengthen our recommendation to wait until fall or winter before signing a new lease.

![Seasonal Component of Rent](../../blob/main/images/seasonalcomp.png)

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
```
.
├── data : data used for modeling
├── images : images used in README
├── README.md : project information and repository structure
├── index.ipynb : jupyter notebook used for modeling
├── presentation.pdf : presentation for stakeholders
└── requirements.txt: requirements file
```
