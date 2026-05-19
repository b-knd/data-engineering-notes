# 1.3 Outline Snowflake’s catalog and objects

## Stages

Secure, temporary storage locations used to load or unload data between external storage and Snowflake tables

**Internal stages - managed by Snowflake**

- User Stage (~)
    - Unique, private stage assigned to every user
    - Ideal for personal data staging
- Table Stage (%)
    - Automatically created for every table
    - Ideal for loading data specifically into that table
- Named Internal Stage
    - Explicitly created using ‘CREATE STAGE’
    - Flexibility to share among users and tables

**External Stages - pointing to external buckets**

- Referenced data stored in external cloud storage
- Require external stage objects to hold location and credential details

```sql
CREATE STAGE ...
ALTER STAGE ...
DROP STAGE ...
LIST @<stage_name>
```

## Table types

**Permanent Table (default)**

- Standard table type
- Persist until explicitly dropped
- Full 7-days Fail-safe period + up to 90 days Time Travel (Standard only 1 day)

**Temporary Table**

- Session-based
- Not visible to other users/sessions
- Time travel: 0/1 (capped at session length)
- No fail safe
- Use case: ETL staging, session-specific intermediate data
- *Note: temporary table takes precedence over any permanent table with same name in same schema within session*

**Transient Table**

- Persist until explicitly dropped
- Visible to all users with appropriate privileges
- No fail safe period (After time travel period, data is unrecoverable)
- Time travel: 0/1 day
- Use case: large intermediate/staging data, where storage cost matters

**External Table**

- Read-only tables whose files are stored in external stage
- Source of truth stays in external cloud storage
- Use case: query data lake files without ingestion to Snowflake

**Hybrid Table**

- Optimised for read and write operations in transactional and hybrid workloads
- Support primary keys and unique constraints
- Use case: OLTP-style workload + analytical queries
- Cannot be temporary or transient

**Iceberg Table**

- Uses Apache Iceberg open table format with data in external cloud storage
- Can use Snowflake as Iceberg catalog, external catalog/create table from files in object storage
- Use case: Open-format interoperability with other data platforms

**Dynamic table**

- Declarative, fully managed table type
- Automatically materialises results of query, keeping data updated based on defined TARGET_LAG
- Automatically handling incremental refreshes, complex transformations

## View Types

Saved queries that act as virtual tables, combining or protecting data without storing it physically

**Standard Views**

- Created on-demand
- Execute underlying SQL everytime they are accessed
- Real-time data without storing results

**Materialised Views**

- Pre-computed results
- Ideal for frequently accessed, complex queries in single tables
- *Temporary suspended when column is dropped*

**Secure Views**

- Prevent users from seeing underlying DDL or data from tables they cannot access

**Semantic Views**

- Metadata layer for AI and BI tools
- Definition of business metrics and dimensions to ensure data consistency

### Key characteristics

- Virtual nature
    - Don’t store data (except materialised)
    - Data always current but slower execution than materialised views
- Security
    - Protects against data leaks by hiding underlying structure which is vital when sharing data
- Management
    - Created and managed via SQL or Snowsight

```sql
CREATE VIEW ...
CREATE MATERIALIZED VIEW ...
CREATE SECURE VIEW...

# details on existing views
INFORMATION_SCHEMA.VIEWS
```

## Streams

- Objects that enable Change Data Capture, tracking all insertions, updates and deletions on table to facilitate incremental data processing
- Snapshot of changes, allowing eficient ELT/ETL pipelines, and often paired with tasks for automated, real-time data integration
- Key aspects
    - Type of streams
        - Standard - tracks all DML changes
        - Append-only - tracks only row insertion, efficient for append-heavy tables
        - Inserts-only - specifically for tracking insert-only operations on external tables
    - How it works
        - Does not contain actual data, but rather metadata about changes (offset tracking)
        - It add three metadata columns (METADATA$ACTION, METADATA$ISUPDATE, METADATA$ISINSERT) to source records
    - Staleness prevention - streams must be consumed within source table’s time travel retention period, or become stale
    - Use cases - incremental ETL, real-time data sync, and auditing
- Streams can be created to querychange data on the following objects: Standard tables, including shared tables. Views, including secure views, Directory tables, Dynamic tables, Apache Iceberg™ tables with Limitations. Event tables External tables

## User Defined Functions (UDFs)

- Allow user to extend Snowflake’s built-in functionality by writing custom logic in various programming languages
- Types
    - Scalar functions - return one output value for each input rows
    - UDTFs - return set of rows (tabular result) for each input row
    - Aggregate functions - perform calculations across multiple rows to return single summary value
    - Vectorised UDFs - process batches of row as Pandas DataFrames, improving performance for Python-based logics
- Supported languages
    - Python
    - SQL
    - Javascript
    - Java/Scala
- Key features
    - Security - UDFs execute within secure, isolated sandbox on Snowflake server
    - External functions - special type of UDF that can call external APIs or services outside of Snowflake
    - Ease of Use - once registered using CREATE FUNCTION command, they can be called like built-in functions

```sql
CREATE FUNCTION area_of_circle(radius FLOAT)
	RETURNS FLOAT
	AS
		$$
		pi() * radius * radius
		$$;

--calling
SELECT area_of_circle(1.0);
```

## Data Types

**Numeric Data Types**

- Fixed-point number - exact values
    - NUMBER(p, s) (precision up to 38, s for scale)
    - Alias - DECIMAL, NUMERIC, INT, INTEGER, BIGINT, SMALLINT, TINYINT, BYTEINT
- Floating-point numbers - approximate values
    - FLOAT, FLOAT4, FLOAT8 - 64bit double-precision
    - Aliases - DOUBLE, DOUBLE PRECISION, REAL
- DECFLOAT - decimal floating-point type for exact base-10 representation up to 38 digits

**String & Binary data types**

- VARCHAR - variable-length Unicode characters, default length is 16MB with maximum 128MB
    - Alias - STRING, TEXT, NVARCHAR, CHAR, CHARACTER
- BINARY - stores variable-length binary data up to 8MB
    - Alias - VARBINARY

**Date and Time data types**

- DATE
- TIME (up to nanosecond precision)
- TIMESTAMP
    - TIMESTAMP_NTZ - no timezone
    - TIMESTAMP_LTZ - local timezone
    - TIMESTAMP_TX - with timezone

**Semi-structured and Structured data types**

- VARIANT - store any other data type (including JSON-like object)
- ARRAY - list of elements, which can be of a specific structured type
- OBJECT - collection of key-value pairs
- MAP - associates keys with values of specific data type

**Others specialised types**

- BOOLEAN - TRUE, FALSE, NULL
- GEOGRAPHY/GEOMETRY
- VECTOR (for AI/ML embeddings, with max 4096 dimensions)