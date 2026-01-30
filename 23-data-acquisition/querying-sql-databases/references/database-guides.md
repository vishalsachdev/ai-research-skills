# Database-Specific Guides

Detailed guides for working with specific database systems using pandas and SQLAlchemy.

## PostgreSQL

PostgreSQL is a powerful open-source relational database with advanced features.

### Installation

```bash
# Install PostgreSQL driver
pip install psycopg2-binary

# Or use psycopg2 (requires compilation)
pip install psycopg2
```

### Connection

```python
from sqlalchemy import create_engine

# Basic connection
engine = create_engine('postgresql://user:password@localhost:5432/mydb')

# With specific driver
engine = create_engine('postgresql+psycopg2://user:password@localhost/mydb')

# Production configuration
engine = create_engine(
    'postgresql://user:password@localhost/mydb',
    pool_size=20,
    max_overflow=40,
    pool_recycle=3600,
    pool_pre_ping=True,
    connect_args={
        'connect_timeout': 10,
        'application_name': 'my_data_app'
    }
)
```

### PostgreSQL-Specific Features

#### COPY Command for Fast Bulk Loading

```python
from io import StringIO
import pandas as pd

# Prepare data
df = pd.DataFrame({
    'id': range(1, 100001),
    'name': [f'User {i}' for i in range(1, 100001)],
    'value': range(100000)
})

# Method 1: Using COPY (Fastest)
csv_buffer = StringIO()
df.to_csv(csv_buffer, index=False, header=False, sep='\t')
csv_buffer.seek(0)

with engine.raw_connection() as conn:
    cursor = conn.cursor()
    # COPY is 10-100x faster than INSERT
    cursor.copy_from(csv_buffer, 'my_table', sep='\t', null='')
    conn.commit()

# Method 2: Using copy_expert with custom options
csv_buffer = StringIO()
df.to_csv(csv_buffer, index=False, header=True)
csv_buffer.seek(0)

with engine.raw_connection() as conn:
    cursor = conn.cursor()
    cursor.copy_expert(
        "COPY my_table FROM STDIN WITH CSV HEADER",
        csv_buffer
    )
    conn.commit()
```

#### PostgreSQL Data Types

```python
from sqlalchemy.dialects.postgresql import (
    UUID, JSONB, ARRAY, HSTORE, INET, CIDR
)

df.to_sql(
    'postgres_table',
    engine,
    dtype={
        'id': UUID,
        'data': JSONB,
        'tags': ARRAY(String),
        'config': HSTORE,
        'ip_address': INET
    }
)
```

#### Working with JSON/JSONB

```python
import pandas as pd
from sqlalchemy import text

# Query JSONB columns
query = text("""
    SELECT 
        id,
        data->>'name' as name,
        data->'address'->>'city' as city,
        data->'tags' as tags
    FROM users
    WHERE data->>'status' = :status
""")

df = pd.read_sql(query, engine, params={'status': 'active'})

# Insert DataFrames with JSON columns
import json

df = pd.DataFrame({
    'id': [1, 2],
    'data': [
        json.dumps({'name': 'John', 'age': 30}),
        json.dumps({'name': 'Jane', 'age': 25})
    ]
})

from sqlalchemy.dialects.postgresql import JSONB
df.to_sql('users', engine, dtype={'data': JSONB})
```

#### Arrays

```python
# Query arrays
query = text("""
    SELECT id, tags
    FROM products
    WHERE 'electronics' = ANY(tags)
""")

df = pd.read_sql(query, engine)

# Insert arrays
from sqlalchemy.dialects.postgresql import ARRAY

df = pd.DataFrame({
    'id': [1, 2],
    'tags': [['electronics', 'computers'], ['books', 'fiction']]
})

df.to_sql('products', engine, dtype={'tags': ARRAY(String)})
```

#### Full Text Search

```python
# Create full text search index
with engine.begin() as conn:
    conn.execute(text("""
        CREATE INDEX idx_fts ON articles 
        USING GIN (to_tsvector('english', title || ' ' || body))
    """))

# Query with full text search
query = text("""
    SELECT id, title, ts_rank(to_tsvector('english', title || ' ' || body), 
                               to_tsquery('english', :search)) as rank
    FROM articles
    WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('english', :search)
    ORDER BY rank DESC
""")

df = pd.read_sql(query, engine, params={'search': 'data & science'})
```

## MySQL/MariaDB

MySQL is a popular open-source relational database.

### Installation

```bash
# Install MySQL driver
pip install pymysql

# Or use mysqlclient (faster, requires compilation)
pip install mysqlclient
```

### Connection

```python
# Using PyMySQL
engine = create_engine('mysql+pymysql://user:password@localhost/mydb')

# Using mysqlclient
engine = create_engine('mysql+mysqldb://user:password@localhost/mydb')

# With charset (important for Unicode)
engine = create_engine('mysql+pymysql://user:password@localhost/mydb?charset=utf8mb4')

# Production configuration
engine = create_engine(
    'mysql+pymysql://user:password@localhost/mydb',
    pool_size=10,
    max_overflow=20,
    pool_recycle=3600,  # MySQL drops idle connections after 8 hours
    pool_pre_ping=True,  # Check connection health
    connect_args={
        'charset': 'utf8mb4',
        'connect_timeout': 10
    }
)
```

