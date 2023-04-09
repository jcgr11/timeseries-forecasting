# Timerseries forecasting with Meta's Prophet algorithm
#### The base regression formula used by the model is a decomposable time series model (Harvey & Peters 1990) with three main model components: trend, seasonality, and holidays. The base regression formula for the prophet algorithm is: 
<p align="center">
$$y(t) = g(t) + s(t) + h(t) + ε<sub>t</sub>$$
</p>

* $g(t)$ is the trend function which models non-periodic changes in the value of the
time series, $s(t)$ represents periodic changes (e.g., weekly and yearly seasonality), and
$h(t)$ represents the effects of holidays which occur on potentially irregular schedules over
one or more days. The error term $ε<sub>t</sub>$ represents any idiosyncratic changes which are not
accommodated by the model.
#### References
* https://research.facebook.com/blog/2017/2/prophet-forecasting-at-scale/#:~:text=At%20its%20core%2C%20the%20Prophet,selecting%20changepoints%20from%20the%20data.
* https://facebook.github.io/prophet/docs/quick_start.html#python-api
* https://peerj.com/preprints/3190/
* Harvey, A. & Peters, S. (1990), ‘Estimation procedures for structural time series models’, Journal of Forecasting 9, 89–108.
* Lewinson, E. (2022). Python for finance cookbook: Over 80 powerful recipes for effective financial data analysis. Packt Publishing.

## Set up:
* Download necessary packages (see requirements file)
* Download historical daily price dataframe using yf.download() functon for defined ticker, start, and end date. Both attempts generated a balanced fit trendline at approximately 3 years of data for the 2 two securities tested. More or less data seemed display imbalanced nature (over/under fit).
* To standardize the dataframe output for easier manipulation the time stamp was removed. The Date index was moved to the first column of the dataframe using df.reset_index(). The timestamp was redacted for the date values in new "Date" column
* The the column names were adjusted to match the variables used in the prophet algorithm "ds" = date series and "y" = value series being forecasted.

## Defining your training and testing sets:
* The train_indices variable is used to define a boolean array called by comparing the date values in the ds column of the axp dataframe with the date string "2023-04-02". The result is True for the rows with dates earlier than "2023-04-02" and False for the rows with dates on or after "2023-04-02".
* The df_train variable pulls the rows from the axp dataframe where the train_indices array is True, (rows with dates earlier than "2023-04-02"). 
* The .loc function is used to select rows by boolean indexing. The resulting dataframe is assigned to df_train. 
* The .dropna() function is then used to remove any rows with missing values from the df_train dataframe.
* the df_variable selects the rows from the axp dataframe where the train_indices array is False, meaning the rows have dates on or after "2023-04-02". The ~ symbol inverts the boolean values in train_indices, so that False becomes True and vice versa. The resulting dataframe is assigned to df_test. The .reset_index(drop=True) function is used to reset the index of df_test so that it starts from 0 and drops the old index.

## Setting Prophet's parameters:
* Create a Prophet object and set the changepoint_range parameter to 0.9. The changepoint_range parameter controls the flexibility of the trend, with higher values indicating a more flexible trend.
* Add US holidays to the model as an additional regressor using the .add_country_holidays() function from the Prophet package. This can help capture any seasonal variations in the data that are not accounted for by the model's built-in seasonality and account for non-trading days outside of weekends.
* Since the data being used is daily price data a weekly seasonality component is added to the model to optimize it for the granularity of the price data. The period parameter then specifies the length of the seasonality in days (in this case, 365.25/7 = 52.18 weeks) and the fourier_order parameter controls the flexibility of the seasonality, with higher values indicating a more flexible seasonality.

## Begin the analysis:
* The original timeseries is plotted using the .plot() function.
* A new dataframe called df_future is created with 120 periods (i.e., 120 days) of future dates, with business day frequency (i.e., excluding weekends and holidays). Using the .make_future_dataframe() function which is a Prophet function that generates a dataframe with future dates based on the frequency specified.
* The Prophet model is then called to make predictions for the future dates in the df_future dataframe. The .predict() function used generates predictions based on the fitted model and the future dates provided.
* The .plot() function is called to generate a time series plot of the predicted values.
* The pd.date_range() function generates a sequence of dates with monthly frequency from the first date in the historical data to the last date in the df_future dataframe. The strftime() function formats the dates to show only the year and month. The rotation=45 argument rotates the x-axis tick labels by 45 degrees for better readability.
* peak at the df_pred columns to ensure the needed attributes are present.
* The df_pred dataframe is called again but this time re-assigned to the variable name fig which is then used in the add_changepoints_to_plot function to display vertical lines at points which the model has identified as changepoints in the timeseries. The fig.gca() function gets the current axis of the figure, and the prophet and df_pred arguments provide the fitted Prophet model and the predicted values, respectively.
* .plot_components() is called on the predefined "prophet" variable with the df_pred as the first argument to decompose the timeseries into its trend, seasonality, and noise components. This decomposition helps understand the underlying patterns and variations in the time series, and can be used to identify potential anomalies or irregularities in the data.

## Prepare your new data after further analyzing the timeseries for plotting you predicted forecast: 
* A py list of the columns you want to keep and assigned to the selected_cols variables
* the df_pred variable is redefined but selected all rows and only the columns stored in the selected_cols list variable. The index is reset and dropped to then used set_index on the ds column so the dateseries is now the index.
* the same preocess mentioned above to remove the time stamp from the date series is ran again on the redefined df_pred dataframe variable. 
* The original timeseries dataframe is merged with the predictions generated by the prophet algorithm stored in the df_pred dataframe using the pandas .merge() funciton in which the original dataframe axp and predicted values in the df_pred dataframe. the on= and how= arguments are set to ds and outer join to join the two dataframes on the dataseries. This is stored in the combined_df variable. 
* The combined_df columns are renamed for plotting and the ds column (now called Date) is reset as the index for the combined dataframe. 
* A figure with one subplot using subplots() function of the matplotlib.pyplot module is created. The figsize is set to match previous plots in the analysis.
* The columns to be plotted are save to a list for easier calling in the following steps (Actual, Predicted, Upper Bound, and Lower Bound). 
* The sns.lineplot() function from seaborn is used to create the line plot. The data parameter specifies the data to be plotted in this case the combined dataframe (combined_df), and the ax parameter specifies the subplot to be used.
* The .fill_between function is called to fill the speace between the upper and lower bounds of the plot. 
* The last lines of this scripts lines of code format the x-axis ticks, set the x-axis limits, add gridlines, and set the title and axis labels of the plot.
* The resulting plot shows the actual timeseries (blue line) and predicted timeseries (orange line) values of the time series, along with the upper (Green boundary) and lower (Red boundary) bounds of the uncertainty interval (shaded blue area). 
* The forecasted_value variable holds the last value (predicted stock price on the last date of the prediction) which is located using .iloc[-1, 1] to located the value at the intersection of the last row and second column of the dataframe.
