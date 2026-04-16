# Homework 7 – Data Warehousing with Azure Synapse

## Overview

In Homework 6, you used Apache Spark to transform Silver data into **Gold layer datasets** stored as Parquet files in Azure Data Lake Storage. These datasets are clean, aggregated, and ready for analysis.

However, many organizations require a structured **data warehouse layer** to support reporting, BI tools, and standardized analytics workflows.

In Homework 7, you will take your Gold datasets and build a **data warehouse using Azure Synapse SQL**.

Azure Synapse SQL Overview
https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview
What is a Data Warehouse?
https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/data-warehousing

---

## How This Builds on HW6

In HW6, you created datasets such as:

- daily_weather_summary  
- daily_aqi_summary  
- daily_pollutant_summary  
- temperature_extremes  

These datasets are:

- clean  
- aggregated  
- stored as files  

In HW7, you will transform these into:

Gold Parquet Files  
↓  
External Tables (SQL access to files)  
↓  
Internal Warehouse Tables (Fact + Dimensions)  

---

## Architecture Context

Gold Layer (Parquet in ADLS)  
↓  
Synapse SQL External Tables  
↓  
Internal Warehouse Tables  
↓  
Analytics / BI  

---

## Tools Used

- Azure Synapse Analytics  
- Synapse SQL  
- Azure Data Lake Storage  

---

## Cost Control Guidelines

- Use the smallest SQL pool available  
- Start the pool only when needed  
- Pause the pool immediately after use  

---

# Part 1 – Create External Tables

## Objective

Expose Gold datasets through SQL without moving the data.

## Requirements

Create external tables for at least:

- daily_weather_summary  
- daily_aqi_summary  

These tables must:

- reference correct ADLS paths  
- use Parquet format  
- match the schema of the Gold datasets  

## Here are some helpful links

- Create External Tables (Synapse SQL)
https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/create-use-external-tables

- External Data Sources
https://learn.microsoft.com/en-us/sql/t-sql/statements/create-external-data-source-transact-sql

- External File Formats (Parquet)
https://learn.microsoft.com/en-us/sql/t-sql/statements/create-external-file-format-transact-sql

- Query Data in ADLS using Synapse
https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/query-data-storage

---

# Part 2 – Create Warehouse Tables

## Objective

Create a simple warehouse schema.

## Required Tables

### Fact Table

- FactWeather

### Dimension Tables

- DimDate  
- DimLocation  
- DimWeatherCondition  

## Helpful Links
- Star Schema Design (Microsoft)
https://learn.microsoft.com/en-us/power-bi/guidance/star-schema

- Dimensional Modeling Basics
https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/oltp-data-modeling

- Designing Tables in Synapse SQL
https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-overview

---

# Part 3 – Load Data

## Objective

Load data from external tables into internal tables.

## Requirements

- Use INSERT or CTAS statements  
- Populate FactWeather from daily_weather_summary  
- Populate DimDate using distinct dates  
- Populate DimLocation (can be static if needed)  

## Helpful Links
- INSERT INTO (T-SQL)
https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql
- CTAS (Create Table As Select)
https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-develop-ctas

---

# Part 4 – Validate Data

## Objective

Ensure tables contain valid data.

## Required Validation

Run queries such as:

SELECT TOP 10 * FROM FactWeather;  
SELECT COUNT(*) FROM FactWeather;  

You should confirm:

- tables are populated  
- values are reasonable  
- schema is clean  

## Helpful Links
- SELECT Statement (T-SQL)
https://learn.microsoft.com/en-us/sql/t-sql/queries/select-transact-sql
- Aggregate Functions
https://learn.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql
---

# Deliverables

Submit:

1. SQL script(s) used to create:
   - external tables  
   - warehouse tables  
   - data loads  

2. Screenshot showing warehouse tables in Synapse  

3. Screenshot of successful query results  

***Make sure all screenshots have your username in the top right corner***

---

# Grading Breakdown

| Category | Points |
|----------|--------|
| External tables created correctly | 20 |
| Fact table design and creation | 20 |
| Dimension tables | 20 |
| Data loading | 20 |
| Validation queries | 10 |
| Organization and clarity | 10 |
| Total | 100 |

---

# Things to Consider (Not Graded)

- Why separate fact and dimension tables?  
- What advantages does a warehouse provide over querying files directly?  
- When would you query Gold directly instead of building a warehouse?  

---

# What Success Looks Like

You should be able to explain:

“I exposed Gold data through external tables, structured it into a fact/dimension model, and loaded it into a warehouse for analytics.”

