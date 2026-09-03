# 📘 dbt Learning Notes

A structured collection of notes, concepts, and interview preparation material for **dbt (data build tool)** — built while learning from scratch.

> **Goal:** Understand dbt deeply enough to answer interview questions with confidence and eventually work with dbt on real data warehouse projects (Snowflake, BigQuery, etc.)

---

## 🗂️ Notes Index

| # | File | Topic | Status |
|---|------|-------|--------|
| 01 | [What is dbt?](notes/01_what_is_dbt.md) | Definition, ELT vs ETL, supported platforms, dbt Core vs Cloud | ✅ |
| 02 | [dbt Architecture](notes/02_dbt_architecture.md) | How dbt works internally, Snowflake connection, DAG intro, key commands | ✅ |
| 03 | [dbt Project Structure](notes/03_dbt_project_structure.md) | Folder layout, dbt_project.yml, models, seeds, snapshots, macros, tests | ✅ |
| 04 | [dbt Models In Depth](notes/04_dbt_models_in_depth.md) | Materializations (view/table/incremental/ephemeral), ref(), adapter | ✅ |
| 05 | ref() + source() + DAG | Dependency graph, source() vs ref(), lineage, multi-env resolution | 🔜 |
| 06 | dbt Testing | Generic tests, singular tests, schema.yml, custom tests | 🔜 |
| 07 | Jinja & Macros | Jinja templating, macros, variables, conditionals in SQL | 🔜 |
| 08 | Incremental Models (Deep Dive) | is_incremental(), {{ this }}, unique_key, strategies | 🔜 |
| 09 | Snapshots (SCD Type 2) | Tracking history, valid_from/valid_to, snapshot config | 🔜 |
| 10 | dbt Seeds | Loading CSV data, when to use, limitations | 🔜 |

---

## 🧱 What is dbt?

> **dbt (data build tool)** is a transformation framework that enables data engineers and analytics engineers to transform data **inside** data warehouses and lakehouses using **SQL and Jinja**. It also provides built-in features for **testing, documentation, dependency management, and deployment** of data transformations.

### Where dbt fits in the Modern Data Stack

```
Source Systems  (CRM, ERP, APIs, logs)
       |
   Ingestion  (Fivetran, Airbyte, Stitch)
       |
Data Warehouse / Lakehouse  (raw layer)
       |
      dbt   <-- transforms data here
       |
Analytics-ready Data  (BI tools, ML, dashboards)
```

---

## ⚡ Key Concepts at a Glance

### Materializations

| Type          | Creates in Warehouse | Use Case                          |
|---------------|----------------------|-----------------------------------|
| `view`        | Virtual view         | Lightweight transformations       |
| `table`       | Physical table       | Frequently queried models         |
| `incremental` | Physical table       | Large tables, process new rows only |
| `ephemeral`   | Nothing (CTE only)   | Intermediate reusable SQL         |

### Project Folder Structure

```
my_dbt_project/
|-- dbt_project.yml      # Master config
|-- models/              # SQL transformations (core of dbt)
|-- tests/               # Data quality tests
|-- seeds/               # Static CSV reference data
|-- snapshots/           # SCD Type 2 historical tracking
|-- macros/              # Reusable Jinja functions
`-- analyses/            # Ad-hoc queries (not materialised)
```

### Supported Platforms

Snowflake · BigQuery · Redshift · Databricks · Azure Synapse · PostgreSQL · DuckDB · Spark

---

## 🎯 Interview Quick Reference

| Question | One-line Answer |
|----------|-----------------|
| What is dbt? | SQL transformation framework with testing, docs, and dependency management |
| Where does dbt execute SQL? | On the data platform (Snowflake, BigQuery, etc.) — not inside dbt itself |
| What is a dbt model? | A `.sql` file containing a SELECT statement that dbt compiles and runs |
| What does `ref()` do? | Declares model dependency, builds DAG, resolves relation name dynamically |
| View vs Table? | View = SQL definition stored; Table = result data physically stored |
| What is an incremental model? | Processes only new/changed rows instead of rebuilding entire table |
| What is ephemeral? | No DB object created — SQL injected as CTE into downstream models |
| What is the DAG? | Directed Acyclic Graph — dbt's dependency map that determines run order |

---

## 🛠️ Setup Used

- **Data Platform:** Snowflake
- **dbt version:** dbt Core
- **Learning approach:** Concept → Notes → Interview Q&A → Practice

---

## 📌 Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Notes complete |
| 🔜 | Coming up next |
| ⭐ | High-value interview topic |