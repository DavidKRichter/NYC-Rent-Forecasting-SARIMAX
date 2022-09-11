# Using SARIMAX Modeling to Predict the New York Rental Market

2022 is a hard year to be a renter in New York City. Rental prices have reached record highs and news sources including the [Times](https://www.nytimes.com/2022/07/11/business/economy/rent-inflation-interest-rates.html) report that potential homeowners priced out of a similarly hot housing market may be driving up prices even further. A more recent Times [story](https://www.nytimes.com/2022/08/01/nyregion/why-its-so-hard-to-find-an-affordable-apartment-in-new-york.html) blames New York's restrictive zoning regulations for the city's failure to keep up with increasing demand. So how can NYC Renters save money in this hot rental market?


![Rental Index Chart](../../blob/main/images/rental_index.png)

![Rental Increase Chart](../../blob/main/images/monthly_change.png)

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

![Best SARIMA](../../blob/main/images/BestSARIMAAIC.png)

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

![Best SARIMAX](../../blob/main/images/BestSARIMAXAIC12.png)


## Using our Model for Forecasting

In order to make forecasts through July 2023, I needed to extrapolate additional values for two of the exogenous predictors: Federal Funds Rate 4 Month Lag 1 Difference and Manhattan Sale Price Index 9 Month Lag 1 Difference. To do this, I determined minimum and maximum values for the past year of data and generated two sets of extrapolated data: one that would minimize the predictions made by the SARIMAX model and one that out maximize the predictions made by the model. However, because the exogenous regressors only made small contributions to the model, differences between their predictions were not significantly different. 

Below we see the model's forecasts through July 2023:

![Forecast](../../blob/main/images/forecast.png)

## Recommendations and Caveats

Assuming the mean prediction to be the correct one, we would advise renters to wait until the fall or winter to sign a new lease. While our model suggests a strong likelihood that rents could rise slightly during this period rather than fall, we're unlikely to see rises comparable to those we've seen in the past six months.

That said, we need to account for some of the limitations of our SARIMAX model. 

Based on the model's mean prediction, we should expect a 14% year over year increase in NYC rental prices between July 2022 and July 2023. This means that renters should try to lock in new leasers sooner rather than later. It also means that if they have the opportunity to sign a multiyear lease, they should not agree to pay more than 14% more in rent than they would for a single year lease for the same apartment.

While these predictions are the best available given the models we've examined, our best SARIMAX model still has a considerable amount of forecasting error and the prediction intervals for its forecasts are consequently quite large. It's possible that testing out the use of other regressors or interactions between regressors might improve our model further. However, it's also possible that the limited size of this time series and the complexity of the factors influencing it may make it impossible to improve our model significantly.

## Sources

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
