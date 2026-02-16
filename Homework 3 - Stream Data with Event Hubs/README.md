# Homework 3 – Real-Time Data Ingestion with Event Hubs and Azure Functions

## Overview

In Homework 3, you will build a **real-time data ingestion pipeline** using Azure Event Hubs and Azure Functions. This assignment shifts from the **batch processing** paradigm you implemented in HW1 and HW2 to **event-driven, serverless architecture**. For this assignment, we are simulating data capture by having a Function App (small bit of code) make the API request from OpenWeather, and then send the data to be received by the Event Hub. In HW4, this data will then be sent to Stream Analytics for further processing of the data and storage in the Bronze Zone. 

You will:

1. **Configure Azure Event Hubs** as a managed streaming platform to receive real-time weather and air pollution data
2. **Develop and deploy an Azure Function** that runs on a schedule (hourly), fetches data from OpenWeather API, and publishes it to Event Hubs
3. **Validate end-to-end message flow** by inspecting Event Hub metrics and message content
4. **Manage cloud costs** by properly disabling resources and calculating actual expenses

Although it would be valuable to learn the complete setup of the Azure Function and code, that would likely be more time consuming and isn't necessarily something you'd do that often as a data scientist or data engineer. Instead, we provide scaffolding for the Azure Functions and Event Hubs infrastructure so you can focus on the **data engineering logic**: API integration, data combination, schema design, and error handling.

> **How to succeed**
> - Start with the provided template code - don't write from scratch
> - Test locally before deploying to Azure (faster iteration)
> - Validate each step: API calls work → data combines correctly → Event Hubs receives messages
> - Build evidence as you go: logs, screenshots, test outputs
> - **Disable resources immediately after validation** to minimize costs

### Context: Real-Time vs Batch Processing

In HW2, you ingested **historical data** using Azure Data Factory with a looping pattern - perfect for backfilling large time ranges on a predictable schedule.

In HW3, you're building a **real-time ingestion** system where:
- Data is collected **continuously** (every hour, on the hour)
- Processing happens with **low latency** (minutes, not hours)
- Downstream systems can **react immediately** to new data

**Key architectural difference:**
- **Batch (HW2)**: ADF pulls data → writes to ADLS → Spark processes on-demand
- **Real-time (HW3)**: Azure Function pushes data → Event Hubs buffers → consumers process continuously

Event Hubs acts as a **decoupling layer**: your data producer (Azure Function) and consumers (Stream Analytics in HW4) operate independently. If your consumer crashes, Event Hubs retains messages so the consumer can catch up when it restarts.

**Reference:**
- Batch vs streaming architectures: https://learn.microsoft.com/azure/architecture/data-guide/big-data/#batch-processing
- Event-driven architecture patterns: https://learn.microsoft.com/azure/architecture/guide/architecture-styles/event-driven

---

## Learning Objectives

After completing this assignment, you should be able to:

- **Explain the role of Event Hubs** in decoupling producers and consumers in a streaming architecture
- **Reason about partitioning and message retention** and their impact on throughput, cost, and fault tolerance
- **Integrate multiple APIs** into a unified data pipeline with proper error handling
- **Design message schemas** that combine data from multiple sources
- **Deploy serverless functions** to Azure and configure them with environment variables
- **Validate message flow** using Event Hub metrics and message inspection
- **Articulate tradeoffs** between batch and streaming ingestion for different use cases
- **Manage cloud costs** through proper resource lifecycle management

---

## Part 1: Azure Event Hubs Setup and Configuration

### Objective

Create an Event Hubs namespace and instance configured to receive real-time weather and air pollution messages from your Azure Function.

---

### Key Design Constraints (Read Carefully)

Your Event Hubs setup must meet **all** requirements below:

