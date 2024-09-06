# Snowflake Unique Features
Snowflake offers several unique SQL features that are distinct from other traditional databases:
- [Snowflake Time Travel](#snowflake-time-travel)
- [Cloning](#cloning)
- [Streams and Tasks](#streams-and-tasks)
- [Semi-structured Data types](#semi-structured-data-types)
- [Snowflake Scripting](#snowflake-scripting-stored-procedures)
- [Snowflake Data Sharing](#snowflake-data-sharing)
- [Query Tagging](#query-tagging)

## Snowflake Time Travel
Allow querying historical data (up to 90 days depending on fail-safe configurations) as it existed at a previous point in time.
```sql
SELECT * FROM my_table AT (TIMESTAMP => '2023-09-01 00:00:00');
```
_[Back to top](#snowflake-unique-features)_

## Cloning
Snowflake supports zero-copy cloning, allowing the creation of copy of table, schema or database without duplicating the underlying storage
_(Means copy is created without physically duplicating underlying data but create a "view" of same data where changes made are independant)_
```sql
CREATE TABLE my_table_clone CLONE my_table;
```

## Streams and Tasks
Snowflake provides streams to track changes (inserts, updates, deletes) to table
``` sql
CREATE STREAM my_table_streams ON TABLE my_table;
```
Snowflake also provides tasks to automate execution of SQL statements in response to changes in data or on a scheduled basis.
```sql
CREATE TASK my_task
WAREHOUSE = my_warehouse
SCHEDULE = 'USING CRON 0 * * * * UTC'
AS
INSERT INTO another_table SELECT * FROM my_table;
```
_[Back to top](#snowflake-unique-features)_

## Semi-structured Data Types
Snowflake allows working with semi-structured data like JSON, XML and Avro using the `VARIANT`, `OBJECT`, and `ARRAY` data types.
```sql
--Example
SELECT data:VARIANT FROM my_table;
```

## Snowflake Scripting (Stored Procedures)
Snowflake scripting allows to write procedural logic (loops, conditionals) directly
```sql
BEGIN
  LET i INT := 1;
  WHILE i <= 10 DO
    INSERT INTO my_table VALUES (i);
    LET i := i+1;
  END WHILE;
END;
```
_[Back to top](#snowflake-unique-features)_

## Snowflake Data Sharing
Secure data sharing enables sharing live data across accounts without copying data
```sql
CREATE SHARE my_share;
GRANT SELECT ON my_table to SHARE my_share;
```

## Query Tagging
Supports tagging SQL queries to track usage or categorize them for audit purposes
```sql
ALTER SESSION SET QUERY_TAG = 'Report Generation';
SELECT * FROM sales_report;
```
_[Back to top](#snowflake-unique-features)_
