# Homework 4 -- Real-Time Stream Analytics

## Overview

In Homework 3, you built a **real-time ingestion pipeline** using:

-   Azure Function
-   Azure Event Hubs

That pipeline produced a **live stream of weather and air quality
events**.

In Homework 4, you will build the **analytics layer** that processes
this stream in real time.

You will use:

-   Azure Stream Analytics
-   Power BI Service

The goal is to transform streaming events into **live metrics and
dashboards**.

This assignment focuses on:

-   Streaming data transformations
-   Event time vs ingestion time
-   Windowed aggregations
-   Real-time dashboards
-   Cost-conscious cloud engineering

------------------------------------------------------------------------

## Platform Compatibility

This assignment **does not require Power BI Desktop**.

All Power BI work is completed using:

https://app.powerbi.com

This ensures compatibility for:

-   macOS
-   Windows
-   Linux
-   Chromebook

------------------------------------------------------------------------

## Architecture Context

Your pipeline now looks like this:

Weather API\
↓\
Azure Function\
↓\
Event Hub\
↓\
Stream Analytics\
↓\
Power BI

You are adding the **analytics and visualization layer**.

------------------------------------------------------------------------

# Part 0 -- Restore the Streaming Backbone

Homework 3 required you to **delete the Event Hub instance** to reduce
costs.

Before beginning HW4 you must restore the stream.

### 0.1 Recreate the Event Hub

Navigate to your existing **Event Hubs namespace**.

Create a new Event Hub with:

Name: weather-events\
Partition count: 1\
Message retention: 1 hour\
Capture: Disabled

Do **not create additional consumer groups**.

The **Basic tier only supports `$Default`**.

------------------------------------------------------------------------

### 0.2 Restart the Function App

Start the Function App used in Homework 3.

Verify events are arriving in Event Hub:

Event Hub → Metrics → Incoming Messages

The graph should increase as new events arrive.

If no messages appear, the pipeline is not active.

------------------------------------------------------------------------

### Production Consideration

Event Hubs stores events for a limited time.

Ask yourself:

-   What happens if your analytics system is offline longer than the
    retention period?
-   How would you replay lost data?

------------------------------------------------------------------------

# Part 1 -- Create a Stream Analytics Job

Create a new **Azure Stream Analytics job**.

Recommended configuration:

Streaming Units: 1\
Hosting: Cloud\
Region: Same region as Event Hub

Streaming jobs run continuously.

Always start with the **smallest configuration**.

Reference:\
https://learn.microsoft.com/azure/stream-analytics/

------------------------------------------------------------------------

# Part 2 -- Configure the Event Hub Input

Add a new input.

Configuration:

Input type: Event Hub\
Alias: eh-input\
Event Hub: weather-events\
Consumer group: \$Default

Avoid using underscores in alias names.

Use hyphens instead.

When referencing aliases in queries, wrap them in brackets:

\[eh-input\]

Reference:\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-define-inputs

------------------------------------------------------------------------

# Part 3 -- Design the Streaming Query

The Stream Analytics query transforms raw events into structured
metrics.

Your query must:

-   extract temperature
-   extract AQI
-   extract PM2.5
-   preserve event timestamp
-   implement one windowed aggregation

You may use any window type:

-   Tumbling
-   Sliding
-   Hopping

Reference:\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-window-functions

------------------------------------------------------------------------

## Event Time Handling

Streaming systems must define **how time is interpreted**.

You must use event timestamps from the incoming data.

Reference:\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-time-handling

------------------------------------------------------------------------

### Production Consideration

Streaming systems often receive **late or out-of-order events**.

Ask yourself:

-   What happens when events arrive late?
-   How should the system treat delayed data?

------------------------------------------------------------------------

# Part 4 -- Configure Power BI Output

Add a Power BI output.

Configuration:

Output alias: pbi-output\
Dataset: DS562_HW4_Stream\
Table: weather_stream

Authorize Stream Analytics to connect to Power BI.

Reference:\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-define-outputs#power-bi

------------------------------------------------------------------------

### Important Behavior

Power BI **will not create the dataset until the first events arrive**.

If your dataset does not appear immediately, verify that:

-   the Stream Analytics job is running
-   events are flowing through Event Hub

------------------------------------------------------------------------

# Part 5 -- Build the Dashboard

Create a Power BI report using the streaming dataset.

Your dashboard must include:

### Temperature Over Time

Line chart:

X-axis: timestamp\
Y-axis: temperature

------------------------------------------------------------------------

### PM2.5 Over Time

Line chart:

X-axis: timestamp\
Y-axis: pm2_5

------------------------------------------------------------------------

### AQI Indicator

Card visualization displaying:

aqi

------------------------------------------------------------------------

### Windowed Aggregation

Visualization based on your window function.

Example:

average temperature over a 5-minute window

------------------------------------------------------------------------

### Production Consideration

Dashboards must be designed for an audience.

Consider:

-   Is this dashboard for engineers?
-   For operations teams?
-   For executives?

Different audiences require different levels of detail.

------------------------------------------------------------------------

# Part 6 -- Validate the System

Verify the entire pipeline:

Function → Event Hub → Stream Analytics → Power BI

Check:

-   Event Hub metrics show incoming events
-   Stream Analytics input metrics increase
-   Output metrics increase
-   Power BI visuals update

Reference:\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-monitoring

------------------------------------------------------------------------

# Part 7 -- Shutdown and Cost Control

After capturing screenshots:

1.  Stop the Stream Analytics job
2.  Stop the Function App
3.  Delete the Event Hub instance

Do **not delete the namespace**.

Streaming resources incur cost while running.

Responsible shutdown is part of cloud engineering.

Reference:\
https://learn.microsoft.com/azure/cost-management-billing/costs/cost-mgt-best-practices

------------------------------------------------------------------------

# Deliverables

Submit screenshots demonstrating:

1.  Event Hub receiving events
2.  Stream Analytics input configuration
3.  Query producing results
4.  Stream Analytics job running
5.  Power BI dashboard updating
6.  Resources stopped

------------------------------------------------------------------------

# Helpful Resources

Azure Stream Analytics documentation\
https://learn.microsoft.com/azure/stream-analytics/

Window functions\
https://learn.microsoft.com/azure/stream-analytics/stream-analytics-window-functions

Event-driven architecture\
https://learn.microsoft.com/azure/architecture/guide/architecture-styles/event-driven

Power BI streaming datasets\
https://learn.microsoft.com/power-bi/connect-data/service-real-time-streaming
