![cover_photo](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/Super%20Cue%20Classic%20Tea%20n%20Milk%20Tea.jpg)
# Super Cue Boba Shop

*Boba shops have grown exponentially in the bay area since the first boba shop opened in 1997. There are now countless boba shops through out the bay area, and the competition has become very intense for boba shop owners. In this project, I will use timeseries analysis to find out the optimal store hours for Super Cue Cafe's specific store location.*

## 1. Data

Super Cue cafe has started to use Clover POS (point of sales) system since 2013. And this modern POS system stores numerous information, and contains free and pay to access apps for the owners.

As the ex-owner of Super Cue Cafes. I have kept hourly sales of my store locations for a little more than 2 years. Which, I manually gathered the data from the clover's analytics app, and put them into excel files to do analysis as a small business owner.

## 2. Method

Since the data consist of only time and sales, it is certainly a univariable data, and is defintely best suited as a time series model. However, most of the time series dataset have seasonalities, here are the models that works well with time series dataset that has multiple seasonalities:

1. **SARIMAX:** Seasonal AutoRegressive Integrated Moving Averages, Autoregression explores any dependent relationship between an observation and some number of lagged variables. Integrated to make time-series stationery by subtracting or differencing an observation from observation at the previous time step of the same time series. Moving Average explores the relationship between an observation and a residual error by application of moving average to lagged observations, with any given time window. X for exogenous variables. This model is well suited for data sets with multiple seasonalities, and with more than just time as independent variable.

2. **Auto_arima:** Automatically discover the optimal order for an ARIMA model, so it is similar to SARIMAX, but it does not depend on the PACF/Auto-Correlation (manual computation of differencing), but instead, it conducts differencing tests (i.e., Kwiatkowski–Phillips–Schmidt–Shin, Augmented Dickey-Fuller or Phillips–Perron) on its own to determine the order of differencing.

3. **TBATS:** T for trigonometric regressors to model multiple-seasonalities, B for Box-Cox transformations, A for ARMA errors, T for trend, S for seasonality; another model well suited for data sets with multiple seasonalities.


## 3. Data Cleaning

[Data Wrangling](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Capstone%202%20Data%20Wrangling.ipynb)

As I mentioned earlier, the data I personally collected consists of time (date and hour) and sales.

Problems that I had to solve for each sheet

* **Problem 1:** I begin by importing one sheet (1 month of data), and realized there are additional week each month, where that additional week is the month's hourly average by day of week and hour of day.

* **Problem 2:** Even though columns are Day of Weeks, there's an additional column that's the Average. ANd there are also rows that's the sum of the day's AM sales, PM sales and Total sales.

* **Problem 3:** The format isn't time series ready, since the column of the data set is Day of week, while index consists of date (e.g. 01-01-2000) and time (e.g. 11:00).

* **Problem 4:** Sometimes a sheet can have 4 or 5 weeks of data due to usually 4 weeks of data per sheet, while the accumulated dates may result in a month with 5 weeks of data once per 3 to 4 months.

* **Solution:** I defined a function that will clean the sheet with problems mentioned above by: remove empty rows, remove the unwanted columns/rows, break downeach weeks data to such way then combine all the weeks (except last week, since its' the average) data to one. Transpose it, so the data set has index as date and column as hour of day.

Problems after cleaning each sheet

* **Problem 1:** There are missing values or outliers

* **Problem 2:** Need to make them into 1 big dataframe or a dataframe per year, such that the index is time (date and time!) and column is sales for that hour.

* **Solution:** I concat all the datasheets to one dataframe, then checked its standard deviation to find out the outliers and replace them and appropriate hours with that year's average. Then I used this data set where the index is date, and column is hours and Daily sum, to create a Dataframe that's more suitable for time series, where index = date and time while column is sales of that hour; I filled in 0 for hours that the store was closed.

## 4. EDA

[EDA](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Capstone%202%20EDA.ipynb)

* By comparing the sales between 2016 and 2017 in day of week and hour of day, we can see that both of the years have similar results, suggesting daily and weekly seasonality.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/EDA%20DoW.png)
![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/EDA%20hourly.png)