- **Must create a namespace** (not just an Event Hub instance)
- **Partition count must be exactly 1** (you will explain what would change at higher scale)
- **Message retention must be 1 hour** (short retention for cost optimization)
- **Event Hubs Capture must be disabled** (we'll use Stream Analytics in HW4 for persistence)
- **Must use the Basic pricing tier** (cost control)
- **Must be in the same region** as your ADLS storage account from HW1
- **Event Hub instance name must be `weather-events`** (your function code will reference this)

### Why These Constraints?

**Partition count = 1:**
- Partitions enable parallel processing by multiple consumers
- With 1 partition, messages arrive in order and we have simple consumer logic
- For this homework: 1 message/hour = low throughput, 1 partition is appropriate
- **At scale:** 1000 messages/second would need multiple partitions to distribute load

**Message retention = 1 hour:**
- Event Hubs is a temporary buffer, not permanent storage
- Retention determines how long messages are available if a consumer needs to replay them
- **Cost optimization:** Shorter retention reduces cost
- **Tradeoff:** 1 hour is enough for validation; production might need 24+ hours for fault tolerance

**Basic tier:**
- Sufficient for homework (1 consumer group, 100 connections, 256 KB messages)
- Production would use Standard or Premium for higher throughput

**Reference:**
- Event Hubs quotas and limits: https://learn.microsoft.com/azure/event-hubs/event-hubs-quotas
- Event Hubs pricing: https://azure.microsoft.com/pricing/details/event-hubs/

---

### Configuration Steps

#### 1. Create Event Hubs Namespace

1. In Azure Portal, search for "Event Hubs" and click "Create"
2. **Basics tab:**
   - **Subscription:** Your Azure for Students subscription
   - **Resource Group:** Use the same resource group from HW0/HW1
   - **Namespace name:** `ds562-<yourname>-namespace` (must be globally unique)
   - **Location:** Same region as your ADLS Gen2 account
   - **Pricing tier:** Basic
   - **Throughput units:** 1 (default for Basic tier)

3. **Advanced tab:**
   - Leave all defaults

4. **Networking tab:**
   - **Connectivity method:** Public endpoint (all networks)

5. Click **Review + Create** → **Create**
6. Wait for deployment (1-2 minutes)

**Reference:**
- Creating Event Hubs: https://learn.microsoft.com/azure/event-hubs/event-hubs-create

---

#### 2. Create Event Hub Instance

1. Navigate to your Event Hubs namespace
2. In left sidebar under **Entities**, click **Event Hubs**
3. Click **+ Event Hub**
4. **Name:** `weather-events`
5. **Partition Count:** 1
6. **Message Retention:** 1 (hour)
7. **Event Hubs Capture:** Off (leave disabled)
8. Click **Create**

---

#### 3. Get Connection String

Your Azure Function needs to authenticate to Event Hubs using a connection string.

1. In your Event Hubs **namespace** (not the instance), go to **Settings** → **Shared access policies**
2. Click **RootManageSharedAccessKey**
3. **Copy** the **Primary Connection String**
   - Looks like: `Endpoint=sb://ds562-yourname-namespace.servicebus.windows.net/;SharedAccessKeyName=...`
4. **Save it securely** - you'll need it in Part 2

**Important:** Do NOT commit this to GitHub or include in screenshots.

**Reference:**
- Shared access policies: https://learn.microsoft.com/azure/event-hubs/authenticate-shared-access-signature

---

### Validation Checklist

Before proceeding to Part 2, verify:

- [ ] Event Hubs namespace exists with status "Active"
- [ ] Namespace uses Basic pricing tier
- [ ] Event Hub instance `weather-events` exists
- [ ] Instance shows 1 partition and 1 hour retention
- [ ] You have the connection string saved securely

**Screenshot requirements:**
1. Namespace overview showing name, region, and Basic tier
2. Event Hub instance showing partition count = 1 and retention = 1 hour

---

## Part 2: Azure Function Development and Deployment

### Objective

Deploy an Azure Function that fetches weather and air pollution data on an hourly schedule and publishes combined messages to Event Hubs.

**What we provide:** Template code that handles Azure Functions infrastructure and Event Hubs communication
**What you implement:** API integration, data combination, and error handling logic

---

### Required Message Schema

Your function must produce JSON messages with this exact structure:

```json
{
  "city": "Boston",
  "latitude": 42.3601,
  "longitude": -71.0589,
  "timestamp": "2026-02-14T07:00:00-05:00",
  "weather": {
    "coord": {"lon": -71.0589, "lat": 42.3601},
    "weather": [{"id": 800, "main": "Clear", "description": "clear sky"}],
    "main": {"temp": 2.5, "feels_like": -1.2, "pressure": 1013, "humidity": 65},
    "wind": {"speed": 4.1, "deg": 270},
    "clouds": {"all": 0},
    "dt": 1708005600,
    "name": "Boston"
  },
  "pollution": {
    "coord": {"lon": -71.0589, "lat": 42.3601},
    "list": [{
      "dt": 1708005600,
      "main": {"aqi": 2},
      "components": {
        "co": 230.31,
        "no": 0.01,
        "no2": 4.56,
        "o3": 68.66,
        "so2": 0.89,
        "pm2_5": 6.47,
        "pm10": 7.32,
        "nh3": 0.64
      }
    }]
  }
}
```

**Schema requirements:**
- `timestamp` in **EST timezone** with offset (e.g., `-05:00`)
- `weather` contains the **complete** OpenWeather `/data/2.5/weather` API response
- `pollution` contains the **complete** OpenWeather `/data/2.5/air_pollution` API response

---

### Local Development Setup

#### Prerequisites

Install the following on your computer:

1. **Python 3.9 or higher** - Verify: `python --version`
2. **Visual Studio Code** - https://code.visualstudio.com/
3. **Azure Functions extension for VS Code** - https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions
4. **Azure Functions Core Tools** - https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools

**Verification:**
```bash
python --version  # Should show 3.9+
func --version    # Should show 4.x
```

---

### Step 1: Download Template Code

We provide a complete Azure Functions project template. Download it from the course repository:

**Template structure:**
```
hw3-weather-function/
├── .vscode/                    # VS Code settings (provided)
├── .gitignore                  # Prevents committing secrets (provided)
├── host.json                   # Function app settings (provided)
├── requirements.txt            # Python dependencies (provided)
├── local.settings.json.template  # Template for local config (YOU CONFIGURE)
├── WeatherIngestion/           # Your function folder
│   ├── function.json          # Timer trigger config (provided)
│   ├── __init__.py            # Entry point (DO NOT MODIFY)
│   └── weather_ingestion.py  # YOUR CODE GOES HERE
└── README.md                   # Setup instructions
```

**Download link:** `[INSTRUCTOR: Insert repository URL or provide ZIP file]`

---

### Step 2: Configure Local Environment

1. **Rename the template file:**
   ```bash
   cp local.settings.json.template local.settings.json
   ```

2. **Edit `local.settings.json` and fill in your values:**
   ```json
   {
     "IsEncrypted": false,
     "Values": {
       "AzureWebJobsStorage": "UseDevelopmentStorage=true",
       "FUNCTIONS_WORKER_RUNTIME": "python",
       "EVENT_HUB_CONNECTION_STRING": "YOUR_CONNECTION_STRING_FROM_PART_1",
       "OPENWEATHER_API_KEY": "YOUR_API_KEY_FROM_HW1"
     }
   }
   ```

3. **Verify `.gitignore` includes `local.settings.json`**
   - Run `git status` - you should NOT see `local.settings.json` listed
   - This prevents accidentally committing secrets

---

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

**What gets installed:**
- `azure-functions` - Core framework
- `requests` - For calling OpenWeather API
- `azure-eventhub` - For publishing to Event Hubs
- `pytz` - For timezone conversions

---

### Step 4: Implement Your Code

**Open `WeatherIngestion/weather_ingestion.py` - this is where you write your code.**

The template provides:
- ✅ Azure Functions infrastructure (you don't touch this)
- ✅ Event Hubs publishing logic (already implemented)
- ✅ Function signatures with clear TODOs
- ❌ API integration (you implement)
- ❌ Data combination (you implement)
- ❌ Error handling (you implement)

**Template code structure:**

```python
"""
WeatherIngestion - Hourly data collection for Boston weather and air pollution

YOU IMPLEMENT:
- fetch_weather() - Call OpenWeather weather API
- fetch_pollution() - Call OpenWeather pollution API  
- combine_data() - Merge both datasets into required schema
- Error handling in main()

DO NOT MODIFY:
- send_to_eventhub() - Already implemented
- Anything in __init__.py
"""

import logging
import requests
import json
import os
from datetime import datetime
import pytz

# Boston coordinates
LATITUDE = 42.3601
LONGITUDE = -71.0589

# OpenWeather API endpoints
WEATHER_URL = "http://api.openweathermap.org/data/2.5/weather"
POLLUTION_URL = "http://api.openweathermap.org/data/2.5/air_pollution"


def fetch_weather(lat, lon, api_key):
    """
    Fetch current weather data from OpenWeather API.
    
    Args:
        lat (float): Latitude
        lon (float): Longitude
        api_key (str): OpenWeather API key
    
    Returns:
        dict: Weather data JSON response, or None if request fails
    
    TODO: Implement this function
    
    Hints:
    - Use requests.get() to call WEATHER_URL
    - Pass parameters: lat, lon, appid (api_key), units='metric'
    - Handle exceptions (network errors, API errors)
    - Return response.json() if successful, None if failed
    - Log errors using logging.error()
    
    Example:
        weather_data = fetch_weather(42.3601, -71.0589, "your_api_key")
        if weather_data:
            print(weather_data['main']['temp'])  # Temperature in Celsius
    """
    # YOUR CODE HERE
    pass


def fetch_pollution(lat, lon, api_key):
    """
    Fetch current air pollution data from OpenWeather API.
    
    Args:
        lat (float): Latitude
        lon (float): Longitude
        api_key (str): OpenWeather API key
    
    Returns:
        dict: Pollution data JSON response, or None if request fails
    
    TODO: Implement this function (similar to fetch_weather)
    
    Hints:
    - Use requests.get() to call POLLUTION_URL
    - Pass parameters: lat, lon, appid (api_key)
    - Handle exceptions
    - Return response.json() if successful, None if failed
    """
    # YOUR CODE HERE
    pass


def get_est_timestamp():
    """
    Get current timestamp in EST timezone as ISO 8601 string.
    
    Returns:
        str: Timestamp in format "2026-02-14T07:00:00-05:00"
    
    TODO: Implement this function
    
    Hints:
    - Get current UTC time: datetime.now(datetime.timezone.utc)
    - Convert to EST: use pytz.timezone('America/New_York')
    - Format as ISO string: .isoformat()
    
    Example:
        timestamp = get_est_timestamp()
        print(timestamp)  # "2026-02-14T07:00:00-05:00"
    """
    # YOUR CODE HERE
    pass


def combine_data(weather, pollution, timestamp):
    """
    Combine weather and pollution data into the required message schema.
    
    Args:
        weather (dict): Weather API response
        pollution (dict): Pollution API response
        timestamp (str): EST timestamp string
    
    Returns:
        dict: Combined message matching the required schema
    
    TODO: Implement this function
    
    Required schema:
    {
      "city": "Boston",
      "latitude": 42.3601,
      "longitude": -71.0589,
      "timestamp": "<timestamp parameter>",
      "weather": <weather parameter>,
      "pollution": <pollution parameter>
    }
    
    Hints:
    - Create a dictionary with the required fields
    - Use LATITUDE and LONGITUDE constants
    - weather and pollution should be the complete API responses
    """
    # YOUR CODE HERE
    pass


def send_to_eventhub(data):
    """
    Send data to Event Hubs.
    
    *** ALREADY IMPLEMENTED - DO NOT MODIFY ***
    
    This function handles all Event Hubs communication.
    Just call it with your combined data dictionary.
    
    Args:
        data (dict): Message to send (will be JSON-serialized)
    
    Raises:
        Exception: If Event Hub connection or send fails
    """
    from azure.eventhub import EventHubProducerClient, EventData
    
    connection_str = os.getenv("EVENT_HUB_CONNECTION_STRING")
    eventhub_name = "weather-events"
    
    if not connection_str:
        raise ValueError("EVENT_HUB_CONNECTION_STRING environment variable not set")
    
    try:
        producer = EventHubProducerClient.from_connection_string(
            conn_str=connection_str, 
            eventhub_name=eventhub_name
        )
        
        event_data_batch = producer.create_batch()
        event_data_batch.add(EventData(json.dumps(data)))
        producer.send_batch(event_data_batch)
        producer.close()
        
        logging.info("✓ Data sent to Event Hubs successfully")
    except Exception as e:
        logging.error(f"✗ Error sending data to Event Hub: {e}")
        raise


def main(mytimer):
    """
    Main function that runs on hourly schedule.
    
    TODO: Implement the orchestration logic
    
    Required logic:
    1. Get API key from environment variable
    2. Call fetch_weather() for Boston coordinates
    3. Call fetch_pollution() for Boston coordinates
    4. Get current timestamp in EST
    5. If BOTH API calls succeeded:
         - Call combine_data()
         - Call send_to_eventhub()
         - Log success
    6. If either API call failed:
         - Log which one failed
         - Do NOT send partial data
    
    Error handling strategy:
    - If an API fails: log the error, don't send message, function completes normally
    - If Event Hub send fails: let the exception propagate (function marked as failed)
    
    Why this strategy?
    - Skipping one hour of data is OK (we'll get data next hour)
    - But if Event Hubs is down, we want Azure to know the function failed
    """
    logging.info("=== WeatherIngestion function started ===")
    
    # Get API key from environment variable
    api_key = os.getenv("OPENWEATHER_API_KEY")
    if not api_key:
        logging.error("✗ OPENWEATHER_API_KEY environment variable not set")
        return
    
    # YOUR CODE HERE
    # 
    # Implement the orchestration logic described above
    # Use the functions you implemented: fetch_weather, fetch_pollution, 
    # get_est_timestamp, combine_data, send_to_eventhub
    
    pass
```

**Reference for implementation:**
- OpenWeather Current Weather API: https://openweathermap.org/current
- OpenWeather Air Pollution API: https://openweathermap.org/api/air-pollution
- Python requests library: https://requests.readthedocs.io/en/latest/
- Python datetime and timezones: https://docs.python.org/3/library/datetime.html

---

### Step 5: Test Locally

1. **Open the project in VS Code:**
   ```bash
   code hw3-weather-function
   ```

2. **Start the function locally:**
   - Press `F5` (Start Debugging)
   - Or: Terminal → Run Task → "func: host start"

3. **Trigger the function manually** (don't wait for the timer):
   - In VS Code: Right-click function in Azure extension → "Execute Function Now..."
   - Or use HTTP: `curl -X POST http://localhost:7071/admin/functions/WeatherIngestion`

4. **Check that the logs in the terminal look something like the following:**
   ```
   [2026-02-14T12:00:00.000Z] Executing 'WeatherIngestion'
   [2026-02-14T12:00:00.123Z] === WeatherIngestion function started ===
   [2026-02-14T12:00:01.234Z] ✓ Weather data retrieved
   [2026-02-14T12:00:01.345Z] ✓ Pollution data retrieved
   [2026-02-14T12:00:01.456Z] ✓ Data sent to Event Hubs successfully
   [2026-02-14T12:00:01.567Z] Executed 'WeatherIngestion' (Succeeded)
   ```

**Expected behavior:**
- No errors in logs
- Both API calls succeed
- Event Hub send succeeds
- Function completes

**Troubleshooting tips if you get errors:**

| Error | Cause | Fix |
|-------|-------|-----|
| `KeyError: 'OPENWEATHER_API_KEY'` | Environment variable not set | Check `local.settings.json` has correct key name |
| `401 Unauthorized` from API | Invalid API key | Verify API key is active at openweathermap.org |
| `ConnectError` to Event Hubs | Wrong connection string | Verify connection string format in `local.settings.json` |
| `ModuleNotFoundError` | Dependencies not installed | Run `pip install -r requirements.txt` |

**Reference:**
- Running functions locally: https://learn.microsoft.com/azure/azure-functions/functions-run-local
- Debugging in VS Code: https://learn.microsoft.com/azure/azure-functions/functions-develop-vs-code?tabs=python#debugging

---

### Step 6: Deploy to Azure

#### Create Function App

1. **In VS Code Azure extension**, click "Create Function App in Azure..." (cloud icon)
2. **Configuration:**
   - **Name:** `ds562-<yourname>-weather-func` (globally unique)
   - **Runtime:** Python 3.9 (or your version)
   - **Region:** Same as Event Hubs namespace
   - **Hosting plan:** Consumption
   - **Storage account:** Use existing
   - **Application Insights:** No (to save cost)

**What gets created:**
- Function App (serverless compute)
- Consumption plan (pay only when function runs)

---

#### Deploy Your Code

1. **Right-click your Function App** in Azure extension
2. **Select "Deploy to Function App..."**
3. **Confirm deployment**
4. **Wait for completion** (1-2 minutes)

**Verification:**
- In Azure Portal → Your Function App → Functions
- You should see `WeatherIngestion` listed

---

#### Configure Environment Variables in Azure

**Important:** `local.settings.json` is NOT deployed to Azure. You must set environment variables in the Portal.

1. **Azure Portal** → Your Function App → **Settings** → **Environment variables**
2. **Under "App settings"**, click **+ Add**
3. **Add both variables:**
   - Name: `EVENT_HUB_CONNECTION_STRING`, Value: `<your connection string>`
   - Name: `OPENWEATHER_API_KEY`, Value: `<your API key>`
4. Click **Apply** → **Confirm**

**Verification:**
- Go to your function → **Code + Test**
- Add a test: Click **Run**
- Check logs - should show function executing successfully

**Reference:**
- Deploying from VS Code: https://learn.microsoft.com/azure/azure-functions/functions-develop-vs-code?tabs=python#republish-project-files
- Managing app settings: https://learn.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings

---

### Step 7: Validate Scheduled Execution

Your function is configured to run **every hour at minute :00** (NCRONTAB: `0 0 * * * *`).

**Wait for the next hour boundary**, then check:

1. **Monitor page:**
   - Function App → Functions → WeatherIngestion → **Monitor**
   - Should show invocation at :00 minutes
   - Status: Success
   - Duration: < 10 seconds

2. **Execution logs:**
   - Click the invocation → View logs
   - Should show same messages as local testing

3. **Event Hub metrics:**
   - Event Hubs namespace → **Metrics**
   - Add metric: "Incoming Messages"
   - Time range: Last 1 hour
   - Should show 1 message at function execution time

**Reference:**
- Monitoring Azure Functions: https://learn.microsoft.com/azure/azure-functions/functions-monitoring

---

### Debugging Guide

If something doesn't work, debug in this order:

#### Level 1: Test APIs Outside Your Code

Use a browser or Postman:

```
Weather:
http://api.openweathermap.org/data/2.5/weather?lat=42.3601&lon=-71.0589&appid=YOUR_KEY&units=metric

Pollution:
http://api.openweathermap.org/data/2.5/air_pollution?lat=42.3601&lon=-71.0589&appid=YOUR_KEY
```

Expected: HTTP 200, JSON response

---

#### Level 2: Debug Locally in VS Code

1. Set breakpoints in `weather_ingestion.py`
2. Press F5
3. Step through and inspect variables
4. Verify API responses are valid dicts (not None)

---

#### Level 3: Check Azure Logs

**Function logs:**
- Function → Monitor → Click invocation → Logs tab
- Look for error messages

**Common deployment issues:**

| Log Message | Cause | Fix |
|-------------|-------|-----|
| `KeyError: 'OPENWEATHER_API_KEY'` | Environment variable not set in Azure | Add to Settings → Environment variables |
| `401 Client Error` | API key not in Azure | Add OPENWEATHER_API_KEY |
| `None` returned from API | Exception not logged | Add more detailed logging in try/except |

---

#### Level 4: Validate Event Hubs

**Check metrics:**
- Event Hubs namespace → Metrics
- Metric: "Incoming Messages"
- Should match number of function executions

**If metrics show 0 but function logs show success:**
- Verify `eventhub_name = "weather-events"` in code
- Verify connection string points to correct namespace

---

### Validation Checklist

Before proceeding to Part 3:

- [ ] Function runs successfully locally
- [ ] Function deployed to Azure
- [ ] Environment variables configured in Azure
- [ ] Function executes on schedule (Monitor shows invocations at :00 minutes)
- [ ] No errors in logs
- [ ] Event Hub metrics show incoming messages
- [ ] At least **2 messages** received, ~1 hour apart

**Screenshot requirements:**
1. Local testing terminal showing successful execution
2. Function App overview (name, region, Consumption plan)
3. Monitor page showing 2+ successful executions
4. Event Hub metrics showing incoming messages

---

## Part 3: Message Validation and Inspection

### Objective

Verify that messages reaching Event Hubs match the required schema.

---

### Inspect a Sample Message

**Method A: Event Hubs Real-Time Insights**

1. Event Hub instance → **Process data** → Enable real-time insights
2. Wait for next function execution
3. View message in preview pane
4. Copy the JSON
5. **Disable real-time insights immediately** (costs money)

**Method B: Event Hubs Capture (Temporary)**

1. Event Hub instance → Enable Capture
2. Configure output to Azure Blob Storage
3. Wait for next function execution
4. Download Avro file and extract JSON

**Method C: Wait for HW4**

If you can't inspect now, you'll validate when Stream Analytics reads from Event Hubs in HW4.

---

### Verify Schema

Check that your message has:

- ✅ `city` = "Boston"
- ✅ `latitude` = 42.3601
- ✅ `longitude` = -71.0589
- ✅ `timestamp` in EST (offset `-05:00` or `-04:00`)
- ✅ `weather` object with `main`, `weather`, `wind`, `clouds`
- ✅ `pollution` object with `list` array containing `components`

**Screenshot requirement:**
- One complete message (formatted for readability)
- Annotations showing the key schema elements

**Reference:**
- Event Hubs Capture: https://learn.microsoft.com/azure/event-hubs/event-hubs-capture-overview

---

## Part 4: Resource Management and Cost Analysis

### Objective

Minimize costs by stopping resources immediately after validation.

---

### Required Actions (Graded)

#### 1. Stop Function App

**As soon as you have your screenshots:**

1. Azure Portal → Function App → **Overview**
2. Click **Stop**
3. Verify status = "Stopped"

---

#### 2. Delete Event Hub Instance

**Keep the namespace, delete the instance:**

1. Event Hubs namespace → Event Hubs
2. Select `weather-events`
3. Click **Delete**

**Why:**
- You'll recreate the instance in HW4
- Reduces namespace cost slightly
- Namespace name remains reserved for you

---

## Part 5: Reflection Questions

Be able to answer the following:

### Question 1: Partitioning at Scale

You configured 1 partition (1 message/hour = low throughput).

**Scenario:** Your company expands to ingest data for **500 cities** every hour (500 messages/hour).

a) How many partitions would you configure? Justify your answer.

b) How would you ensure all messages for the same city go to the same partition? (Research partition keys)

c) What changes in Stream Analytics (HW4) when consuming from multiple partitions?

