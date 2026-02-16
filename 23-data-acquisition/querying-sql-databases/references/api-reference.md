# SQL Database API Reference

Complete API reference for pandas and SQLAlchemy database operations.

## pandas Database Functions

### pd.read_sql()

General function for reading SQL queries or tables.

```python
pandas.read_sql(
    sql,                    # SQL query string or table name
    con,                    # SQLAlchemy engine or connection
    index_col=None,        # Column(s) to set as index
    coerce_float=True,     # Convert decimal to float
    params=None,           # Query parameters (dict or list)
    parse_dates=None,      # Columns to parse as dates
    columns=None,          # Columns to select
    chunksize=None,        # Return iterator with chunk size
    dtype=None,            # Data types for columns
    dtype_backend=None     # Backend for nullable dtypes
)
```

**Example**:
```python
df = pd.read_sql(
    "SELECT * FROM users WHERE age > :min_age",
    engine,
    params={'min_age': 18},
    parse_dates=['created_at']
)
```

### pd.read_sql_query()

Read SQL query into DataFrame.

```python
pandas.read_sql_query(
    sql,                   # SQL query string
    con,                   # Database connection
    index_col=None,
    coerce_float=True,
    params=None,
    parse_dates=None,
    chunksize=None,
    dtype=None
)
```

**When to use**: Specifically for SELECT queries, slightly faster than read_sql().

### pd.read_sql_table()

Read SQL database table into DataFrame.

```python
pandas.read_sql_table(
    table_name,            # Name of SQL table
    con,                   # SQLAlchemy engine
    schema=None,           # Schema name
    index_col=None,
    coerce_float=True,
    parse_dates=None,
    columns=None,          # List of columns to read
    chunksize=None
)
```

**Example**:
```python
# Read entire table
df = pd.read_sql_table('users', engine)

# Read specific columns
df = pd.read_sql_table('users', engine, columns=['id', 'name', 'email'])

# From specific schema
df = pd.read_sql_table('users', engine, schema='public')
```

### DataFrame.to_sql()

Write DataFrame to SQL database.

```python
DataFrame.to_sql(
    name,                  # Table name
    con,                   # SQLAlchemy engine
    schema=None,           # Schema name
    if_exists='fail',      # {'fail', 'replace', 'append'}
    index=True,            # Write DataFrame index
    index_label=None,      # Label for index column
    chunksize=None,        # Rows per batch
    dtype=None,            # Column types (dict of {col: sqlalchemy.types})
    method=None            # {None, 'multi', callable}
)
```

**if_exists options**:
- `'fail'`: Raise error if table exists (default)
- `'replace'`: Drop and recreate table
- `'append'`: Insert new rows

**method options**:
- `None`: Default insert (one row at a time)
- `'multi'`: Multiple rows per INSERT (faster)
- `callable`: Custom insert function

**Example**:
```python
from sqlalchemy.types import Integer, String, Float

df.to_sql(
    'products',
    engine,
    if_exists='replace',
    index=False,
    method='multi',
    dtype={
        'product_id': Integer,
        'name': String(255),
        'price': Float
    }
)
```

## SQLAlchemy Core API

### create_engine()

Create database engine with connection pooling.

```python
from sqlalchemy import create_engine

engine = create_engine(
    url,                          # Database URL
    echo=False,                   # Log SQL statements
    pool_size=5,                  # Connection pool size
    max_overflow=10,              # Extra connections above pool_size
    pool_timeout=30,              # Seconds to wait for connection
    pool_recycle=-1,              # Seconds before recycling connection
    pool_pre_ping=False,          # Test connections before use
    connect_args={},              # Driver-specific arguments
    isolation_level=None,         # Transaction isolation level
    poolclass=None                # Custom pool class
)
```

**Connection URLs**:
```python
# PostgreSQL
create_engine('postgresql://user:pass@host:5432/db')
create_engine('postgresql+psycopg2://user:pass@host/db')

# MySQL
create_engine('mysql+pymysql://user:pass@host/db')

# SQLite
create_engine('sqlite:///path/to/db.db')
create_engine('sqlite:///:memory:')  # In-memory

# SQL Server
create_engine('mssql+pyodbc://user:pass@host/db?driver=ODBC+Driver+17+for+SQL+Server')
```

**Common pool configurations**:
```python
# Production web app
engine = create_engine(
    url,
    pool_size=20,
    max_overflow=40,
    pool_recycle=3600,
    pool_pre_ping=True
)

# Data analysis script
engine = create_engine(
    url,
    pool_size=1,
    max_overflow=0,
    pool_pre_ping=True
)

# High concurrency
engine = create_engine(
    url,
    pool_size=50,
    max_overflow=100,
    pool_timeout=60
)
```

