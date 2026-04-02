# dbt Analytics Engineering Associate — Comprehensive Study Notes

> **Target:** dbt Analytics Engineering Certification (Associate)
> **Last updated:** April 2026

---

## Table of Contents

1. [Exam Overview](#1-exam-overview)
2. [dbt Core Fundamentals](#2-dbt-core-fundamentals)
3. [Project Structure & Configuration](#3-project-structure--configuration)
4. [Models](#4-models)
5. [Sources](#5-sources)
6. [Seeds](#6-seeds)
7. [Snapshots](#7-snapshots)
8. [Tests](#8-tests)
9. [Documentation](#9-documentation)
10. [Macros & Jinja](#10-macros--jinja)
11. [Packages](#11-packages)
12. [Exposures](#12-exposures)
13. [Analyses](#13-analyses)
14. [Hooks & Operations](#14-hooks--operations)
15. [Materializations](#15-materializations)
16. [Incremental Models (Deep Dive)](#16-incremental-models-deep-dive)
17. [Environments & Targets](#17-environments--targets)
18. [dbt Cloud vs dbt Core](#18-dbt-cloud-vs-dbt-core)
19. [Selectors & Node Selection](#19-selectors--node-selection)
20. [Metadata & Artifacts](#20-metadata--artifacts)
21. [Best Practices & Anti-Patterns](#21-best-practices--anti-patterns)
22. [Quick-Reference Cheat Sheet](#22-quick-reference-cheat-sheet)

---

## 1. Exam Overview

### Exam Format
- **~65 multiple-choice questions**
- **~120 minutes**
- Passing score: ~65% (varies slightly by cohort)
- Administered online via Certiverse

### Domain Breakdown (approximate weightings)
| Domain | Weight |
|---|---|
| dbt project fundamentals (models, sources, seeds, snapshots) | ~30% |
| Testing & documentation | ~20% |
| Jinja, macros & packages | ~15% |
| dbt Cloud features (jobs, environments, IDE) | ~15% |
| Advanced modeling patterns (incremental, materializations) | ~10% |
| Governance & best practices | ~10% |

### Key Study Resources
- [dbt official docs](https://docs.getdbt.com)
- [dbt Learn courses](https://courses.getdbt.com)
- [dbt Discourse](https://discourse.getdbt.com)
- [dbt Slack community](https://community.getdbt.com)

---

## 2. dbt Core Fundamentals

### What dbt Does
- **T** in ELT — dbt handles the Transform step only
- Writes SQL `SELECT` statements; dbt compiles them and runs them against the data warehouse
- Manages dependencies between models using `ref()` and `source()`
- Does **not** move data into the warehouse (that's the E and L)

### How dbt Works
```
Raw SQL files (models)
        ↓
  Jinja templating (compile)
        ↓
  Compiled SQL
        ↓
  Run against DW (CREATE TABLE / VIEW / etc.)
        ↓
  Materialized results
```

### dbt Adapters
dbt connects to warehouses via **adapters**. Common adapters:
- `dbt-snowflake`
- `dbt-bigquery`
- `dbt-redshift`
- `dbt-databricks`
- `dbt-postgres`
- `dbt-duckdb`

Each adapter is installed separately: `pip install dbt-snowflake`

### The DAG (Directed Acyclic Graph)
- dbt infers the DAG from `ref()` and `source()` calls
- No cycles allowed
- The DAG is exposed in the dbt Docs lineage graph and in `manifest.json`

---

## 3. Project Structure & Configuration

### `dbt_project.yml` — the project manifest
```yaml
name: 'my_project'
version: '1.0.0'
config-version: 2

profile: 'my_profile'          # links to profiles.yml

model-paths: ["models"]
seed-paths: ["seeds"]
test-paths: ["tests"]
analysis-paths: ["analyses"]
macro-paths: ["macros"]
snapshot-paths: ["snapshots"]
docs-paths: ["docs"]

target-path: "target"
clean-targets: ["target", "dbt_packages"]

models:
  my_project:
    staging:
      +materialized: view
    marts:
      +materialized: table
```

Key `dbt_project.yml` knobs:
| Key | Purpose |
|---|---|
| `name` | Project identifier |
| `profile` | Which profiles.yml profile to use |
| `model-paths` | Where to look for model SQL files |
| `+materialized` | Default materialization per folder |
| `+schema` | Custom schema suffix per folder |
| `+tags` | Apply tags to all models in a folder |
| `+enabled` | Enable/disable nodes |
| `+pre-hook` / `+post-hook` | Hooks at folder level |
| `+meta` | Attach arbitrary metadata |

### `profiles.yml`
- Lives in `~/.dbt/profiles.yml` by default (outside the project for security)
- Can be overridden with `--profiles-dir` flag
- Contains **connection credentials** per target (dev, prod)

```yaml
my_profile:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: xyz.us-east-1
      user: "{{ env_var('DBT_USER') }}"
      password: "{{ env_var('DBT_PASSWORD') }}"
      role: TRANSFORMER
      database: DEV_DB
      warehouse: COMPUTE_WH
      schema: dbt_myname
      threads: 4
    prod:
      type: snowflake
      ...
      schema: analytics
```

### `packages.yml`
Declares external dbt packages:
```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
  - package: dbt-labs/codegen
    version: 0.12.1
  - git: https://github.com/org/repo
    revision: main
```
Install with: `dbt deps`

---

## 4. Models

### What is a Model?
- A `.sql` file in the `models/` directory
- Contains a single `SELECT` statement
- dbt wraps it in DDL (`CREATE TABLE AS`, `CREATE VIEW AS`, etc.)

### The `ref()` Function
```sql
-- models/marts/fct_orders.sql
SELECT *
FROM {{ ref('stg_orders') }}  -- references models/staging/stg_orders.sql
```
- Creates DAG dependency
- Automatically resolves to the correct schema/database per environment
- **Always use `ref()` to reference other models — never hardcode schema names**

### Model Naming Convention (community standard)
```
stg_<source>__<entity>.sql     # staging
int_<entity>__<verb>.sql       # intermediate
fct_<entity>.sql               # fact
dim_<entity>.sql               # dimension
```

### Config Block in Model
```sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    schema='finance',
    tags=['finance', 'daily'],
    meta={'owner': 'finance_team'}
) }}

SELECT ...
```

Config precedence (highest to lowest):
1. In-model `{{ config(...) }}` block
2. `.yml` file property
3. `dbt_project.yml` folder config
4. dbt defaults

### YAML Properties File
```yaml
# models/staging/_stg_sources.yml  (or any .yml in models/)
version: 2

models:
  - name: stg_orders
    description: "Staged orders from the source system."
    config:
      materialized: view
    columns:
      - name: order_id
        description: "Primary key."
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'returned']
```

---

## 5. Sources

### Defining Sources
Sources represent raw tables loaded by your EL tool.
```yaml
# models/staging/sources.yml
version: 2

sources:
  - name: jaffle_shop          # source name
    database: raw              # optional override
    schema: jaffle_shop        # schema in warehouse
    loader: fivetran
    loaded_at_field: _etl_loaded_at   # for freshness
    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: orders
        description: "Raw orders table."
        columns:
          - name: id
            tests:
              - unique
              - not_null
      - name: customers
```

### Referencing Sources
```sql
SELECT *
FROM {{ source('jaffle_shop', 'orders') }}
-- resolves to: raw.jaffle_shop.orders
```

### Source Freshness
```bash
dbt source freshness        # checks freshness of all sources with freshness config
dbt source freshness --select source:jaffle_shop
```
- Requires `loaded_at_field` to be defined
- Results stored in `sources.json` artifact

---

## 6. Seeds

### What are Seeds?
- CSV files in the `seeds/` directory loaded as tables into the warehouse
- Good for static reference data (country codes, mapping tables, etc.)
- **Not for large datasets**

### Commands
```bash
dbt seed                    # load all seeds
dbt seed --select country_codes
dbt seed --full-refresh     # drop and recreate
```

### Seed Configuration (`dbt_project.yml`)
```yaml
seeds:
  my_project:
    +schema: seed_data
    country_codes:
      +column_types:
        country_code: varchar(2)
        country_name: varchar(100)
```

### Referencing Seeds
```sql
SELECT *
FROM {{ ref('country_codes') }}   -- same as referencing a model
```

---

## 7. Snapshots

### What are Snapshots?
- Capture **Type 2 Slowly Changing Dimensions (SCD2)** — historical changes in mutable source data
- Live in `snapshots/` directory

### Snapshot Strategies

**`timestamp` strategy** — uses an `updated_at` column:
```sql
{% snapshot orders_snapshot %}
  {{ config(
      target_schema='snapshots',
      unique_key='id',
      strategy='timestamp',
      updated_at='updated_at'
  ) }}
  SELECT * FROM {{ source('jaffle_shop', 'orders') }}
{% endsnapshot %}
```

**`check` strategy** — compares a list of columns:
```sql
{% snapshot orders_snapshot %}
  {{ config(
      target_schema='snapshots',
      unique_key='id',
      strategy='check',
      check_cols=['status', 'amount']      -- or check_cols='all'
  ) }}
  SELECT * FROM {{ source('jaffle_shop', 'orders') }}
{% endsnapshot %}
```

### Snapshot Metadata Columns
| Column | Description |
|---|---|
| `dbt_scd_id` | Unique surrogate key for each snapshot row |
| `dbt_updated_at` | When the row was last updated in the snapshot |
| `dbt_valid_from` | When this record version became valid |
| `dbt_valid_to` | When this version was superseded (`NULL` = current) |

### Running Snapshots
```bash
dbt snapshot
dbt snapshot --select orders_snapshot
```

---

## 8. Tests

### Two Types of Tests

#### 1. Generic Tests (built-in + packages)
Defined in YAML, apply to columns or models:
```yaml
columns:
  - name: order_id
    tests:
      - unique
      - not_null
  - name: status
    tests:
      - accepted_values:
          values: ['placed', 'shipped', 'returned']
  - name: customer_id
    tests:
      - relationships:
          to: ref('stg_customers')
          field: customer_id
```

Built-in generic tests:
- `unique`
- `not_null`
- `accepted_values`
- `relationships`

#### 2. Singular Tests
- Custom SQL files in `tests/` directory
- The query **must return 0 rows** to pass
```sql
-- tests/assert_total_payment_amount_is_positive.sql
SELECT order_id, sum(amount) as total_amount
FROM {{ ref('fct_payments') }}
GROUP BY 1
HAVING total_amount < 0
```

### Test Configuration
```yaml
models:
  - name: stg_orders
    tests:
      - dbt_utils.expression_is_true:
          expression: "amount >= 0"
    columns:
      - name: order_id
        tests:
          - unique:
              config:
                severity: warn      # warn | error (default: error)
                warn_if: ">= 10"
                error_if: ">= 100"
```

### Running Tests
```bash
dbt test                            # all tests
dbt test --select stg_orders        # tests on one model
dbt test --select source:jaffle_shop  # tests on source
dbt test --select tag:finance       # tests by tag
dbt build                           # run + test together (in DAG order)
```

### `dbt build`
Runs seeds → snapshots → models → tests in DAG order. If a model fails its test, downstream models are skipped.

### Storing Test Results
```yaml
# dbt_project.yml
tests:
  +store_failures: true         # saves failed rows to warehouse
  +store_failures_as: view      # view | table | ephemeral
```

---

## 9. Documentation

### Adding Descriptions
```yaml
version: 2

models:
  - name: fct_orders
    description: "One record per order. Fact table."
    columns:
      - name: order_id
        description: "Primary key — surrogate key from source."
```

### Doc Blocks
For longer descriptions, use `docs()` blocks in `.md` files:
```markdown
{% docs order_status %}
The status of the order. Possible values:
- `placed`: order created
- `shipped`: order in transit
- `returned`: order returned
{% enddocs %}
```

Reference in YAML:
```yaml
- name: status
  description: "{{ doc('order_status') }}"
```

### Generating & Serving Docs
```bash
dbt docs generate    # creates catalog.json
dbt docs serve       # local webserver at localhost:8080
```

### What the Docs Site Shows
- Model descriptions and column descriptions
- Lineage graph (DAG)
- Source freshness
- Test status
- Referenced macros

---

## 10. Macros & Jinja

### Jinja Basics in dbt
| Syntax | Purpose |
|---|---|
| `{{ ... }}` | Expression — outputs a value |
| `{% ... %}` | Statement — logic (if, for, set) |
| `{# ... #}` | Comment |

### Common dbt Jinja Functions
```sql
{{ ref('model_name') }}
{{ source('source_name', 'table_name') }}
{{ config(materialized='table') }}
{{ this }}                   -- current model's relation (schema.table)
{{ target.name }}            -- current target (dev / prod)
{{ target.schema }}
{{ target.database }}
{{ env_var('MY_VAR') }}
{{ env_var('MY_VAR', 'default_value') }}
{{ var('my_variable') }}     -- from --vars flag or dbt_project.yml
{{ run_query('SELECT ...') }}
{{ log('message', info=True) }}
```

### Variables
Define in `dbt_project.yml`:
```yaml
vars:
  payment_method: 'credit_card'
  start_date: '2020-01-01'
```

Override at runtime:
```bash
dbt run --vars '{"payment_method": "bank_transfer", "start_date": "2021-01-01"}'
```

Use in SQL:
```sql
WHERE payment_method = '{{ var("payment_method") }}'
```

### Writing Macros
```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name, scale=2) %}
    ({{ column_name }} / 100)::numeric(16, {{ scale }})
{% endmacro %}
```

Call in a model:
```sql
SELECT
    {{ cents_to_dollars('amount') }} as amount_dollars
FROM {{ ref('stg_payments') }}
```

### Dispatch (adapter polymorphism)
`dbt.dispatch('macro_name', 'package_name')` is used to create adapter-specific macro overrides.

### `generate_schema_name` macro
Controls how dbt names schemas. Default: `target_schema` + `custom_schema`.
Override in `macros/generate_schema_name.sql` to change behavior (e.g., in production use only the custom schema name).

```sql
{% macro generate_schema_name(custom_schema_name, node) -%}
    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}
        {{ default_schema }}
    {%- else -%}
        {{ custom_schema_name | trim }}
    {%- endif -%}
{%- endmacro %}
```

---

## 11. Packages

### What are Packages?
- Reusable collections of models, macros, and tests
- Installed via `dbt deps`
- Defined in `packages.yml`

### Key Packages to Know

**`dbt-labs/dbt_utils`** — utility macros & tests
- `dbt_utils.surrogate_key([cols])` → hashed surrogate key
- `dbt_utils.generate_series(n)` → integer series
- `dbt_utils.date_spine(...)` → date spine table
- `dbt_utils.union_relations([refs])` → UNION ALL multiple models
- `dbt_utils.pivot(...)` → pivot rows to columns
- Test: `dbt_utils.expression_is_true`
- Test: `dbt_utils.unique_combination_of_columns`
- Test: `dbt_utils.not_null_proportion`

**`dbt-labs/codegen`**
- Generates `source` YAML, `model` YAML boilerplate

**`calogica/dbt_date`**
- Date utility macros

**`dbt-labs/dbt_audit_helper`**
- Compare two models or query results for data auditing

### Installing Packages
```bash
dbt deps          # installs packages from packages.yml into dbt_packages/
```

---

## 12. Exposures

### What are Exposures?
- Document downstream consumers of your dbt models (dashboards, ML models, apps)
- Appear in lineage graph to show end-to-end data flow

```yaml
# models/exposures.yml
version: 2

exposures:
  - name: weekly_orders_dashboard
    type: dashboard              # dashboard | notebook | analysis | ml | application
    maturity: high               # low | medium | high
    url: https://bi.company.com/dashboard/orders
    description: "Weekly orders KPI dashboard for the ops team."
    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')
    owner:
      name: Jane Smith
      email: jane@company.com
```

---

## 13. Analyses

### What are Analyses?
- `.sql` files in `analyses/` — compiled but **not** materialized
- Used for exploratory queries, ad-hoc analysis, audit queries
- Can use `ref()` and `source()`
- Compiled output goes to `target/compiled/`

```bash
dbt compile     # compiles all SQL (models + analyses) without running
```

---

## 14. Hooks & Operations

### Hooks
SQL statements that run at specific points in the dbt lifecycle.

| Hook | When it runs |
|---|---|
| `pre-hook` | Before a model/seed/snapshot runs |
| `post-hook` | After a model/seed/snapshot runs |
| `on-run-start` | Before any nodes run (project-level) |
| `on-run-end` | After all nodes run (project-level) |

```yaml
# dbt_project.yml
on-run-start:
  - "CREATE SCHEMA IF NOT EXISTS audit"

on-run-end:
  - "{{ log_run_results() }}"

models:
  my_project:
    marts:
      +post-hook: "GRANT SELECT ON {{ this }} TO ROLE reporter"
```

### Operations
- `dbt run-operation macro_name` runs a macro directly (not as part of a model)
- Used for warehouse admin tasks (grants, vacuums, etc.)

```bash
dbt run-operation grant_select --args '{"role": "reporter"}'
```

---

## 15. Materializations

### The 4 Built-in Materializations

| Materialization | What dbt Does | When to Use |
|---|---|---|
| `view` | `CREATE OR REPLACE VIEW` | Staging, light transforms; fresh every query |
| `table` | `CREATE TABLE AS SELECT` (drop & recreate) | When query is expensive; downstream needs speed |
| `incremental` | `INSERT` / `MERGE` new/changed rows | Large fact tables; append-only or upsert patterns |
| `ephemeral` | CTE injected inline | Intermediate logic; no warehouse object created |

### Choosing a Materialization
```
Is it staging?         → view
Is it a mart/fact?     → table or incremental
Is it intermediate?    → ephemeral or view
Is it huge & growing?  → incremental
```

### Custom Materializations
Can be written in `macros/materializations/`.

---

## 16. Incremental Models (Deep Dive)

### Basic Pattern
```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

SELECT
    order_id,
    customer_id,
    amount,
    created_at
FROM {{ source('jaffle_shop', 'orders') }}

{% if is_incremental() %}
    WHERE created_at > (SELECT MAX(created_at) FROM {{ this }})
{% endif %}
```

- `{{ this }}` refers to the **existing** table in the warehouse
- `is_incremental()` is `False` on first run or `--full-refresh`

### Incremental Strategies
| Strategy | Behavior | Best For |
|---|---|---|
| `append` | Insert only, no dedup | Immutable event logs |
| `merge` | MERGE (upsert) on `unique_key` | Mutable tables |
| `delete+insert` | Delete matching rows, re-insert | Partitioned tables |
| `insert_overwrite` | Replace partitions | BigQuery, Spark |

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id',
    incremental_strategy='merge'
) }}
```

### Full Refresh
```bash
dbt run --full-refresh                   # rebuilds all incremental tables
dbt run --full-refresh --select fct_events
```

### `on_schema_change`
Controls what happens when model columns change:
```sql
{{ config(
    materialized='incremental',
    on_schema_change='sync_all_columns'  # ignore | fail | append_new_columns | sync_all_columns
) }}
```

---

## 17. Environments & Targets

### Environments
A **target** = one set of connection credentials (schema, database, warehouse, etc.)

Common setup:
| Target | Schema | Purpose |
|---|---|---|
| `dev` | `dbt_<username>` | Individual dev sandbox |
| `ci` | `dbt_ci_<pr_number>` | Automated PR checks |
| `prod` | `analytics` | Production models |

### Dynamic Schema with `target`
```sql
{% if target.name == 'prod' %}
    schema: analytics
{% else %}
    schema: dbt_{{ target.schema }}
{% endif %}
```

### Environment Variables
```sql
{{ env_var('DBT_ENV_SECRET_PASSWORD') }}
```
- In dbt Cloud: set under **Environment Variables** (per environment or globally)
- Prefixed `DBT_ENV_SECRET_` values are masked in logs

---

## 18. dbt Cloud vs dbt Core

### dbt Core
- Open-source CLI tool
- Run via terminal: `dbt run`, `dbt test`, etc.
- Self-managed infrastructure
- No UI (except `dbt docs serve`)

### dbt Cloud
Hosted platform on top of dbt Core. Key features:

| Feature | Details |
|---|---|
| **IDE** | Browser-based SQL editor with autocomplete and lineage |
| **Jobs** | Scheduled or API-triggered runs |
| **Environments** | Separate dev / prod credentials |
| **CI/CD** | Slim CI — runs only changed models on PRs |
| **Metadata API** | Query dbt Cloud metadata via GraphQL |
| **Semantic Layer** | Define metrics centrally (formerly dbt Metrics) |
| **Explorer** | Enhanced lineage + documentation UI (replaces legacy docs) |
| **Webhooks** | Trigger external systems on job completion |

### dbt Cloud Job Configuration
- **Run steps**: sequence of dbt commands (`dbt build`, `dbt test`, etc.)
- **Schedule**: cron syntax or interval
- **Environment**: links to a defined environment (credentials)
- **Generate docs**: can auto-generate on each run
- **Notifications**: Slack, email on success/failure

### Slim CI (dbt Cloud)
```bash
# Only run models changed vs the production manifest
dbt run --select state:modified+
dbt test --select state:modified+
```
Requires `--state` flag pointing to previous production `manifest.json`.

---

## 19. Selectors & Node Selection

### Selection Syntax
```bash
dbt run --select <selector>
```

| Selector | Meaning |
|---|---|
| `stg_orders` | Exact model name |
| `staging.*` | All models in staging folder |
| `tag:finance` | All nodes with tag `finance` |
| `+stg_orders` | stg_orders and all upstream nodes |
| `stg_orders+` | stg_orders and all downstream nodes |
| `+stg_orders+` | stg_orders + all upstream + all downstream |
| `1+stg_orders` | stg_orders + 1 level upstream |
| `stg_orders+2` | stg_orders + 2 levels downstream |
| `source:jaffle_shop` | All models sourced from jaffle_shop |
| `source:jaffle_shop.orders` | Specific source table |
| `path:models/staging` | Models in a path |
| `config.materialized:incremental` | All incremental models |
| `state:modified` | Changed since last state |
| `state:new` | New since last state |
| `exposure:weekly_orders+` | All models feeding an exposure |
| `result:error` | Models that errored in last run |

### Set Operators
```bash
# Union (space = OR)
dbt run --select stg_orders stg_customers

# Intersection (comma = AND)
dbt run --select tag:finance,config.materialized:table

# Exclusion
dbt run --select staging.* --exclude stg_payments
```

### YAML Selectors
Define reusable selectors in `selectors.yml`:
```yaml
selectors:
  - name: finance_tables
    definition:
      union:
        - method: tag
          value: finance
        - method: config.materialized
          value: table
```

---

## 20. Metadata & Artifacts

### Key Artifacts (in `target/`)
| File | Contents |
|---|---|
| `manifest.json` | Full project graph — all nodes, configs, tests, dependencies |
| `run_results.json` | Status, timing, row counts for last run |
| `catalog.json` | Column-level metadata from warehouse (after `dbt docs generate`) |
| `sources.json` | Source freshness results |

### Using Artifacts for State
```bash
dbt run --select state:modified+ --state /path/to/previous/target
```

### dbt Cloud Metadata API
- GraphQL API: `https://metadata.getdbt.com/graphql`
- Query model runs, test results, source freshness across all jobs
- Requires dbt Cloud API token

---

## 21. Best Practices & Anti-Patterns

### ✅ Best Practices

**Project structure:**
- Separate staging, intermediate, marts layers
- One model per file, one `SELECT` per model
- Name files consistently: `stg_`, `int_`, `fct_`, `dim_`
- Store credentials in `profiles.yml` outside the project repo
- Use `env_var()` for secrets

**Modeling:**
- Always use `ref()` and `source()` — never hardcode schema.table
- Keep staging models 1:1 with sources (rename, recast, basic clean only)
- Add surrogate keys in staging with `dbt_utils.surrogate_key()`
- Document models and columns in YAML

**Testing:**
- At minimum: `not_null` + `unique` on every primary key
- `relationships` tests across models
- `accepted_values` for status/type columns
- Run `dbt build` (not just `dbt run`) to ensure tests pass with each run

**Performance:**
- Use `incremental` for large tables
- Set appropriate `threads` in profiles.yml
- Use `ephemeral` to avoid unnecessary warehouse objects

### ❌ Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Hardcoded schema names | Breaks across environments | Use `ref()` / `source()` |
| Logic in staging models | Hard to debug, violates layer separation | Push logic to intermediate/marts |
| No tests | Silent data quality failures | Add `not_null`, `unique` at minimum |
| Overusing `ephemeral` | Long compile times, hard to debug | Use `view` for debugging phases |
| Huge monolithic models | Slow to run, hard to test | Break into smaller models |
| Credentials in `dbt_project.yml` | Security risk | Use `profiles.yml` + `env_var()` |
| `SELECT *` in marts | Implicit column exposure | Explicitly select columns |

---

## 22. Quick-Reference Cheat Sheet

### CLI Commands
```bash
dbt init <project_name>         # scaffold new project
dbt debug                       # test connection
dbt compile                     # compile SQL without running
dbt run                         # run models
dbt test                        # run tests
dbt build                       # seed + snapshot + run + test
dbt seed                        # load seed files
dbt snapshot                    # run snapshots
dbt source freshness            # check source freshness
dbt docs generate               # build catalog.json
dbt docs serve                  # serve docs locally
dbt deps                        # install packages
dbt clean                       # delete target/ and dbt_packages/
dbt ls                          # list nodes matching selector
dbt run-operation <macro>       # run a macro directly
```

### Useful Flags
```bash
--select / -s               # node selection
--exclude                   # exclude nodes
--profiles-dir              # custom profiles.yml path
--profile                   # override profile name
--target / -t               # override target (dev/prod)
--vars                      # pass variables as JSON
--full-refresh              # drop and recreate incremental/snapshots
--fail-fast                 # stop on first error
--threads                   # override thread count
--state                     # path to previous artifacts (for slim CI)
--no-version-check          # skip dbt version check
```

### Jinja Quick Reference
```sql
{{ ref('model') }}
{{ source('name', 'table') }}
{{ this }}
{{ target.name }} / {{ target.schema }} / {{ target.database }}
{{ env_var('VAR') }}
{{ var('var_name') }}
{{ config(...) }}
{{ is_incremental() }}
{% if condition %} ... {% else %} ... {% endif %}
{% for item in list %} ... {% endfor %}
{% set x = value %}
{{ log('msg', info=True) }}
```

### Materialization Summary
```
view        → CREATE OR REPLACE VIEW     → always fresh, no storage cost
table       → DROP + CREATE TABLE AS     → fast reads, full refresh each run
incremental → INSERT/MERGE new rows      → efficient for large tables
ephemeral   → CTE injected inline        → no warehouse object, debug-unfriendly
```

### Test Types Summary
```
Generic (YAML):  unique | not_null | accepted_values | relationships
Singular (SQL):  SELECT rows that FAIL — must return 0 rows to pass
Package tests:   dbt_utils.unique_combination_of_columns | expression_is_true | etc.
```
