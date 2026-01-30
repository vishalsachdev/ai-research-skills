# SQL Database Troubleshooting Guide

Common issues and solutions when working with SQL databases in pandas and SQLAlchemy.

## Connection Issues

### Cannot Connect to Database

**Symptom**: `OperationalError: could not connect to server`

**Common Causes**:

1. **Wrong connection string**:
```python
# ❌ Wrong
engine = create_engine('postgresql://localhost/mydb')  # Missing user/password

# ✅ Correct
engine = create_engine('postgresql://user:password@localhost:5432/mydb')
```

2. **Database server not running**:
```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Start PostgreSQL
sudo systemctl start postgresql

# Check MySQL
sudo systemctl status mysql
```

3. **Firewall blocking connection**:
```bash
# Check if port is open (PostgreSQL default: 5432, MySQL: 3306)
telnet localhost 5432

# Allow port in firewall (Ubuntu)
sudo ufw allow 5432
```

4. **Wrong host/port**:
```python
# Check database is listening on correct interface
# PostgreSQL: edit postgresql.conf
listen_addresses = '*'  # or specific IP

# Check pg_hba.conf for access rules
# Add: host all all 0.0.0.0/0 md5
```

**Solution**:
```python
# Test connection step by step
from sqlalchemy import create_engine, text

try:
    engine = create_engine('postgresql://user:password@localhost:5432/mydb')
    with engine.connect() as conn:
        result = conn.execute(text("SELECT 1"))
        print("Connection successful!")
except Exception as e:
    print(f"Connection failed: {e}")
```

### SSL/TLS Connection Issues

**Symptom**: `SSL connection required` or `SSL error`

**Solution**:
```python
# PostgreSQL - require SSL
engine = create_engine(
    'postgresql://user:password@host/db',
    connect_args={'sslmode': 'require'}
)

# PostgreSQL - disable SSL (development only)
engine = create_engine(
    'postgresql://user:password@host/db',
    connect_args={'sslmode': 'disable'}
)

# MySQL - with SSL
engine = create_engine(
    'mysql+pymysql://user:password@host/db',
    connect_args={
        'ssl': {
            'ssl_ca': '/path/to/ca-cert.pem',
            'ssl_cert': '/path/to/client-cert.pem',
            'ssl_key': '/path/to/client-key.pem'
        }
    }
)
```

### Authentication Failed

**Symptom**: `authentication failed for user`

**Solutions**:

1. **Check credentials**:
```python
import os

# Use environment variables
engine = create_engine(
    f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASS')}@host/db"
)
```

2. **Check database user permissions**:
```sql
-- PostgreSQL: Create user with permissions
CREATE USER myuser WITH PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;

-- MySQL: Create user
CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'%';
FLUSH PRIVILEGES;
```

3. **Special characters in password**:
```python
from urllib.parse import quote_plus

password = "p@ssw0rd!#$%"
engine = create_engine(
    f'postgresql://user:{quote_plus(password)}@host/db'
)
```

## Performance Issues

### Slow Queries

**Symptom**: Queries taking too long to execute

**Diagnosis**:
```python
# PostgreSQL: Use EXPLAIN ANALYZE
explain_query = """
    EXPLAIN ANALYZE
    SELECT * FROM large_table WHERE column = 'value'
"""
plan = pd.read_sql(explain_query, engine)
print(plan)

# MySQL: Use EXPLAIN
explain_query = "EXPLAIN SELECT * FROM large_table WHERE column = 'value'"
plan = pd.read_sql(explain_query, engine)
```

**Solutions**:

1. **Add indexes**:
```python
with engine.begin() as conn:
    # Single column index
    conn.execute(text("CREATE INDEX idx_column ON table_name(column)"))
    
    # Composite index
    conn.execute(text(
        "CREATE INDEX idx_multi ON table_name(col1, col2)"
    ))
    
    # Unique index
    conn.execute(text(
        "CREATE UNIQUE INDEX idx_email ON users(email)"
    ))
```