## 5. Algorithms & Machine Learning

Due to having multiple seasonalities, it may be a good idea to breakdown the datasets into different time frequencies. So I have two modeling sections

[Modeling (day)](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Capstone%202%20modeling%20(day).ipynb)

I tried models that aren't suitable for multi-seasonality datasets as well (ARIMA, SARIMA, Holtz). But ultimately, the models that were able to capture multiple seasonalities were:
TBATS, SARIMAX, Auto_arima, which were the method mentioned earlier that are recommended for time series with multiple seasonalities.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/Model%20Metric%20Day.png)

>***NOTE:** I choose ForeMAE as the accuracy metric over other metric scoring, because the dataset isn't smoothed, and is likely to have outliers (due to other variables not included in data, such as temperature and holiday) and RMSE score will be influenced more than MAE score from outliers. Then ForeMAE is calculated with entire year as testing data, and thus would capture the yearly seasonality much better than MAE which only accounted for one season (3 months) of testing data.

[Modeling (hour)](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Capstone%202%20modeling%20(hour).ipynb)

From the Model (day) experiences, TBATS and auto_arima had the best results, so the modeling for hourly data focused on trying different exogenous variable settings for auto_arima. And using predictions from TBATS to multiply ratios calculated in different ways.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/Model%20Score%20Metrics.png)

>***NOTE:** Due to having 24 times (24 hours a day) more data than daily sales, the fitting time became much longer, and concerning. And thus the fitting time became a big part for the scoring metric as well. Even though TBATSxR has N/A for its fitting time, it is definitely less time consuming than auto_arima models in both prepping (exog variables is needed for auto_arima, but not TBATS) and fitting the model.

**WINNER: TBATSxR**

This model uses the results from TBATS then multiply the reslt by a ratio calculated by another model or method, in this case, the ratio was calculated based on training data's average daily sales by month. And thus create a yearly seasonality effect based on month.


## 6. Which Dataset to choose and how long for training data?

It is recommened to have 3 years or more of data for a proper time series analysis. Which trains the first 2 years and test on the 3rd year. However, we only have 2 years of data! Also, I have already tried several training/testing periods, because auto_arima with exog variables may consum a lot of memeory, the hourly models were mainly train=2016, and predict the 2017 to compare with the test. And from the result of training/testing various models, we have concluded earlier that TBATSxR is most suited for this project. And fortunately, TBATS doesn't take as much time and memory as auto_arima, so it was able to train the whole two years of data.
But as for the missing 3rd year's data, I used the average of 2016 and 2017's data based on hour and day of week starting with Monday's 00:00.

## 7. Predictions

[Finalized codes](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Capstone%202%20Finalized%20Codes.ipynb)

In the finalized codes, it contains the codes on how I got the results for model(day) and model(hour), then the rest of it was used to make predictions for 2018. The following is the test vs prediciton for 2018.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/TBATSxTR_pred2018%20test%20vs%20prediction.png)

## 8. Recommendation for Super Cue's store hours

* There were only two specific store hours where the sales is predicted to be having less than $40 in revenue, one of them is at 11:00AM, but I won't recommend to close during these hours, because the predicted value is very close to 40.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/less%20than%2040%20at%2011.png)

* The other predicted store hours where the revenue is less than 40 is at 11:00PM, which is on Friday and Saturday only, but as you can see from the counts (104), that's every accountable Friday & Saturday for 2018 (52 weeks per year times 2 days in a week), and with max predicted value of 13.05, I highly recommened the store to be closed at 11PM.

![](https://github.com/tc18fwd/SpringBoard/blob/master/Capstone%20Two/Readme/less%20than%2040%20at%2023.png)
