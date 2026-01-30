# pandas API Reference

## Essential DataFrame Methods

### Data Input/Output

**Reading Data**:
```python
pd.read_csv(filepath, sep=',', header=0, names=None, index_col=None, usecols=None, dtype=None, parse_dates=False, chunksize=None)
pd.read_excel(filepath, sheet_name=0, header=0, names=None, index_col=None, usecols=None, dtype=None)
pd.read_json(filepath, orient='columns', lines=False)
pd.read_parquet(filepath, engine='auto', columns=None)
pd.read_sql(sql, con, index_col=None, coerce_float=True, params=None, parse_dates=None, chunksize=None)
pd.read_html(url_or_html, match='.+', flavor=None, header=None, index_col=None, skiprows=None, attrs=None, parse_dates=False)
```

**Writing Data**:
```python
df.to_csv(filepath, sep=',', index=True, header=True, mode='w', encoding='utf-8')
df.to_excel(filepath, sheet_name='Sheet1', index=True, header=True, engine=None)
df.to_json(filepath, orient='columns', lines=False, indent=None)
df.to_parquet(filepath, engine='auto', compression='snappy', index=None)
df.to_sql(name, con, schema=None, if_exists='fail', index=True, index_label=None, chunksize=None, dtype=None, method=None)
df.to_html(filepath, index=True, classes=None, header=True, border=None)
```

### Data Inspection

```python
df.head(n=5)           # First n rows
df.tail(n=5)           # Last n rows
df.info()              # Data types, non-null counts, memory usage
df.describe()          # Statistical summary
df.shape               # (rows, columns)
df.columns             # Column names
df.dtypes              # Data types
df.index               # Index information
df.memory_usage()      # Memory consumption per column
df.sample(n=10)        # Random sample of n rows
```

### Selection and Filtering

```python
# Column selection
df['column']           # Single column (Series)
df[['col1', 'col2']]  # Multiple columns (DataFrame)

# Row selection by position
df.iloc[0]             # First row
df.iloc[0:5]          # First 5 rows
df.iloc[:, 0:3]       # All rows, first 3 columns

# Row selection by label
df.loc['label']        # Row with index label
df.loc[:, 'col1']     # All rows, specific column
df.loc[df['value'] > 100, ['col1', 'col2']]  # Boolean + column selection

# Boolean indexing
df[df['value'] > 100]
df[(df['value'] > 100) & (df['category'] == 'A')]
df.query('value > 100 and category == "A"')

# Conditional selection
df.where(df > 0)       # Keep values where condition is True
df.mask(df > 0)        # Replace values where condition is True
```

### Data Cleaning

```python
# Missing values
df.isna()              # Boolean mask of missing values
df.notna()             # Boolean mask of non-missing values
df.isnull()            # Alias for isna()
df.notnull()           # Alias for notna()
df.dropna()            # Drop rows with any missing values
df.dropna(subset=['col1'])  # Drop rows where specific columns are missing
df.fillna(value)       # Fill missing values
df.ffill()             # Forward fill
df.bfill()             # Backward fill
df.interpolate()       # Interpolate missing values

# Duplicates
df.duplicated()        # Boolean mask of duplicate rows
df.drop_duplicates()   # Remove duplicate rows
df.drop_duplicates(subset=['col1'], keep='first')

# Data type conversion
df.astype({'col': 'int64'})
pd.to_datetime(df['date'])
pd.to_numeric(df['value'], errors='coerce')
df['category'].astype('category')
```

### Aggregation and Grouping

```python
# Basic aggregation
df.sum()               # Column-wise sum
df.mean()              # Column-wise mean
df.median()            # Column-wise median
df.std()               # Standard deviation
df.var()               # Variance
df.min()               # Minimum
df.max()               # Maximum
df.count()             # Count non-null values
df.nunique()           # Count unique values
df.value_counts()      # Frequency counts

# GroupBy operations
df.groupby('category').sum()
df.groupby(['col1', 'col2']).agg(['sum', 'mean', 'count'])
df.groupby('category').agg({
    'value1': 'sum',
    'value2': ['mean', 'std']
})

# Window functions
df.groupby('category')['value'].cumsum()
df.groupby('category')['value'].shift(1)
df.groupby('category')['value'].rank()
```

### Merging and Joining

