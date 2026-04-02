# Homework 6 – Distributed Data Processing with Spark

## Overview

In Homework 5, you transformed raw Bronze JSON into clean Silver Parquet datasets. These datasets are now usable, but still too detailed for efficient analytics.

In Homework 6, you will use Apache Spark in Azure Synapse to process the Silver datasets and produce Gold layer analytical datasets.

This assignment focuses on:
- loading structured Parquet data with Spark
- validating schemas and data quality
- computing daily aggregations
- writing curated outputs to the Gold layer
- managing Spark compute costs responsibly

---

## Architecture Context

Silver Layer (Parquet)
        ->
Azure Synapse Spark Notebook
        ->
Aggregations and Transformations
        ->
Gold Layer (Parquet)

---

## Tools Used

- Azure Synapse Analytics
- Apache Spark
- PySpark DataFrames
- Azure Data Lake Storage

---

## Cost Control Guidelines

- Use the smallest Spark pool possible
- Avoid unnecessary reruns
- Detach notebook when done
- Ensure Spark pool is stopped after use

---

# Part 1 – Load and Validate Silver Data

## Required Inputs

- /silver/weather
- /silver/air_pollution

## Required Steps

1. Load both datasets using Spark
2. Print schema and preview data
3. Count total rows
4. Create a daily date column
5. Check null values for key fields

---

# Part 2 – Create Gold Datasets

You must create the following datasets:

## 1. Daily Weather Summary
Includes:
- date
- avg temperature
- avg humidity
- avg wind speed
- max temperature
- min temperature
- record count

## 2. Daily AQI Summary
Includes:
- date
- avg AQI
- max AQI
- min AQI
- record count

## 3. Daily Pollutant Summary
Includes averages for:
- PM2.5
- PM10
- O3
- NO2
- CO
- SO2

## 4. Temperature Extremes
Includes:
- date
- max temperature
- min temperature

---

## Required Spark Operations

- groupBy
- avg, max, min, count
- derived date columns
- optional joins

---

# Part 3 – Write Gold Outputs

Write each dataset to:

/gold/

Example structure:

/gold/daily_weather_summary/
/gold/daily_aqi_summary/
/gold/daily_pollutant_summary/
/gold/temperature_extremes/

Format: Parquet

---

# Part 4 – Validate Outputs

1. Read each dataset back into Spark
2. Print schema
3. Show sample rows
4. Confirm data exists in ADLS
5. Verify values are reasonable

---

# Notebook Structure

1. Imports
2. Paths
3. Load data
4. Validation
5. Transformations
6. Write outputs
7. Validate outputs

---

# Deliverables

1. Exported Synapse notebook (.html or .ipynb)
2. Screenshot of Gold layer in ADLS
3. Screenshot of successful notebook execution

---

# Grading Breakdown

| Category | Points |
|----------|--------|
| Data loading and validation | 20 |
| Daily Weather Summary | 15 |
| Daily AQI Summary | 15 |
| Daily Pollutant Summary | 15 |
| Temperature Extremes | 10 |
| Writing outputs | 10 |
| Validation | 10 |
| Notebook organization | 5 |
| Total | 100 |

---

# Reflection (Not Graded)

- Why is distributed computing useful?
- When should Spark be used instead of SQL?
- Why separate Silver and Gold layers?

---

# What Success Looks Like

You should be able to explain:

“I loaded Silver data, validated it, created four daily summaries, and wrote them to the Gold layer for analytics.”
