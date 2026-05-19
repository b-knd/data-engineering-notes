# 1.2 Outline key Snowflake tools and user interfaces

## Snowsight

Snowflake’s web UI not CLI (SnowSQL) and not a driver/connector

- Query & SQL editor
    - Write and run SQL queries
    - View query history
    - Access query profile
    - Use explain plans
- Monitoring & admin
    - Warehouse load monitoring
    - Resource monitor setup
    - Cost insights feature
    - Snowflake budgets service
- Data exploration
    - Browse database and schemas
    - preview table data
    - View object details
    - Access Marketplace
- Worksheets & collaboration
    - Create and save SQL worksheets
    - Share worksheets with teammates
    - Support for multiple roles/warehouses
    - Result caching visible in profile
- Dashboards & charts
    - Build visual dashboards from queries
    - Chart query results inline
    - Schedule refreshes for dashboards
    - Share insights across teams

## SnowSQL (legacy) /CLI client

- Command-line client for connecting to Snowflake to execute SQL queries and perform DDL and DML operations, including loading data into and unloading data out of database tables
- Can be run as interactive shell or batch mode through stdin or using -f option
- Application developed using Snowflake Connector for Python (but not needed to install SnowSQL)
- How to connect
    - Install locally
    - Connect via terminal via account identifier
    - Support MFA and SSO login
- Core capabilities
    - Run DDL & DML statements
    - Execute scripts
    - Load data with PUT command
    - Unload data with GET command
- Key use cases
    - Automation & scripting
    - Batch data loading/unloading
    - CI/CD pipeline integration
    - Admin tasks via scripts
- Key SnowSQL commands to know
    
    ```bash
    snowsql -a <account> -u <user> # connect to Snowflake account
    PUT file://path/to/file@stage_name # upload local file to a stage
    GET @stage_name file://local/path # download file from stage
    !exit /!quit # exit the SnowSQL session
    snowsql -f script.sql # execute a SQL file non-interactively
    ```
    

## Snowflake Connectors

Facilitate seamless data integration between Snowflake and external systems

### Key Snowflake Connector Types & Use Cases

- Snowflake connector for python
- Snowflake Kafka connector
- Snowflake connector for spark
- Snowflake JDBC drivers
- Native connectors (marketplace)

### Key features and capabilities

- Security - support various authentication methods, including OAuth and External OAuth
- Performance - offers bulk loading capabilities for high-performance data loading
- Automation - simplifies data workflows through automatic table creation, schema management, and data ingestion from SaaS applications
- Development - provides dedicated tools for custom application development

## Snowflake Drivers

Libraries that allow applications and tools to connect to Snowflake programmatically using standard interfaces

- JDBC, ODBC, Node.js, .NET, Go & other drivers (Spark, PHP, Kafka)
- Python connector

Drivers are low-level connection protocol - app talks directly to Snowflake

Connectors - high-level integration where it bridge platform/service and Snowflake

```sql
SELECT CURRENT_CLIENT() # to check current driver's version
```

## Snowpark

- Developer framework that allows data engineers, data scientists and developers to securely process data in Python, Java and Scala directly within Snowflake
- Run on Snowflake’s elastic engine without moving data, optimise performance and reduce infrastructure maintenence
- Key components
    - Snowpark libraries - to write code in Python, Java or Scala using familiar DataFrame-style programming
    - Server-side execution - execute code in Snowflake
    - Dataframes - lazy-evaluated API for querying and processing data, similar to Spark or Pandas
    - UDFs and Stored Procedures - enable custom logic implementation for data transformation, ML scoring and complex ETL/ELT tasks
    - Snowpark ML API - for machine learning modelling and operations
    - Snowpark container services - allow running containerised applications within Snowflake
- Advantages
    - No data movement
    - Flexible language support
    - Seamless integration (with existing IDEs)
    - Simplified architecture - reduce the need for external data processing engine like Spark

## SnowCD

## Streamlit in Snowflake

- Allow you to build, deploy and share interactive Python-based web applications directly within Snowflake platform
- Combines streamlit’s open-source library with snowflake’s security
- Benefits
    - Security and governance - app run natively inside Snowflake, data secured (utilise RBAC)
    - Simplified structure - Snowflake manage underlying compute and storage
    - Seamless integration - work with Snowpark, UDFs and stored procedures, process data using Python without moving
- Ways to run
    - Warehouse runtime - run on standard Snowflake warehouses, on-demand
    - Container runtime - run as long-running service using Snowpark Container Services (dedicated instance shared among all viewers for faster initial access)

## Cortex

Suite of AI features that use large language models to understand unstructured data, answer freeform questions and provide intelligent assistance - all within Snowflake’s secure governed platform

### Key Components

1. Cortex AI Functions (AISQL)
    1. Unify structured and unstructured analytics within single platform using familiar SQL syntax
    2. Functions
        1. AI_COMPLETE - direct LLM completions within SQL
        2. AI_FILTER - filter rows using plain-language conditions
        3. AI_EXTRACT - extract info from text, documents, and images
        4. AI_CLASSIFY - sort text or image inputs into categories
        5. AI_TRANSCRIBE - transcribe audio/video
        6. AI_EMBED - generate embedding vectors for similarity search
        7. AI_SIMILARITY - calculate similarity between two inputs
        8. AI_REDACT - detects and removes PII from unstructured text automatically
        9. AI_TRANSLATE - translate text between languages
2. Cortex Agents
    1. Orchestrate across structured and unstructured data sources to deliver insights
    2. Plan tasks, use tools to execute them
    3. Generate responses (Cortex Analyst - structured data, Cortex Search - unstructured data) alongside LLM
3. Cortex Analyst
    1. Converts natural language to SQL with high accuracy
    2. Enable business to query data by asking questions without writing SQL
4. Cortex Search
    1. Fully managed hybrid search (semantic + keyword) for finding information by asking questions within set of documents
    2. Built-in text embedding and retrieval
5. Snowflake Intelligence
    1. Build data agents that answer questions about structured and unstructured data
    2. Take action across third party platforms like Salesforce and Google Workspace
6. Cortex Code
    1. AI coding assistant built directly into Snowsight
    2. Can ask to explain Snowflake resources, optimise complex queries or tweak filters by describing task in natural language

**Key benifits**

- No data movement - AI runs inside Snowflake’s security perimeter
- SQL-native
- Multi-model - all in same query
- Serverless
- Governed - respect Snowflake’s existing role-based access controls, data policies and audit trails

## Snowflake SQL API