**Reference:**
- Event Hubs partitioning: https://learn.microsoft.com/azure/event-hubs/event-hubs-features#partitions

---

### Question 2: Error Handling

Your implementation only sends a message if BOTH APIs succeed.

**Scenario A:** Weather API succeeds, pollution API fails.
- What happens to the weather data?
- Is this the right design choice? Why?

**Scenario B:** Both APIs succeed, Event Hubs is down.
- Does your function retry?
- Should it?

**Scenario C:** Function times out after 50 seconds.
- Should you increase timeout or optimize API calls?
- What risks does longer timeout introduce?

---

### Question 3: Batch vs Real-Time

Compare HW2 (batch via ADF) with HW3 (real-time via Functions).

a) **Latency:** Which provides fresher data? Quantify the difference.

b) **Debuggability:** Which is easier to debug when something breaks? Why?

c) **Cost:** Which is more cost-effective for 1 message/hour?

d) **Your Project:** Which approach for your final project? Why?

---

### Question 4: Message Retention

You configured 1-hour retention.

a) What happens if Stream Analytics (HW4) is down for 2 hours? Can it recover?

b) If you increased retention to 7 days:
   - What benefits?
   - What costs?
   - When is it worth it?

c) Design a hybrid approach for long-term retention without paying for 7-day Event Hubs retention.