```python
# Concatenate
pd.concat([df1, df2], axis=0)  # Vertical (row-wise)
pd.concat([df1, df2], axis=1)  # Horizontal (column-wise)

# Merge (SQL-style joins)
pd.merge(df1, df2, on='key')                    # Inner join
pd.merge(df1, df2, on='key', how='left')        # Left join
pd.merge(df1, df2, on='key', how='right')       # Right join
pd.merge(df1, df2, on='key', how='outer')       # Full outer join
pd.merge(df1, df2, left_on='key1', right_on='key2')

# Join (index-based)
df1.join(df2, how='left')
df1.join(df2, on='key')
```

### Reshaping

```python
# Pivot
df.pivot(index='row_col', columns='col_col', values='value_col')
df.pivot_table(values='value', index='row', columns='col', aggfunc='mean')

# Melt (unpivot)
pd.melt(df, id_vars=['id'], value_vars=['col1', 'col2'])

# Stack/Unstack
df.stack()             # Pivot column level to row level
df.unstack()           # Pivot row level to column level

# Transpose
df.T                   # Transpose rows and columns
```

### Sorting

```python
df.sort_values('column')
df.sort_values(['col1', 'col2'], ascending=[True, False])
df.sort_index()
df.nlargest(n=5, columns='value')
df.nsmallest(n=5, columns='value')
```

### String Methods

```python
# Access with .str accessor
df['text'].str.lower()
df['text'].str.upper()
df['text'].str.strip()
df['text'].str.replace('old', 'new')
df['text'].str.contains('pattern')
df['text'].str.startswith('prefix')
df['text'].str.endswith('suffix')
df['text'].str.split(',')
df['text'].str.len()
df['text'].str.extract(r'(\d+)')  # Regex extraction
```

### DateTime Methods

```python
# Access with .dt accessor
df['date'].dt.year
df['date'].dt.month
df['date'].dt.day
df['date'].dt.hour
df['date'].dt.dayofweek
df['date'].dt.quarter
df['date'].dt.is_month_end
df['date'].dt.strftime('%Y-%m-%d')
```

### Apply and Map

```python
# Apply function to each element
df['column'].apply(lambda x: x * 2)
df.apply(lambda row: row['a'] + row['b'], axis=1)

# Map values
df['category'].map({'A': 1, 'B': 2, 'C': 3})
df['value'].map(lambda x: x ** 2)

# applymap (deprecated, use map)
df.map(lambda x: x.upper())  # pandas 2.1+
```

## Time Series Specific

```python
# Resampling
df.resample('D').mean()      # Daily
df.resample('W').sum()       # Weekly
df.resample('M').mean()      # Monthly
df.resample('Q').sum()       # Quarterly
df.resample('Y').mean()      # Yearly

# Rolling windows
df['value'].rolling(window=7).mean()
df['value'].rolling(window=7).sum()
df['value'].rolling(window=7).std()
df['value'].rolling(window='7D').mean()  # Time-based window

# Expanding windows
df['value'].expanding().mean()
df['value'].expanding().sum()

# Exponential weighted functions
df['value'].ewm(span=7).mean()
df['value'].ewm(alpha=0.1).mean()

# Shifting
df['value'].shift(1)        # Lag by 1
df['value'].shift(-1)       # Lead by 1
df['value'].diff()          # First difference
df['value'].pct_change()    # Percentage change
```

## Performance Optimizations

```python
# Categorical data
df['category'] = df['category'].astype('category')

# Downcast numeric types
df['int_col'] = pd.to_numeric(df['int_col'], downcast='integer')
df['float_col'] = pd.to_numeric(df['float_col'], downcast='float')

# Query for complex filters (faster than boolean indexing)
df.query('value > @threshold and category in @valid_categories')

# Eval for vectorized computations
df.eval('result = a + b * c', inplace=True)

# Chunked reading
for chunk in pd.read_csv('large.csv', chunksize=10000):
    process(chunk)

# Memory usage inspection
df.memory_usage(deep=True)
df.info(memory_usage='deep')
```

## Advanced Indexing

```python
# MultiIndex (hierarchical indexing)
df.set_index(['level1', 'level2'])
df.xs('key', level='level1')
df.loc[('level1_val', 'level2_val')]

# Index operations
df.reset_index()
df.set_index('column')
df.reindex(new_index)
df.rename(columns={'old': 'new'})
```

## Configuration

```python
# Display options
pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 50)
pd.set_option('display.width', 200)
pd.set_option('display.float_format', '{:.2f}'.format)

# Computation options
pd.set_option('mode.chained_assignment', 'warn')
pd.set_option('compute.use_numexpr', True)
```