### Engine Methods

#### engine.connect()

Get a new connection from the pool.

```python
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users"))
    rows = result.fetchall()
```

#### engine.begin()

Get connection with automatic transaction management.

```python
with engine.begin() as conn:
    conn.execute(text("INSERT INTO users (name) VALUES (:name)"), {'name': 'John'})
    # Auto-commit on success, auto-rollback on exception
```

#### engine.dispose()

Close all connections in pool.

```python
engine.dispose()
```

#### engine.execute()

Execute SQL statement (deprecated, use connection.execute()).

```python
# Deprecated
result = engine.execute("SELECT * FROM users")

# Use instead
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users"))
```

### text()

Create textual SQL statement with parameter binding.

```python
from sqlalchemy import text

# Basic query
stmt = text("SELECT * FROM users WHERE age > :min_age")

# Execute with parameters
with engine.connect() as conn:
    result = conn.execute(stmt, {'min_age': 18})
    rows = result.fetchall()

# Multiple parameters
stmt = text("""
    SELECT * FROM users 
    WHERE age BETWEEN :min_age AND :max_age 
    AND country = :country
""")

result = conn.execute(stmt, {
    'min_age': 18,
    'max_age': 65,
    'country': 'USA'
})
```

### inspect()

Database introspection.

```python
from sqlalchemy import inspect

inspector = inspect(engine)

# List tables
tables = inspector.get_table_names()
tables_in_schema = inspector.get_table_names(schema='public')

# Get columns
columns = inspector.get_columns('users')
for col in columns:
    print(f"{col['name']}: {col['type']} (nullable: {col['nullable']})")

# Get primary key
pk = inspector.get_pk_constraint('users')
print(pk['constrained_columns'])

# Get foreign keys
fks = inspector.get_foreign_keys('orders')
for fk in fks:
    print(f"{fk['constrained_columns']} -> {fk['referred_table']}.{fk['referred_columns']}")

# Get indexes
indexes = inspector.get_indexes('users')
for idx in indexes:
    print(f"{idx['name']}: {idx['column_names']}")

# Check if table exists
has_users = inspector.has_table('users')
has_products = inspector.has_table('products', schema='inventory')
```

### MetaData

Database schema container.

```python
from sqlalchemy import MetaData, Table, Column, Integer, String

# Create metadata
metadata = MetaData()

# Reflect existing database
metadata.reflect(bind=engine)

# Access tables
users_table = metadata.tables['users']

# Or reflect single table
users_table = Table('users', metadata, autoload_with=engine)

# Define new table
products_table = Table(
    'products',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(255)),
    Column('price', Integer)
)

# Create all tables
metadata.create_all(engine)

# Drop all tables
metadata.drop_all(engine)
```

## Connection and Transaction Management

### Context Managers

```python
# Connection context manager
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users"))
    # Connection automatically returned to pool

# Transaction context manager
with engine.begin() as conn:
    conn.execute(text("INSERT INTO users (name) VALUES ('John')"))
    conn.execute(text("UPDATE stats SET count = count + 1"))
    # Auto-commit on success, auto-rollback on error

# Nested transactions
with engine.begin() as conn:
    conn.execute(text("INSERT INTO users (name) VALUES ('John')"))
    
    with conn.begin_nested():  # SAVEPOINT
        conn.execute(text("INSERT INTO logs (message) VALUES ('User added')"))
        # Can rollback to this savepoint
```

### Manual Transaction Control

```python
conn = engine.connect()
trans = conn.begin()

try:
    conn.execute(text("INSERT INTO users (name) VALUES ('John')"))
    conn.execute(text("UPDATE stats SET count = count + 1"))
    trans.commit()
except Exception as e:
    trans.rollback()
    raise
finally:
    conn.close()
```

## SQLAlchemy Types

Common SQLAlchemy column types for use with `dtype` parameter in `to_sql()`.

```python
from sqlalchemy.types import (
    Integer, BigInteger, SmallInteger,
    String, Text, Unicode,
    Float, Numeric,
    Boolean,
    Date, DateTime, Time, Interval,
    JSON, ARRAY,
    LargeBinary
)

# Usage in to_sql()
df.to_sql(
    'table_name',
    engine,
    dtype={
        'id': BigInteger,
        'name': String(255),
        'description': Text,
        'price': Numeric(10, 2),
        'quantity': Integer,
        'active': Boolean,
        'created_at': DateTime,
        'metadata': JSON
    }
)
```

### Type Details

