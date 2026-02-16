# Common pandas Patterns and Recipes

## Data Loading Patterns

### CSV with Custom Settings
```python
df = pd.read_csv(
    'data.csv',
    sep=',',
    encoding='utf-8',
    na_values=['NA', 'null', ''],
    thousands=',',
    decimal='.',
    parse_dates=['date'],
    date_format='%Y-%m-%d',
    dtype={'id': 'int32', 'category': 'category'}
)
```

### Excel with Multiple Sheets
```python
# Read all sheets
excel_file = pd.ExcelFile('data.xlsx')
sheets = {sheet: excel_file.parse(sheet) for sheet in excel_file.sheet_names}

# Or specific sheets
df_sales = pd.read_excel('data.xlsx', sheet_name='Sales')
df_inventory = pd.read_excel('data.xlsx', sheet_name='Inventory')
```

### JSON Lines (JSONL)
```python
df = pd.read_json('data.jsonl', lines=True)
```

## Data Cleaning Patterns

### Remove Outliers Using IQR
```python
Q1 = df['value'].quantile(0.25)
Q3 = df['value'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

df_clean = df[(df['value'] >= lower_bound) & (df['value'] <= upper_bound)]
```

### Standardize Column Names
```python
# Convert to lowercase and replace spaces with underscores
df.columns = df.columns.str.lower().str.replace(' ', '_')

# Remove special characters
df.columns = df.columns.str.replace('[^a-z0-9_]', '', regex=True)
```

### Handle Mixed Data Types
```python
# Force numeric conversion, replace errors with NaN
df['numeric_col'] = pd.to_numeric(df['numeric_col'], errors='coerce')

# Then handle NaN values
df['numeric_col'] = df['numeric_col'].fillna(df['numeric_col'].median())
```

## Analysis Patterns

### Calculate Running Total
```python
df['cumulative_sales'] = df.groupby('category')['sales'].cumsum()
```

### Rank Within Groups
```python
df['rank'] = df.groupby('category')['value'].rank(method='dense', ascending=False)
```

### Calculate Percentage of Total
```python
total = df['value'].sum()
df['pct_of_total'] = (df['value'] / total * 100).round(2)

# Within groups
df['pct_in_group'] = df.groupby('category')['value'].apply(
    lambda x: (x / x.sum() * 100).round(2)
)
```

### Window Functions
```python
# Lag/Lead
df['prev_value'] = df.groupby('category')['value'].shift(1)
df['next_value'] = df.groupby('category')['value'].shift(-1)

# Rolling sum within groups
df['rolling_sum_7d'] = df.groupby('category')['value'].rolling(7).sum().reset_index(0, drop=True)
```

### Binning Continuous Values
```python
# Equal-width bins
df['age_group'] = pd.cut(df['age'], bins=[0, 18, 35, 60, 100], labels=['Youth', 'Adult', 'Middle Age', 'Senior'])

# Equal-frequency bins (quantiles)
df['value_quartile'] = pd.qcut(df['value'], q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
```

## Reshaping Patterns

### Wide to Long
```python
# Wide format
# id | jan | feb | mar
# 1  | 100 | 110 | 120

# Convert to long
df_long = pd.melt(
    df,
    id_vars=['id'],
    value_vars=['jan', 'feb', 'mar'],
    var_name='month',
    value_name='sales'
)

# Long format
# id | month | sales
# 1  | jan   | 100
# 1  | feb   | 110
# 1  | mar   | 120
```

### Long to Wide
```python
# Long to wide
df_wide = df_long.pivot(index='id', columns='month', values='sales')

# Or with aggregation if duplicates
df_wide = df_long.pivot_table(index='id', columns='month', values='sales', aggfunc='sum')
```

### Explode Lists
```python
# Before: tags column contains lists
# id | tags
# 1  | ['python', 'data']
# 2  | ['sql', 'analytics']

df_exploded = df.explode('tags')

# After:
# id | tags
# 1  | python
# 1  | data
# 2  | sql
# 2  | analytics
```

