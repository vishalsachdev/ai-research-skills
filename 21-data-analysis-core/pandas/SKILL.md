---
name: pandas-data-analysis
description: Provides comprehensive data manipulation and analysis using pandas DataFrames. Use when working with tabular data, performing exploratory analysis, data cleaning, transformation, aggregation, or time series analysis in Python. Industry standard for data wrangling with 2M+ downloads per day.
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Analysis, pandas, DataFrame, Data Manipulation, Data Cleaning, Time Series, Exploratory Analysis, Python]
dependencies: [pandas>=2.0.0, numpy>=1.24.0]
---

# pandas - Data Analysis and Manipulation

## Quick start

pandas is the foundational library for data analysis in Python, providing high-performance DataFrame structures for tabular data manipulation.

**Installation**:
```bash
pip install pandas numpy
```

**Basic usage**:
```python
import pandas as pd
import numpy as np

# Read data
df = pd.read_csv('data.csv')

# Quick exploration
print(df.head())
print(df.info())
print(df.describe())

# Filter and select
filtered = df[df['value'] > 100]
selected = df[['column1', 'column2']]

# Group and aggregate
summary = df.groupby('category')['value'].agg(['mean', 'sum', 'count'])
```

## Common workflows

### Workflow 1: Data exploration and cleaning

Copy this checklist:

```
Data Cleaning Workflow:
- [ ] Step 1: Load and inspect data
- [ ] Step 2: Handle missing values
- [ ] Step 3: Fix data types
- [ ] Step 4: Remove duplicates
- [ ] Step 5: Validate results
```

**Step 1: Load and inspect data**

```python
import pandas as pd

# Read from various formats
df = pd.read_csv('data.csv')
# df = pd.read_excel('data.xlsx')
# df = pd.read_json('data.json')
# df = pd.read_parquet('data.parquet')

# Initial inspection
print(f"Shape: {df.shape}")
print(f"\nColumns: {df.columns.tolist()}")
print(f"\nData types:\n{df.dtypes}")
print(f"\nMissing values:\n{df.isnull().sum()}")
print(f"\nFirst few rows:\n{df.head()}")
```

**Step 2: Handle missing values**

```python
# Check missing data
missing_summary = df.isnull().sum()
print(f"Missing values:\n{missing_summary[missing_summary > 0]}")

# Strategy 1: Drop rows with any missing values
df_clean = df.dropna()

# Strategy 2: Drop rows where specific columns are missing
df_clean = df.dropna(subset=['important_column'])

# Strategy 3: Fill with specific value
df['numeric_col'] = df['numeric_col'].fillna(0)
df['category'] = df['category'].fillna('Unknown')

# Strategy 4: Fill with mean/median/mode
df['value'] = df['value'].fillna(df['value'].mean())
df['category'] = df['category'].fillna(df['category'].mode()[0])

# Strategy 5: Forward fill or backward fill
df['value'] = df['value'].ffill()  # pandas 2.0+ syntax
```

**Step 3: Fix data types**

```python
# Convert to appropriate types
df['date'] = pd.to_datetime(df['date'])
df['category'] = df['category'].astype('category')
df['value'] = pd.to_numeric(df['value'], errors='coerce')
df['is_active'] = df['is_active'].astype(bool)

# Handle string columns
df['text'] = df['text'].str.strip()
df['text'] = df['text'].str.lower()
```

**Step 4: Remove duplicates**

```python
# Check for duplicates
print(f"Duplicate rows: {df.duplicated().sum()}")

# Remove duplicates (keep first)
df = df.drop_duplicates()

# Remove duplicates based on specific columns
df = df.drop_duplicates(subset=['id'], keep='first')
```

**Step 5: Validate results**

```python
# Final validation
assert df.isnull().sum().sum() == 0, "Still have missing values"
assert df.duplicated().sum() == 0, "Still have duplicates"
print(f"\nCleaned data shape: {df.shape}")
print(f"Data types:\n{df.dtypes}")
```

