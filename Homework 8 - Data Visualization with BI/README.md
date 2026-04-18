# Homework 8 – Data Visualization and Analytics with Power BI

## Overview

In Homework 7, you built a lightweight data warehouse using Azure Synapse serverless SQL pool. You exposed Gold datasets through external tables and created persistent analytical outputs using CETAS.

At this stage, your data platform is capable of supporting analytics.

In Homework 8, you will take the final step in the pipeline:

You will connect your warehouse outputs to **Power BI** and build a simple, effective analytical report.

---

## How This Builds on HW7

In HW7, you created:

- FactWeather  
- DimDate  
- DimLocation  
- DimWeatherCondition  

These objects represent a structured analytical layer.

In HW8, you will:

Warehouse Tables (Synapse)  
↓  
Power BI Connection  
↓  
Data Model  
↓  
Visualizations  
↓  
Insights  

This is the step where your data becomes **actionable**.

---

## Architecture Context

Bronze → Silver → Gold → Warehouse → BI / Analytics  

You are now working in the final stage: **consumption and insight generation**.

---

## Tools Used

- Azure Synapse Analytics (serverless SQL pool)
- Power BI (Desktop or Service)
- Azure Data Lake Storage (indirectly)

---

## Documentation Resources

### Power BI Basics
https://learn.microsoft.com/en-us/power-bi/fundamentals/power-bi-overview

### Connecting Power BI to Azure Synapse
https://learn.microsoft.com/en-us/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect

### Data Modeling in Power BI
https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-data-modeling

### Visualization Best Practices
https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-types-for-reports-and-q-and-a

---

## Environment Options (Important)

You may use:

### Option A – Power BI Desktop (Windows only)
- Full feature set
- Recommended if available

### Option B – Power BI Service (Web)
- Required for Mac users
- Use Dataset connection

Both options are acceptable.

---

## Cost Control Guidelines

- Avoid repeatedly refreshing large datasets
- Use small preview queries first when testing
- Do not leave unnecessary Azure resources running
- Prefer Import mode unless DirectQuery is needed

---

# Part 1 – Connect Power BI to Your Data

## Objective

Connect Power BI to your Synapse data.

Use:

- Serverless SQL endpoint
- Database name

You may use:
- Import mode
- DirectQuery (optional)

## Required Tables

Load at least:

- FactWeather  
- DimDate  

Optional:

- DimLocation  
- DimWeatherCondition  

---

# Part 2 – Build a Data Model

## Objective

Create relationships between your tables.

## Requirements

At minimum:

- FactWeather → DimDate (on full_date)

Optional:

- FactWeather → DimLocation  
- FactWeather → DimWeatherCondition  

---

## Expectation

Your model should support filtering and aggregation.

---

# Part 3 – Create Visualizations

## Objective

Build meaningful visuals using your data.

## Requirements

Create at least **3 visuals**, such as:

- Temperature trends over time  
- AQI trends over time  
- Comparison of min/max temperature  
- Monthly or weekly summaries  

---

## Expectation

Your visuals should:

- use appropriate chart types  
- have clear labels  
- be easy to interpret  

---

# Part 4 – Build a Report / Dashboard

## Objective

Combine your visuals into a single report.

## Requirements

- Arrange visuals logically  
- Add titles and labels  
- Ensure readability  

---

## Optional Enhancements

- Add filters or slicers  
- Add basic formatting (colors, layout)  

---

# Part 5 – Validate Outputs

## Objective

Confirm your report is functional and meaningful.

## Required Validation

You must demonstrate:

- visuals display correctly  
- data updates based on filters (if used)  
- results are consistent with your HW7 outputs  

---

# Deliverables

Submit:

1. Screenshots of:
   - Power BI connection  
   - Data model (relationships view)  
   - At least 3 visuals  
   - Final report/dashboard  

2. Short explanation (3–5 sentences):

   - What insights can be derived from your report?  
   - What was one challenge you encountered?  

---

# Grading Breakdown

| Category | Points |
|----------|--------|
| Data connection | 15 |
| Data model | 20 |
| Visualizations | 25 |
| Report/dashboard | 20 |
| Validation | 10 |
| Explanation | 10 |
| Total | 100 |

---

# Things to Consider (Not Graded)

- What makes a visualization effective or ineffective?  
- How would business users interact with your report?  
- What additional data would improve your analysis?  

---

# What Success Looks Like

You should be able to explain:

“I connected my warehouse data to Power BI, built a simple data model, created visualizations, and used them to communicate insights.”
