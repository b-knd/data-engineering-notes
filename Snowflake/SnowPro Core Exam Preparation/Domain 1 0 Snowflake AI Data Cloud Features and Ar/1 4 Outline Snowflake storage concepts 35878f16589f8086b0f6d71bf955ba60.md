# 1.4 Outline Snowflake storage concepts

## Micro-partitions

- Automatically dividing of data into small, contiguous units of storage, each 50MB - 500MB of uncompressed data
- Rows grouped into micro-partitions and stored in columnar fashion
- Snowflake maintain metadata for each partition including column ranges, number of distinct values and additional properties for optimisation and efficiency
- Benefits
    - Automatic
    - No skew - can overlap to avoid data skew
    - Columnar storage - compressed independently, only referenced columns get scanned
    - Fine-grained pruning - ideal filtering of 10% value should ideally scan 10% partitions

## Data Clustering

- Optimise large-table query performance by organising data into sorted, 50-500MB micro-partitions
- Use case: good for frequently queried, slowly changing tables as it consumes credits for maintenance
- Key aspects
    - Micro-partitions - snowflake divide data into contiguous, columnar based unit to allow fine-grained pruning of data when querying large table
    - Clustering keys - specific columns/expressions to act as key, organising data within micro-partitions to speed up filter, join or aggregation operations
    - Automatic clustering - snowflake manage reclustering process after DML to ensure optimisation is maintained
    - Performance and costs - cost incurred for maintenance
    - Best practice - max of 3-4 columns/expressions is recommended

```sql
CREATE TABLE ... CLUSTER BY (column1, column2);
ALTER TABLE ... CLUSTER BY (column1, column2);
```