2. **Filter in SQL, not pandas**:
```python
# ❌ Slow: Load everything then filter
df = pd.read_sql("SELECT * FROM large_table", engine)
df = df[df['date'] >= '2024-01-01']

# ✅ Fast: Filter in database
df = pd.read_sql(
    "SELECT * FROM large_table WHERE date >= :date",
    engine,
    params={'date': '2024-01-01'}
)
```

3. **Select only needed columns**:
```python
# ❌ Slow: Select all columns
df = pd.read_sql("SELECT * FROM wide_table", engine)

# ✅ Fast: Select specific columns
df = pd.read_sql("SELECT id, name, amount FROM wide_table", engine)
```

4. **Use database aggregations**:
```python
# ❌ Slow: Aggregate in pandas
df = pd.read_sql("SELECT * FROM sales", engine)  # 10M rows
total = df.groupby('product')['amount'].sum()

# ✅ Fast: Aggregate in database
total = pd.read_sql(
    "SELECT product, SUM(amount) as total FROM sales GROUP BY product",
    engine
)
```

### Memory Errors

**Symptom**: `MemoryError` when loading large tables

**Solution**: Use chunking
```python
# Process in chunks
chunk_size = 10000
results = []

for chunk in pd.read_sql(
    "SELECT * FROM large_table",
    engine,
    chunksize=chunk_size
):
    # Process each chunk
    processed = chunk[chunk['value'] > 100]
    aggregated = processed.groupby('category').sum()
    results.append(aggregated)

# Combine results
final = pd.concat(results).groupby(level=0).sum()
```

Alternative: Filter in database
```python
# Only load what you need
df = pd.read_sql("""
    SELECT category, SUM(value) as total
    FROM large_table
    WHERE value > 100
    GROUP BY category
""", engine)
```

### Connection Pool Exhausted

**Symptom**: `QueuePool limit of size X overflow Y reached, connection timed out`

**Causes**:
- Creating too many connections
- Not closing connections
- Pool size too small

**Solutions**:

1. **Increase pool size**:
```python
engine = create_engine(
    connection_string,
    pool_size=20,        # Increase from default 5
    max_overflow=40,     # Increase from default 10
    pool_timeout=60      # Increase timeout
)
```

2. **Reuse engine**:
```python
# ❌ Bad: Creating new engine each time
for i in range(100):
    engine = create_engine(connection_string)
    df = pd.read_sql(query, engine)

# ✅ Good: Reuse single engine
engine = create_engine(connection_string)
for i in range(100):
    df = pd.read_sql(query, engine)
engine.dispose()
```

3. **Use context managers**:
```python
# Ensures connections are returned to pool
with engine.connect() as conn:
    result = conn.execute(text(query))
    df = pd.DataFrame(result.fetchall())
```

4. **Dispose engine when done**:
```python
try:
    # Do work
    df = pd.read_sql(query, engine)
finally:
    engine.dispose()
```

## Data Type Issues

### Type Conversion Errors

**Symptom**: `DataError: invalid input syntax for type`

**Solutions**:

1. **Explicit type conversion in query**:
```python
query = text("""
    SELECT 
        id,
        CAST(date_string AS DATE) as date,
        CAST(amount_string AS NUMERIC) as amount
    FROM table_name
""")
```

2. **Handle NULLs**:
```python
# Read with NULL handling
df = pd.read_sql(query, engine)

# Replace NaN appropriately
df['amount'].fillna(0, inplace=True)
df['name'].fillna('Unknown', inplace=True)
```

3. **Specify types when writing**:
```python
from sqlalchemy.types import Integer, String, Numeric, DateTime

df.to_sql(
    'table_name',
    engine,
    dtype={
        'id': Integer,
        'name': String(255),
        'amount': Numeric(10, 2),
        'created_at': DateTime
    }
)
```

### Date/Time Parsing Issues

**Symptom**: Dates loaded as strings

**Solution**:
```python
# Parse dates during read
df = pd.read_sql(
    query,
    engine,
    parse_dates=['created_at', 'updated_at', 'date_column']
)

# Or parse after reading
df['date_column'] = pd.to_datetime(df['date_column'])

# Handle different date formats
df['date_column'] = pd.to_datetime(df['date_column'], format='%Y-%m-%d')

# Handle timezone-aware datetimes
df['timestamp'] = pd.to_datetime(df['timestamp'], utc=True)
```

