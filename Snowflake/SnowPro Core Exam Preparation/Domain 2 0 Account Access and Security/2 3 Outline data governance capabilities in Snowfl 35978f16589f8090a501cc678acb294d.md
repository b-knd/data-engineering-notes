# 2.3 Outline data governance capabilities in Snowflake

## Secure Views

- Specialised views designed for data privacy, preventing unauthorised users from viewing underlying SQL definition or inferring data structure through optimisation techniques
- Essential for sharing sensitive data securely, as they hide view definitions, support column/row-level security and prevent predicate pushdown optimisation
- Key feature and use case
    - Security - only view owner can see DDL via GET_DDL or information schema
    - Data sharing - secure views are required when sharing data via Snowflake Shares
    - Performance - secure views may perform slower than standard views because they disable certain optimiser optimiser to ensure security
    - Implementation - CREATE SECURE VIEW
- Best practice
    - Use CREATE SECURE VIEW specifically for restricting access to PII or sensitive columns
    - Do not use secure view for simple convenience views (just to rename columns) as it adds unnecessary performance overhead
    - To test, use SIMILATED_DATA_SHARING_CONSUMER session parameter to see what a consumer will experience

```sql
CREATE SECURE VIEW restrictied_customer_view AS
SELECT name, masked_email
FROM customers;
```

## Access History

- Enterprise or higher feature
- Tracks when user query reads data and when SQL perform write operations (INSERT, UPDATE, DELETE, COPY)
- Accessible via ACCESS_HISTORY view in ACCOUNT_USAGE and ORGANISATION_USAGE schema
- What gets tracked per record
    - Each row = one SQL statement, capturing
        - direct_objects_accessed - object referenced
        - base_objects_accessed - underlying source objects data come from (eg base table behind view)
        - objects_modified - what was written
        - object_modified_by_ddl - CREATE/ALTER/DROP including policy and tag changes
        - policy_referenced - masking/row access policies enforced during query
        - parent_query_id/root_query_id - tracing nested stored procedure calls
- Use cases
    - Compliance auditing
    - Data discovery - find unused tables/columns to archive or clean up
    - Sensitive data tracking
    - Data validation
    - Policy auditing
- Column lineage extends objects_modified to show how data flows from source columns to target columns in write operations (INSERT/MERGE/CTAS)
- Level
    - Account level - free, query ACCOUNT_USAGE.ACCESS_HISTORY
    - Organisation level - premium view to aggregate access history across all accounts, incurs additional compute and storage costs

## Object tags

- Schema-level object that can be assigned to another Snowflake object
- Tag associated with arbitrary string value - stored as key-value pair
- Tag must be unique within schema
- Characteristics
    - Object can have multiple tag
    - Single tag can be assigned to different object types (warehouse and tables at once)
    - Same tag can have duplicate values across multiple objects (same tag name, different value)
- Features
    - Tag inheritance - tag at higher-level applied to all child objects (eg applied to table → applied to all columns in that table)
    - Automatic propagation - from source to target (when view build from tagged table)
    - Replication - tag replicated to ensure assignment consistency
- Use case
    - Data protection
        - Discovery of sensitive data
        - Can combine with masking to protect further
    - Resource usage monitoring
        - Group by tag to analyse activities
- Quotas
    - max 50 tags on single object
    - max 50 different tags across all columns of single table
    - max 100 unique tag-entity association
- Basic tagging applicable to all edition, but tag propagation and masking policies only enterprise or higher
- Future grants of privileges on tags are not supported, grant APPLY TAG to custom role to allow applying tag to other objects

## Overview of row/column-level security

- Fire grained data access control
- Allow administrators to restrict data visibility based on user roles and attributes without altering underlying data

**Row-level security (RLS)**

- Restrict which rows user can see in table or view
- Uses row access policies (schema-level objects that return boolean value) for each row during query
- Policies are applied at runtime during SELECT, UPDATE, DELETE and MERGE
- Implementation methods
    - Row access policies - centralised policies that can be applied to multiple tables, making management easier
    - Mapping tables - mapping table to link user roles to specific data subsets
    - Secure views - older method that filters data via WHERE clause based on CURRENT_ROLE() or CURRENT_USER()

**Column-level security**

- Dynamic data masking policies to obfuscate or hide sensitive data (like PII or PHI)
- When user queries column with masking policy, Snwoflake checks user’s role and return either original value, partially masked value or completely masked value
- Implementation methods
    - Dynamic data masking - defined at schema level and applied directly to columns
    - Tag-based masking - policies applied to columns that have specific object tags
    - External tokenisation - external providers to tokenise data before it enters Snowflake at query time
- Key benefits
    - Centralised management - policies created once and can be applied to many tables
    - Data integrity - raw data remains unchanged in storage, only the result of query are modified
    - Compliance - help meet regulatory requirements like GDPR, HIPAA, SOC2 by protecting sensitive information