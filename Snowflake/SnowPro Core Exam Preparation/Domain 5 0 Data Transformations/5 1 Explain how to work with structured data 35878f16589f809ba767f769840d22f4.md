# 5.1 Explain how to work with structured data

## Table Functions

INFER_SCHEMA

- Automatically detects file metadata schema in a set of staged data files containing semi-structured data and retrieves column definitions
- Supported file formats: Parquet, Avro, ORC, JSON and CSV
- Key arguments
    - LOCATION
    - FILE_FORMAT
    - FILES
    - IGNORE_CASE
    - MAX_FILE_COUNT
    - MAX_RECORDS_PER_FILE
    - KIND
- Output columns: COLUMN_NAME, TYPE, NULLABLE, EXPRESSION, FILENAMES and ORDER_ID
- Can use the output with CREATE TABLE … USING TEMPLATE to automatically create Snowflake table/external table/Iceberg table
- Note
    - For CSV, set PARSE_HEADER = TRUE in file format to use first row as column names otherwise named c1, c2…
    - All timestamp types returned as TIMESTAMP_NTZ
    - For CSV, all columns are identified as NULLABLE regardless of actual data
    - Only first level nesting is supported for nested data types - deeper levels aren’t detected
    - If inferring Parquet files for Iceberg table, always use KIND ⇒ ‘ICEBERG’, otherwise column definitions may be incorrect
    - Table stages are not supported - only named internal/external stages and user stages