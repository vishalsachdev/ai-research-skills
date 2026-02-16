# pandas Performance Optimization Guide

## Memory Optimization

### 1. Use Appropriate Data Types

**Problem**: Default dtypes consume unnecessary memory
```python
df.info(memory_usage='deep')  # Check current usage
```

**Solutions**:

**Use categorical for repeated strings** (70-90% memory reduction):
```python
# Before: object dtype
df['category'].memory_usage(deep=True)  # e.g., 40 MB

# After: category dtype
df['category'] = df['category'].astype('category')
df['category'].memory_usage(deep=True)  # e.g., 5 MB
```

**Downcast numeric types**:
```python
# Integer downcasting
df['int_col'] = pd.to_numeric(df['int_col'], downcast='integer')
# int64 -> int32, int16, or int8

# Float downcasting
df['float_col'] = pd.to_numeric(df['float_col'], downcast='float')
# float64 -> float32

# Example
df = pd.DataFrame({'value': [1, 2, 3, 4, 5]})
df['value'].dtype  # int64 (8 bytes per value)
df['value'] = pd.to_numeric(df['value'], downcast='integer')
df['value'].dtype  # int8 (1 byte per value)
```

**Use sparse arrays for mostly-zero data**:
```python
df['sparse_col'] = pd.arrays.SparseArray(df['col'])
```

### 2. Chunked Processing

For files too large to fit in memory:

```python
# Process in chunks
chunk_size = 10000
results = []

for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # Process each chunk
    processed = chunk[chunk['value'] > 100]
    results.append(processed)

# Combine results
final_df = pd.concat(results, ignore_index=True)
```

### 3. Efficient File Formats

**Parquet** (compressed, columnar, fast):
```python
# Write
df.to_parquet('data.parquet', compression='snappy')

# Read
df = pd.read_parquet('data.parquet')

# Benefits: 5-10x smaller files, 2-3x faster read/write
```

**HDF5** (good for append operations):
```python
# Write
df.to_hdf('data.h5', key='df', mode='w')

# Append
df2.to_hdf('data.h5', key='df', mode='a', append=True)

# Read
df = pd.read_hdf('data.h5', key='df')
```

## Computation Speed Optimization

### 1. Vectorization (10-100x faster)

**Avoid loops**:
```python
# Bad (slow)
for i in range(len(df)):
    df.loc[i, 'result'] = df.loc[i, 'a'] + df.loc[i, 'b']

# Good (fast)
df['result'] = df['a'] + df['b']
```

**Use NumPy for complex calculations**:
```python
import numpy as np

# Vectorized computation
df['result'] = np.where(
    df['value'] > 100,
    df['value'] * 1.1,
    df['value'] * 0.9
)
```

### 2. Use query() for Filtering

**For complex boolean indexing** (2-3x faster on large DataFrames):
```python
# Standard boolean indexing
result = df[(df['value'] > 100) & (df['category'] == 'A')]

# query() - faster
result = df.query('value > 100 and category == "A"')

# With variables
threshold = 100
valid_cats = ['A', 'B']
result = df.query('value > @threshold and category in @valid_cats')
```

### 3. Use eval() for Expressions

**For complex arithmetic** (2-5x faster):
```python
# Standard
df['result'] = df['a'] + df['b'] * df['c'] - df['d']

# eval() - faster
df.eval('result = a + b * c - d', inplace=True)
```

### 4. Optimize GroupBy Operations

**Use built-in aggregations**:
```python
# Fast (optimized C code)
df.groupby('category')['value'].sum()

# Slower (Python function calls)
df.groupby('category')['value'].apply(lambda x: x.sum())
```

**Use transform for element-wise results**:
```python
# Calculate group mean and assign back
df['group_mean'] = df.groupby('category')['value'].transform('mean')
```

### 5. Index Operations

**Set index for repeated lookups**:
```python
# Without index (slow for repeated lookups)
for id in ids:
    value = df[df['id'] == id]['value'].iloc[0]

# With index (fast)
df_indexed = df.set_index('id')
for id in ids:
    value = df_indexed.loc[id, 'value']
```

## Parallel Processing

### 1. Using Dask (for larger-than-memory data)

```python
import dask.dataframe as dd

# Read with Dask
ddf = dd.read_csv('large_file.csv')

# Same API as pandas
result = ddf.groupby('category')['value'].mean().compute()
```

### 2. Using Modin (drop-in pandas replacement)

```python
import modin.pandas as pd

# Same code, automatic parallelization
df = pd.read_csv('file.csv')
result = df.groupby('category').mean()
```

### 3. Manual Parallelization

```python
from multiprocessing import Pool
import numpy as np

def process_chunk(chunk):
    # Process chunk
    return chunk[chunk['value'] > 100]

# Split DataFrame
chunks = np.array_split(df, 4)

# Process in parallel
with Pool(4) as pool:
    results = pool.map(process_chunk, chunks)

# Combine
final_df = pd.concat(results)
```

## I/O Optimization

### 1. Specify dtypes When Reading

```python
# Let pandas infer (slow)
df = pd.read_csv('data.csv')

# Specify dtypes (faster, less memory)
dtypes = {
    'id': 'int32',
    'value': 'float32',
    'category': 'category'
}
df = pd.read_csv('data.csv', dtype=dtypes)
```

### 2. Use usecols to Read Subset

```python
# Read only needed columns
df = pd.read_csv('data.csv', usecols=['id', 'value', 'category'])
```

### 3. Parse Dates Efficiently

```python
# Specify date format
df = pd.read_csv(
    'data.csv',
    parse_dates=['date'],
    date_format='%Y-%m-%d'  # pandas 2.0+
)
```

## Profiling Performance

### 1. Time Individual Operations

```python
# Using %%timeit in Jupyter
%%timeit
df.groupby('category')['value'].mean()

# Using %time for single run
%time df.groupby('category')['value'].mean()
```

### 2. Profile Memory Usage

```python
# Check memory usage
df.memory_usage(deep=True).sum() / 1024**2  # MB

# Check by dtype
df.memory_usage(deep=True).groupby(df.dtypes).sum() / 1024**2
```

### 3. Use pandas Profiler

```python
from ydata_profiling import ProfileReport

profile = ProfileReport(df, minimal=True)
profile.to_file('report.html')
```

## Best Practices Summary

1. **Use appropriate dtypes** - categorical for strings, downcast numerics
2. **Vectorize operations** - avoid loops, use NumPy
3. **Use query/eval** - faster than boolean indexing for large data
4. **Read efficiently** - specify dtypes, use parquet/hdf5
5. **Index wisely** - set index for repeated lookups
6. **Process in chunks** - for larger-than-memory data
7. **Monitor memory** - use memory_usage(deep=True)
8. **Profile first** - identify bottlenecks before optimizing
