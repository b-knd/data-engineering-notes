# 1.1 Outline key features of the Snowflake AI Data Cloud

## What is Snowflake

- A self-managed service
- Advantages
    - No need manual set-up, configurations or managements of any software or hardware
    - All maintenance, upgrades, management and tuning handled by Snowflake/

## Snowflake Interoperable Storage

- Making data accessible across multiple engines, clouds and catalogs - without copying or moving data
- Key table types for interoperability
    - Iceberg tables - open format, Snowflake/externally managed, full ACID + snapshots
        - Combine snowflake query performance with external cloud storage
        - Open format features
            - ACID transaction
            - Schema evolution
            - Hidden partitioning
            - Table snapshots
        - Storage model
            - Data in Apache Parquet files
            - Stored on cloud storage
            - No Snowflake storage cost
            - No fail-safe period
        - Connectivity
            - External volume (*named account-level object, connect Snowflake to cloud storage via IAM; required for all iceberg tables)*
            - Catalog integration object (*named account-level object, stores metadata info when NOT using Snowflake as catalog)*
            - Cross-cloud/region supported
            - Egress cost may apply
    - External tables - query data in S3/GCS/Azure without ingestion
    - Dynamic tables - support iceberg as storage format to simplify data processing for data lakes and lakehouses

## Snowflake elastic compute

- Snowflake’s ability to scale compute resources independently from storage
- Elastic compute, exposed as virtual warehouses
- Different sizing depending of the amount of compute power needed
- Dynamically sized multi-cluster warehouses are also available (enterprise onwards)
- Leveraging elasticity for performance
    - Vertical scaling: adjust warehouse size to handle more complex queries and large data volumes (scale down/up)
    - Horizontal scaling: adjust number of clusters in warehouse to manage workload concurrency (scale in/out)
    - Query acceleration services: augmenting warehouse with serverless resources to handle unpredictable, outlier queries (enterprise edition onwards)

## Snowflake architecture/ Snowflake Layers

- Hybrid of **shared-disk** and **shared-nothing** architecture
- Shared-disk architecture
    - Central data repository for persisted data
    - Accessible from all compute nodes in platform
    - Pro: data management simplicity
    - Which layers: Storage + Cloud service layer
- Shared-nothing architecture
    - Massively parallel processing compute clusters
    - Each node stores portion of data set locally
    - Pro: not tightly coupled, flexibility to scale in/out

### Database Storage

- Types of data
    - Structured data — such as rows and columns in a table (tabular schema)
    - Semi-structured data - such as JSON file or XML file (flexible schema)
    - Unstructured data - documents, images or audio file (no inherent schema)
- Types of tables
    - [Snowflake tables](https://docs.snowflake.com/en/user-guide/tables-micro-partitions) - will cover this later
        - Internally optimised, compressed, columnar format
        - Automatically handled - organisation, file size, structure, compression, metadata, and statistics
        - Automatically divided into micro-partitions (continuous units of storage) to improve efficiency
        - Can handle all data type (including unstructured through FILE data type) - cover this later
    - [Apache Iceberg table](https://docs.snowflake.com/en/user-guide/tables-iceberg)
        - Combined performance and query semantics of typical snowflake tables + external cloud storage
        - Ideal for existing data lakes/lakehouses that are not residing in Snowflake
        - Data stored in external cloud storage location: Amazon S3, Google Cloud Storage, Microsoft Azure Storage
        - Both structured/semi-structured data
    - [Hybrid tables](https://docs.snowflake.com/en/user-guide/intro-key-concepts#:~:text=more%20information%2C%20see-,Hybrid%20tables,-.)
        - Optimized for low latency and high throughput by using index-based random reads and writes
        - Supports row locking and enforce unique and referential integrity constraints (transactional workload)

### Compute

- [Virtual warehouse](https://docs.snowflake.com/en/user-guide/warehouses) - cluster of compute resources
    - Process SQL statement
    - Run code in Java/Python/Scala through Snowpark
    - Shared-nothing architecture, each clusters don’t share resources with other warehouses
    - Performance are independent of other warehouses

### Cloud Services

- Manage services:
    - Security, authentication and access control
    - Snowflake Horizon Catalog
    - Infrastructure management with cloud platforms
    - Metadata management, including SNOWFLAKE database and information schema
    - Query parsing and optimization
    - Regulatory compliance

## Overview of Snowflake Editions

### Standard Edition

- Introductory level offering
- Full, unlimited access to all Snowflake’s standard features

### Enterprise Edition

- All from standard edition
- Designed for needs of large-scale enterprises and organisations
- From this onwards:
    - 24-hour early access to weekly new release - to be used for testing or validation before deployment of release
    - Security and governance
        - Synthetic data
        - Extended time travel
        - Periodic rekeying of encrypted data
        - Columnar/Row-level security
        - Aggregation policy
        - Projection policies
        - Data classification
        - ACCESS_HISTORY
    - Compute resource management
        - Multi-cluster virtual warehouse
    - SQL
        - Query acceleration
        - Search optimization
        - Materialized views

### Business Critical Edition

- Formerly known as Enterprise for Sensitive Data (ESD)
- High level of data protection to support extreme sensitive data (PHI) that must comply with HIPAA and HITRUST CSF
- Account failover/failback to support business continuity and disaster recovery
- *Required by HIPAA and HITRUST, before any PHI data can be stored, a signed business associate agreement (BAA) must be in place between agency and Snowflake Inc.*
- From this onwards:
    - Security and governance
        - Customer-managed encryption key
        - Private connection to Snowflake service/Snowflake internal stages/external stages/external volumes for Apache Iceberg tables
        - Private connection through AWS PrivateLink, Azure Private Link or Google Cloud Private Service Connect
        - Cross region connectivity for AWS PrivateLink
        - PHI data/PCI CSS
    - Data replication and failover
        - Failover and failback between snowflake account
        - Redirecting client connection
    - Data sharing
        - Create and manage own Snowflake Data Clean Rooms

### Virtual Private Snowflake (VPS)

- Highest lever of security
- Use case: financial institutions/ large enterprises that collect, analyse and share highly sensitive data
- A completely separate Snowflake environemnt, isolated from all other Snowflake account (no hardware resource sharing)
- Only this:
    - Security and governance
        - Dedicated metadata store and pool of compute resources