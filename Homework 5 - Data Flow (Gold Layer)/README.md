# Homework 5 – Building the Silver Layer with Azure Data Factory

## Objective
In Homework 1 and Homework 2 you ingested raw weather and air pollution data into Azure Data Lake Storage. These datasets currently exist in a **Bronze layer**, which represents raw, unrefined data directly from source systems.

In this assignment you will transform that raw data into a **cleaned and standardized Silver layer** using **Azure Data Factory Dataflows**.

The Silver layer prepares data for analytics by performing transformations such as:
- flattening nested JSON
- correcting timestamps
- standardizing schema
- removing invalid records
- creating derived fields

The result will be **structured Parquet datasets** that are easier to query and process in downstream systems such as Spark and Synapse.

---

## Architecture Context
Your architecture should now resemble the following:

OpenWeather API  
↓  
Azure Data Factory Pipeline  
↓  
Bronze Layer (Raw JSON)  
↓  
ADF Dataflows  
↓  
Silver Layer (Clean Parquet)

This follows the **Medallion Architecture pattern** used in modern data platforms.

Reference:  
https://www.databricks.com/glossary/medallion-architecture

---

## Tools Used
- Azure Data Factory
- Azure Data Lake Storage
- ADF Dataflows
- Parquet file format

ADF Dataflow Documentation:  
https://learn.microsoft.com/en-us/azure/data-factory/concepts-data-flow-overview

---

## Cost Control Guidelines
Cloud resources consume money even when idle. Follow these guidelines carefully.

- Do not leave debug sessions running
- Run pipelines only when needed
- Avoid repeated executions of large transformations
- Monitor activity runs before triggering again

You can review resource usage in:

Azure Portal → Cost Management → Cost Analysis

---

## Assignment Tasks

### 1. Weather Data Transformation
Create an **ADF Dataflow** that processes weather data from the Bronze layer.

Required transformations:
- Flatten nested JSON fields
- Create a corrected timestamp column
- Convert timestamps into readable datetime values
- Create a unique identifier for each observation
- Convert temperature values into Celsius and Fahrenheit
- Remove rows containing invalid or missing critical values

Output location:
/silver/weather/

Output format:
Parquet

Documentation:  
https://learn.microsoft.com/en-us/azure/data-factory/data-flow-flatten

---

### 2. Air Pollution Data Transformation
Create a second **ADF Dataflow** for air pollution data.

Required transformations:
- Flatten nested API fields
- Standardize timestamp values
- Generate unique row identifiers
- Round pollutant measurements
- Remove records with invalid timestamps

Output location:
/silver/air_pollution/

---

### 3. Pipeline Orchestration
Create pipelines to execute both dataflows.

Each pipeline should:
- trigger the dataflow
- write results to the Silver layer
- complete successfully without manual intervention

Monitor execution using the **ADF Monitor tab**.

Documentation:  
https://learn.microsoft.com/en-us/azure/data-factory/monitor-visually

---

## Production Considerations (Reflection Only)
These questions are intended to help you think about real-world production environments. They are **not graded deliverables**.

- What types of data quality issues did you encounter while transforming the data?
- Why is Parquet commonly used for analytics datasets instead of JSON?
- If this pipeline processed millions of records per hour, what additional design considerations might be necessary?

---

## Deliverables
Submit the following:

1. Screenshot of the Weather Dataflow design
2. Screenshot of the Air Pollution Dataflow design
3. Screenshot of successful pipeline runs
4. Screenshot showing the Silver layer datasets in ADLS

## Grading

Category                      Points    What TAs Will Look For
Dataflow Architecture         25        Two dataflows created (weather + air pollution) with logical transformations
Correct Transformations       30        Flatten, derived columns, filtering, and column selection implemented correctly
Silver Data Output            25        Clean Parquet output written to /silver/weather/ and /silver/air_pollution/
Data Quality                  10        Invalid records filtered, timestamps converted properly
Documentation / Evidence      10        Screenshots or evidence of successful pipeline runs and output data

