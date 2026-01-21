# Homework 2 – Student Troubleshooting Guide
*(Historical Weather Ingestion + Spark EDA)*

This guide is aligned to the **current Homework 2 README** and covers common issues encountered during:
- Historical weather ingestion using a loop in Azure Data Factory
- Exploratory Data Analysis using Azure Synapse and Spark

Use this guide to unblock yourself before asking for help.

---

## Azure Data Factory – Historical Weather Pipeline

### Issue: Only one output file is created
**Likely cause**
- Sink file name is static
- Overwrite is enabled

**What to check**
- File name includes a loop-specific value
- Overwrite is disabled
- ForEach shows multiple iterations in the run view

---

### Issue: ForEach activity runs only once
**Likely cause**
- Items field does not evaluate to an array

**What to check**
- Items expression produces a list (for example, week offsets)
- ForEach is not accidentally wrapped inside another activity

Reference:
https://learn.microsoft.com/azure/data-factory/control-flow-for-each-activity

---

### Issue: Pipeline fails mid-run
**Likely causes**
- API rate limiting
- Request window too large

**What to try**
- Reduce window size
- Enable retry policies
- Run sequentially instead of in parallel

---

### Issue: Files exist but look identical
**Likely cause**
- Time parameters not changing per iteration

**What to check**
- Start and end times depend on the loop value
- API URL reflects dynamic parameters

---

## Spark / Synapse Setup Issues

### Issue: Spark pool will not start
**Likely causes**
- Node size too large
- Subscription limits reached

**What to try**
- Use smallest available node size
- Enable auto-scale and auto-pause

---

### Issue: Data cannot be loaded from ADLS
**Likely cause**
- Incorrect abfss path
- Wrong container or storage account name

**What to check**
- Container names (`bronze`) are correct
- Using `.dfs.core.windows.net` endpoint

Reference:
https://learn.microsoft.com/azure/synapse-analytics/spark/synapse-spark-read-write-data

---

## Spark EDA Issues

### Issue: Schema is deeply nested and confusing
**What to do**
- Use `printSchema()`
- Identify only fields needed for analysis
- Flatten selectively

---

### Issue: Joins return very few rows
**Likely cause**
- Timestamp mismatch

**What to try**
- Bucket or round timestamps (for example, hourly)
- Explain join strategy in notebook

---

### Issue: Notebook crashes during plotting
**Likely cause**
- Dataset too large for visualization

**What to do**
- Sample before converting to Pandas
- Plot subsets, not full datasets

---

## When Asking for Help, Include
- Screenshot of pipeline run (showing iterations)
- Screenshot of Bronze folder contents
- Clear description of what you expected vs what happened
