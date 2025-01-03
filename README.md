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
This project leverages the [IATA and ICAO Codes API](https://rapidapi.com/vacationist/api/iata-and-icao-codes) to develop an interactive R Shiny web application. The app enables users to search, analyze, and visualize global airport and airline data. By integrating API-driven data retrieval with advanced data manipulation and visualization techniques, the project provides an intuitive platform for exploring global airport information, featuring dynamic charts and interactive visuals to enhance user experience.



Installation
------------

Use the below API to be able to download the endpoints into your desired dataframe using [Rapid API](https://rapidapi.com/hub)

``` r
library(httr)
library(jsonlite)

# Function to get data from the API
get_airport_data <- function(query, api_key) {
  url <- paste0("https://vacationist.p.rapidapi.com/", query)
  response <- GET(url, add_headers(
    "X-RapidAPI-Key" = api_key,
    "X-RapidAPI-Host" = "vacationist.p.rapidapi.com"
  ))
  content <- content(response, as = "text")
  data <- fromJSON(content, flatten = TRUE)
  return(data)
}

# Example: Fetching airports
api_key <- "YOUR_API_KEY"
airport_data <- get_airport_data("airports", api_key)
print(airport_data)
```



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