### Unicode/Encoding Issues

**Symptom**: Special characters displayed incorrectly (�, ?, etc.)

**Solutions**:

1. **MySQL - Use utf8mb4**:
```python
engine = create_engine(
    'mysql+pymysql://user:pass@host/db?charset=utf8mb4'
)

# Or in connect_args
engine = create_engine(
    'mysql+pymysql://user:pass@host/db',
    connect_args={'charset': 'utf8mb4'}
)
```

2. **PostgreSQL - Set client encoding**:
```python
engine = create_engine(
    'postgresql://user:pass@host/db?client_encoding=utf8'
)
```

3. **Ensure database encoding is correct**:
```sql
-- PostgreSQL: Check database encoding
SELECT datname, pg_encoding_to_char(encoding) FROM pg_database;

-- Create database with UTF-8
CREATE DATABASE mydb ENCODING 'UTF8';

-- MySQL: Check table encoding
SHOW CREATE TABLE table_name;

-- Convert table to utf8mb4
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## Write Operation Issues

### Table Already Exists

**Symptom**: `ValueError: Table 'table_name' already exists`

**Solution**: Use `if_exists` parameter
```python
# Append to existing table
df.to_sql('table_name', engine, if_exists='append', index=False)

# Replace existing table
df.to_sql('table_name', engine, if_exists='replace', index=False)

# Fail if exists (default)
df.to_sql('table_name', engine, if_exists='fail', index=False)
```

### Constraint Violations

**Symptom**: `IntegrityError: duplicate key value violates unique constraint`

**Solutions**:

1. **Check for duplicates before insert**:
```python
# Remove duplicates
df_unique = df.drop_duplicates(subset=['id'])

# Append only new records
existing_ids = pd.read_sql("SELECT id FROM table_name", engine)['id']
new_records = df[~df['id'].isin(existing_ids)]
new_records.to_sql('table_name', engine, if_exists='append', index=False)
```

2. **Use upsert pattern (PostgreSQL)**:
```python
from sqlalchemy.dialects.postgresql import insert

with engine.begin() as conn:
    for _, row in df.iterrows():
        stmt = insert(table).values(row.to_dict())
        stmt = stmt.on_conflict_do_update(
            index_elements=['id'],
            set_=row.to_dict()
        )
        conn.execute(stmt)
```

3. **Use ON DUPLICATE KEY UPDATE (MySQL)**:
```python
with engine.begin() as conn:
    conn.execute(text("""
        INSERT INTO table_name (id, name, value)
        VALUES (:id, :name, :value)
        ON DUPLICATE KEY UPDATE
            name = VALUES(name),
            value = VALUES(value)
    """), df.to_dict('records'))
```

### Permission Denied

**Symptom**: `PermissionError` or `InsufficientPrivilege`

**Solution**: Grant appropriate permissions
```sql
-- PostgreSQL
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO myuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO myuser;

-- MySQL
GRANT SELECT, INSERT, UPDATE, DELETE ON database.* TO 'myuser'@'%';
FLUSH PRIVILEGES;
```

## Transaction Issues

### Deadlocks

**Symptom**: `DeadlockDetected` error

**Solutions**:

1. **Implement retry logic**:
```python
from sqlalchemy.exc import OperationalError
import time

def execute_with_retry(engine, query, max_retries=3):
    for attempt in range(max_retries):
        try:
            return pd.read_sql(query, engine)
        except OperationalError as e:
            if 'deadlock' in str(e).lower() and attempt < max_retries - 1:
                time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
                continue
            raise
```

2. **Access tables in consistent order**:
```python
# Always access tables in same order to prevent circular waits
with engine.begin() as conn:
    # Always table1 before table2
    conn.execute(text("UPDATE table1 SET ..."))
    conn.execute(text("UPDATE table2 SET ..."))