**Numeric Types**:
- `Integer`: Standard integer
- `BigInteger`: Large integer (64-bit)
- `SmallInteger`: Small integer (16-bit)
- `Float`: Floating point
- `Numeric(precision, scale)`: Fixed precision decimal

**String Types**:
- `String(length)`: VARCHAR with length
- `Text`: Unlimited length text
- `Unicode(length)`: Unicode VARCHAR

**Date/Time Types**:
- `Date`: Date only
- `DateTime`: Date and time
- `Time`: Time only
- `Interval`: Time interval

**Other Types**:
- `Boolean`: True/False
- `JSON`: JSON data (PostgreSQL, MySQL 5.7+)
- `ARRAY`: Array type (PostgreSQL)
- `LargeBinary`: Binary data (BLOB)

## Database-Specific Features

### PostgreSQL

```python
# Use specific PostgreSQL driver
engine = create_engine('postgresql+psycopg2://user:pass@host/db')

# COPY for fast bulk insert
from io import StringIO
csv_buffer = StringIO()
df.to_csv(csv_buffer, index=False, header=False)
csv_buffer.seek(0)

with engine.raw_connection() as conn:
    cursor = conn.cursor()
    cursor.copy_from(csv_buffer, 'table_name', sep=',')
    conn.commit()

# Use PostgreSQL-specific types
from sqlalchemy.dialects.postgresql import UUID, JSONB, ARRAY

df.to_sql('table', engine, dtype={
    'id': UUID,
    'data': JSONB,
    'tags': ARRAY(String)
})
```

### MySQL

```python
# Use PyMySQL driver
engine = create_engine('mysql+pymysql://user:pass@host/db?charset=utf8mb4')

# Handle MySQL-specific issues
engine = create_engine(
    'mysql+pymysql://user:pass@host/db',
    pool_recycle=3600,  # Reconnect after 1 hour (MySQL timeout)
    connect_args={'charset': 'utf8mb4'}
)
```

### SQLite

```python
# File-based database
engine = create_engine('sqlite:///path/to/database.db')

# In-memory database
engine = create_engine('sqlite:///:memory:')

# SQLite doesn't need connection pooling
engine = create_engine(
    'sqlite:///db.db',
    connect_args={'check_same_thread': False}  # For multi-threading
)
```

## Error Handling

Common exceptions and how to handle them.

```python
from sqlalchemy.exc import (
    OperationalError,      # Connection/operational errors
    IntegrityError,        # Constraint violations
    ProgrammingError,      # SQL syntax errors
    DataError,            # Data value errors
    DatabaseError         # General database errors
)

try:
    df = pd.read_sql(query, engine)
except OperationalError as e:
    # Connection failed, retry logic
    print(f"Database connection error: {e}")
except IntegrityError as e:
    # Constraint violation (unique, foreign key, etc.)
    print(f"Data integrity error: {e}")
except ProgrammingError as e:
    # SQL syntax error
    print(f"SQL syntax error: {e}")
except DatabaseError as e:
    # General database error
    print(f"Database error: {e}")
```

## Performance Tips

### Query Optimization

```python
# Use EXPLAIN to analyze queries
explain_query = f"EXPLAIN ANALYZE {your_query}"
plan = pd.read_sql(explain_query, engine)
print(plan)

# Create indexes for frequently queried columns
with engine.begin() as conn:
    conn.execute(text("CREATE INDEX idx_user_email ON users(email)"))
    conn.execute(text("CREATE INDEX idx_order_date ON orders(order_date)"))

# Use query hints (database-specific)
query = "SELECT /*+ INDEX(users idx_email) */ * FROM users WHERE email = :email"
```

### Connection Pooling Best Practices

```python
# Match pool size to workload
# For web apps: pool_size = (number_of_cores * 2) + effective_spindle_count
# For scripts: pool_size = 1-5

# Enable connection pre-ping for reliability
engine = create_engine(url, pool_pre_ping=True)

# Set appropriate timeouts
engine = create_engine(
    url,
    pool_timeout=30,      # Wait 30s for connection
    pool_recycle=3600     # Recycle after 1 hour
)
```

### Batch Operations

```python
# Batch inserts for better performance
chunk_size = 1000
for i in range(0, len(df), chunk_size):
    chunk = df.iloc[i:i+chunk_size]
    chunk.to_sql('table', engine, if_exists='append', index=False, method='multi')

# Use executemany for bulk operations
with engine.begin() as conn:
    data = [{'name': 'John', 'age': 30}, {'name': 'Jane', 'age': 25}]
    conn.execute(
        text("INSERT INTO users (name, age) VALUES (:name, :age)"),
        data
    )
```
