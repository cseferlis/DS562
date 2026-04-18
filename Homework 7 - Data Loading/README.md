# Homework 7 – Data Warehousing with Azure Synapse (Serverless SQL Pool)

## Overview

In Homework 6, you used Apache Spark to transform Silver data into **Gold layer datasets** stored as Parquet files in Azure Data Lake Storage. These datasets are clean, aggregated, and ready for analysis.

In Homework 7, you will use **Azure Synapse serverless SQL pool** to expose those Gold datasets through SQL and create a lightweight warehouse layer that supports downstream analytics and BI.

Because **serverless SQL pool does not support regular internal `CREATE TABLE` warehouse tables**, this assignment uses a combination of:

- **external tables** over Gold data
- **views** for reusable dimensional logic
- **persistent external tables created with CETAS** for your warehouse outputs

This keeps the assignment aligned with a **low-cost, serverless architecture** while still teaching core warehouse concepts.

---

## How This Builds on HW6

In HW6, you created datasets such as:

- `daily_weather_summary`
- `daily_aqi_summary`
- `daily_pollutant_summary`
- `temperature_extremes`

These datasets are:

- clean
- aggregated
- stored as Parquet files in ADLS

In HW7, you will transform these into:

Gold Parquet Files  
↓  
External Tables / Views (SQL access to files)  
↓  
Persistent External Warehouse Tables (CETAS)  
↓  
Analytics / BI

---

## Architecture Context

Gold Layer (Parquet in ADLS)  
↓  
Synapse Serverless SQL Pool  
↓  
External Tables + Views  
↓  
Persistent External Warehouse Tables  
↓  
Analytics / BI  

---

## Tools Used

- Azure Synapse Analytics
- Serverless SQL Pool
- Azure Data Lake Storage
- T-SQL

---

## Documentation Resources

### Serverless SQL Pool Overview
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/on-demand-workspace-overview

### External Tables
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/create-use-external-tables
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-external-tables

### Views in Serverless SQL Pool
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/create-use-views

### CETAS (Create External Table As Select)
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-tables-cetas

### Star Schema / Dimensional Modeling
- https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

---

## Cost Control Guidelines

This assignment is designed around **serverless SQL pool** to reduce cost.

You are expected to:

- use serverless SQL pool, not dedicated SQL pool
- avoid repeatedly scanning the same files unnecessarily
- validate queries with small result sets first
- stop other billable resources (Spark pools, SQL pools) if they are running from earlier homeworks

---

# Part 1 – Create External Tables Over Gold Data

## Objective

Expose your Gold datasets through SQL without moving the data.

## Requirements

Create external tables for at least:

- `daily_weather_summary`
- `daily_aqi_summary`

These tables must:

- reference the correct ADLS paths
- use Parquet format
- match the schema of your HW6 Gold outputs

## Expectation

Your schemas must match **your actual HW6 outputs**, not a generic example.

---

# Part 2 – Create a Reusable Date Dimension as a View

## Objective

Create a reusable `DimDate` object using a **view**, not a regular table.

## Why

In serverless SQL pool, regular internal warehouse tables are not supported. A view allows you to create reusable dimensional logic without trying to store internal table data.

## Requirements

Create a view called:

- `DimDate`

It should include a date range appropriate for your project and fields such as:

- `date`
- `year`
- `month`
- `day`
- `quarter`
- `month_name`
- `day_name`
- `is_weekend`

---

# Part 3 – Create Three Persistent Warehouse Tables Using CETAS

## Objective

Create **three persistent external warehouse tables** backed by files in ADLS.

These are the three persistent warehouse objects for this assignment:

- `FactWeather`
- `DimLocation`
- `DimWeatherCondition`

## Why CETAS

Since serverless SQL pool supports **CETAS** rather than regular internal warehouse tables, you will use CETAS to materialize query results into external Parquet-backed tables in storage.

## Requirements

### 3.1 FactWeather
Create a persistent external table called `FactWeather` using CETAS.

It should be built from your weather-oriented Gold data and include measures such as:

- `date`
- `location`
- average temperature
- average humidity
- average wind speed
- max temperature
- min temperature
- record count

### 3.2 DimLocation
Create a persistent external table called `DimLocation`.

A simple implementation is acceptable. If your project only includes Boston, this dimension may include:

- `location`
- `latitude`
- `longitude`

### 3.3 DimWeatherCondition
Create a persistent external table called `DimWeatherCondition`.

You may derive this from available weather condition fields in your existing datasets. A simple implementation is acceptable if your source data is limited.

---

# Part 4 – Validate the Warehouse Objects

## Objective

Confirm that all SQL objects were created successfully and return sensible results.

## Required Validation

You must run validation queries for:

- your external source tables
- `DimDate` view
- `FactWeather`
- `DimLocation`
- `DimWeatherCondition`

Examples:

```sql
SELECT TOP 10 * FROM FactWeather;
SELECT TOP 10 * FROM DimDate;
SELECT COUNT(*) FROM FactWeather;
```

You should confirm:

- objects exist
- schemas are readable
- results are non-empty where expected
- values are reasonable

---

# Deliverables

Submit:

1. SQL script(s) used to create:
   - external source tables
   - `DimDate` view
   - CETAS-based warehouse tables

2. Screenshot showing the SQL objects in Synapse

3. Screenshot of successful query results from:
   - `FactWeather`
   - `DimDate`
   - one additional warehouse object of your choice

---

# AI Usage Expectations

This assignment is designed so that generic AI-generated SQL will not work without customization.

You must:

- align schemas to **your actual HW6 Gold outputs**
- provide screenshots from **your environment**
- use **your actual ADLS paths**
- include at least **one small design choice** that reflects your implementation (for example, a chosen date range for `DimDate` or a selected set of condition fields in `DimWeatherCondition`)

---

# Required Explanation (Graded)

Answer briefly:

1. Why did you choose the columns included in `FactWeather`?
2. Why is `DimDate` implemented as a view in this assignment?
3. What challenge did you encounter aligning your SQL objects to your HW6 outputs?

---

# Grading Breakdown

| Category | Points |
|----------|--------|
| External source tables | 15 |
| DimDate view | 15 |
| FactWeather (CETAS) | 20 |
| DimLocation (CETAS) | 10 |
| DimWeatherCondition (CETAS) | 10 |
| Validation queries | 10 |
| Required explanation | 10 |
| Organization and clarity | 10 |
| Total | 100 |

---

# Things to Consider (Not Graded)

- What are the tradeoffs between querying Gold files directly and creating reusable warehouse objects?
- Why is a date dimension often modeled independently from any single fact table?
- What would change if you were using a dedicated SQL pool instead of serverless SQL pool?

---

# What Success Looks Like

You should be able to explain:

“I exposed Gold data through external tables, created a reusable date dimension as a view, materialized three persistent warehouse tables with CETAS, and validated the results for analytics.”

