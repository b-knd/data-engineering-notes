# 5.3 Explain how to work with unstructured data

## Define and use directory tables

- Implicit object layers on top of a stage (either internal or external) that stores file-level metadata about the data files stored there
- Similar to external table but specifically designed to catalog and manage files
- Handle unstructured data like PDFs, images and audio files
- Features
    - Metadata storage - automatically captures file details such as RELATIVE_PATH, SIZE, LAST_MODIFIED and a FILE_URL for secure access
    - No standalone objects - enabled as property of stage, cannot be created
    - Refresh option - can be updated manually using (ALTER STAGE … REFRESH) or automatically for external stages via cloud event notifications
    - Billing - automatic refreshes on external stages incur Snowpipe charges, else generally free
- Common use cases
    - Unstructured data pipeline - query this table to get file URLs and pass to Snowpark API or external functions for processing
    - Integrated views - join directory table with Snowflake tables to create single view that combines FILE_URLs with business metadata
    - Secure access - use FILE_URL to allow user to access files securely without giving direct access to underlying cloud storage

**Code**

```sql
# creation
CREATE STAGE my_stage
	DIRECTORY = (ENABLE = TRUE);
	
# query metadata
SELECT * FROM DIRECTORY(@my_stage);

# manual refresh
ALTER STAGE my_stage REFRESH;
```

## SQL file functions

- build-in scalar functions used to handle and reference files stored in internal or external stages
- Interact with staged files - those on Amazon s3, GCS or Azure directly within queries
- Functions
    - GET_PRESIGNED_URL - generate temporary, pre-authenticated URL to file in stage, user can download or view without needing Snowflake credentials
    - GET_RELATIVE_PATH - returns relative path of file on stage, given its absolute path/URL
    - GET_STAGE_LOCATION - extract stage identifier (e.g. @my_stage) from file’s full path
    - BUILD_SCOPED_FILE_URL - generate permanent URL for a file located in stage
- Querying staged files directly
    - SELECT $1, $2 FROM @my_stage/file.csv;
    - Query file metadata using METADATA$FILENAME and METADATA$FILE_ROW_NUMBER

## Types of URLs used to access data files

**Scoped URL**

- Encoded, temporary link that allows user to access staged file without needing specific privilege on stage itself
- Expired 24 hours (or until query result cache expires
- Only user who generated the URL can use it
- Best for secure, short-term data sharing within specific session
- Use BUILD_SCOPED_FILE_URL

**File URL (Stage File URL)**

- Permanent, persistent link that identifies specific path to file (database, schema, stage and path)
- Remains valid as long as file exists
- User with role (READ on internal stages or USAGE on external stages
- Best for long-term internal references and REST_API programmatic access
- Use BUILD_STAGE_FILE_URL or query directory table to generate

**Pre-signed URL**

- Simple HTTPs link that allows anyone with the link to access file via web browser without authenticating into Snowflake
- Configurable at time of generation, 60 seconds/1 hour…)
- Open to anyone with URL, no Snowflake credentials or authorisation tokens are required
- Best for sharing data with external users or business intelligence tools that cannot log into Snowflake
- Use GET_PRESIGNED_URL function