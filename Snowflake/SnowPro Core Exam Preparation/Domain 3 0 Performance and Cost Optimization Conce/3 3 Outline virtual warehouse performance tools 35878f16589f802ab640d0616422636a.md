# 3.3 Outline virtual warehouse performance tools

## Query Acceleration Service

- Speed up long-running, scan-heavy queries by offloading heavy lifting to shared, serverless compute resources

**Core functionality**

- Targeted acceleration - identify parallelizable parts of query plan (large table scans and selective filters) and process them using extra compute capacity outside virtual warehouse
- Automatic detection - snowflake automatically determines if query is eligible for acceleration
- Dynamic scaling - temporary scale-up for specific outlier queries, prevent them from blocking other workload or requiring permanent increase in warehouse size

**Benefits**

- Cost efficiency - billed per second for the serverless resources used, no markup over regular warehouse charges (*cheaper than over-provisioning large warehouse for occasional heavy loads)*
- Improved throughput - offload heavy queries - reduce overall queueing and latency
- No code change - enabled on warehouse, QAS works automatically

**Implementation and Limit**

- Requires Enterprise Edition or higher

```sql
ALTER WAREHOUSE my_wh SET ENABLE_QUERY_ACCELERATION = TRUE;
```

- Scale factor - control costs by setting QUERY_ACCELERATION_MAX_SCALE_FACTOR (default is 8 - allow warehouse to lease up to 8x its normal compute power for specific query)
- Eligibility check - use SYSTEM$ESTIMATE_QUERY_ACCELERATION to see if past query would benefit from QAS

**Use case**

- Ideal - ad-hoc analytics, unpredictable data volumes, queries scanning hundreds of GB or TB of data with highly selective filters
- Less ideal - queries with small scans, heavy sorting or grouping on high-cardinality columns where data reduction is minimal

## Scaling up vs scaling out

**Scale up - bigger warehouse**

- Changing to a larger warehouse size (small → large → x-large)
- Size doubles up compute resources (and credit consumption)
- Good for complex, heavy queries that need more CPU and memory (eg. large joins, heavy aggregations, data spilling issues, window functions)
- Change is new-instant
- When
    - Query slow due to complexity not volume
    - Data spillage to local/remote disk
    - Single large query is bottleneck

**Scale out - more clusters**

- Adding more warehouse clusters running in parallel
- Additional clusters is the same size as original
- Good for high concurrency
- Helps
    - Query queueing, peak usage periods, many simultaneous users
- Enterprise and above