---

## Deliverables

Submit to Gradescope:

### 1. Code (ZIP)

**Include:**
- `requirements.txt`
- `WeatherIngestion/weather_ingestion.py` (your implementation)
- `function.json`
- `host.json`

**Exclude:**
- `local.settings.json` (contains secrets)
- `.vscode/`
- `__pycache__/`

---

### 2. Screenshots (PDF)

One PDF with labeled screenshots:

1. Event Hubs namespace overview
2. Event Hub instance settings (partition=1, retention=1hr)
3. Local testing terminal output
4. Function App overview
5. Monitor page (2+ successful executions)
6. Event Hub metrics (incoming messages)
7. Sample message content

---

## Grading Rubric

| Category | Points | Criteria |
|----------|--------|----------|
| **Event Hubs Config** | 15 | Correct settings, evidence provided |
| **Function Implementation** | 30 | All functions implemented, schema correct, error handling |
| **Local Testing** | 10 | Evidence of successful local execution |
| **Azure Deployment** | 15 | Deployed correctly, runs on schedule |
| **Message Validation** | 15 | Schema verified, timestamps correct |
| **Resource Management** | 15 | Stopped promptly |

| **Total** | **100** | |

---

## FAQ

**"My function works locally but not in Azure"**
→ Check environment variables are set in Azure Portal

**"How do I know messages reached Event Hubs?"**
→ Check Event Hubs metrics (Incoming Messages count)

**"Can I use a different city?"**
→ Yes, update coordinates and city name in code

**"I've used up my Azure credits"**
→ Delete old resources from HW1/HW2, or contact TA

---

## Additional Resources

- Azure Functions Python guide: https://learn.microsoft.com/azure/azure-functions/functions-reference-python
- Event Hubs features: https://learn.microsoft.com/azure/event-hubs/event-hubs-features
- OpenWeather API docs: https://openweathermap.org/api
- Python requests library: https://requests.readthedocs.io/

---

**End of Homework 3**

Remember: Test locally first, validate each step, and stop resources immediately after getting evidence!
