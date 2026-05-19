# 6.2 Outline Snowflake data sharing with Snowflake

## Secure Data Sharing

**Direct Share**

- Secure data-sharing method that allows provider to grant consumer read-only access to live data without moving or copying it
- REFERENCE_USAGE privilege needed to reference objects in other database when sharing secure views in the share
- Feature
    - Zero copy - data not physically transferred, but shared via metadata layer
    - Live data - updates immediately visible to consumer
    - Read-only - cannot modify or delete shared data

```sql
# creating share
CREATE SHARE my_share;

# grant usage
GRANT USAGE ON DATABASE database_a TO SHARE my_share;
GRANT USAGE ON SCHEMA database_a.schema_a TO SHARE my_share;

# grant REFERENCE_USAGE on secondary database (referenced by secure views)
GRANT REFERENCE_USAGE ON DATABASE database_b TO SHARE my_share;

# grant SELECT on secure view to share
GRANT SELECT ON VIEW database_a.schema_a.my_secure_view TO SHARE my_share;

# add consumer account to share
ALTER SHARE my_share ADD ACCOUNTS = <consumer_account_locator>';
```

**Data listings**

- A way to share datasets with consumers through Data Exchange - either freely or on personalised or request basis
- Listing types
    - Free - consumer access data immediately after accepting terms
    - Personalised - consumer submit request, provider review and fulfils it
- Key consideration
    - Only role created the share can attach it to listing, one share to one listing (even if listing got deleted later on)
    - All sample queries auto-validated before free listing is published
    - Data must be legally shareable
- Cross-region listings
    - Automatically replicated across regions, but data not, need to replicate data to selected region
    - For free listings, data replicated before publishing
    - Personalised listings, data replicated upon consumer request
    - To share data in region, need to have account in that region and all accounts belong to the same organisations

## Data Exchange

- Private, secure data hub allowing organisation to share data with invited partners, suppliers or internal teams
- Centralised control over who can provide or consume data
- Requires invitation to join and setup involves contacting Snowflake Support
- Key features
    - Controlled access - private ecosystem
    - Direct and secure sharing - data not copied or transferred
    - Centralised hub
    - Support for diverse data - semi-structured/structured data
- To establish
    - Contact Snowflake Support
    - Provide business case, name for exchange and account URL
    - ACCOUNTADMIN - manage provider profiles (can be delegated to custom roles)

**Differences from Other Snowflake Sharing Options:**
• **Data Exchange (Private):** Invitation-only, curated exchange for private groups (e.g., a company and its suppliers).
• **Snowflake Marketplace (Public):** Open to all Snowflake users, featuring third-party and public data.
• **Direct Share:** One-to-one secure sharing between two specific parties

## Reader Account

- Allows providers to share data with consumers who are not Snowflake customers, without require any setup/costs
- Reader account created, owned and managed by providers
- Cost charges incurred by consumers bear by provider
- Constraints:
    - Can only consume data from provider account created it
    - Unlimited number of credits per month, billed to provider (resource monitor is suggested)
    - Limited to 20 reader accounts per provider, 7 days retention period after dropping before freeing up another slot
- Feature:
    - Can query data, create materialised views, unload using COPY INTO
    - Not allowed to upload/modify data, creating pipes, stages, shares, masking/row access policies, streamlit app or image repositories
- Creation and manage
    - via snowsight UI (Admin → Accounts → Reader Accounts) or SQL
    - Need ACCOUNTADMIN/CREATE ACCOUNT global privilege to other roles

```sql
CREATE MANAGED ACCOUNT #create reader account
DROP MANAGED ACCOUNT #irreversible, immediately removes all access
SHOW MANAGED ACCOUNTS
```

## Privileges required for working with shares

- By default, ACCOUNTADMIN role handles these tasks
- Core privileges include CREATE SHARE (provider), MODIFY SHARE (provider), USAGE on database/schema, and REFERENCE_USAGE for shared database
- Privileges
    - CREATE_SHARE - global privilege needed to create new share
    - USAGE/SELECT - needed on database, schema, and specific objects being shared
    - MODIFY/ADD/REMOVE - specific share level privileges needed to alter share, such as adding consumer accounts
    - REFERENCE_USAGE - if share contains objects from multiple database
- Privileges for data consumer
    - IMPORT SHARE - global privilege to import share
    - CREATE DATABASE - to create database from inbound share
    - REFERENCE_USAGE on database - to create database from share