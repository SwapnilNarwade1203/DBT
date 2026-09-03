# dbt Models — In Depth

A dbt model is a SQL file that defines a transformation of data.

```
models/
└── staging/
    └── stg_customers.sql
```

```sql
-- stg_customers.sql
SELECT
    customer_id,
    first_name,
    last_name,
    email
FROM raw.customers;
```

---

## 1. What Happens When `dbt run` Executes?

```
stg_customers.sql
       |
     dbt
       |
Compile SQL + Jinja
       |
Generate platform SQL
       |
Adapter (platform integration)
       |
Snowflake executes SQL
       |
Creates / updates database object
```

> **Key point:** dbt orchestrates and compiles the SQL.
> **Snowflake (or any warehouse) performs the actual execution.**

---

## 2. The Four Materializations

> This is **extremely important** for interviews.

| Materialization | Physical object?   | Typical use                                  |
|-----------------|--------------------|----------------------------------------------|
| `view`          | View (definition)  | Lightweight, frequently changing models      |
| `table`         | Physical table     | Frequently queried, expensive transformations|
| `incremental`   | Physical table     | Large datasets with new/changed data         |
| `ephemeral`     | None               | Intermediate reusable SQL (CTE injection)    |

---

## 3. View Materialization

```yaml
# dbt_project.yml
models:
  ecommerce:
    staging:
      +materialized: view
```

dbt generates and runs:

```sql
CREATE VIEW staging.stg_customers AS
SELECT
    customer_id,
    first_name,
    last_name
FROM raw.customers;
```

### Characteristics

- Stores the **SQL definition**, not the result data
- Cheap to create
- Always reflects the latest underlying data when queried
- Can be expensive to query if the transformation is complex
  (underlying SQL re-executes on every query)

---

## 4. Table Materialization

```yaml
models:
  ecommerce:
    marts:
      +materialized: table
```

dbt generates:

```sql
CREATE TABLE mart.fct_orders AS
SELECT
    order_id,
    customer_id,
    amount
FROM staging.stg_orders;
```

### Characteristics

- **Physically stores** the transformed result
- Better query performance for downstream consumers
- Entire table is **rebuilt** on every `dbt run`
- Can be expensive for very large datasets (full rebuild)

---

## 5. Incremental Materialization ⭐ (High Interview Value)

### The Problem

```
orders table = 500 million rows
New data daily = 2 million rows

Rebuilding 500M rows every day = wasteful
```

### The Solution

```
Existing table (500M rows)
         +
New/changed rows (2M)
         |
    Merge / append
         |
Updated table (502M rows)
```

### Example Model

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM {{ ref('stg_orders') }}

{% if is_incremental() %}

WHERE updated_at > (
    SELECT MAX(updated_at)
    FROM {{ this }}
)

{% endif %}
```

### Key Functions

| Function          | Purpose                                                     |
|-------------------|-------------------------------------------------------------|
| `is_incremental()` | Returns `true` on subsequent runs (not the first full build) |
| `{{ this }}`      | Refers to the existing target table in the warehouse        |
| `unique_key`      | Used for update/merge behaviour (not always required)       |

### Incremental run flow

```
First run:
  → Full table build (no WHERE filter)

Subsequent runs:
  → Only new/changed rows processed
  → Appended or merged into existing table
```

> **Watermark** (`MAX(updated_at)`) and **unique_key** are common techniques,
> not mandatory for every incremental model.

---

## 6. Ephemeral Materialization

```yaml
materialized: ephemeral
```

```sql
-- int_customer.sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM {{ ref('stg_orders') }}
GROUP BY customer_id
```

### What happens

- **No table or view is created** in the warehouse
- dbt injects the SQL into downstream models as a **CTE**

```sql
-- downstream model compiled output
WITH int_customer AS (

    SELECT
        customer_id,
        COUNT(*) AS order_count
    FROM staging.stg_orders
    GROUP BY customer_id

)

SELECT *
FROM int_customer;
```

### Use when

- The model is an intermediate step not needed as a standalone object
- You want to keep the warehouse clean (no extra objects)
- The logic is only consumed by one or two downstream models

---

## 7. How to Specify Materialization

### Option A — Inline config block (model-level)

```sql
{{ config(materialized='table') }}

SELECT *
FROM {{ ref('stg_orders') }}
```

### Option B — dbt_project.yml (directory-level)

```yaml
models:
  ecommerce:
    staging:
      +materialized: view      # all models in staging/
    marts:
      +materialized: table     # all models in marts/
```

### Override priority (most specific wins)

```
Project (dbt_project.yml)
       |
  Directory config
       |
   Model config    <-- wins
```

---

## 8. `ref()` — Why It Matters

Instead of hardcoding:

```sql
FROM analytics.stg_orders       -- BAD: hardcoded schema
```

You write:

```sql
FROM {{ ref('stg_orders') }}    -- GOOD: dbt resolves it
```

### ref() does THREE things

```
1. Creates a dependency
   stg_orders --> fct_orders
   (dbt knows to build stg_orders first)

2. Determines execution order
   Builds the DAG so models run in correct sequence

3. Dynamically resolves the relation name
   Resolves to the correct database/schema/table
   based on environment (dev vs prod)
```

---

## 9. Adapter — Clarification

> The adapter is **not just a SQL translator**.

```
                dbt
                 |
        +--------+--------+
        |                 |
  Core framework       Adapter
                          |
               Platform integration
                          |
          +-----------+---+----------+
          |           |              |
      Snowflake    BigQuery      Redshift
```

The adapter handles:
- Platform-specific connection management
- Platform-appropriate SQL generation (DDL, DML)
- Database object management (create, drop, swap)
- Not a simple line-by-line SQL translation

---

## Interview Q&A

**Q1. What is a dbt model?**

> A dbt model is a SQL-based transformation file that dbt compiles and executes
> on the target data platform. The resulting database object depends on the model's
> materialization — view, table, incremental, or ephemeral.

**Q2. What are the four common dbt materializations?**

> View, table, incremental, and ephemeral.

**Q3. Difference between table and incremental?**

> A table materialization rebuilds the entire result on every run.
> An incremental model processes only new or changed records,
> avoiding a full rebuild and reducing compute cost for large datasets.

**Q4. Does an ephemeral model create a table?**

> No. An ephemeral model creates no database object. Its SQL is incorporated
> into downstream models as a CTE at compile time.

**Q5. Why use `ref()` instead of hardcoding table names?**

> `ref()` creates a dependency in the DAG, lets dbt determine the correct
> execution order, and dynamically resolves the relation name based on the
> environment — avoiding hardcoded schema or database names.

---

## Score Card (Self-Evaluation — 8.9/10)

| Question              | Score  | Key correction                                   |
|-----------------------|--------|--------------------------------------------------|
| Model definition      | 9/10   | SQL file doesn't create object — dbt does        |
| dbt run flow          | 9.5/10 | Adapter = platform integration, not just transfer |
| Execution location    | 10/10  | Snowflake executes ✅                             |
| Materializations      | 8.5/10 | View ≠ "frequently accessed"; it's "cheap build" |
| View vs Table         | 7.5/10 | Compare explicitly, don't define separately      |
| Incremental           | 9.5/10 | watermark + unique_key are techniques, not rules |
| Ephemeral             | 10/10  | CTE injection ✅                                  |
| ref()                 | 7.5/10 | DAG + execution order + name resolution          |

---

## Next: ref() + source() + DAG -> 05_ref_source_dag.md

Topics to cover:
- ref() deep dive
- source() and why it differs from ref()
- DAG visualisation
- Why hardcoded names break multi-environment setups
- Model lineage
