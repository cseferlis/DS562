# Homework 2 – Historical Weather Ingestion and Exploratory Data Analysis with Spark

## Overview

In Homework 2, you will extend the Azure data platform you built in Homework 1. You will:

1. **Ingest historical weather data using a looping pattern in Azure Data Factory (ADF)**, writing **multiple JSON files** to your data lake.
2. **Explore and join** your **historical weather** and **air pollution** datasets using **Apache Spark** in **Azure Synapse**.
3. Produce a short EDA summary describing what you found, what is missing or suspicious, and what you would do next.

In this assignment, analytics are not mathematically complex, but the **engineering thinking** is advanced: batching, schema inspection, semi-structured data shaping, time alignment, and validation.

> **How to succeed**
> - Don’t aim for speed. Aim for **repeatability** and **evidence**.
> - Treat the pipeline and notebook like products: build small, validate, then scale up.
> - When something breaks, isolate one layer (API → ADF → Storage → Spark) and verify it before moving on.

---

## Learning Objectives

After completing this assignment, you should be able to:

- Design looping ingestion pipelines in Azure Data Factory
- Reason about time windows and offsets (and why APIs force batching)
- Debug multi-iteration pipelines using evidence (activity output, ADLS files, schema inspection)
- Use Spark to load semi-structured JSON from ADLS Gen2
- Flatten nested arrays/structs into analysis-ready tables
- Align time-series data on a consistent grain (hourly) and join datasets
- Perform basic EDA (missingness, summaries, correlations, visualizations) and communicate insights

---

## Part 1: Historical Weather Ingestion Using Azure Data Factory

### Objective

Use ADF to call the OpenWeather **Historical Weather (City)** endpoint repeatedly (batched by time window) and store results as multiple JSON files in:

```
/bronze/historical_weather/
```

### Key Design Constraints (Read Carefully)

Your solution must meet **all** constraints below:

- **Must use a ForEach loop** in ADF.
- Each loop iteration must retrieve **~1 window of data** (for example, 7 days).
- The loop must run **many times** (enough iterations to cover ~1 year).
- Each iteration must write **a separate JSON file** to ADLS.
- Your source must be an **HTTP dataset** (or HTTP linked service + dataset).
- Your pipeline must be **parameterized** (no hard-coded latitude/longitude/key in the dataset body).

### Pipeline Structure (High-Level Guidance)

Your pipeline should look like this (conceptually):

1. **Pipeline parameters** (values that are constant for the run)
   - `p_lat`, `p_lon`, `p_apiKey`
   - `p_windowDays` (example: 7)
   - Optional: `p_numWindows` (example: 52)

2. **ForEach activity**
   - Items: a numeric range representing window offsets (e.g., 0..51)
   - Inside: **Copy activity**

3. **Copy activity**
   - Source: HTTP (OpenWeather)
   - Sink: ADLS Gen2 (write JSON to `/bronze/historical_weather/`)
   - Uses **dynamic content** to pass the correct time window for that iteration

> **Parameter placement guidance (the thing that trips most people)**
> - **Dataset parameters** are placeholders (lat/lon/start/end/apiKey). They do not “compute.”
> - **Pipeline parameters** hold run-level constants (lat/lon/apiKey/windowDays).
> - **Activity dynamic content** (inside Copy) is where you bind placeholders to values and compute start/end per iteration.

### Recommended Naming Conventions

Use names that make debugging easier:

- Pipeline: `PL_HistoricalWeather_Loop`
- ForEach: `FE_TimeWindows`
- Copy activity: `CPY_HistoricalWeather_ToADLS`
- HTTP dataset: `DS_HTTP_OpenWeather_History`
- ADLS dataset: `DS_ADLS_HistoricalWeather_JSON`

### HTTP Dataset Setup (What to Configure)

We have change the base URL of our api to fetch the historical data rather than current aqi. As you create your HTTP connector, keep the following in mind.

- Base URL in the linked service: `https://history.openweathermap.org`
- Relative URL in dataset: `/data/2.5/history/city`
- Query parameters in dataset **or** in the Copy activity “Additional headers / query” (depending on your UI)

**OpenWeather documentation (reference):**
- OpenWeather APIs: https://openweathermap.org/api

### Looping + Time Window Logic (What You Need to Implement)

You need a consistent definition of “window i” such that each iteration computes:

- `start_i` (UNIX seconds)
- `end_i` (UNIX seconds)

Rules:
- `end_i` should be later than `start_i`
- Windows should not overlap (unless you intentionally choose overlap and explain it)
- Windows should cover ~1 year total

