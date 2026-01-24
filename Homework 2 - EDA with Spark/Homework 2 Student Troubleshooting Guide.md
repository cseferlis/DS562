
# Homework 2 – Student Troubleshooting & Help Guide

This guide highlights common issues students encounter while completing Homework 2.

---

## Azure Data Factory (ADF)

### Pipeline Only Produces One File

If your pipeline runs successfully but you only see one file in your output folder, check the following:

1. **Copy Activity placement**
   - Make sure the Copy activity is *inside* the ForEach loop, not after it.

2. **Loop configuration**
   - Confirm the ForEach `Items` expression produces multiple values (for example, a range of numbers).

3. **File naming**
   - Ensure the sink file name is dynamic so each loop iteration writes a different file.
   - If every iteration writes to the same file name, earlier files will be overwritten.

4. **Verify results**
   - After the pipeline completes, refresh the ADLS folder and confirm multiple files exist.

📘 Helpful reference:  
https://learn.microsoft.com/azure/data-factory/control-flow-for-each-activity

---

### API Returns Empty or Partial Data
Cause:
- Time window too large
- Incorrect timestamp math

Fix:
- Verify UNIX timestamps are in seconds
- Reduce the window size

Help:
https://openweathermap.org/api/history-weather-api

---

### Confusion About Parameters (Dataset vs Pipeline vs Activity)

If you’re stuck on **where parameters belong** in Azure Data Factory, use this simple rule:

#### 1) Dataset parameters = placeholders
Use dataset parameters to define **what values will be supplied later**, not the values themselves.

Example: the HTTP dataset might define placeholders like:
- `lat`
- `lon`
- `start`
- `end`
- `apikey`

You are not “setting” the real values here. You are just creating slots for them.

---

#### 2) Pipeline parameters = constant values for the whole run
Use pipeline parameters for values that stay the same throughout the pipeline execution, like:
- latitude
- longitude
- API key
- window size (days)

These are usually entered once when you run the pipeline (or set as defaults).

---

#### 3) Activity expressions = dynamic values that change each iteration
Use activity-level dynamic content (inside the Copy activity) for values that change per loop iteration, such as:
- calculated `start` timestamp
- calculated `end` timestamp
- dynamic file name

---

### Quick sanity check
If a value:
- **never changes** during the run → pipeline parameter  
- **changes every loop** → activity expression  
- is just a **slot to be filled** → dataset parameter  

📘 Helpful reference (ADF parameters):  
https://learn.microsoft.com/azure/data-factory/parameters

---

### Spark & Notebook Issues (Common Problems + Quick Fixes)

This section covers the most common problems students run into once they start working in Synapse/Spark.

---

#### 1) “My ABFSS path doesn’t work” (or “invalid authority”)
**What it usually means**
- The `abfss://` path is formatted incorrectly (container vs folder confusion), or
- The storage account name/container name is wrong.

**What to do**
- Confirm the pattern matches:
abfss://<container>@<storage_account>.dfs.core.windows.net/<folder_path>/

Example (your names will differ):
abfss://bronze@mydatalake.dfs.core.windows.net/bronze/air_pollution/

📘 Help: Read/write ADLS Gen2 in Synapse Spark  
https://learn.microsoft.com/azure/synapse-analytics/spark/synapse-spark-read-write-data

---

#### 2) “Spark loads the weather JSON but I only see ONE row”
**What it usually means**
- The hourly records are stored inside an array (commonly `list`), and you haven’t exploded it yet.

**What to do**
- Inspect the schema (`printSchema()`) and look for an array field like `list`.
- Use `explode()` to convert array elements into rows.

📘 Help: explode()  
https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.explode.html

---

#### 3) “My join returns 0 rows”
**What it usually means**
- Your timestamps aren’t aligned (seconds-level mismatch).
- One dataset is hourly and the other is slightly offset.

**What to do**
- Create an hourly join key by truncating timestamps to the hour in both datasets.
- Join on the truncated hour key (not raw timestamps).

📘 Help: Spark date/time functions  
https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html

---

#### 4) “toPandas() crashes or runs out of memory”
**What it usually means**
- You are converting too much Spark data into pandas.

**What to do**
- Always sample *before* calling `toPandas()`.
- Only bring the columns you need for plotting.

Tip:
- If you’re unsure, start with a very small sample and increase later.

---

#### 5) “My plots look weird / time series is scrambled”
**What it usually means**
- Your data isn’t sorted by timestamp after converting to pandas.

**What to do**
- After converting to pandas, sort by timestamp before plotting.

---

### Notebook Structure Tip (How many cells should I use?)
Use **separate cells for separate steps** so you can debug easily.

Recommended pattern:
1. Imports + paths
2. Load raw data
3. Inspect schema
4. Flatten air
5. Flatten weather
6. Join + validation
7. EDA stats
8. Plotting (one plot per cell)

This makes it much easier to isolate and fix issues.
