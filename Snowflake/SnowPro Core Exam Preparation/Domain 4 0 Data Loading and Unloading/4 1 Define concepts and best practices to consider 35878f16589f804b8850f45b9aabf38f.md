# 4.1 Define concepts and best practices to consider when loading data

## DML Error Logging

- By default, when running set of DML statements and one of the statement fails, DML ends and changes rolled back
- To continue and log error, turn on DML error logging for table (that table = base table)
- Condition to enable
    - ERROR_LOGGING = TRUE
    - OPT_OUT_ERROR_LOGGING = FALSE

**Types of errors logged**

- NOT NULL constraint violations
- Type conversion errors
- Incompatible precision/scale/length values
- Expression failures like division by zero

**Error table**

- Automatically created linked to base table
- Each logged row captured timestamp, query ID, error code, error metadata and actual data causing failure
- Only base table owner/ users granted ‘SELECT ERROR TABLE’ privilege can query
- Can only run SELECT and TRUNCATE on error tables

## Snowpipe

**Snowpipe**

Continuous, serverless data ingestion service that loads data in micro-batches from cloud storage (S2, Azure, GCS) into tables as soon as files are available

- Support semi-structure data (JSON, Avro)
- Components and features
    - Pipe object - snowflake object containing COPY statement that defines data source and target table
    - Automated ingestion - use cloud messaging to detect new files instantly, enabling serverless loading
    - REST API - REST endpoint to notify Snowflake of new files allowing customised routine
    - Snowpipe streaming - alternative for low-latency, row-based ingestion directly into table without staging files first
- How it works
    - Files loaded into external cloud storage stage
    - Cloud storage notifies Snowflake of new file
    - Snowpipe reads file using COPY statement and load into target table
    - Ingested files tracked for period to prevent reloading

**Snowpipe streaming**

- Snowflake real-time ingestion service
- Enable applications to load streaming data directly into Snowflake tables as rows arrive without staging files/managing intermediate storage
- Data available for query within seconds of ingestion
- Performance stats:
    - Up to 10GB/s throughput per table, as low as 5 second end-to-end ingest-to-query latency
- Key features
    - Exactly-once delivery - build-in offset token tracking prevents duplicate data and data loss
    - Ordered ingestion - rows ingested in order, maps naturally to source partitions
    - In-flight transformations - cleanse, reshape, filter rows, reorder columns, cast types, and apply expression during ingestion with no separate ETL steps needed
    - Schema evolution - add new columns detected in incoming stream without manual DDL changes
    - Pre-clustering at ingest time - sort data during ingestion for optimised query performance on tables with clustering keys
    - Iceberg table support - stream data directly into Snowflake-managed Iceberg tables including v2 and v3
    - Serverless - compute scales automatically, no infrastructure to manage

**Snowpipe vs Snowpipe streaming**

|  | Snowpipe | Snowpipe Streaming |
| --- | --- | --- |
| Data form | Files | Rows |
| Ordering | Not supported | Ordered per channel |
| Transformations | Not supported | In-flight |