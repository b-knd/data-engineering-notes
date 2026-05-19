# 3.1 Explain the use of the Query Profile

## Data Spilling

- When warehouse runs our of memory while executing query, memory bytes spill onto local disk storage, if need even more, spills onto remote cloud-provider storage → worse performance
- Worse to best performance: remote storage (slow) → local disk → in-memory (fast)
- Where to check
    - *snowflake.account_usage.query_history* filtering *bytes_spilled_to_local_storage/bytes_spilled_to_remote_storage*
    - Only ACCOUNTADMIN roles has the privileges needed by default
- How to fix it
    - Use larger warehouse
    - Processing data smaller batches
    - Use query profile to identify which specific operation nodes causing the spill and optimise
- *Note: when QAS is enabled, Snowflake writes small amount of data to remote storage for each eligible query even if QAS not used*

## Micro-partition pruning

- Mechanism to skip unnecessary storage blocks during query
- Role of metadata
    - Snowflake records minimum and maximum values for every column within each micro-partition, along with number of distinct values and other optimization properties
    - Optimiser job: query optimiser compares filter to min/max metadata of every partition (if out of range, partition is pruned)
- Factors that improved pruning
    - Data clustering - pruning efficient when data sorted/order along natural dimensions (date/geographic regions)
    - Clustering key - to minimise overlapping values and allow more data skipping
    - Clustering depth - smaller the depth, more efficient the pruning

## Query history

- Quick visual check, programmatic access to recent data, or long-term autiding
- Three ways to access
    - Snowsight UI (Monitoring tab)
        - Monitoring >> Query history
        - Displays queries from last 14 days
        - Can filter by user, warehouse, status and date range
        - Can see full SQL text and Query Profile for performance troubleshooting
    - Information schema
        - For real-time monitoring and programmatic checks of recent queries
        - Only for last 7 days
        - Minimal latency, includes queries that are currently running
        - Returns up to 100 queries by default, can be increased up to 10,000 using RESULT_LIMIT argument
    - Account usage (shared view)
        - For long-term auditing, reporting and cost analysis
        - Query SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
        - Stores up to 365 dats
        - Typical 45 minute delay before data appears in this view

```sql
SELECT * FROM table(information_schema.query_history())
ORDER BY start_time DESC;
```