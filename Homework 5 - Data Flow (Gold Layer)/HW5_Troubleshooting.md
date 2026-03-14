
# Homework 5 – Student Troubleshooting Guide
## Silver Layer Transformations with Azure Data Factory

This guide addresses common problems students encounter while building **ADF Dataflows** and writing results to the **Silver layer**.

---

## 1. Dataflow Debug Mode Running Too Long

### Problem
Debug mode runs indefinitely and consumes resources.

### Fix
Turn off debug after testing.

**Steps**
1. Open your Dataflow
2. Toggle **Debug** off
3. Save the Dataflow

Documentation  
https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-debug-mode

---

## 2. Flatten Transformation Produces No Columns

### Problem
Nested JSON fields do not appear after flattening.

### Fix
Ensure the correct **array column** is selected in the Flatten transformation.

Example fields often requiring flattening:

```
list
list.main
list.weather
```

Documentation  
https://learn.microsoft.com/en-us/azure/data-factory/data-flow-flatten

---

## 3. Dataflow Writes Empty Output

### Possible Causes

• Source path incorrect  
• Schema drift issues  
• Incorrect filter conditions

### How to Diagnose

Use **Data Preview** inside the Dataflow to verify records exist before writing output.

---

## 4. Timestamp Conversion Errors

### Problem
Unix timestamps appear as large integers.

### Fix
Convert using timestamp functions.

Example transformation expression:

```
toTimestamp(dt)
```

---

## 5. Pipeline Runs But No Files Appear

Check the following:

• Sink dataset path is correct  
• ADLS container exists  
• Output format set to **Parquet**

Documentation  
https://learn.microsoft.com/en-us/azure/data-factory/format-parquet

---

## 6. Cost Management Tips

ADF debugging can consume resources.

Before logging off:

• Stop debug sessions  
• Ensure pipelines are not scheduled repeatedly
