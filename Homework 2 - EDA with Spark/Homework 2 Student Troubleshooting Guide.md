
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

### Dataset vs Pipeline Parameters
Rule:
- Dataset parameters = placeholders
- Pipeline parameters = values
- Expressions = logic

Help:
https://learn.microsoft.com/azure/data-factory/parameters

---

## Spark & Notebook Issues

### Cannot Read ADLS Path
Use:
abfss://container@storageaccount.dfs.core.windows.net/path/

Help:
https://learn.microsoft.com/azure/synapse-analytics/spark/synapse-spark-read-write-data

---

### Only One Row Loaded
Cause:
- explode() not applied

Help:
https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.explode.html

---

### Pandas Memory Errors
Always sample before calling toPandas().