```

3. **Use appropriate isolation levels**:
```python
# READ COMMITTED prevents some deadlocks
engine = create_engine(
    connection_string,
    isolation_level="READ COMMITTED"
)
```

### Uncommitted Transactions

**Symptom**: Changes not visible to other connections

**Solution**: Use explicit transactions
```python
# Ensure transaction is committed
with engine.begin() as conn:
    df.to_sql('table_name', conn, if_exists='append', index=False)
    # Automatically commits on success

# Or manually commit
conn = engine.connect()
trans = conn.begin()
try:
    df.to_sql('table_name', conn, if_exists='append', index=False)
    trans.commit()
except:
    trans.rollback()
    raise
finally:
    conn.close()
```

## Query Errors

### SQL Syntax Errors

**Symptom**: `ProgrammingError: syntax error at or near`

**Solutions**:

1. **Test query in database client first**
2. **Check SQL dialect differences**:
```python
# PostgreSQL uses LIMIT/OFFSET
query = "SELECT * FROM table LIMIT 10 OFFSET 20"

# SQL Server uses TOP/OFFSET FETCH
query = """
    SELECT * FROM table
    ORDER BY id
    OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY
"""

# MySQL uses LIMIT with offset
query = "SELECT * FROM table LIMIT 20, 10"
```

3. **Use parameterized queries**:
```python
# ❌ String formatting can cause syntax errors
query = f"SELECT * FROM {table_name} WHERE value > {threshold}"

# ✅ Use parameters
query = text("SELECT * FROM table_name WHERE value > :threshold")
df = pd.read_sql(query, engine, params={'threshold': threshold})
```

### Table or Column Does Not Exist

**Symptom**: `ProgrammingError: relation "table_name" does not exist`

**Solutions**:

1. **Check table exists**:
```python
from sqlalchemy import inspect

inspector = inspect(engine)
tables = inspector.get_table_names()
print(f"Available tables: {tables}")

if 'my_table' in tables:
    df = pd.read_sql("SELECT * FROM my_table", engine)
```

2. **Specify schema**:
```python
# PostgreSQL: Use schema.table
df = pd.read_sql("SELECT * FROM public.table_name", engine)

# Or set search_path
with engine.connect() as conn:
    conn.execute(text("SET search_path TO myschema"))
    df = pd.read_sql("SELECT * FROM table_name", conn)
```

3. **Check column names**:
```python
# Get column info
inspector = inspect(engine)
columns = inspector.get_columns('table_name')
column_names = [col['name'] for col in columns]
print(f"Available columns: {column_names}")
```

## Debugging Tips

### Enable SQL Logging

```python
import logging

# Enable SQLAlchemy logging
logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)

# Now all SQL statements are printed
engine = create_engine(connection_string, echo=True)
df = pd.read_sql(query, engine)
```

### Test Connection

```python
def test_connection(engine):
    """Test database connection and gather info"""
    try:
        with engine.connect() as conn:
            # Test query
            result = conn.execute(text("SELECT 1"))
            print("✓ Connection successful")
            
            # Database version
            if 'postgresql' in engine.url.drivername:
                version = conn.execute(text("SELECT version()")).scalar()
            elif 'mysql' in engine.url.drivername:
                version = conn.execute(text("SELECT VERSION()")).scalar()
            elif 'sqlite' in engine.url.drivername:
                version = conn.execute(text("SELECT sqlite_version()")).scalar()
            
            print(f"✓ Database version: {version}")
            
            # List tables
            inspector = inspect(engine)
            tables = inspector.get_table_names()
            print(f"✓ Found {len(tables)} tables: {tables[:5]}...")
            
            return True
    except Exception as e:
        print(f"✗ Connection failed: {e}")
        return False

test_connection(engine)
```

### Profile Query Performance

```python
import time

def profile_query(query, engine):
    """Profile query execution time"""
    start = time.time()
    df = pd.read_sql(query, engine)
    elapsed = time.time() - start
    
    print(f"Rows: {len(df)}")
    print(f"Columns: {len(df.columns)}")
    print(f"Memory: {df.memory_usage(deep=True).sum() / 1024 / 1024:.2f} MB")
    print(f"Time: {elapsed:.2f} seconds")
    
    return df
```
