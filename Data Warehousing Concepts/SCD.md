# Slowly Changing Dimensions (SCD)

Definition: Refers to how data changes over time in data warehouse

### Type 0
- Dimensions that remains unchanged 
- Usually are mapping tables/ dim_table that are used for joining

### Type 1
- Data overwritten by new data without keeping historical record of old data
- Cannot keep track of changes
- Only do this implementation if no trends tracking is needed

### Type 2
- Create a new record
- For analytical perspective, to get the active record, several options:
    - CDC implementation
        - Keep something like status flag to indicate true or false to get active record
    - Timestamp column
        - Keep a timestamp column so that to get active records can filter the most recent one and assume as active

### Type 3
- Adding a new column instead of new record to maintain single primary key but need to track change
- Something like a before after, when update, shift current to previous and update current
- But can only use to track single change in a record, not suitable when need to track multiple changes within one record

### Type 4
- Create tables to store historical data and another one as the active records
