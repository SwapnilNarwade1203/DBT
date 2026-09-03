# dbt Architecture — How dbt Works Internally

## High-Level Architecture

```
+-------------------+       profiles.yml        +--------------------+
|   dbt Project     |  -----------------------> |  Data Platform     |
|                   |   (connection config)      |  (Snowflake /      |
|  models/          |                            |   BigQuery /       |
|  tests/           |       SQL queries          |   Redshift etc.)   |
|  macros/          |  -----------------------> |                    |
|  snapshots/       |                            |  Runs SQL on its   |
|  seeds/           |  <----------------------- |  own compute       |
|  dbt_project.yml  |   results / metadata       |                    |
+-------------------+                            +--------------------+
```

dbt is a **compiler + orchestrator**.
- It compiles your `.sql` model files into raw SQL
- It sends that SQL to the connected data platform to execute
- **dbt itself does NOT process any data** — the warehouse does

---

## Core Components of a dbt Project

```
my_dbt_project/
|
|-- dbt_project.yml       # Project config (name, version, paths, vars)
|-- profiles.yml          # Connection credentials (outside project dir)
|
|-- models/               # SQL transformation logic (SELECT statements)
|   |-- staging/
|   |   |-- stg_orders.sql
|   |   |-- stg_customers.sql
|   |-- marts/
|       |-- dim_customers.sql
|       |-- fct_orders.sql
|
|-- tests/                # Custom data tests (.sql)
|-- macros/               # Reusable Jinja functions
|-- snapshots/            # SCD Type 2 tracking
|-- seeds/                # CSV reference data
|-- analyses/             # Ad-hoc SQL queries (not materialised)
|-- target/               # Compiled SQL output (auto-generated)
|-- packages.yml          # dbt packages / dependencies
```

---

## How dbt Connects to Snowflake

Connection is defined in **profiles.yml** (stored in `~/.dbt/profiles.yml`):

```yaml
my_snowflake_project:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: your_account.region          # e.g. xy12345.us-east-1
      user: your_username
      password: your_password               # or use key-pair / SSO
      role: TRANSFORMER                     # Snowflake role
      database: ANALYTICS                   # Target database
      warehouse: COMPUTE_WH                 # Snowflake virtual warehouse
      schema: DEV_SWAPNIL                   # Your dev schema
      threads: 4                            # Parallel model runs
```

When you run `dbt run`:
1. dbt reads `profiles.yml` → establishes connection to Snowflake
2. dbt compiles your model SQL (resolving `ref()` and `source()` functions)
3. dbt sends the compiled SQL to Snowflake
4. Snowflake executes it and creates tables / views in your schema
5. dbt collects results and displays them in your terminal

---

## The DAG — How dbt Manages Dependencies

dbt builds a **Directed Acyclic Graph (DAG)** from your models automatically.

```
raw.orders  -->  stg_orders  -->  fct_orders  -->  (BI tool)
                                      |
raw.customers --> stg_customers --> dim_customers
```

You declare dependencies using `ref()`:

```sql
-- fct_orders.sql
select
    o.order_id,
    c.customer_name,
    o.amount
from {{ ref('stg_orders') }}   AS o   -- depends on stg_orders
join {{ ref('stg_customers') }} AS c  -- depends on stg_customers
    on o.customer_id = c.customer_id
```

dbt automatically:
- Figures out run order (stg_ models before fct_ models)
- Runs independent models in parallel (controlled by `threads`)
- Re-runs only downstream models when upstream changes

---

## Materialisation Types

| Type          | What it creates in Snowflake         | When to use                        |
|---------------|--------------------------------------|------------------------------------|
| `view`        | A virtual view (default)             | Light, frequently changing models  |
| `table`       | A full physical table (recreated)    | Final mart / reporting tables      |
| `incremental` | Appends/merges only new rows         | Large fact tables                  |
| `ephemeral`   | CTE — no object created in warehouse | Intermediate logic, not exposed    |

Set per model in `dbt_project.yml` or inline:

```sql
-- models/marts/fct_orders.sql
{{ config(materialized='table') }}

select ...
```

---

## dbt Run Flow — Step by Step

```
dbt run
  |
  1. Parse dbt_project.yml + profiles.yml
  |
  2. Compile all model .sql files
  |   - Resolve {{ ref() }} and {{ source() }}
  |   - Apply Jinja macros
  |
  3. Build the DAG (dependency graph)
  |
  4. Execute models in topological order
  |   - Connect to Snowflake via profiles.yml
  |   - Run compiled SQL (CREATE TABLE / CREATE VIEW / MERGE)
  |
  5. Report results (pass / error / skip)
```

---

## Key dbt Commands (for Snowflake)

| Command              | What it does                                          |
|----------------------|-------------------------------------------------------|
| `dbt debug`          | Test connection to Snowflake                          |
| `dbt run`            | Compile & run all models                              |
| `dbt test`           | Run all data quality tests                            |
| `dbt docs generate`  | Generate documentation site                          |
| `dbt docs serve`     | Open docs in browser                                  |
| `dbt seed`           | Load seed CSV files into Snowflake                    |
| `dbt snapshot`       | Run SCD Type 2 snapshots                              |
| `dbt compile`        | Compile SQL without running (inspect output in target/)|

---

## Next: dbt Models in Depth -> 03_dbt_models.md