### MySQL-Specific Features

#### Handling Connection Timeouts

```python
# MySQL drops connections after wait_timeout (default 8 hours)
# Always use pool_recycle
engine = create_engine(
    connection_string,
    pool_recycle=3600,  # Recycle connections every hour
    pool_pre_ping=True  # Verify connection before use
)
```

#### LOAD DATA INFILE for Fast Bulk Loading

```python
import pandas as pd

# Export to CSV
df.to_csv('/tmp/data.csv', index=False)

# Use LOAD DATA INFILE (much faster than INSERT)
with engine.begin() as conn:
    conn.execute(text("""
        LOAD DATA LOCAL INFILE '/tmp/data.csv'
        INTO TABLE my_table
        FIELDS TERMINATED BY ','
        ENCLOSED BY '"'
        LINES TERMINATED BY '\n'
        IGNORE 1 ROWS
    """))
```

#### Working with MySQL JSON Type

```python
from sqlalchemy.dialects.mysql import JSON

# MySQL 5.7+ supports JSON type
df.to_sql('users', engine, dtype={'metadata': JSON})

# Query JSON
query = text("""
    SELECT id, metadata->>'$.name' as name
    FROM users
    WHERE JSON_EXTRACT(metadata, '$.age') > :min_age
""")

df = pd.read_sql(query, engine, params={'min_age': 18})
```

#### ON DUPLICATE KEY UPDATE

```python
# Upsert pattern with ON DUPLICATE KEY UPDATE
from sqlalchemy import text

with engine.begin() as conn:
    # Insert or update
    conn.execute(text("""
        INSERT INTO products (id, name, price, stock)
        VALUES (:id, :name, :price, :stock)
        ON DUPLICATE KEY UPDATE
            price = VALUES(price),
            stock = VALUES(stock)
    """), [
        {'id': 1, 'name': 'Product A', 'price': 19.99, 'stock': 100},
        {'id': 2, 'name': 'Product B', 'price': 29.99, 'stock': 50}
    ])
```

## SQLite

SQLite is a lightweight, file-based database ideal for development and small applications.

### Connection

```python
# File-based database
engine = create_engine('sqlite:///path/to/database.db')

# In-memory database (fast, for testing)
engine = create_engine('sqlite:///:memory:')

# Multi-threading support
engine = create_engine(
    'sqlite:///database.db',
    connect_args={'check_same_thread': False}
)
```

### SQLite-Specific Features

#### No Connection Pooling Needed

```python
# SQLite doesn't benefit from connection pooling
# Use NullPool for scripts
from sqlalchemy.pool import NullPool

engine = create_engine('sqlite:///db.db', poolclass=NullPool)
```

#### Enable Foreign Keys

```python
# SQLite doesn't enforce foreign keys by default
from sqlalchemy import event

@event.listens_for(engine, "connect")
def set_sqlite_pragma(dbapi_conn, connection_record):
    cursor = dbapi_conn.cursor()
    cursor.execute("PRAGMA foreign_keys=ON")
    cursor.close()
```

#### Backup Database

```python
import sqlite3
import shutil

# Method 1: File copy (database must not be in use)
shutil.copy('database.db', 'backup.db')

# Method 2: Using SQLite backup API
source = sqlite3.connect('database.db')
backup = sqlite3.connect('backup.db')
source.backup(backup)
backup.close()
source.close()
```

#### Full Text Search with FTS5

```python
# Create FTS5 virtual table
with engine.begin() as conn:
    conn.execute(text("""
        CREATE VIRTUAL TABLE articles_fts USING fts5(title, body)
    """))
    
    # Insert data
    conn.execute(text("""
        INSERT INTO articles_fts(title, body)
        SELECT title, body FROM articles
    """))

# Full text search
query = text("""
    SELECT * FROM articles_fts
    WHERE articles_fts MATCH :search
    ORDER BY rank
""")

df = pd.read_sql(query, engine, params={'search': 'data science'})
```

## Microsoft SQL Server

### Installation

```bash
# Install ODBC driver
pip install pyodbc

# On Linux, install Microsoft ODBC driver first:
# https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server
```

### Connection

```python
# Using Windows Authentication
engine = create_engine(
    'mssql+pyodbc://host/database?driver=ODBC+Driver+17+for+SQL+Server&trusted_connection=yes'
)

# Using SQL Server Authentication
engine = create_engine(
    'mssql+pyodbc://user:password@host/database?driver=ODBC+Driver+17+for+SQL+Server'
)

# Production configuration
engine = create_engine(
    connection_string,
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
    connect_args={
        'timeout': 30,
        'autocommit': False
    }
)
```

### SQL Server-Specific Features

