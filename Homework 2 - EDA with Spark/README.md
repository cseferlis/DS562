# Homework 2 – Historical Weather Ingestion and Exploratory Data Analysis with Spark

## Overview

In Homework 2, you will extend the Azure data platform you built in Homework 1.
You will ingest **historical weather data using a looping pattern**, then perform **exploratory data analysis (EDA)** using Apache Spark.

This assignment introduces two core data engineering skills:
- Handling **API limitations** using loops and parameterized pipelines
- Exploring semi-structured data at scale using **Spark**

---

## Learning Objectives

After completing this assignment, you should be able to:
- Design looping ingestion pipelines in Azure Data Factory
- Reason about time windows and offsets
- Debug multi-iteration pipelines
- Perform exploratory analysis on time-series data
- Join datasets and reason about relationships

---

## Part 1: Historical Weather Ingestion Using Azure Data Factory

### Objective

In this section, you will ingest approximately **one year of historical weather data** from the OpenWeather API.

Unlike Homework 1, this API **cannot return large date ranges in a single request**. You must design a pipeline that **breaks the year into smaller windows** and retrieves data iteratively.

This mirrors real-world ingestion challenges where APIs impose limits on request size and frequency.

---

### Key Design Constraints (Read Carefully)

Your solution **must** satisfy all of the following:

- Use a **new pipeline** in the same Azure Data Factory instance
- Use a **looping construct** (ForEach)
- Make **multiple API calls** (not one)
- Each API call retrieves a **small time window** (for example, about one week)
- Each iteration writes a **separate JSON file**
- Files must **not overwrite each other**
- All output must land in:
```
/bronze/historical_weather/
```
There is more than one valid implementation, but solutions that violate these constraints will not receive full credit.

---

### Pipeline Structure (High-Level Guidance)

Your pipeline should follow this general pattern:

1. **Generate a list of time offsets**
   - For example, one offset per week
   - The loop should execute approximately **52 times**

2. **For each iteration**
   - Compute a **start time** and **end time** for that window
   - Call the historical weather API using those times
   - Write the response to a uniquely named file

Think in terms of:

> “For each time window in the past year, fetch the weather data for that window.”

---

### Hints (Not a Step-by-Step)

Use these hints to guide your design, not as a recipe:

- The **ForEach** activity expects an array. Many students use a list of integers to represent time offsets.
- Each iteration has access to the current loop value using `item()`.
- Azure Data Factory provides built-in date functions such as:
  - `utcNow()`
  - `addDays()`
- The OpenWeather API expects **timestamps**, not date strings.
- File overwrites usually happen because the **file name is static**.

If your pipeline runs successfully but produces only **one file**, your loop is likely working but your sink configuration is not.

---

### Debugging Guidance (Strongly Recommended)

Before running the full pipeline:

- Temporarily reduce the number of iterations (for example, 2–3 windows)
- Confirm that:
  - The loop executes multiple times
  - Multiple output files are created
  - File names differ across iterations

Once validated, scale up to the full year.

---

### Validation Checklist (What “Done” Looks Like)

Your historical ingestion is complete when:

- The pipeline run shows **multiple loop iterations**
- The `/bronze/historical_weather/` folder contains **many JSON files**
- File sizes look reasonable (not empty, not identical)
- File timestamps roughly correspond to different historical periods

---

### Common Mistakes to Avoid

- Editing the Homework 1 pipeline instead of creating a new one
- Using a loop that executes only once
- Requesting too much data in a single API call
- Overwriting the same output file repeatedly
- Attempting to “fix” API errors by hardcoding dates

---

## Part 2: Exploratory Data Analysis Using Spark

### Azure Synapse and Spark Setup

### Objective

In this section, you will configure **Apache Spark** using **Azure Synapse Analytics** and prepare the environment for exploratory data analysis.

You are not expected to be an expert in Spark yet. The goal is to:
- Understand how Spark fits into a modern data platform
- Learn how to load and explore data stored in a data lake
- Use distributed compute to explore semi-structured datasets

---

### Step 1: Create an Azure Synapse Workspace

If you have not already done so:

1. In the Azure Portal, search for **Azure Synapse Analytics**
2. Click **Create**
3. Select:
   - Your existing **resource group**
   - Link to your existing **Data Lake Storage Gen2 account**
4. Use default settings for SQL admin credentials
5. Review and create

You will use this workspace for **all Spark work** in this homework.

---

### Step 2: Create a Spark Pool

1. Open your Synapse workspace
2. Navigate to **Manage → Apache Spark pools**
3. Create a new Spark pool with:
   - **Small node size**
   - **Auto-scale disabled**
   - **Auto-pause enabled**

This configuration minimizes cost while still supporting exploratory analysis.

---

### Step 3: Create a Spark Notebook

1. In Synapse Studio, go to **Develop → Notebooks**
2. Create a new notebook
3. Language: **Python**
4. Attach the notebook to your Spark pool

This notebook will contain **all analysis for Homework 2**.

---

## Part 3: Exploratory Data Analysis Using Spark

### Objective

Use Apache Spark to explore and understand the datasets you have ingested:

- Air pollution data (Homework 1)
- Historical weather data (Homework 2)

The purpose of EDA is **understanding**, not cleaning, modeling, or optimization.