### Workflow 2: Data aggregation and grouping

Copy this checklist:

```
Aggregation Workflow:
- [ ] Step 1: Group by categories
- [ ] Step 2: Apply aggregation functions
- [ ] Step 3: Create pivot tables
- [ ] Step 4: Export results
```

**Step 1: Group by categories**

```python
# Single column grouping
grouped = df.groupby('category')

# Multiple column grouping
grouped = df.groupby(['region', 'category'])

# Aggregate single column
result = df.groupby('category')['sales'].sum()

# Multiple aggregations
result = df.groupby('category').agg({
    'sales': ['sum', 'mean', 'count'],
    'quantity': ['sum', 'max'],
    'customer_id': 'nunique'
})
```

**Step 2: Apply aggregation functions**

```python
# Common aggregations
summary = df.groupby('category').agg({
    'value': ['mean', 'median', 'std', 'min', 'max'],
    'count': 'sum',
    'id': 'count'
})

# Custom aggregation function
def custom_agg(x):
    return x.quantile(0.95)

result = df.groupby('category')['value'].agg(custom_agg)

# Multiple functions including custom
result = df.groupby('category')['value'].agg([
    'mean',
    'std',
    ('p95', lambda x: x.quantile(0.95)),
    ('cv', lambda x: x.std() / x.mean())
])
```

**Step 3: Create pivot tables**

```python
# Basic pivot table
pivot = df.pivot_table(
    values='sales',
    index='region',
    columns='product',
    aggfunc='sum'
)

# Pivot with multiple aggregations
pivot = df.pivot_table(
    values='sales',
    index='region',
    columns='product',
    aggfunc=['sum', 'mean', 'count']
)

# Pivot with margins (totals)
pivot = df.pivot_table(
    values='sales',
    index='region',
    columns='product',
    aggfunc='sum',
    margins=True,
    margins_name='Total'
)
```

**Step 4: Export results**

```python
# Export to CSV
result.to_csv('analysis_results.csv')

# Export to Excel with multiple sheets
with pd.ExcelWriter('report.xlsx') as writer:
    summary.to_excel(writer, sheet_name='Summary')
    pivot.to_excel(writer, sheet_name='Pivot')
    df.to_excel(writer, sheet_name='Raw Data', index=False)
```

### Workflow 3: Time series analysis

Copy this checklist:

```
Time Series Workflow:
- [ ] Step 1: Parse dates and set index
- [ ] Step 2: Resample data
- [ ] Step 3: Calculate rolling statistics
- [ ] Step 4: Handle time-based operations
```

**Step 1: Parse dates and set index**

```python
# Parse dates during read
df = pd.read_csv('data.csv', parse_dates=['date'])

# Convert to datetime
df['date'] = pd.to_datetime(df['date'])

# Set datetime as index
df = df.set_index('date')
df = df.sort_index()
```

**Step 2: Resample data**

```python
# Resample to different frequencies
daily = df.resample('D').mean()      # Daily average
weekly = df.resample('W').sum()      # Weekly sum
monthly = df.resample('M').mean()    # Monthly average
quarterly = df.resample('Q').sum()   # Quarterly sum

# Multiple aggregations
resampled = df.resample('M').agg({
    'sales': 'sum',
    'quantity': 'sum',
    'price': 'mean'
})
```

**Step 3: Calculate rolling statistics**

```python
# Rolling mean (moving average)
df['ma_7d'] = df['value'].rolling(window=7).mean()
df['ma_30d'] = df['value'].rolling(window=30).mean()

# Rolling sum
df['rolling_sum'] = df['value'].rolling(window=7).sum()

# Rolling standard deviation
df['rolling_std'] = df['value'].rolling(window=7).std()

# Exponential weighted moving average
df['ewma'] = df['value'].ewm(span=7).mean()
```

**Step 4: Handle time-based operations**

