# Homework 6 – Distributed Data Processing with Spark

## Objective
The Silver layer contains cleaned data that is suitable for analysis. However, many analytical workloads require **large-scale aggregations and transformations** that benefit from distributed computing.

In this assignment you will use **Apache Spark in Azure Synapse** to process Silver layer data and create **Gold layer analytical datasets**.

These datasets will power downstream analytics and visualization workloads.

---

## Architecture Context

Bronze Layer (Raw Data)  
↓  
Silver Layer (Clean Data)  
↓  
Apache Spark Transformations  
↓  
Gold Layer (Aggregated Data)

The **Gold layer** contains curated datasets designed specifically for analytics and reporting.

---

## Tools Used
- Azure Synapse Analytics
- Apache Spark
- PySpark DataFrames
- Azure Data Lake Storage

Spark Documentation:  
https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-overview

---

## Cost Control Guidelines
Spark clusters consume compute resources while running.

Important practices:
- Shut down clusters when finished
- Avoid leaving notebooks running
- Run only the cells required for testing

Stop clusters in:  
Synapse Studio → Manage → Apache Spark Pools

---

## Assignment Tasks

### 1. Load Silver Layer Data
Load the following datasets:

silver/weather  
silver/air_pollution

Documentation:  
https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-dataframe

---

### 2. Perform Aggregations
Create at least **four analytical datasets** using Spark.

Examples include:
- daily weather summary
- daily AQI average
- pollutant averages
- temperature extremes

Use Spark operations such as:
- groupBy
- aggregations
- joins
- filtering

Spark SQL Functions:  
https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html

---

### 3. Write Results to the Gold Layer
Save the resulting datasets to:

/gold/

Use:
Parquet format

Documentation:  
https://learn.microsoft.com/en-us/azure/data-factory/format-parquet

---

## Deliverables
Submit:

1. Exported Synapse notebook (.html or .ipynb)
2. Screenshot of Gold layer datasets in ADLS
3. Screenshot showing Spark notebook execution

---

## Production Considerations (Reflection Only)
These questions are intended to help you think about real-world production environments.

- Why is distributed computing useful for large-scale data processing?
- What challenges might arise if datasets grow to billions of records?
- When would you choose Spark instead of SQL for data transformations?
