# Types of Snowflake Tables
- [Dynmamic Table](#dymanic-table)
- [Directory Table](#directory-table)
- [Event Table](#event-table)
- [External Table](#external-table)
- [Hybrid Table](#hybrid-table)
- [Iceberg Table](#iceberg-table)
- [Permanent Table](#permanent-table)
- [Temporary Table](#temporary-table)
- [Transient Table](#transient-table)


## Dynamic Table
- **Description**: Table that automatically update based on result of query and are always kept in sync with underlying data sources
- **Use Cases**:
  - Automating data synchronisation without manual intervention
  - Maintaining a continuously updated view of data for real-time analytics
  - Managing complex dependencies in data pipelines
- Syntax
  
  ```sql
  CREATE OR REPLACE DYNAMIC TABLE DEMO_DYNAMIC_TABLE_CUSTOMER
  TARGET_LAG = '1 minute'
  WAREHOUSE = COMPUTE_WH
  AS
  SELECT 
  cst.c_name,
  cst_addr.ca_street_number,
  cst_addr.ca_street_name,
  cst_addr.ca_street_type,
  cst_addr.ca_suite_number
  FROM 
  DEMO_DB.DEMO_SCHEMA.DEMO_CUSTOMER_ADDRESS cst_addr 
  INNER JOIN DEMO_DB.DEMO_SCHEMA.DEMO_CUSTOMER cst
  ON cst_addr.ca_address_id=cst.c_address;
  ```
  _** Target lag: time lag between data refresh_

## Directory Table
- **Description**: Store file-level metadata about tiles in stage (internal/external). There is an option to include directory table upon creating a stage
- Syntax
  ```sql
  /*Let us consider we have multiple files stored in Snowflake internal stage
  Below is how we can see all the metadata details of the file.*/
  
  SELECT * FROM DIRECTORY( @DEMO_INTERNAL_STAGE_01 );  --> This is how this table is accessed.
  
  /*We can even query the table as given below */
  SELECT * FROM DIRECTORY( @DEMO_INTERNAL_STAGE_01 ) where size>2000;
  
  -- The output has the exact file URL where we do have our data stored.


  /* Some more syntaxes with directory tables are as follows */
  
  CREATE STAGE mystage
    DIRECTORY = (ENABLE = TRUE)    -->This is the keyword
    FILE_FORMAT = myformat;
  
  CREATE STAGE mystage
    URL='s3://load/files/'
    STORAGE_INTEGRATION = my_storage_int
    DIRECTORY = (ENABLE = TRUE);     -->This is the keyword
  ```
_[Back to top](#type-of-snowflake-tables)_

## Event Table
- **Description:** Table designed to capture logs and evnets natively within platform
- **Use cases**:
  - To track DML operation
  - Capture log at session level, easily tracked event within particular session
  - Can receive notification about new events and errors in application
- Syntax
  ```sql
  --Step 1(Creating an event table)::
  CREATE EVENT TABLE DEMO_DB.DEMO_SCHEMA.EVENT_TBL_V1;
  
  --Step 2(Associate event table with an account)::
  ALTER ACCOUNT SET EVENT_TABLE = DEMO_DB.DEMO_SCHEMA.EVENT_TBL_V1;
  
  --Step 3(setting the log level)
  ALTER SESSION SET LOG_LEVEL = INFO; 
  ```
  _** Other values: trace, debug, info, warm, error_

## External Table
- **Description:** Tables that allow data querying from data stored in external locations like Amazon S3, Azure Blob Storage or Google Cloud Storage without loading data into Snowflake
- **Use cases:**
  - Use feature on Snowflake without importing data natively onto Snowflake
  - Read-only so format does not change and can be use by other tools/technologies
  - Query multiple files by joining them into single table
  - Efficient connectivity to data lake
- **Cons/Limitations**
  - Not as performant as internal tables
  - Read only so cannot perform DML operations
  - Additional method to refresh external tables
  - File skipped if encountered scanning error.
- Syntax
  ```sql
  --Create the file format.
  create or replace file format demo_csv_format
  type = 'csv' field_delimiter = ',' skip_header = 1
  field_optionally_enclosed_by = '"' 
  null_if = ('NULL', 'null') 
  empty_field_as_null = true;
  
  --create the external stage via storage integration object.
  create or replace stage demo_db.demo_schema.ext_stage_snow_demo url="s3://demostorageint/" 
  STORAGE_INTEGRATION=s3_external_table_storageint     ---This is the storage integration object
  file_format = demo_csv_format;        
  
  
  --Create the external tables.
  create or replace external TABLE demo_customer_tbl (
          CUST_SK varchar AS (value:c1::varchar),
          CUST_ID varchar AS (value:c2::varchar)
      )
  with location=@ext_stage_snow_demo
  auto_refresh = false
  file_format = (format_name = demo_csv_format)   
  
  
  --Query the external tables.
  select * from demo_customer_tbl;
  ```
  _** There will be another column named metadata$filename which stores info about file in external table_
_[Back to top](#type-of-snowflake-tables)_

  ## Hybrid Table
  - **Description**: Table support all transactional capability needs. Support fast single row operations and work on new row-based storage engine
  - **Key features**
    - Storage: stored as 2 copies (one in row storage, another in column storage)
    - Locking: happens at row level, higher concurrency execution of single row updates
    - Indexes: use B-Tree indexes for primary key and secondary indexes
    - Compatible with connectors
  - **Cons**
    - Cost: because of the 2-copy storage
    - Scalability: size limit
  - Syntax
    ```sql
    CREATE HYBRID TABLE Customers (
        CustomerKey number(38,0) PRIMARY KEY,
        Customername varchar(50)
    );
    
    
    -- Create order table with foreign key referencing the customer table
    CREATE OR REPLACE HYBRID TABLE Orders (
        Orderkey number(38,0) PRIMARY KEY,
        Customerkey number(38,0),
        Orderstatus varchar(20),
        Totalprice number(38,0),
        Orderdate timestamp_ntz,
        Clerk varchar(50),
    CONSTRAINT fk_o_customerkey FOREIGN KEY (Customerkey) REFERENCES Customers(Customerkey),
    INDEX index_o_orderdate (Orderdate)); -- secondary index to accelerate time-based lookups
    ```

## Iceberg Table
- **Description:** New table powered by Apache iceberg open table formats to store data and metadata all in customer managed storage and can use standard Snowflake features like DMLs, encryption
- **Use cases:**
  - To build open data lakehouse
  - Any need to not bring data natively to Snowflake can be solved by this table
- **Limitation:**
  - Cloning, replication, applying tags is not yet available as of now
  - Need to access external volume directly as storage is not supported as of today
  - Support processing of Parquet filt format only
- Syntax
  ```sql
  CREATE ICEBERG TABLE myTable
  CATALOG='SNOWFLAKE'
  EXTERNAL_VOLUME='myIcebergVolume'
  BASE_LOCATION='relative/path/from/extvol/location/';
  ```
_[Back to top](#type-of-snowflake-tables)_

## Permanent Table
- **Description**: Default table in Snowflake, data is persisted indefintely unless explicitly deleted
- **Use cases:**
  - Store historical data that needs to be queried over time
  - Keep critical business data that requires strong data durability
  - Tables that participate in regular ETL processes
- Syntax
  ```SQL
  --The syntax for creating the permanent table is given as below:
  create table student (id number, name varchar(100));  
  ```

## Temporary Table
- **Description:** Table that are session-specific and are automatically dropped at the end of session in which they were created
- **Use cases:**
  - Intermediate storage for data processing within a session
  - Temporary staging of data for complex transformation or calculations
  - Scenarios where data does not need to persist beyond sessions
- Syntax
  ```sql
  create temporary table student (id number, name varchar(100));
  ```
 
## Transient Table
- **Description:** Table similar to permanent but does not have fail-safe recovery period. They persist beyond session but are most cost-effective as they do not need long term data protection
- **Use cases:**
  - Store non-critical data that can be regenerated if lost
  - Intermediate data storage for ETL processes
  - Reduce storage costs for short-lived data that does not need full recovery options
- Syntax
  ```sql
  create transient table student (id number, name varchar(100));
  ```
_[Back to top](#type-of-snowflake-tables)_
