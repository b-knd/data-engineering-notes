# 3.4 Optimise query performance

## Search Optimisation Service

- Enterprise or higher
- Improves performance of certain types of lookup and analytical queries
- Transparent to users
- What query types - selective point lookup (returning one or small number of distinct rows)
    - Point lookup - equality filters, IN predicates, NULL checks on structured columns
    - Text searches - using SEARCH and SEARCH_IP functions
    - Substring and regex - LIKE, ILIKE, RLIKE
    - Semi-structured data - equality, IN, ARRAY_CONTAINS, ARRAYS_OVERLAP, full-text search, NULL checks on VARIANT/OBJECT/ARRAY
    - Structured data - equality, IN, substring predicates on structured ARRAY/OBJECT/MAP
    - Geospatial queries - selected geospatial function on GEOGRAPHY values
- How it works
    - Service creates and maintains persistent data structure - search access path
        - Track which values might be found in each micro-partition - allowing micro-partitions to be skipped entirely during scans instead of scanning everything
    - Key behavious
        - Build search access path happens in background - but queries not accelerated until fully build
        - When table data changes - maintenance automatically updates search access path (queries might be slower)
        - No virtual warehouse needed to run the maintenance service - but has storage/compute cost

```sql
ALTER TABLE my_table ADD SEARCH OPTIMIZATION;
```

- Check progress via search_optimization_progress column in SHOW TABLES
- Cast on column cannot use search optimization

![image.png](3%204%20Optimise%20query%20performance/image.png)

## Understanding the impact of different types of caching

**Result cache (persisted query results)**

- Query result cached for 24 hours to avoid regeneration of results when nothing changes
- Also can be used to post-process results by layering new query on top of query result
- When cached result reused, 24 hour period reset (*but resetting period only limit up to 31 days, then cache purged and start fresh)*
- For result larger than 100KB, security token to access them expires after 6 hours (new token can be retrieved while cache is still valid) *small results don’t use access token*
- Condition for cache reuse
    - New query matches previous one exactly (different casing/table aliases will not trigger cache reuse)
    - Query doesn’t use non-reusable functions like UUID_STRING, RANDOM, RANDSTR
    - Query doesn’t use external functions or query hybrid tables
    - Underlying table data hasn’t changed (No new DML/DDL operations)
    - Cached result hasn’t expired
    - Role has required privileges on all tables used in cached query
- Control
    - By default, caching is enabled, b
    - To disable, set USE_CACHED_RESULT at user/account level
- Post-processing cached results
    - Use RESULT_SCAN table function to treat previous query’s result as table
    - Useful for post-processing outputs from SHOW, DESCRIBE or CALL
    - Alternative to RESULT_SCAN, use pipe operator (- >>) to process result without having to display initial output

**Warehouse cache (Local SSD Cache/Data Cache)**

- Resides within compute layer
- Consists of raw data from micro-partitions stored on local SSDs of compute nodes
- When query executed, warehouse pulled micro-partitions from central storage layer and stores them locally, subsequent queries can read from local SSD rather than pulling from remote storage
- Lifecycle
    - Purged on suspension
    - *Virtual warehouse = elastic resource, when suspended should release back to shared pool*
    - When warehouse resumed, start with “cold cache”, slow first query
- Cache management
    - Flushing policy: when full, Least-Recently Used (LRU) algo to flush older data and make room for new data
    - Data spilling: if local memory insufficient → remote disk spilling
- Additional notes
    - Pruning: cache works alongside micro-partition pruning to speeds up retrieval of micro-partition
    - Warm queries: run specific query to populate to local SSD to improve subsequent analytical workload