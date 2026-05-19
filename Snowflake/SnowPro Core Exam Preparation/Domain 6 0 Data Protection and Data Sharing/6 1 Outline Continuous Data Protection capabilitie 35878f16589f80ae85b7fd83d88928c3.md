# 6.1 Outline Continuous Data Protection capabilities

## Fail Safe

- non-configurable 7-day data retention period that begins after Time Travel retention period ends
- Last resort disaster recovery mechanism for permanent tables, allowing Snowflake Support to recover data lost due to extreme system failures or accidental deletion
- Key aspects
    - Purpose - ensure historical data is protected and recoverable following major outage or accidental data loss beyond Time Travel window
    - Duration - fixed 7 days period
    - Access - only Snowflake Support can restore data from Fail-safe
    - Applicability - only applies to Permanent tables, schemas and databases
    - Cost - storage costs for the 7 days

## Time travel

- Data protection feature that allows you to access and restore historical data in tables, schemas, and databases within defined retention period
- Key capabilities
    - Query historical data - view data exactly as it looked at a specific time in past
    - Restore objects - use UNDROP command to immediately recover accidentally deleted tables, schemas or databases
    - Cloning - create snapshot clone of entire database or table at specific point in time for testing or historical records
- Use case
    - AT - to query changes at specified point
    - BEFORE - refer to state immediately preceding specified point
- Method to specified point in time
    - Timestamp - SELECT * FROM my_table AT(TIMESTAMP ⇒ ‘2024-08-24 12:00:00’::timestamp_tz)
    - Offset (time in seconds from now) - SELECT * FROM my_table AT(OFFSET ⇒ -3600)
    - Statement ID - query data as it existed before specific SQL statement was executed using Query ID

**Retention Periods & Costs**

- Length of time data is preserved depends on Snowflake Edition
    - Standard edition - default is 1 day (cannot be increased)
    - Enterprise edition - configurable from 0 to 90 days
- Storage cost - charge for additional data storage to required to maintain these historical “micro-partitions”
- Set by MIN_DATA_RETENTION_TIME_IN_DAYS

## Cloning

**Zero-copy cloning**

- Metadata-only operations
- Creation of exact replicas of databases, schemas and table instantly without physically duplicating underlying data
- Snowflake creates new set of metadata that points to same immutable micro-partitions used by original source object
- Initially sharing same storage block as the source table
- When there is data change, Snowflake creates new micro-partitions to store changes and leave the original partitions untouched
- Incremental costs when data changes
- Limitations
    - Internal named stages are not cloned when schema/database is cloned
    - Error table content won’t be cloned (truncated)
    - Cloned table can be “promoted” using the SWAP WITH to swap metadata of two tables

## Data Encryption

- Comprehensive end-to-end encryption
- Data encrypted by default at rest and in transit using AES 256 bit encryption
- Utilise hierarchical key model - rooted in cloud provider-hosted hardware security modules (HSM) - where root key protect account keys
- Key aspect
    - Default protection - all data, including backups, metadata and files in internal stages automatically encrypted
    - Hierarchical key model - multiple level of keys
    - Tri-secret secure - enable customers to combine their own keys with Snowflake’s keys
    - Data-in-transit - communication between cloud services, compute and storage also encrypted
    - Client-side encryption - supports encrypting data before uploading to stage using tools like AWS_CSE
    - Key pair authentication - offers RSA key pair as secure alternative to password