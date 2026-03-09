# HW4 -- Student Troubleshooting Guide

Real-Time Stream Analytics with Azure

This guide helps you diagnose common problems when building the
streaming pipeline for Homework 4.

Pipeline architecture:

Weather API\
↓\
Azure Function\
↓\
Event Hub\
↓\
Stream Analytics\
↓\
Power BI

Troubleshoot in this order. Problems earlier in the pipeline will break
everything downstream.

------------------------------------------------------------------------

# 1. No Events Appearing in Event Hub

## Symptoms

-   Event Hub metrics show **0 incoming messages**
-   Stream Analytics input shows **0 events**

## Things to Check

### 1.1 Function App Running

Go to:

Function App → Overview

Status must be:

Running

If it is stopped:

Click **Start**.

------------------------------------------------------------------------

### 1.2 Verify the Function Executed

Navigate to:

Function App → Functions → your function → Monitor

You should see recent executions.

If there are no executions: - wait for the next timer trigger - or
manually run the function (if available)

------------------------------------------------------------------------

### 1.3 Check Event Hub Name

If the function sends to the wrong Event Hub name, no messages will
arrive.

Confirm the function uses:

weather-events

------------------------------------------------------------------------

### 1.4 Check Event Hub Metrics

Navigate to:

Event Hub → weather-events → Metrics

Select metric:

Incoming Messages

You should see values increasing when the function runs.

------------------------------------------------------------------------

# 2. Stream Analytics Input Shows No Data

## Symptoms

-   Stream Analytics job running
-   Input metrics remain **0**

## Things to Check

### 2.1 Event Hub Input Configuration

Verify:

Input type: Event Hub\
Event hub: weather-events\
Consumer group: \$Default

Basic Event Hub tier only supports the `$Default` consumer group.

------------------------------------------------------------------------

### 2.2 Stream Analytics Job Running

Navigate to:

Stream Analytics Job → Overview

Status must be:

Running

If not:

Click **Start**.

------------------------------------------------------------------------

### 2.3 Input Alias Naming

Avoid underscores.

Example:

Correct:

eh-input

Incorrect:

eh_input

Use brackets in queries:

\[eh-input\]

------------------------------------------------------------------------

# 3. Query Returns No Results

## Symptoms

Test query returns nothing.

## Things to Check

### 3.1 Validate Field Names

Use **Test Query** in Stream Analytics.

Start with a simple query:

SELECT \*\
FROM \[eh-input\]

Confirm the schema and field names.

------------------------------------------------------------------------

### 3.2 Check Timestamp Usage

If using event time:

TIMESTAMP BY timestamp

Ensure the field exists in the event data.

------------------------------------------------------------------------

### 3.3 Window Queries Need Time to Pass

Window queries may not produce results immediately.

Example:

TumblingWindow(minute,5)

This requires events across the window period before returning rows.

------------------------------------------------------------------------

# 4. Power BI Dataset Not Appearing

## Symptoms

Dataset does not appear in Power BI Service.

## Important Behavior

Power BI **does not create the dataset until the first rows arrive.**

This is expected.

------------------------------------------------------------------------

## Things to Check

### 4.1 Stream Analytics Output Events

Navigate to:

Stream Analytics → Monitoring → Output

Verify:

Output events \> 0

If output events are zero, the query is not producing rows.

------------------------------------------------------------------------

### 4.2 Power BI Authorization

Navigate to:

Stream Analytics → Outputs → pbi-output

Click:

Re-authorize

Then restart the Stream Analytics job.

------------------------------------------------------------------------

### 4.3 Confirm Dataset Location

Open:

https://app.powerbi.com

Navigate to:

My Workspace → Datasets

Look for:

DS562_HW4_Stream

------------------------------------------------------------------------

# 5. Power BI Visuals Not Updating

## Symptoms

Dataset exists but charts do not change.

## Things to Check

### 5.1 Timestamp Data Type

Ensure the timestamp field is interpreted as **DateTime**.

If the field is text, charts may not update correctly.

------------------------------------------------------------------------

### 5.2 Query Producing New Rows

Verify in Stream Analytics:

Monitoring → Output events increasing

If output events are not increasing, the stream is not producing new
results.

------------------------------------------------------------------------

# 6. Stream Analytics Job Fails to Start

## Symptoms

Job start fails or errors appear.

## Things to Check

### 6.1 Input or Output Authentication

Re-authorize the Power BI output.

------------------------------------------------------------------------

### 6.2 Incorrect Query Syntax

Check for alias issues.

Correct example:

SELECT \*\
INTO \[pbi-output\]\
FROM \[eh-input\]

Aliases containing hyphens must use brackets.

------------------------------------------------------------------------

# 7. Cost Control

Streaming services run continuously.

After completing the assignment:

1.  Stop the Stream Analytics job
2.  Stop the Function App
3.  Delete the Event Hub instance
4.  Keep the Event Hub namespace

Stopping resources prevents unnecessary cloud costs.

------------------------------------------------------------------------

# 8. Recommended Debugging Order

Always debug the pipeline in this order:

1.  Function execution
2.  Event Hub receiving messages
3.  Stream Analytics input events
4.  Stream Analytics query output
5.  Power BI dataset
6.  Power BI visuals

Most problems originate earlier in the pipeline.

------------------------------------------------------------------------

# Helpful Documentation

Stream Analytics documentation\
https://learn.microsoft.com/azure/stream-analytics/

Event Hub monitoring\
https://learn.microsoft.com/azure/event-hubs/

Power BI streaming datasets\
https://learn.microsoft.com/power-bi/connect-data/service-real-time-streaming
