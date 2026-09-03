# dbt Project Structure

A dbt project is the complete directory containing your dbt code, configuration,
tests, macros, and other project resources.

---

## Typical Project Layout

```
my_dbt_project/
|
|-- dbt_project.yml
|
|-- models/
|   |-- staging/
|   |   |-- stg_customers.sql
|   |   `-- stg_orders.sql
|   |
|   |-- intermediate/
|   |   `-- int_customer_orders.sql
|   |
|   `-- marts/
|       |-- dim_customers.sql
|       `-- fct_orders.sql
|
|-- seeds/
|   `-- country_codes.csv
|
|-- snapshots/
|   `-- customers_snapshot.sql
|
|-- tests/
|   `-- test_order_amount.sql
|
|-- macros/
|   `-- generate_schema_name.sql
|
`-- analyses/
    `-- customer_analysis.sql
```

---

## 1. dbt_project.yml

The **main configuration file** of a dbt project.

```yaml
name: ecommerce
version: '1.0.0'
config-version: 2

profile: ecommerce

model-paths:    ["models"]
test-paths:     ["tests"]
seed-paths:     ["seeds"]
macro-paths:    ["macros"]
snapshot-paths: ["snapshots"]
```

Tells dbt:
- Project name and version
- Where models, tests, seeds, macros, snapshots are located
- Model-level configurations (materialisation defaults, tags, etc.)

> **Interview Q:** What is `dbt_project.yml`?
>
> `dbt_project.yml` is the primary configuration file of a dbt project. It defines
> project-level settings including project name, directory paths, model configurations,
> and other project behaviour.

---

## 2. models/

The **most important directory** — contains all SQL transformation logic.

```
models/
|-- staging/
|   `-- stg_customers.sql      <- cleans raw source data
|
|-- intermediate/
|   `-- int_customer_orders.sql  <- joins / business logic
|
`-- marts/
    `-- fct_orders.sql           <- final analytics-ready table
```

A model is just a `SELECT` statement:

```sql
-- stg_customers.sql
SELECT
    customer_id,
    first_name,
    last_name
FROM {{ source('raw', 'customers') }}
```

Running `dbt run` builds this as a table or view in your warehouse.

---

## 3. seeds/

CSV files loaded into the warehouse via `dbt seed`.

```
seeds/
`-- country_codes.csv
```

```
country_code,country_name
IN,India
US,United States
UK,United Kingdom
```

### When to use seeds

| Good use                    | Bad use                            |
|-----------------------------|------------------------------------|
| Small reference/lookup data | Millions of transactional records  |
| Static mappings             | Continuously changing source data  |
| Country codes               | Operational datasets               |
| Status mappings             |                                    |

---

## 4. snapshots/

Used to **track changes in records over time** — implements Slowly Changing Dimensions (SCD Type 2).

### Example

Original record:
```
customer_id | name | city
101         | John | Pune
```

After update:
```
customer_id | name | city
101         | John | Mumbai
```

Snapshot preserves history:
```
customer_id | city   | valid_from | valid_to
101         | Pune   | Jan 1      | Mar 10
101         | Mumbai | Mar 10     | NULL
```

> `valid_to = NULL` means it is the **current active record**.

---

## 5. tests/

Contains **singular (custom) data tests** — SQL queries that should return **zero rows**.

```sql
-- tests/test_order_amount.sql
SELECT *
FROM {{ ref('fct_orders') }}
WHERE order_amount < 0
```

- If this returns **0 rows** → test **passes** ✅
- If this returns **any rows** → test **fails** ❌

### Two types of tests (covered later)

```
Generic Tests          Singular Tests
(schema.yml)           (tests/ folder)
- not_null             - Custom SQL queries
- unique               - Complex business rules
- accepted_values
- relationships
```

---

## 6. macros/

**Reusable Jinja functions** — avoid repeating complex SQL logic.

```sql
-- macros/cents_to_dollars.sql
{% macro cents_to_dollars(column_name) %}
    {{ column_name }} / 100.0
{% endmacro %}
```

Used in a model:

```sql
SELECT
    {{ cents_to_dollars('amount_cents') }} AS amount
FROM {{ ref('stg_orders') }}
```

> Macros are one of the **most important topics for advanced dbt interviews**.

---

## 7. analyses/

SQL queries for **ad-hoc analysis** — not materialised into the warehouse.

```
analyses/
`-- monthly_revenue_analysis.sql
```

- Useful for exploratory queries
- Can use `ref()` and `source()` like models
- `dbt compile` will resolve them but `dbt run` will NOT execute them

---

## Priority Focus for Interviews

| Directory         | Purpose                          | Interview Weight |
|-------------------|----------------------------------|-----------------|
| `models/`         | SQL transformations              | ⭐⭐⭐⭐⭐          |
| `tests/`          | Data quality validation          | ⭐⭐⭐⭐           |
| `macros/`         | Reusable Jinja/SQL logic         | ⭐⭐⭐⭐           |
| `seeds/`          | Load CSV reference data          | ⭐⭐⭐            |
| `snapshots/`      | Track historical changes (SCD2)  | ⭐⭐⭐            |
| `dbt_project.yml` | Project configuration            | ⭐⭐⭐            |
| `analyses/`       | Ad-hoc queries                   | ⭐⭐             |

---

## Star Interview Answer

**Q: Explain the structure of a dbt project.**

> "A dbt project contains a `dbt_project.yml` configuration file and directories
> such as `models`, `tests`, `seeds`, `snapshots`, `macros`, and `analyses`.
> Models contain SQL transformations, tests validate data quality, seeds contain
> static CSV reference data, snapshots track historical changes using SCD Type 2,
> macros provide reusable Jinja logic, and `dbt_project.yml` holds project-level
> configuration including directory paths and model settings."

---

## Summary Map

```
dbt Project
     |
     |-- models/       -> Transform data (SQL SELECT statements)
     |-- tests/        -> Validate data quality
     |-- seeds/        -> Load static CSV reference data
     |-- snapshots/    -> Track historical record changes (SCD2)
     |-- macros/       -> Reusable Jinja/SQL functions
     |-- analyses/     -> Ad-hoc queries (not materialised)
     `-- dbt_project.yml -> Master config file
```

---

## Next: dbt Models in Depth -> 04_dbt_models_in_depth.md

Topics: what happens on `dbt run`, how `.sql` becomes a Snowflake table/view,
and all four materialisation types (view, table, incremental, ephemeral).
