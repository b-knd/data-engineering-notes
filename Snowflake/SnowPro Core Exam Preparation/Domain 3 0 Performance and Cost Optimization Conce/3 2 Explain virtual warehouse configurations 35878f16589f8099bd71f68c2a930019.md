# 3.2 Explain virtual warehouse configurations

## Multi-cluster warehouses

- Only available for Enterprise edition and above
- Allow user to scale compute resources to manage user and query concurrency needs as they change, such as during peak and off hours

### What is multi-cluster warehouse

- By default, virtual warehouse consists of single cluster of compute resources
- When resources are not sufficient to execute all queries submitted, queries will be queue until resources free up
- Multi-cluster warehouse supports allocating (statically/dynamically) additional clusters to make larger pool of compute resources
    - Maximum number of clusters (>1)
        - Highest value depends on warehouse size
    - Minimum number of clusters (≤ maximum)
- Supports same properties and actions as single-cluster warehouses
    - Specify warehouse size
    - Resizing warehouse at any time
    - Auto-suspending running warehouse due to inactivity (applied to all multi-clusters - wait for query to complete executed)
    - Auto-resuming suspended warehouse when new queries are submitted

### Scaling Mode

1. Maximised
    1. specify same value for both maximum and minimum number of clusters (>1)
    2. Snowflake starts all clusters the moment warehouse is started
    3. Use case: control available resource, if known to have large number of concurrent user sessions and queries do not fluctuate significantly
2. Auto-scale
    1. Specify different value for maximum and minimum number of clusters
    2. Snowflake starts/stops clusters as needed
    3. As concurrent sessions increases and query starts queueing up, Snowflake starts additional clusters up to maximum number defined
    4. When load decreases, Snowflake shuts down clusters

### Scaling Policy

To control usage of credits in auto-scale mode

1. Standard
    1. Minimize queueing over conserving credits
    2. When query gets queued, snowflake estimate not enough resources, will increase cluster automatically
        1. *max_cluster_count ≤ 10, snowflake starts new warehouses one by one*
        2. *max_cluster_count > 10, snowflake starts multiple clusters at once*
    3. After sustained period of low load, snowflake shuts down 1 or more least-loaded clusters once queries on them finished
2. Economy
    1. Conserve credit (can allow cluster fully-loaded or even slight queueing)
    2. Only if estimated new cluster can stay busy for ≥ 6 minutes
    3. Snowflake mark shut down if < 6 minutes of work left to do

### Monitoring

1. Compute >> warehouses
2. Select warehouse name
    1. View queries running/queued
    2. Monitoring >> query history to view across multiple warehouses

## Warehouse sizing

- XS (1) → 4XL (128)
- Each size up = doubles compute and cost
- Key behavious
    - Minimum billing - min 60 seconds billed, even if query finishes in 5 seconds
    - Auto-suspend
    - Auto-resume
    - Per-second billing after 60 seconds
    - Resizing - can resize on the fly, running queries finishes on current size, new queries use new size
- Choosing right size
    - Start small ad scale up
    - Use query profile to check if there is spilling
    - Watch for queueing
- Type
    - Standard
    - Snowpark-optimise - ML training, large memory workload (16X more memory per node)
    - Economy - cost-sensitive, non-urgent background jobs (use space capacity)