## Joining Patterns

### Join on Index
```python
result = df1.join(df2, how='left', on='id')
```

### Join with Different Column Names
```python
result = pd.merge(
    df1,
    df2,
    left_on='customer_id',
    right_on='id',
    how='left'
)
```

### Join with Indicator
```python
result = pd.merge(
    df1,
    df2,
    on='id',
    how='outer',
    indicator=True
)

# Check which rows are in both, left only, or right only
print(result['_merge'].value_counts())
```

### Fuzzy Matching
```python
from fuzzywuzzy import process

def fuzzy_merge(df1, df2, key1, key2, threshold=90):
    s = df2[key2].tolist()
    
    m = df1[key1].apply(lambda x: process.extractOne(x, s))
    df1['matches'] = m.apply(lambda x: x[0])
    df1['match_score'] = m.apply(lambda x: x[1])
    
    # Filter by threshold
    df1 = df1[df1['match_score'] >= threshold]
    
    return pd.merge(df1, df2, left_on='matches', right_on=key2, how='left')
```

## Time Series Patterns

### Resample with Custom Aggregation
```python
df.resample('M').agg({
    'sales': 'sum',
    'quantity': 'sum',
    'price': 'mean',
    'customer_id': 'nunique'
})
```

### Fill Missing Dates
```python
# Create complete date range
date_range = pd.date_range(start=df['date'].min(), end=df['date'].max(), freq='D')

# Reindex to include all dates
df_complete = df.set_index('date').reindex(date_range)

# Fill missing values
df_complete = df_complete.fillna(0)
```

### Calculate Year-over-Year Growth
```python
df['yoy_growth'] = df['value'].pct_change(periods=12) * 100  # For monthly data
```

### Seasonal Decomposition
```python
from statsmodels.tsa.seasonal import seasonal_decompose

# Decompose time series
result = seasonal_decompose(df['value'], model='additive', period=12)

df['trend'] = result.trend
df['seasonal'] = result.seasonal
df['residual'] = result.resid
```

## Advanced Aggregation Patterns

### Multiple Aggregations with Custom Names
```python
result = df.groupby('category').agg(
    total_sales=('sales', 'sum'),
    avg_sales=('sales', 'mean'),
    num_transactions=('id', 'count'),
    unique_customers=('customer_id', 'nunique')
)
```

### Conditional Aggregation
```python
# Sum only positive values
df.groupby('category')['value'].apply(lambda x: x[x > 0].sum())

# Count values meeting condition
df.groupby('category').apply(lambda x: (x['value'] > 100).sum())
```

### Aggregation with Filtering
```python
# Filter groups after aggregation
result = df.groupby('category')['sales'].sum()
result = result[result > 10000]
```

## Export Patterns

### Excel with Formatting
```python
with pd.ExcelWriter('report.xlsx', engine='xlsxwriter') as writer:
    df.to_excel(writer, sheet_name='Data', index=False)
    
    # Get workbook and worksheet
    workbook = writer.book
    worksheet = writer.sheets['Data']
    
    # Add formatting
    header_format = workbook.add_format({'bold': True, 'bg_color': '#D7E4BD'})
    
    # Write headers with format
    for col_num, value in enumerate(df.columns.values):
        worksheet.write(0, col_num, value, header_format)
```

### CSV with Compression
```python
df.to_csv('data.csv.gz', compression='gzip', index=False)
```

### JSON with Specific Orientation
```python
# Records orientation (list of dicts)
df.to_json('data.json', orient='records', lines=True)

# Split orientation (dict of index, columns, data)
df.to_json('data.json', orient='split')
```

## Chain Method Pattern

```python
result = (
    df
    .query('value > 100')
    .assign(
        value_squared=lambda x: x['value'] ** 2,
        category_upper=lambda x: x['category'].str.upper()
    )
    .groupby('category')
    .agg({'value': 'mean', 'value_squared': 'sum'})
    .sort_values('value', ascending=False)
    .head(10)
)
```