**Suggested mental model:**
- Treat the ForEach item as an index `i`
- Work backward from “now” (UTC) in chunks of `windowDays`
- Compute:
  - end = now - i*windowDays
  - start = now - (i+1)*windowDays

> **Note:** ADF expression language is strict. Small syntax errors often produce confusing template errors. Build and validate with 2–3 iterations first before scaling.

**ADF help links**
- ForEach activity: https://learn.microsoft.com/azure/data-factory/control-flow-for-each-activity
- Expression language functions: https://learn.microsoft.com/azure/data-factory/control-flow-expression-language-functions

### Hints (Not a Step-by-Step)

Use these as checkpoints rather than a recipe:

- Start with **2 iterations** (small range) and confirm **2 distinct files** appear in ADLS.
- Make sure each iteration writes to a unique filename (include the window index or start timestamp).
- Confirm the API response JSON contains an hourly `list` array (or equivalent) before you scale to ~1 year.
- If your pipeline “succeeds” but produces 1 file, your loop likely isn’t varying the request or sink path.
- Your files must have unique names, otherwise they will be over-written. So, part of the looping exercise needs to yield 52 unique names for 52 weeks of data. Think of expressions you could use to create this scenario.

### Debugging Guidance (Strongly Recommended)

When something breaks, debug in this order:

1. **API sanity check** (outside ADF)
   - Use a browser or Postman to call the endpoint with a small time window.
   - Confirm you get JSON back and the `list` contains multiple records.

2. **ADF Copy activity output**
   - Inspect the activity run output (status, duration, bytes written).
   - Confirm the request URL being executed is what you expect.

3. **ADLS file inspection**
   - Confirm multiple files exist.
   - Open a file and confirm it contains the expected JSON structure.

4. **Scale gradually**
   - 2 windows → 5 windows → 20 windows → full year

**Postman reference (optional but helpful)**
- Postman “Send your first request”: https://learning.postman.com/docs/getting-started/first-steps/sending-the-first-request/
- Postman variables: https://learning.postman.com/docs/sending-requests/variables/

### Validation Checklist (What “Done” Looks Like)

You are “done” with Part 1 when you can show:

- ADF pipeline run succeeded
- ForEach executed multiple iterations
- `/bronze/historical_weather/` contains multiple JSON files
- File contents are valid JSON and include hourly records (not just metadata)

### Common Mistakes to Avoid

- Hard-coding lat/lon/apiKey directly into the dataset instead of parameterizing
- Using the wrong host name (DNS resolution failures)
- Forgetting to vary the sink filename (everything overwrites one file)
- Creating overlapping windows unintentionally (duplicates)
- Mixing local time with UTC without noticing

---

## Part 2: Spark Environment Setup within Azure Synapse

### Objective

Create a Synapse Spark environment and a notebook to read your ADLS data and perform analysis.

### Azure Synapse and Spark Setup

#### Step 1: Create an Azure Synapse Workspace

- Create a Synapse workspace in the same region as your storage where possible.
- Link it to your ADLS Gen2 account.
- Ensure your workspace has permission to read the data lake.

Synapse overview:
- https://learn.microsoft.com/azure/synapse-analytics/overview-what-is

#### Step 2: Create a Spark Pool

Use a small pool with:
- Auto-scale disabled
- Auto-pause after 10 minutes enabled

Spark pool configurations:
- https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-pool-configurations

#### Step 3: Create a Spark Notebook

- Develop → Notebooks → New notebook
- Language: Python

---

## Part 3: Exploratory Data Analysis Using Spark

### Objective

Load air pollution + historical weather data from ADLS, flatten the nested JSON into analysis-ready tables, align on hourly timestamps, join, and perform EDA.

---

### Step 1: Load Data from the Data Lake

#### Loading Data from Azure Data Lake into Spark

Use `abfss://` paths to read all JSON files in each folder.

Reference:
- Synapse Spark read/write data: https://learn.microsoft.com/azure/synapse-analytics/spark/synapse-spark-read-write-data
- ABFS URI format: https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction-abfs-uri
- Spark JSON datasets: https://spark.apache.org/docs/latest/sql-programming-guide.html#json-datasets

**Checkpoint:** You should be able to run `.printSchema()` and `.show(1, truncate=False)` on both raw DataFrames.

---

### Step 2: Inspect Schema and Structure

#### Inspecting Schemas and Nested JSON

