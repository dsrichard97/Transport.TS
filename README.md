# Transport Trend Analysis <a href='https://dsrichard97.github.io/Transport.TS/'><img src='photos/Transport.png' style="float: right; width: 300px; height: auto; margin-left: 20px; margin-top: 10px;" /></a>


### Quick Links
<ul style="list-style: none; padding: 0;">
  <li><a href="#overview" style="text-decoration: none; color: #007acc;">Overview</a></li>
  <li><a href="#business-case" style="text-decoration: none; color: #007acc;">Business-Case</a></li>
  <li><a href="#aboutdata" style="text-decoration: none; color: #007acc;">About Data</a></li>
  <li><a href="#install" style="text-decoration: none; color: #007acc;">Installation</a></li>
  <li><a href="#usage" style="text-decoration: none; color: #007acc;">Usage</a></li>
  <li><a href="https://example.com" style="text-decoration: none; color: #007acc;" target="_blank">Github Repo</a></li>
</ul>

<!-- badges: start -->

[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/TSstudio)](https://cran.r-project.org/package=TSstudio)
[![Total Downloads](https://cranlogs.r-pkg.org/badges/grand-total/TSstudio)](https://cran.r-project.org/package=TSstudio)
[![Downloads](http://cranlogs.r-pkg.org/badges/TSstudio)](https://cran.r-project.org/package=TSstudio)
[![Lifecycle:Retired](https://img.shields.io/badge/Lifecycle-Retired-d45500)](https://cran.r-project.org/package=TSstudio)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/license/mit/)

<!-- badges: end -->

Overview
------------

The [RSocrata](https://dev.socrata.com/) package provides a set of tools tha tallows you open data from governments, non-profits, and NGOs around the world. The goal of this project is to showcase how fun it is to get data from online open sources and repackage data. This project focuses on the utility of using time series data, [tableau](https://www.tableau.com/), and [forecast](https://CRAN.R-project.org/package=forecast).

More information available on the package [RSocrata](https://dev.socrata.com/) or for more information about the transportatin developer site please visit [US Department of Transportation](https://www.transportation.gov/developer).

Overview
------------
## *Hypothetical Business Case*
### Stakeholder Ask
### Stakeholder Requests

The stakeholders have outlined the following key deliverables:

### Goals of the Project
- **Demonstrate Feasibility:** Show how open data can be leveraged to address real-world business problems.
- **Empower Decision-Making:** Provide stakeholders with data-driven forecasts and visualizations.
- **Promote Open Source Tools:** Highlight the ease and effectiveness of using open-source tools like RSocrata and forecast for business applications.

### Expected Deliverables
1. A cleaned and processed dataset pulled from an open data source using the RSocrata package.
2. Forecasted time series trends using the `forecast` package.
3. A Tableau dashboard that visualizes both historical and forecasted data.
4. A final report summarizing insights, recommendations, and methodologies.

### Technical Stack
- **Data Retrieval:** [RSocrata](https://dev.socrata.com/)
- **Analysis and Forecasting:** [forecast](https://CRAN.R-project.org/package=forecast) in R
- **Visualization:** [Tableau](https://www.tableau.com/)
- **Programming Language:** R

---

# Next Steps
1. Identify an open data source relevant to the business case (e.g., city traffic data, weather patterns, or retail sales).
2. Set up the RSocrata package and retrieve the data.
3. Perform data cleaning and exploratory data analysis (EDA).
4. Develop and validate a time series forecasting model.
5. Create a Tableau dashboard with visualizations.
6. Present findings and actionable recommendations to stakeholders.


Installation
------------

Use the below API to be able to download the endpoints into your desired dataframe

``` r
## Install the required package with:
## install.packages("RSocrata")

library("RSocrata")

## API Token Acess from the data.transportation.gov site
df <- read.socrata(
  "https://data.transportation.gov/resource/xgub-n9bw.json",
  app_token = "YOURAPPTOKENHERE",
  email     = "user@example.com",
  password  = "fakepassword"
)
```

or for more detailed informaiton from the [SODA DEVLOPER SITE](https://dev.socrata.com/foundry/data.transportation.gov/xgub-n9bw):



Usage
-----

### Plotting time series data

``` r
library(TSstudio)
data(USgas)

# Ploting time series object
ts_plot(USgas, 
        title = "US Monthly Natural Gas Consumption",
        Ytitle = "Billion Cubic Feet")
```
<img src="man/figures/USgas_plot.png" width="100%" />

### Seasonality analysis
``` r
# Seasonal plot
ts_seasonal(USgas, type = "all")
```
<img src="man/figures/USgas_seasonal.png" width="100%" />

``` r

# Heatmap plot

ts_heatmap(USgas)
```
<img src="man/figures/USgas_heatmap.png" width="100%" />


### Correlation analysis

``` r
# ACF and PACF plots
ts_cor(USgas, lag.max = 60)
```
<img src="man/figures/USgas_acf.png" width="100%" />

``` r
# Lags plot
ts_lags(USgas, lags = 1:12)
```

<img src="man/figures/USgas_lags.png" width="100%" />

``` r
# Seasonal lags plot
ts_lags(USgas, lags = c(12, 24, 36, 48))
```
<img src="man/figures/USgas_lags2.png" width="100%" />

### Training forecasting models

``` r
# Forecasting applications
# Setting training and testing partitions
USgas_s <- ts_split(ts.obj = USgas, sample.out = 12)
train <- USgas_s$train
test <- USgas_s$test

# Forecasting with auto.arima
library(forecast)
md <- auto.arima(train)
fc <- forecast(md, h = 12)

# Plotting actual vs. fitted and forecasted
test_forecast(actual = USgas, forecast.obj = fc, test = test)
```
<img src="man/figures/USgas_test_f.png" width="100%" />

``` r
# Plotting the forecast 
plot_forecast(fc)
```
<img src="man/figures/USgas_forecast.png" width="100%" />

``` r
# Run horse race between multiple models
methods <- list(ets1 = list(method = "ets",
                            method_arg = list(opt.crit = "lik"),
                            notes = "ETS model with opt.crit = lik"),
                ets2 = list(method = "ets",
                            method_arg = list(opt.crit = "amse"),
                            notes = "ETS model with opt.crit = amse"),
                arima1 = list(method = "arima",
                              method_arg = list(order = c(2,1,0)),
                              notes = "ARIMA(2,1,0)"),
                arima2 = list(method = "arima",
                              method_arg = list(order = c(2,1,2),
                                                seasonal = list(order = c(1,1,1))),
                              notes = "SARIMA(2,1,2)(1,1,1)"),
                hw = list(method = "HoltWinters",
                          method_arg = NULL,
                          notes = "HoltWinters Model"),
                tslm = list(method = "tslm",
                            method_arg = list(formula = input ~ trend + season),
                            notes = "tslm model with trend and seasonal components"))
# Training the models with backtesting
md <- train_model(input = USgas,
                  methods = methods,
                  train_method = list(partitions = 6, 
                                      sample.out = 12, 
                                      space = 3),
                  horizon = 12,
                  error = "MAPE")
# A tibble: 6 x 7
  model_id model       notes                                         avg_mape avg_rmse `avg_coverage_80%` `avg_coverage_95%`
  <chr>    <chr>       <chr>                                            <dbl>    <dbl>              <dbl>              <dbl>
1 arima2   arima       SARIMA(2,1,2)(1,1,1)                            0.0557     167.              0.583              0.806
2 hw       HoltWinters HoltWinters Model                               0.0563     163.              0.736              0.889
3 ets1     ets         ETS model with opt.crit = lik                   0.0611     172.              0.681              0.903
4 ets2     ets         ETS model with opt.crit = amse                  0.0666     186.              0.458              0.833
5 tslm     tslm        tslm model with trend and seasonal components   0.0767     220.              0.417              0.667
6 arima1   arima       ARIMA(2,1,0)                                    0.188      598.              0.875              0.958

```


``` r
# Plot the performance of the different models on the testing partitions
plot_model(md)
```

<img src="man/figures/plot_model.gif" width="100%" />


``` r
# Holt-Winters tunning parameters with grid search
hw_grid <- ts_grid(USgas, 
                   model = "HoltWinters",
                   periods = 6,
                   window_space = 6,
                   window_test = 12,
                   hyper_params = list(alpha = seq(0,1,0.1),
                                       beta = seq(0,1,0.1),
                                       gamma = seq(0,1,0.1)))
                                       
plot_grid(hw_grid, type = "3D")
```

<img src="man/figures/hw_grid.png" width="100%" />
