# 3.5 Describe cost optimization concepts and best practices in Snowflake

## Attribute cost

### Use of ACCOUNT_USAGE schema

- Within SNOWFLAKE shared database
- Enable querying object metadata and historical usage data for account and all associated reader accounts (READER_ACCOUNT_USAGE)
- Two key schemas
    - Account_usage - provide comprehensive metadata at account level
    - READER_ACCOUNT_USAGE - smaller subset of views scoped to reader account created
- Key views
    - Query and performance - QUERY_HISTORY, WAREHOUSE_METERING_HISTORY, WAREHOUSE_LOAD_HISTORY
    - Storage - STORAGE_USAGE, DATABASE_STORAGE_USAGE_HISTORY, TABLE_STORAGE_METRICS
    - Security & Access - LOGIN_HISTORY, ACCESS_HISTORY, GRANTS_TO_USERS, GRANTS_TO_ROLES
    - Objects - TABLES, VIEWS, DATABASES, SCHEMAS, STAGES, FUNCTIONS
    - Data loading - LOAD_HISTORY, COPY_HISTORY, PIPE_USAGE_HISTORY
    - Tasks & pipelines - TASK_HISTORY, COMPLETE_TASK_GRAPHS
    - Governance - POLICY_REFERENCES, TAG_REFERENCES, DATA_TRANSFER_HISTORY

Key Differences vs information schema

|  | ACCOUNT_USAGE | INFORMATION_SCHEMA |
| --- | --- | --- |
| SCOPE | Account-wide (all databases) | Per database |
| DROPPED OBJECTS | Included | Not included |
| DATA RETENTION | 365 days | 7 days - 6 months |
| LATENCY | 45 mins - 3 hours | Real-time |
| ACCESS | ACCOUNTADMIN only (by default) | Role-based |

## Cost considerations when moving data among regions

### Replication

- Enables data, databases, and account objects to be copied and synchronised across multiple accounts, regions, or cloud providers for disaster recovery and business continuity
- Support incremental refreshes of data, with options to manage costs by scheduling, and provides automated failover for Business Critical Edition accounts
- Key aspects
    - Primary/secondary model - data flows from primary account (read/write) to secondary account (read-only)
    - Objects supported - databases, shares, users, roles and warehouses
    - Cost - billed on target account, covering data transfer and compute
- Setting up
    - Enable replication - use SQL command ALTER ACCOUNT SET PARAMETER ENABLE_ACCOUNT_DATABASE_REPLICATION = TRUE on both source and target accounts or via Snowflake UI
    - Create groups - create replication/failover group to select with databases, roles, and objects are included
    - Refresh - schedule automatic refreshes or manuall refresh via ALTER DATABASE … REFRESH
- Considerations
    - Temporary, external, and hybrid tables are not supported
    - Replication acts as incremental pull rather than continuous streaming, save cost but introduce time lag based on schedule
    - Best for cross-regional, cross-cloud or cross-business unit data availability

## Cost considerations when using serverless features

```sql
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE my_role;
GRANT SELECT ON ALL VIEWS IN SCHEMA SNOWFLAKE.ACCOUNT_USAGE TO ROLE my_role;
```

## Cost considerations when using serverless features

- Snowflake serverless features (Snowpipe, Automatic Clustering, Materialised views) optimise performance by automatically scaling resources managed by Snowflake, billing per-second based on usage rather than fixed warehouse uptime
- Key cost considerations include 1-credit-per-compute-hour rate, potential high costs during high data churn, and additional Cloud Services fees
- Key serverless cost considerations
    - Usage-based billing - unlike warehouse, serverless features uses Snowflake-managed compute, billed per second with no minimum 60-second duration for some feature
    - Multipler effect - feature use different multiplier for credit consumption
        - 1X - Snowpipe, Snowpipe streaming, query acceleration, replication
        - 2X - materialised views, search optimisation service
    - High churn impact - feature like automatic clustering and search optimisation cost more when tables have high data volatility (frequent insert/update/delete)
    - Cloud services layer - serverless task use cloud service layer, charged if daily usage exceeds 10% of daily virtual warehouse usage
    - Maintenance overhead - materialised views and search optimisation require continuous background maintenance, which accumulates costs over time, regardless of whether they are actively queried
- How to optimise
    - Monitor cost - ACCOUNT_USAGE and ORGANISATION_USAGE views to track specific sertverless feature comsumption
    - Right-size feature - use clustering for high-latency table queries but avoid over-clustering which increases maintenance costs
    - Manage materialised views - create on table that are queried frequently but changed infrequently to minimise maintenance cost
    - Use resource monitor - set up monitor to notify or suspend services when usage thresholds are met