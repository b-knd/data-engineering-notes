# 4.4 Outline commands used to unload data and when to use them

## COPY INTO

- Export data from Snowflake table or query into specified stage

```sql
COPY INTO @my_external_stage/export/
	FROM my_table
	FILE_FORMAT = (TYPE = 'PARQUET')
	OVERWRITE = TRUE;
```

## GET

- Download data files from one of the following internal stage types to local directory or folder on client machine
    - Named internal stage
    - Internal stage for specified table
    - Internal stage for current user