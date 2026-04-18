# Homework 7 – Data Warehousing with Azure Synapse

## Overview

In Homework 6, you used Apache Spark to transform Silver data into **Gold layer datasets** stored as Parquet files in Azure Data Lake Storage. These datasets are clean, aggregated, and ready for analysis.

In Homework 7, you will take your Gold datasets and build a **data warehouse using Azure Synapse SQL**.

---

## How This Builds on HW6

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

## Documentation Resources

### Synapse & Warehousing
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview  
- https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/data-warehousing  

### External Tables
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/create-use-external-tables  
- https://learn.microsoft.com/en-us/sql/t-sql/statements/create-external-data-source-transact-sql  
- https://learn.microsoft.com/en-us/sql/t-sql/statements/create-external-file-format-transact-sql  

### Data Modeling
- https://learn.microsoft.com/en-us/power-bi/guidance/star-schema  

### Data Loading
- https://learn.microsoft.com/en-us/sql/t-sql/statements/insert-transact-sql  
- https://learn.microsoft.com/en-us/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-develop-ctas  

---

## Cost Control Guidelines

- Use the smallest SQL pool available  
- Start only when needed  
- Pause immediately after use  

---

# Part 1 – Create External Tables

## Objective
Expose Gold datasets through SQL without moving the data.

## Requirements

Create external tables for:
- daily_weather_summary  
- daily_aqi_summary  

## Expectation

Your schema MUST match your HW6 output.

---

# Part 2 – Create Warehouse Tables

## Required Tables

### Fact Table
- FactWeather

### Dimension Tables
- DimDate  
- DimLocation  
- DimWeatherCondition  

---

# Part 3 – Load Data

- Load FactWeather from external tables  
- Populate DimDate from distinct values  
- Populate DimLocation (static allowed)  

---

# Part 4 – Validate Data

Run:
SELECT TOP 10 * FROM FactWeather;  
SELECT COUNT(*) FROM FactWeather;  

---

# Deliverables

1. SQL scripts  
2. Screenshots of tables  
3. Query results  

---

# AI Usage Expectations

This assignment is designed so generic AI answers will not work without customization.

You must:

- Align schemas to YOUR HW6 outputs  
- Provide screenshots from YOUR environment  
- Include at least ONE custom column or transformation  

---

# Required Explanation (Graded)

Answer briefly:

- Why did you choose the columns in FactWeather?  
- What challenges did you face aligning schema with HW6?  

---

# Grading Breakdown

| Category | Points |
|----------|--------|
| External Tables | 20 |
| Fact Table | 20 |
| Dimensions | 20 |
| Data Load | 20 |
| Validation | 10 |
| Explanation | 10 |
| Total | 100 |

---

# What Success Looks Like

“I exposed Gold data through external tables, structured it into a warehouse, and validated it for analytics.”
