---
name: querying-sql-databases
description: Provides guidance for querying and managing SQL databases using pandas and SQLAlchemy, including connection management, query optimization, and data extraction workflows for data analysis tasks
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Acquisition, SQL, Pandas, SQLAlchemy, Database, Python]
dependencies: [pandas>=2.0.0, sqlalchemy>=2.0.0, psycopg2-binary>=2.9.0, pymysql>=1.1.0]
---

# Querying SQL Databases

This skill provides expert guidance for working with SQL databases in Python using pandas and SQLAlchemy. You'll learn efficient data extraction patterns, connection management, query optimization, and best practices for integrating database data into analytical workflows.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Installation & Setup](#installation--setup)
- [Basic Workflows](#basic-workflows)
- [Advanced Patterns](#advanced-patterns)
- [When to Use vs Alternatives](#when-to-use-vs-alternatives)
- [Common Issues & Solutions](#common-issues--solutions)
- [Performance Optimization](#performance-optimization)

## Core Concepts

### Database Connection Methods

**SQLAlchemy Engine** (Recommended):
- Connection pooling and management
- Database-agnostic API
- ORM capabilities
- Transaction control

**Direct pandas.read_sql()** (Simple queries):
- Quick one-off queries
- Minimal configuration
- Limited connection control

### Key Components

1. **Connection Strings**: Database URLs for different systems
2. **Engines**: SQLAlchemy connection pools
3. **Query Methods**: SQL execution approaches
4. **Data Types**: SQL to pandas type mapping
5. **Transactions**: ACID compliance for writes

## Installation & Setup

### Install Database Drivers

```bash
# PostgreSQL
pip install pandas sqlalchemy psycopg2-binary

# MySQL/MariaDB
pip install pandas sqlalchemy pymysql

# SQLite (built-in)
pip install pandas sqlalchemy

# Microsoft SQL Server
pip install pandas sqlalchemy pyodbc

# Multiple databases
pip install pandas sqlalchemy psycopg2-binary pymysql pyodbc
```

### Connection String Formats

```python
# PostgreSQL
postgresql://user:password@host:port/database
postgresql+psycopg2://user:password@host:port/database

# MySQL
mysql+pymysql://user:password@host:port/database

# SQLite
sqlite:///path/to/database.db
sqlite:///absolute/path/database.db

# SQL Server
mssql+pyodbc://user:password@host:port/database?driver=ODBC+Driver+17+for+SQL+Server
```

## Basic Workflows

### Workflow 1: Read Data from Database

**Use Case**: Extract data from SQL database for analysis

**Steps**:

1. **Create database connection**:
```python
from sqlalchemy import create_engine
import pandas as pd

# Create engine with connection pooling
engine = create_engine(
    'postgresql://user:password@localhost:5432/mydb',
    pool_size=5,
    max_overflow=10,
    pool_pre_ping=True  # Verify connections before use
)
```

2. **Execute simple query**:
```python
# Using pandas - simplest approach
query = "SELECT * FROM sales WHERE date >= '2024-01-01'"
df = pd.read_sql(query, engine)

print(f"Loaded {len(df)} rows")
print(df.head())
```

3. **Execute parameterized query** (SQL injection safe):
```python
from sqlalchemy import text

# Parameterized query
query = text("""
    SELECT customer_id, SUM(amount) as total
    FROM sales
    WHERE date BETWEEN :start_date AND :end_date
    GROUP BY customer_id
""")

df = pd.read_sql(
    query,
    engine,
    params={'start_date': '2024-01-01', 'end_date': '2024-12-31'}
)
```

4. **Read with chunking for large datasets**:
```python
# Process large tables in chunks
chunk_size = 10000
chunks = []

for chunk in pd.read_sql(query, engine, chunksize=chunk_size):
    # Process each chunk
    processed = chunk[chunk['amount'] > 100]
    chunks.append(processed)

# Combine results
df = pd.concat(chunks, ignore_index=True)
```

5. **Close connection**:
```python
# Close when done
engine.dispose()
```

**Checklist**:
- [ ] Install required database driver
- [ ] Create SQLAlchemy engine with connection string
- [ ] Test connection with simple query
- [ ] Use parameterized queries for user input
- [ ] Handle large datasets with chunking
- [ ] Dispose engine when finished

### Workflow 2: Write Data to Database

**Use Case**: Save analysis results or processed data to database

**Steps**:

1. **Prepare DataFrame**:
```python
import pandas as pd

# Create or load data
df = pd.DataFrame({
    'product_id': [1, 2, 3],
    'product_name': ['Widget A', 'Widget B', 'Widget C'],
    'price': [19.99, 29.99, 39.99],
    'inventory': [100, 50, 75]
})
```

2. **Write to new table**:
```python
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:password@localhost:5432/mydb')

# Create new table
df.to_sql(
    'products',
    engine,
    if_exists='fail',  # Options: 'fail', 'replace', 'append'
    index=False,
    method='multi'  # Faster bulk insert
)
```

3. **Append to existing table**:
```python
# Add more data to existing table
new_data = pd.DataFrame({
    'product_id': [4, 5],
    'product_name': ['Widget D', 'Widget E'],
    'price': [49.99, 59.99],
    'inventory': [25, 30]
})

new_data.to_sql('products', engine, if_exists='append', index=False)
```

4. **Replace table with validation**:
```python
# Backup and replace pattern
with engine.begin() as conn:
    # Create backup
    conn.execute(text("CREATE TABLE products_backup AS SELECT * FROM products"))
    
    try:
        # Replace data
        df.to_sql('products', engine, if_exists='replace', index=False)
        
        # Validate
        result = pd.read_sql("SELECT COUNT(*) as cnt FROM products", engine)
        assert result.iloc[0]['cnt'] == len(df), "Row count mismatch"
        
        # Drop backup if successful
        conn.execute(text("DROP TABLE products_backup"))
    except Exception as e:
        # Restore from backup
        conn.execute(text("DROP TABLE IF EXISTS products"))
        conn.execute(text("ALTER TABLE products_backup RENAME TO products"))
        raise
```

5. **Write with data type specification**:
```python
from sqlalchemy.types import Integer, String, Float, DateTime

# Specify column types
df.to_sql(
    'products',
    engine,
    if_exists='replace',
    index=False,
    dtype={
        'product_id': Integer,
        'product_name': String(255),
        'price': Float,
        'inventory': Integer
    }
)
```

**Checklist**:
- [ ] Validate DataFrame schema before writing
- [ ] Choose appropriate if_exists strategy
- [ ] Use method='multi' for better performance
- [ ] Specify data types for production tables
- [ ] Test with small dataset first
- [ ] Implement backup strategy for replace operations

### Workflow 3: Advanced Query Patterns

**Use Case**: Complex data extraction with joins, CTEs, and aggregations

**Steps**:

1. **Multi-table joins**:
```python
query = text("""
    SELECT 
        o.order_id,
        o.order_date,
        c.customer_name,
        c.email,
        SUM(oi.quantity * oi.unit_price) as total_amount
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.order_date >= :start_date
    GROUP BY o.order_id, o.order_date, c.customer_name, c.email
    HAVING SUM(oi.quantity * oi.unit_price) > :min_amount
    ORDER BY total_amount DESC
""")

df = pd.read_sql(
    query,
    engine,
    params={'start_date': '2024-01-01', 'min_amount': 1000}
)
```

2. **Common Table Expressions (CTEs)**:
```python
query = text("""
    WITH monthly_sales AS (
        SELECT 
            DATE_TRUNC('month', sale_date) as month,
            product_id,
            SUM(amount) as monthly_total
        FROM sales
        WHERE sale_date >= :start_date
        GROUP BY DATE_TRUNC('month', sale_date), product_id
    ),
    ranked_products AS (
        SELECT 
            month,
            product_id,
            monthly_total,
            RANK() OVER (PARTITION BY month ORDER BY monthly_total DESC) as rank
        FROM monthly_sales
    )
    SELECT * FROM ranked_products WHERE rank <= 10
""")

top_products = pd.read_sql(query, engine, params={'start_date': '2024-01-01'})
```

3. **Window functions**:
```python
query = text("""
    SELECT 
        sale_date,
        product_id,
        amount,
        SUM(amount) OVER (
            PARTITION BY product_id 
            ORDER BY sale_date 
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as rolling_7day_total,
        AVG(amount) OVER (
            PARTITION BY product_id 
            ORDER BY sale_date 
            ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
        ) as rolling_30day_avg
    FROM sales
    WHERE sale_date >= :start_date
    ORDER BY product_id, sale_date
""")

df = pd.read_sql(query, engine, params={'start_date': '2024-01-01'})
```

4. **Dynamic column selection**:
```python
def build_query(table, columns, filters=None):
    """Build dynamic SQL query safely"""
    from sqlalchemy import Table, MetaData, select
    
    metadata = MetaData()
    metadata.reflect(bind=engine)
    table_obj = metadata.tables[table]
    
    # Select specific columns
    cols = [table_obj.c[col] for col in columns]
    query = select(*cols)
    
    # Add filters
    if filters:
        for col, value in filters.items():
            query = query.where(table_obj.c[col] == value)
    
    return query

# Use the builder
query = build_query(
    'sales',
    columns=['sale_date', 'product_id', 'amount'],
    filters={'status': 'completed'}
)

df = pd.read_sql(query, engine)
```

5. **Execute multiple queries**:
```python
# Multiple related queries
queries = {
    'sales': "SELECT * FROM sales WHERE date >= :date",
    'customers': "SELECT * FROM customers WHERE active = true",
    'products': "SELECT * FROM products WHERE inventory > 0"
}

results = {}
for name, query in queries.items():
    results[name] = pd.read_sql(
        text(query),
        engine,
        params={'date': '2024-01-01'}
    )

# Access results
sales_df = results['sales']
customers_df = results['customers']
products_df = results['products']
```

**Checklist**:
- [ ] Test complex queries in database client first
- [ ] Use CTEs for readable complex queries
- [ ] Leverage database window functions vs pandas
- [ ] Validate query results with known aggregates
- [ ] Use EXPLAIN ANALYZE to check query performance

## Advanced Patterns

### Connection Context Managers

```python
from contextlib import contextmanager

@contextmanager
def get_db_connection(conn_string):
    """Context manager for database connections"""
    engine = create_engine(conn_string)
    try:
        yield engine
    finally:
        engine.dispose()

# Usage
with get_db_connection('postgresql://user:pass@host/db') as engine:
    df = pd.read_sql("SELECT * FROM table", engine)
```

### Transaction Management

```python
from sqlalchemy import text

# Explicit transaction control
with engine.begin() as conn:
    # All operations in transaction
    conn.execute(text("DELETE FROM temp_data WHERE date < :cutoff"), 
                 {'cutoff': '2024-01-01'})
    
    df.to_sql('temp_data', conn, if_exists='append', index=False)
    
    # Commit happens automatically if no exception
    # Rollback happens automatically on exception
```

### Reflection and Metadata

```python
from sqlalchemy import MetaData, inspect

# Inspect database schema
inspector = inspect(engine)

# List all tables
tables = inspector.get_table_names()
print(f"Tables: {tables}")

# Get column info for a table
columns = inspector.get_columns('sales')
for col in columns:
    print(f"{col['name']}: {col['type']}")

# Get primary keys
pk = inspector.get_pk_constraint('sales')
print(f"Primary key: {pk['constrained_columns']}")

# Get foreign keys
fks = inspector.get_foreign_keys('sales')
for fk in fks:
    print(f"FK: {fk['constrained_columns']} -> {fk['referred_table']}")
```

## When to Use vs Alternatives

### Use SQL Databases When:
- ✅ Data already stored in relational database
- ✅ Need ACID transactions and data integrity
- ✅ Complex joins and aggregations required
- ✅ Multi-user concurrent access needed
- ✅ Data size exceeds memory (query subsets)
- ✅ Need database-level security and access control

### Consider Alternatives When:
- ❌ **CSV/Parquet files**: Simple datasets, one-time analysis
- ❌ **Cloud data warehouses** (BigQuery, Snowflake): Very large datasets, cloud-native
- ❌ **NoSQL databases**: Unstructured or document data
- ❌ **APIs**: Real-time data or third-party services
- ❌ **Dask/Spark**: Datasets larger than single-machine memory

### pandas vs SQLAlchemy vs raw DB drivers:

| Feature | pandas.read_sql() | SQLAlchemy Core | Raw DB Driver |
|---------|------------------|-----------------|---------------|
| Ease of use | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| Connection pooling | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Transaction control | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Database portability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐ |
| Performance | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

**Recommendation**: Use SQLAlchemy for production code, pandas.read_sql() for quick analysis.

## Common Issues & Solutions

### Issue 1: Connection Pool Exhaustion

**Problem**: "QueuePool limit exceeded" error

```python
# ❌ Bad: Creating new engine for each query
for i in range(100):
    engine = create_engine(connection_string)
    df = pd.read_sql("SELECT * FROM table", engine)
    # engine not disposed
```

**Solution**: Reuse engine with proper pooling

```python
# ✅ Good: Single engine with connection pooling
engine = create_engine(
    connection_string,
    pool_size=10,        # Base pool size
    max_overflow=20,     # Extra connections if needed
    pool_recycle=3600,   # Recycle connections after 1 hour
    pool_pre_ping=True   # Verify connection health
)

try:
    for i in range(100):
        df = pd.read_sql("SELECT * FROM table", engine)
finally:
    engine.dispose()
```

### Issue 2: Memory Errors with Large Tables

**Problem**: "MemoryError" when reading large table

```python
# ❌ Bad: Loading entire table into memory
df = pd.read_sql("SELECT * FROM large_table", engine)  # 100M rows
```

**Solution**: Use chunking or filtering

```python
# ✅ Good: Process in chunks
chunk_size = 50000
processed_chunks = []

for chunk in pd.read_sql("SELECT * FROM large_table", engine, chunksize=chunk_size):
    # Process chunk
    filtered = chunk[chunk['status'] == 'active']
    aggregated = filtered.groupby('category')['amount'].sum()
    processed_chunks.append(aggregated)

result = pd.concat(processed_chunks).groupby(level=0).sum()

# ✅ Better: Filter in database
query = """
    SELECT category, SUM(amount) as total
    FROM large_table
    WHERE status = 'active'
    GROUP BY category
"""
df = pd.read_sql(query, engine)
```

### Issue 3: SQL Injection Vulnerability

**Problem**: Unsafe string formatting in queries

```python
# ❌ DANGEROUS: SQL injection risk
user_input = "'; DROP TABLE users; --"
query = f"SELECT * FROM users WHERE username = '{user_input}'"
df = pd.read_sql(query, engine)  # Could delete entire table!
```

**Solution**: Use parameterized queries

```python
# ✅ Safe: Parameterized query
from sqlalchemy import text

user_input = "john_doe"
query = text("SELECT * FROM users WHERE username = :username")
df = pd.read_sql(query, engine, params={'username': user_input})
```

### Issue 4: Data Type Mismatches

**Problem**: Incorrect data types after reading

```python
# Column types may not match expectations
df = pd.read_sql("SELECT * FROM sales", engine)
print(df.dtypes)
# date_column: object (should be datetime)
# amount: float64 (should be decimal for money)
```

**Solution**: Explicit type conversion

```python
# Specify types during read
df = pd.read_sql(
    "SELECT * FROM sales",
    engine,
    parse_dates=['date_column', 'created_at']
)

# Or convert after reading
df['date_column'] = pd.to_datetime(df['date_column'])
df['amount'] = df['amount'].astype('float64')

# When writing, specify SQL types
from sqlalchemy.types import Date, Numeric

df.to_sql(
    'sales',
    engine,
    dtype={
        'date_column': Date,
        'amount': Numeric(10, 2)
    }
)
```

### Issue 5: Slow Query Performance

**Problem**: Queries taking too long

```python
# Slow query
query = "SELECT * FROM orders WHERE customer_id = 12345"
# Takes 30 seconds for 10M row table
```

**Solution**: Add indexes and optimize query

```python
# Create index (do this once in database)
with engine.begin() as conn:
    conn.execute(text(
        "CREATE INDEX idx_customer_id ON orders(customer_id)"
    ))

# Use EXPLAIN to analyze query
explain_query = "EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 12345"
plan = pd.read_sql(explain_query, engine)
print(plan)

# Optimize by selecting only needed columns
query = """
    SELECT order_id, order_date, total_amount
    FROM orders
    WHERE customer_id = :customer_id
"""
df = pd.read_sql(query, engine, params={'customer_id': 12345})
```

### Issue 6: Encoding Issues

**Problem**: Special characters not displaying correctly

**Solution**: Specify encoding in connection string

```python
# For MySQL with UTF-8
engine = create_engine(
    'mysql+pymysql://user:pass@host/db?charset=utf8mb4'
)

# For PostgreSQL
engine = create_engine(
    'postgresql://user:pass@host/db?client_encoding=utf8'
)
```

## Performance Optimization

### Query Optimization Tips

1. **Select only needed columns**:
```python
# ❌ Slow: Select all columns
df = pd.read_sql("SELECT * FROM large_table", engine)

# ✅ Fast: Select specific columns
df = pd.read_sql("SELECT id, name, amount FROM large_table", engine)
```

2. **Filter in database, not pandas**:
```python
# ❌ Slow: Filter in pandas
df = pd.read_sql("SELECT * FROM sales", engine)
filtered = df[df['date'] >= '2024-01-01']

# ✅ Fast: Filter in SQL
df = pd.read_sql(
    "SELECT * FROM sales WHERE date >= :date",
    engine,
    params={'date': '2024-01-01'}
)
```

3. **Use database aggregations**:
```python
# ❌ Slow: Aggregate in pandas
df = pd.read_sql("SELECT * FROM sales", engine)
result = df.groupby('product_id')['amount'].sum()

# ✅ Fast: Aggregate in database
result = pd.read_sql(
    "SELECT product_id, SUM(amount) as total FROM sales GROUP BY product_id",
    engine
)
```

### Bulk Operations

```python
# Fast bulk insert with method='multi'
df.to_sql('table', engine, if_exists='append', index=False, method='multi')

# Even faster with native tools (PostgreSQL)
from io import StringIO

# Create CSV buffer
csv_buffer = StringIO()
df.to_csv(csv_buffer, index=False, header=False)
csv_buffer.seek(0)

# Use COPY command
with engine.raw_connection() as conn:
    cursor = conn.cursor()
    cursor.copy_from(csv_buffer, 'table', sep=',')
    conn.commit()
```

## Additional Resources

- **API Reference**: [references/api-reference.md](references/api-reference.md)
- **Database-Specific Guides**: [references/database-guides.md](references/database-guides.md)
- **Troubleshooting**: [references/troubleshooting.md](references/troubleshooting.md)

## Summary

This skill covered:
- ✅ Database connections with SQLAlchemy
- ✅ Reading and writing data with pandas
- ✅ Parameterized queries for security
- ✅ Complex queries with joins and CTEs
- ✅ Performance optimization strategies
- ✅ Common issues and solutions

**Key Takeaways**:
1. Always use parameterized queries to prevent SQL injection
2. Filter and aggregate in the database when possible
3. Use connection pooling for better performance
4. Process large datasets with chunking
5. Dispose engines when done to free resources