```python
# Date components
df['year'] = df.index.year
df['month'] = df.index.month
df['day_of_week'] = df.index.dayofweek
df['quarter'] = df.index.quarter

# Lag and lead
df['lag_1'] = df['value'].shift(1)
df['lead_1'] = df['value'].shift(-1)

# Calculate differences
df['diff'] = df['value'].diff()
df['pct_change'] = df['value'].pct_change()
```

## When to use vs alternatives

**Use pandas when**:
- Working with tabular data (CSV, Excel, SQL tables)
- Need quick exploratory data analysis
- Data fits in memory (<100GB typical limit)
- Need rich ecosystem of data science tools
- Performing data cleaning and transformation

**Use alternatives when**:
- **Polars**: Need 5-10x faster performance, lazy evaluation, or better memory efficiency
- **Dask**: Data doesn't fit in memory, need distributed computing
- **PySpark**: Big data processing across cluster (100GB+)
- **SQL databases**: Data is already in database, need to share with non-Python users
- **NumPy**: Pure numerical computing without labels/indexes

## Common issues

### Issue 1: SettingWithCopyWarning

**Problem**: Warning when modifying DataFrame slice
```python
df[df['value'] > 10]['new_col'] = 100  # Warning!
```

**Solution**: Use `.loc` for assignment
```python
df.loc[df['value'] > 10, 'new_col'] = 100
```

### Issue 2: Memory usage with large datasets

**Problem**: DataFrame consuming too much memory

**Solutions**:
```python
# 1. Use categorical dtype for repeated strings
df['category'] = df['category'].astype('category')

# 2. Downcast numeric types
df['int_col'] = pd.to_numeric(df['int_col'], downcast='integer')
df['float_col'] = pd.to_numeric(df['float_col'], downcast='float')

# 3. Read in chunks
for chunk in pd.read_csv('large.csv', chunksize=10000):
    process(chunk)

# 4. Use compression
df.to_parquet('data.parquet', compression='snappy')
```

### Issue 3: Slow performance on large datasets

**Problem**: Operations taking too long

**Solutions**:
```python
# 1. Use vectorized operations instead of loops
# Bad
for idx, row in df.iterrows():
    df.loc[idx, 'result'] = row['a'] + row['b']

# Good
df['result'] = df['a'] + df['b']

# 2. Use query() for filtering
# Faster than boolean indexing for large DataFrames
filtered = df.query('value > 100 and category == "A"')

# 3. Use eval() for complex expressions
df.eval('result = a + b * c', inplace=True)

# 4. Consider switching to Polars for better performance
```

### Issue 4: Date parsing issues

**Problem**: Dates not parsing correctly

**Solutions**:
```python
# Specify date format explicitly
df['date'] = pd.to_datetime(df['date'], format='%Y-%m-%d')

# Handle mixed formats
df['date'] = pd.to_datetime(df['date'], format='mixed')

# Handle errors
df['date'] = pd.to_datetime(df['date'], errors='coerce')  # Invalid -> NaT

# For custom formats
df['date'] = pd.to_datetime(df['date'], format='%d/%m/%Y %H:%M:%S')
```

## Key concepts

**DataFrame vs Series**:
- DataFrame: 2D labeled data structure (like a table)
- Series: 1D labeled array (like a column)

**Index**:
- Row labels for accessing data
- Can be integer positions or custom labels (dates, IDs, etc.)
- Enables fast lookups and alignment

**Vectorization**:
- Apply operations to entire columns at once
- Much faster than Python loops
- Leverages NumPy's C-optimized code

**Method chaining**:
```python
result = (df
    .query('value > 100')
    .groupby('category')['sales']
    .sum()
    .sort_values(ascending=False)
    .head(10)
)
```

## Advanced reference

For detailed documentation on advanced topics:
- **API Reference**: See [references/api.md](references/api.md)
- **Performance Tips**: See [references/performance.md](references/performance.md)
- **Common Patterns**: See [references/patterns.md](references/patterns.md)
