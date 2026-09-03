# What is dbt?

## Interview-Ready Definition

> **dbt (data build tool)** is a transformation framework that enables data engineers and analytics engineers
> to transform data **inside** data warehouses and lakehouses using **SQL and Jinja**.
> It also provides built-in features for **testing, documentation, dependency management, and deployment**
> of data transformations.

---

## Where dbt Fits in the Modern Data Stack

```
Source Systems  (CRM, ERP, APIs, logs...)
       |
   Ingestion  (Fivetran, Airbyte, Stitch)
       |
Data Warehouse / Lakehouse  (raw / staging layer)
       |
      dbt  <-- YOU ARE HERE
       |
 SQL Transformations (models, tests, docs)
       |
Analytics-ready Data  (BI tools, dashboards, ML)
```

---

## Supported Data Platforms

dbt is **not** limited to cloud-only. It supports:

| Platform          | Type            |
|-------------------|-----------------|
| Snowflake         | Cloud DW        |
| BigQuery          | Cloud DW        |
| Redshift          | Cloud DW        |
| Databricks        | Cloud Lakehouse  |
| Azure Synapse     | Cloud DW        |
| PostgreSQL        | On-prem / Cloud |
| DuckDB            | Local / Embedded|
| Spark             | Distributed     |

---

## What dbt Provides (Beyond Just SQL)

| Feature                   | Description                                               |
|---------------------------|-----------------------------------------------------------|
| **Models**                | SELECT statements saved as `.sql` files                   |
| **Testing**               | Built-in & custom data quality checks                     |
| **Documentation**         | Auto-generated docs from schema descriptions              |
| **Dependency Management** | dbt resolves model run-order via a DAG                    |
| **Jinja Templating**      | Macros, variables, conditionals inside SQL                |
| **Incremental Loads**     | Only process new/changed data                             |
| **Snapshots**             | Track slowly changing dimensions (SCD Type 2)             |
| **Seeds**                 | Load CSV files as reference/lookup tables                 |

---

## Key Differentiator — ELT, not ETL

dbt follows the **ELT** (Extract → Load → Transform) pattern:

- Data is first **loaded** into the warehouse *as-is*
- dbt then **transforms** it *inside* the warehouse using SQL
- This leverages the warehouse's own compute — no separate ETL engine needed

---

## Two Flavors of dbt

| dbt Core               | dbt Cloud                          |
|------------------------|------------------------------------|
| Open-source CLI tool   | Managed SaaS platform              |
| Free                   | Paid (has free tier)               |
| Local development      | IDE, scheduler, CI/CD built-in     |
| You manage infra       | Fully managed                      |

---

## Next: dbt Architecture -> 02_dbt_architecture.md