You must explicitly identify:
- where the hourly observations live (usually a `list` array)
- which fields are nested structs (e.g., `main`, `wind`, `clouds`)
- the timestamp field and its unit (UNIX seconds)

Spark schema references:
- Schema inference & data types: https://spark.apache.org/docs/latest/sql-programming-guide.html#schema-inference-and-data-types

---

### Step 3: Prepare Data for Analysis

#### Flattening Data and Handling Arrays

Both datasets are semi-structured. Your analysis-ready goal is:

- **one row per hour**
- top-level numeric columns for pollutants and weather variables
- timestamp columns in Spark timestamp type

Typical operations:
- `explode()` for arrays
- `select()` nested fields into columns

References:
- `explode`: https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.explode.html
- Spark SQL functions: https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html

---

### Step 4: Time Alignment and Dataset Integration

#### Working with Time and Timestamps

You must:
- convert UNIX seconds to timestamps
- create an hourly key using `date_trunc('hour', ts)`
- explain why joining on hour is appropriate

Time functions references:
- Date and time functions: https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html#date-and-time-functions
- SQL date functions: https://spark.apache.org/docs/latest/sql-ref-functions-builtin.html#date-functions

#### Joining DataFrames

Join weather and air pollution on the hourly timestamp key.

Join references:
- Spark joins (programming guide): https://spark.apache.org/docs/latest/sql-programming-guide.html#joins
- SQL join syntax: https://spark.apache.org/docs/latest/sql-ref-syntax-qry-select-join.html

---

### Step 5: Required EDA Tasks (Checklist)

Your notebook must include **all** of the following, in order:

#### A. Validation and Data Quality (required)

- Row counts for:
  - air dataset
  - weather dataset
  - joined dataset
- Duplicate check at hourly grain (`ts_hour`)
- Missing values per column

Missing data reference:
- Handling missing data: https://spark.apache.org/docs/latest/sql-programming-guide.html#handling-missing-data

#### B. Summary Statistics (required)

- Summary stats for:
  - PM2.5
  - at least two weather variables (temp, humidity, wind_speed, clouds, etc.)

DataFrame operations reference:
- https://spark.apache.org/docs/latest/sql-programming-guide.html#dataframe-operations

#### C. Correlation (required)

- Correlation between PM2.5 and:
  - temperature
  - humidity **or** wind speed

Correlation reference:
- `corr`: https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrameStatFunctions.corr.html
- Statistical functions overview: https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql.html#statistical-functions

#### D. Visualizations (required)

You must include **at least 4** visualizations:

1. PM2.5 over time (time series)
2. Scatter: PM2.5 vs temperature (or another weather variable)
3. Distribution of PM2.5 (histogram)
4. One additional plot that you justify (examples: PM2.5 by weather type, rolling average, weekday vs weekend)

> **Expected approach:** sample to pandas for plotting.

Reference:
- `toPandas`: https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.sql.DataFrame.toPandas.html

---

### Step 6: Interpretation and Insight (required)

Provide a short written section (Markdown in the notebook) answering:

- What patterns did you observe in PM2.5 over time?
- Which weather variable showed the strongest relationship with PM2.5 (if any)?
- What data quality issues exist (missingness, duplicates, gaps, suspicious ranges)?
- What would you do next if you had more time or more data?

---

### Important Guidance

- **Use many small cells.** One cell per idea (load, inspect, flatten, join, validate, analyze, plot). This improves debuggability.
- **Avoid expensive actions repeatedly.** Operations like `count()` and `toPandas()` trigger computation.
- **Document your reasoning.** We will grade clarity of thinking, not just code.

---

## Deliverables (Homework 2)

Submit the following:

1. **ADF configurations to be reviewed**
   - Pipeline canvas including ForEach + Copy activity
   - One successful run showing multiple iterations
   - ADLS folder containing multiple output files in `/bronze/historical_weather/`

2. **Spark notebook submission**
   - Executed notebook (ipynb export or Synapse notebook export)
   - Must include Markdown annotations explaining each step

3. **EDA summary PDF (1–2 pages)**
   - 3–6 bullet insights
   - 1–2 charts (can reuse from notebook)
   - Data quality observations
   - Recommended next steps

---

## Grading Reminder

This is a Master’s-level course. We will grade based on:

- Evidence that your pipeline design works (multi-iteration + multiple files)
- Quality of your validation and reasoning
- Correctness of Spark transformations (flatten → align → join)
- Completeness of EDA checklist
- Clarity of communication in notebook and summary

You will not be penalized for small syntax mistakes if your approach is sound and you can explain what you intended.


