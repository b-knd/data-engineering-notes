# 4.2 Outline commands used to load data and when to use them

## PUT

- Upload (stage) local data files from machine to internal stage (User/table/named stage)

```bash
PUT file://<path_to_file>/<filename> @<stage_name>[/<path>];
```

- Functions
    - Upload to internal snowflake stages (not external stages like S3/GCS)
    - Cannot run in Snowsight web interface worksheets, must use SnowSQL/Snowflake driver (Python, JDBC or ODBC)
    - Automatic compression using Gzip (unless AUTO_COMPRESS=FALSE)
    - Data is encrypted on client machine before transmission
- Parameters
    - AUTO_COMPRESS - default TRUE
    - OVERWRITE - default FALSE, TRUE to overwrite existing files in stage with same name
    - PARALLEL - number of threads for uploading
    - SOURCE_COMPRESSION - identifies compression of local source file (AUTO, GZIP or NONE)

## COPY_INTO

- For bulk data operations, load data/unload data
- Load data
    - From stage to table
    - File must be in Snowflake stage (internal or external cloud storage)

```sql
COPY INTO my_table
	FROM @my_stage/folder/
	FILES = ('data_file.csv')
	FILE_FORMAT = (TYPE = 'CSV' FILE_DELIMITER = ',', SKIP_HEADER = 1);
```

- Options
    - PATTERN - use regex to load multiple files matching a naming convention (PATTERN = ‘.*sales.*\.csv’)
    - ON_ERROR - decide what happens if file has errors (CONTINUE, SKIP_FILE, ABORT_STATEMENT)
    - PURGE - set to TRUE to delete files from stage after successful load
    - FORCE - set to TRUE to reload files even if they haven’t changed since last load
- Key considerations
    - Transformation - Can use SELECT statement within COPY INTO command to reorder columns, cast types, or transform data during load process
    - Performance - split files into smaller chunks (100-250 MB compressed) to allow parallel process across warehouses
    - Monitoring - use LOAD_HISTORY view or Snowsight Query History to track status and errors of copy operations