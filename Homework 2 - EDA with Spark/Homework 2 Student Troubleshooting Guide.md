
# Homework 2 – Student Troubleshooting & Help Guide

This guide highlights common issues students encounter while completing Homework 2.

---

## Azure Data Factory (ADF)

### Pipeline Only Produces One File
Cause:
- Copy Activity placed outside the ForEach loop

Fix:
- Ensure the Copy Activity is inside the ForEach

Help:
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
