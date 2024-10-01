# Change Data Capture (CDC)
Definition: Process of identifying and capturing changes made to data in database and deliver changes near real-time to downstream process or system

## How CDC fit into the ETL/ELT pipelines
- Extract
    - Data in source table is continuously updated
    - Inefficient to completely refresh the entire data
    - CDC comes in to capture changes
        - With this, we can only update the one with changes instead of the entire thing
- Transform
    - For ELT, data is loaded immediately and transformed in target system (data warehouse, data lake or data lakehouse
    - ELT can operate on CDC timescale which continually loads data as it changes at source

## Benefits of CDC
- Eliminate bulk load updating by allowing incremental loading or real-time streaming of changes to target repository
- Zero-downtime database migration
- Support real-time analytics, fraud protection and data synchronisation across distributed systems
- Efficient to move data across wide area network, perfect for cloud
- Suited for moving data into stream processing solution (e.g. Apache Kafka)
- Data synchronisation in multiple system

## CDC Methods
- Log-based
    - New transaction gets logged into log file
    - Changes can be pick up from log
- Query-based/Audit columns
    - Query data in source to pick up changes
    - Need to have timestamp in source
    - Use audit columns (date modified/last modified), scan source with date > target table and update accordingly
- Trigger-based
    - Configure source system to trigger the write to change table
    - Reduce database performance (multiple writes each time row is updated, inserted or deleted)

## Reference
- https://www.qlik.com/us/change-data-capture/cdc-change-data-capture#:~:text=Change%20data%20capture%20(CDC)%20refers,a%20downstream%20process%20or%20system.
- https://www.striim.com/blog/change-data-capture-cdc-what-it-is-and-how-it-works/
