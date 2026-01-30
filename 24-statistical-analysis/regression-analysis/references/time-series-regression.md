# Time Series Regression

## Overview

Time series regression models temporal data with trend, seasonality, and autocorrelation. This guide covers ARIMA, SARIMAX, and regression with time-based features using statsmodels.

## Key Concepts

**Components of Time Series:**
- **Trend**: Long-term increase or decrease
- **Seasonality**: Regular periodic fluctuations
- **Cycle**: Non-regular long-term fluctuations
- **Residual**: Random noise

**Stationarity**: Mean, variance, and autocorrelation are constant over time (required for ARIMA)

## Testing for Stationarity

```python
from statsmodels.tsa.stattools import adfuller
import numpy as np
import pandas as pd

# Generate non-stationary time series
np.random.seed(42)
t = np.arange(100)
trend = 0.5 * t
seasonal = 10 * np.sin(2 * np.pi * t / 12)
noise = np.random.normal(0, 3, 100)
y = trend + seasonal + noise

# Augmented Dickey-Fuller test
adf_result = adfuller(y)

print("ADF Statistic:", adf_result[0])
print("p-value:", adf_result[1])
print("Critical Values:")
for key, value in adf_result[4].items():
    print(f"  {key}: {value:.3f}")

if adf_result[1] < 0.05:
    print("\n✓ Series is stationary")
else:
    print("\n✗ Series is non-stationary - differencing needed")
```

## Making Series Stationary

```python
import pandas as pd
import numpy as np

# Create DataFrame
df = pd.DataFrame({'y': y}, index=pd.date_range('2020-01', periods=100, freq='M'))

# Method 1: Differencing
df['y_diff1'] = df['y'].diff()  # First difference
df['y_diff2'] = df['y'].diff().diff()  # Second difference

# Method 2: Log transformation (for exponential growth)
df['y_log'] = np.log(df['y'])
df['y_log_diff'] = df['y_log'].diff()

# Method 3: Detrending
from scipy import signal
df['y_detrended'] = signal.detrend(df['y'])

# Test each transformation
from statsmodels.tsa.stattools import adfuller

for col in ['y', 'y_diff1', 'y_log_diff', 'y_detrended']:
    adf_stat, p_value = adfuller(df[col].dropna())[:2]
    print(f"{col}: ADF={adf_stat:.3f}, p={p_value:.4f}")
```

## ARIMA Models

**ARIMA(p, d, q)** parameters:
- **p**: Autoregressive order (number of lagged values)
- **d**: Differencing order (number of differences to make stationary)
- **q**: Moving average order (number of lagged errors)

```python
from statsmodels.tsa.arima.model import ARIMA
import matplotlib.pyplot as plt

# Fit ARIMA model
model = ARIMA(df['y'], order=(1, 1, 1))  # ARIMA(1,1,1)
results = model.fit()

print(results.summary())

# Forecast future values
forecast = results.forecast(steps=12)
forecast_ci = results.get_forecast(steps=12).conf_int()

# Plot
plt.figure(figsize=(12, 6))
plt.plot(df.index, df['y'], label='Observed')
plt.plot(results.fittedvalues, label='Fitted', alpha=0.7)

forecast_index = pd.date_range(df.index[-1], periods=13, freq='M')[1:]
plt.plot(forecast_index, forecast, label='Forecast', color='red')
plt.fill_between(forecast_index, 
                 forecast_ci.iloc[:, 0], 
                 forecast_ci.iloc[:, 1], 
                 alpha=0.3, color='red')

plt.legend()
plt.title('ARIMA Forecast')
plt.xlabel('Date')
plt.ylabel('Value')
plt.grid(True, alpha=0.3)
plt.savefig('arima_forecast.png', dpi=300, bbox_inches='tight')
```

## SARIMA (Seasonal ARIMA)

For data with seasonal patterns:

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

# SARIMA(p,d,q)(P,D,Q,s)
# s = seasonal period (e.g., 12 for monthly data with yearly seasonality)