---

### Step 1: Load Data from the Data Lake

Load **all JSON files** from the following folders:
```
/bronze/air_pollution/
/bronze/historical_weather/
```
Use **folder-based reads**, not individual file paths, so Spark loads all files automatically.

At a minimum, demonstrate that:
- The data loads successfully
- The record counts are reasonable
- You are reading from the correct storage location

### Loading Data from Azure Data Lake into Spark

Helpful when:
- You’re unsure how Spark reads from ADLS Gen2
- You want to confirm correct path syntax

- Read data from Azure Data Lake Gen2 using Spark  
  https://learn.microsoft.com/azure/synapse-analytics/spark/synapse-spark-read-write-data

- Azure storage URI formats (`abfss://`)  
  https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction-abfs-uri
---

### Step 2: Inspect Schema and Structure

For each dataset:

- Print the schema
- Identify nested fields and arrays
- Identify key attributes such as:
  - timestamps
  - weather measurements
  - pollution metrics

You are not expected to flatten the entire schema. Focus only on fields relevant to analysis.

### Inspecting Schemas and Nested JSON

Helpful when:
- Your schema looks deeply nested
- You’re unsure how to access nested fields

- Working with nested JSON in Spark  
  https://spark.apache.org/docs/latest/sql-programming-guide.html#json-datasets

- Understanding Spark DataFrame schemas  
  https://spark.apache.org/docs/latest/sql-programming-guide.html#schema-inference-and-data-types

---

### Step 3: Prepare Data for Analysis

To make the data usable for EDA:

- Extract commonly used fields into a flatter structure
- Convert timestamps into a consistent format
- If arrays are present, flatten them as needed

Document:
- Which fields you selected
- Why those fields are useful

### Flattening Data and Handling Arrays

Helpful when:
- Fields you want are inside arrays or structs
- You need one row per timestamp or measurement

- Using `select`, dot notation, and `explode()`  
  https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.explode.html

- Spark SQL functions reference  
  https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html

---

### Step 4: Time Alignment and Dataset Integration

To analyze relationships between weather and pollution:

- Align timestamps between datasets
- Join the datasets on time

If timestamps do not match exactly:
- Bucket or round timestamps (for example, hourly)
- Explain your approach in plain language

The correctness of your reasoning matters more than the specific join technique.

### Working with Time and Timestamps

Helpful when:
- Your timestamps don’t align between datasets
- You need to bucket or round time values

- Spark date and timestamp functions  
  https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html#date-and-time-functions

- Time-based aggregation and grouping  
  https://spark.apache.org/docs/latest/sql-ref-functions-builtin.html#date-functions

#### Joining DataFrames

Helpful when:
- Your join returns fewer rows than expected
- You’re unsure which join strategy to use

- Spark DataFrame joins  
  https://spark.apache.org/docs/latest/sql-programming-guide.html#joins

- Understanding join types (inner, left, etc.)  
  https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html
---

### Step 5: Required EDA Tasks

Your Spark notebook must include **all** of the following:

1. **Record counts and schema inspection**
2. **Missing value analysis**
3. **At least four visualizations**, such as:
   - Distributions (histograms)
   - Scatter plots
   - Time series plots
4. **Correlation analysis** across weather and pollution variables
5. Identification of **at least two insights**, supported by visual evidence

You may sample data for plotting if needed.

### Missing Values and Data Quality Checks

Helpful when:
- You need to identify nulls or incomplete records
- You want to reason about data quality

- Handling missing data in Spark  
  https://spark.apache.org/docs/latest/sql-programming-guide.html#handling-missing-data

### Visualization in Spark Notebooks

Helpful when:
- You’re unsure how to visualize Spark data
- You need to sample before plotting

- Converting Spark DataFrames to Pandas  
  https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.toPandas.html

- Best practices for plotting large datasets  
  https://spark.apache.org/docs/latest/sql-programming-guide.html#dataframe-operations

---

### Step 6: Interpretation and Insight

For each major analysis or visualization:

- Describe what you observe
- Explain why the pattern might exist
- Identify any data quality issues or limitations

Focus on **clear reasoning**, not volume or complexity.

### Correlation and Basic Statistical Analysis

Helpful when:
- You need to compute correlations
- You’re unsure what Spark supports natively

- Correlation in Spark DataFrames  
  https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrameStatFunctions.corr.html

- Spark statistical functions overview  
  https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql.html#statistical-functions

### Important Guidance

- These links are **references**, not recipes.
- You are graded on **reasoning and interpretation**, not perfect syntax.
- If you copy code blindly without understanding it, it will show in your analysis.

When in doubt, ask yourself:
> “What question am I trying to answer about the data?”
---

## Deliverables (Homework 2)

Submit:

1. **Screenshots**
   - Historical weather pipeline run
   - Bronze folder showing multiple historical weather files
2. **Spark notebook**
   - Exported as **HTML**
3. **EDA report (PDF, one page)**
   - Key insights
   - Visuals
   - Data quality observations
   - Suggested next steps

---

## Grading Reminder

You are graded on:
- Correct platform setup
- Evidence of thoughtful analysis
- Clear reasoning and communication

You are **not** graded on:
- Perfect Spark code
- Complex transformations
- Advanced statistics or machine learning