#### Bulk Insert with BCP

```python
# Export to CSV
df.to_csv('data.csv', index=False)

# Use BCP utility (fastest for SQL Server)
import subprocess

subprocess.run([
    'bcp', 'dbo.my_table', 'in', 'data.csv',
    '-S', 'server_name',
    '-d', 'database_name',
    '-U', 'username',
    '-P', 'password',
    '-c',  # Character format
    '-t,',  # Field terminator
    '-F', '2'  # Skip header row
])
```

#### Working with IDENTITY Columns

```python
# Insert with IDENTITY_INSERT
with engine.begin() as conn:
    conn.execute(text("SET IDENTITY_INSERT my_table ON"))
    df.to_sql('my_table', conn, if_exists='append', index=False)
    conn.execute(text("SET IDENTITY_INSERT my_table OFF"))
```

#### Pagination with OFFSET/FETCH

```python
# SQL Server 2012+ pagination
query = text("""
    SELECT * FROM large_table
    ORDER BY id
    OFFSET :offset ROWS
    FETCH NEXT :limit ROWS ONLY
""")

page_size = 1000
for page in range(10):
    df = pd.read_sql(
        query,
        engine,
        params={'offset': page * page_size, 'limit': page_size}
    )
    process(df)
```

## Cloud Databases

### Amazon RDS (PostgreSQL/MySQL)

```python
# RDS PostgreSQL
engine = create_engine(
    'postgresql://user:password@mydb.abc123.us-east-1.rds.amazonaws.com:5432/mydb',
    pool_pre_ping=True,
    connect_args={
        'connect_timeout': 10,
        'sslmode': 'require'  # Enforce SSL
    }
)

# RDS MySQL
engine = create_engine(
    'mysql+pymysql://user:password@mydb.abc123.us-east-1.rds.amazonaws.com/mydb',
    pool_recycle=3600,
    pool_pre_ping=True,
    connect_args={
        'ssl': {'ssl_ca': '/path/to/rds-ca-cert.pem'}
    }
)
```

### Google Cloud SQL

```python
# Using Cloud SQL Proxy
engine = create_engine(
    'postgresql://user:password@localhost:5432/mydb'  # Connect through proxy
)

# Direct connection with SSL
engine = create_engine(
    'postgresql://user:password@cloud-sql-ip/mydb',
    connect_args={
        'sslmode': 'require',
        'sslrootcert': 'server-ca.pem',
        'sslcert': 'client-cert.pem',
        'sslkey': 'client-key.pem'
    }
)
```

### Azure SQL Database

```python
# Azure SQL Database
engine = create_engine(
    'mssql+pyodbc://user@server:password@server.database.windows.net/database?driver=ODBC+Driver+17+for+SQL+Server&Encrypt=yes&TrustServerCertificate=no'
)
```

## Performance Comparison

### Bulk Insert Performance (100,000 rows)

| Method | PostgreSQL | MySQL | SQLite | SQL Server |
|--------|-----------|-------|--------|------------|
| to_sql() default | 45s | 38s | 25s | 42s |
| to_sql(method='multi') | 12s | 10s | 8s | 11s |
| COPY/LOAD DATA | 0.5s | 0.8s | - | - |
| BCP | - | - | - | 0.6s |

### Query Performance Tips by Database

**PostgreSQL**:
- Use EXPLAIN ANALYZE to check query plans
- Create appropriate indexes
- Use VACUUM ANALYZE regularly
- Consider partitioning for very large tables

**MySQL**:
- Use EXPLAIN to check query plans
- InnoDB is default and recommended
- Optimize table periodically
- Use composite indexes wisely

**SQLite**:
- Create indexes for queries
- Use ANALYZE to update statistics
- Consider WAL mode for concurrent access
- Keep database file on fast storage

**SQL Server**:
- Use execution plans
- Update statistics regularly
- Consider columnstore indexes for analytics
- Use filtered indexes where appropriate

## Connection String Examples

### With Environment Variables

```python
import os
from sqlalchemy import create_engine

# Read from environment
db_url = os.getenv('DATABASE_URL')
engine = create_engine(db_url)

# Or construct from parts
engine = create_engine(
    f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASS')}@"
    f"{os.getenv('DB_HOST')}:{os.getenv('DB_PORT')}/{os.getenv('DB_NAME')}"
)
```

### With URL Encoding

```python
from urllib.parse import quote_plus

# Encode password with special characters
password = "p@ssw0rd!#$%"
encoded_password = quote_plus(password)

engine = create_engine(
    f'postgresql://user:{encoded_password}@host/database'
)
```

### Connection Pooling Recommendations

| Use Case | pool_size | max_overflow | pool_recycle |
|----------|-----------|--------------|--------------|
| Script/Notebook | 1-2 | 0-2 | 3600 |
| Web App | 20-40 | 40-80 | 3600 |
| Background Jobs | 5-10 | 10-20 | 1800 |
| Analytics | 5-10 | 5-10 | 3600 |