model = SARIMAX(df['y'], 
                order=(1, 1, 1),          # Non-seasonal
                seasonal_order=(1, 1, 1, 12))  # Seasonal
results = model.fit()

print(results.summary())

# Diagnostics
results.plot_diagnostics(figsize=(15, 10))
plt.savefig('sarima_diagnostics.png', dpi=300, bbox_inches='tight')
```

## Auto-selecting ARIMA parameters

```python
from pmdarima import auto_arima

# Auto-select best ARIMA parameters
auto_model = auto_arima(df['y'], 
                        start_p=0, start_q=0, max_p=5, max_q=5,
                        seasonal=True, m=12,  # m = seasonal period
                        d=None,  # Auto-determine differencing
                        trace=True,  # Print search progress
                        error_action='ignore',
                        suppress_warnings=True,
                        stepwise=True)

print(auto_model.summary())
print(f"\nBest model: ARIMA{auto_model.order} x {auto_model.seasonal_order}")
```

## SARIMAX with Exogenous Variables

Include external predictors:

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX
import numpy as np

# Generate exogenous variable (e.g., marketing spend)
exog = np.random.uniform(100, 200, 100)

# Fit SARIMAX
model = SARIMAX(df['y'], 
                exog=exog,
                order=(1, 1, 1),
                seasonal_order=(1, 1, 1, 12))
results = model.fit()

# Forecast with future exogenous values
future_exog = np.random.uniform(100, 200, 12)
forecast = results.forecast(steps=12, exog=future_exog)
```

## Regression with Time Features

Alternative to ARIMA for interpretable models:

```python
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler

# Create time-based features
df['month'] = df.index.month
df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
df['time_index'] = np.arange(len(df))

# Lagged features
df['y_lag1'] = df['y'].shift(1)
df['y_lag12'] = df['y'].shift(12)

# Rolling statistics
df['y_rolling_mean_3'] = df['y'].rolling(window=3).mean()
df['y_rolling_std_3'] = df['y'].rolling(window=3).std()

# Drop NaN rows
df_clean = df.dropna()

# Features and target
features = ['time_index', 'month_sin', 'month_cos', 'y_lag1', 'y_lag12']
X = df_clean[features]
y = df_clean['y']

# Fit regression
model = LinearRegression()
model.fit(X, y)

# Coefficients
coef_df = pd.DataFrame({
    'feature': features,
    'coefficient': model.coef_
}).sort_values('coefficient', key=abs, ascending=False)

print(coef_df)
print(f"\nR²: {model.score(X, y):.4f}")
```

## Vector Autoregression (VAR)

For multiple time series that influence each other:

```python
from statsmodels.tsa.vector_ar.var_model import VAR
import pandas as pd
import numpy as np

# Generate two correlated time series
np.random.seed(42)
n = 100
y1 = np.zeros(n)
y2 = np.zeros(n)

for t in range(1, n):
    y1[t] = 0.5 * y1[t-1] + 0.3 * y2[t-1] + np.random.normal(0, 1)
    y2[t] = 0.4 * y1[t-1] + 0.6 * y2[t-1] + np.random.normal(0, 1)

df_var = pd.DataFrame({'y1': y1, 'y2': y2})

# Fit VAR model
model = VAR(df_var)
results = model.fit(maxlags=5, ic='aic')  # Select lag by AIC

print(results.summary())

# Granger causality test
from statsmodels.tsa.stattools import grangercausalitytests

print("\nGranger Causality Tests:")
print("y1 → y2:")
grangercausalitytests(df_var[['y2', 'y1']], maxlag=5, verbose=True)

print("\ny2 → y1:")
grangercausalitytests(df_var[['y1', 'y2']], maxlag=5, verbose=True)

# Forecast
forecast = results.forecast(df_var.values[-5:], steps=10)
print("\nForecast:")
print(pd.DataFrame(forecast, columns=['y1', 'y2']))
```

## Prophet (Facebook's Time Series Tool)

Easy-to-use alternative for business forecasting:

```python
from prophet import Prophet
import pandas as pd

# Prepare data (Prophet requires 'ds' and 'y' columns)
df_prophet = pd.DataFrame({
    'ds': pd.date_range('2020-01', periods=100, freq='M'),
    'y': y
})

# Fit model
model = Prophet(yearly_seasonality=True, 
                weekly_seasonality=False,
                daily_seasonality=False)
model.fit(df_prophet)

# Create future dataframe
future = model.make_future_dataframe(periods=12, freq='M')

# Predict
forecast = model.predict(future)

# Plot
fig = model.plot(forecast)
plt.savefig('prophet_forecast.png', dpi=300, bbox_inches='tight')

# Plot components
fig_components = model.plot_components(forecast)
plt.savefig('prophet_components.png', dpi=300, bbox_inches='tight')

# Extract forecast values
forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']].tail(12)
```

## Model Evaluation

### Cross-Validation for Time Series

```python
from sklearn.model_selection import TimeSeriesSplit
from sklearn.metrics import mean_squared_error
import numpy as np

# Time series cross-validation
tscv = TimeSeriesSplit(n_splits=5)

mse_scores = []

for train_idx, test_idx in tscv.split(df):
    train_data = df.iloc[train_idx]['y']
    test_data = df.iloc[test_idx]['y']
    
    # Fit ARIMA on training set
    model = ARIMA(train_data, order=(1, 1, 1))
    results = model.fit()
    
    # Forecast test period
    forecast = results.forecast(steps=len(test_data))
    
    # Calculate MSE
    mse = mean_squared_error(test_data, forecast)
    mse_scores.append(mse)

print(f"CV RMSE: {np.sqrt(np.mean(mse_scores)):.4f} ± {np.sqrt(np.std(mse_scores)):.4f}")
```

### Forecast Accuracy Metrics

```python
from sklearn.metrics import mean_absolute_error, mean_squared_error
import numpy as np

def forecast_metrics(y_true, y_pred):
    """Calculate multiple forecast accuracy metrics"""
    mae = mean_absolute_error(y_true, y_pred)
    rmse = np.sqrt(mean_squared_error(y_true, y_pred))
    mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100
    
    # Mean absolute scaled error
    naive_forecast = y_true[:-1]
    mae_naive = mean_absolute_error(y_true[1:], naive_forecast)
    mase = mae / mae_naive
    
    return {
        'MAE': mae,
        'RMSE': rmse,
        'MAPE': mape,
        'MASE': mase
    }

metrics = forecast_metrics(test_data.values, forecast.values)
for metric, value in metrics.items():
    print(f"{metric}: {value:.4f}")
```

## Dealing with Missing Data

```python
import pandas as pd
import numpy as np

# Create series with missing values
df_missing = df.copy()
df_missing.loc[df_missing.sample(frac=0.1).index, 'y'] = np.nan

# Method 1: Forward fill
df_missing['y_ffill'] = df_missing['y'].fillna(method='ffill')

# Method 2: Interpolation
df_missing['y_interp'] = df_missing['y'].interpolate(method='linear')

# Method 3: Seasonal decomposition + imputation
from statsmodels.tsa.seasonal import seasonal_decompose

# Decompose non-missing parts
decomposition = seasonal_decompose(df_missing['y'].dropna(), 
                                   model='additive', 
                                   period=12)

# Impute using seasonal pattern
# (More sophisticated approach would use model-based imputation)
```

## Summary

**Model Selection Guide:**

| Use Case | Model |
|----------|-------|
| Univariate, no seasonality | ARIMA |
| Univariate, with seasonality | SARIMA |
| With external predictors | SARIMAX |
| Multiple related series | VAR |
| Business forecasting, interpretability | Prophet |
| Complex patterns, interpretability | Regression with time features |

**Key Checks:**
1. Test for stationarity (ADF test)
2. Check for seasonality (ACF/PACF plots)
3. Validate with time series cross-validation
4. Examine residual diagnostics
5. Compare to naive forecast baseline
