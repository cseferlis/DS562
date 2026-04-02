
# Homework 6 – Student Troubleshooting Guide
## Spark Processing in Azure Synapse

---

## 1. Spark Cluster Will Not Start

### Possible Causes

• Resource limits reached  
• Region capacity issue

### Fix

Wait a few minutes and retry cluster startup.

You can also try restarting the notebook session.

Documentation  
https://learn.microsoft.com/en-us/azure/synapse-analytics/spark/apache-spark-overview

---

## 2. Spark Cannot Access ADLS

### Error Example

```
Path does not exist
Permission denied
```

### Fix

Verify your path syntax.

Example:

```
abfss://silver@yourstorageaccount.dfs.core.windows.net/weather/
```

Check container names carefully.

---

## 3. explode() Produces Unexpected Results

### Problem

Array fields do not expand correctly.

### Fix

Confirm the column contains arrays.

Example:

```
df.select(explode("list"))
```

Documentation  
https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/functions.html#explode

---

## 4. Notebook Kernel Crashes

Possible causes:

• Large dataset loaded  
• Too many cached DataFrames

### Fix

Restart kernel and re-run only necessary cells.

---

## 5. Spark Writes Only One File

Spark often writes **multiple partition files**.

To force a single file:

```
df.coalesce(1).write.parquet(path)
```

Use sparingly because it removes distributed benefits.
