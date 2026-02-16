# Homework 3 – Student Troubleshooting Guide
*(Real-Time Weather Ingestion with Event Hubs and Azure Functions)*

This guide is aligned to the **current Homework 3 README** and covers common issues encountered during:
- Event Hubs configuration
- Azure Functions local development and deployment
- Message validation and schema verification

Use this guide to unblock yourself before asking for help.

---

## Event Hubs Setup Issues

### Issue: Cannot create Event Hubs namespace
**Likely causes**
- Name already taken globally
- Region does not support Event Hubs Basic tier

**What to try**
- Add numbers or initials to namespace name
- Use same region as your ADLS storage account
- Verify subscription has available quota

Reference:
https://learn.microsoft.com/azure/event-hubs/event-hubs-create

---

### Issue: Event Hub instance not appearing
**Likely cause**
- Created in different namespace
- Refresh delay in Portal

**What to check**
- Navigate to correct namespace first
- Click "Event Hubs" under Entities
- Wait 10 seconds and refresh

---

### Issue: Cannot find connection string
**Where to look**
- Go to **namespace** (not the Event Hub instance)
- Settings → Shared access policies
- Click "RootManageSharedAccessKey"
- Copy "Primary Connection String"

**Common mistake:** Copying from Event Hub instance instead of namespace

---

## Local Development Issues

### Issue: "Connection refused (127.0.0.1:10000)"
**Likely cause**
- Function trying to use Azurite (local storage emulator) which is not running

**Quick fix**
Edit `local.settings.json`:
```json
"AzureWebJobsStorage": "",  // Change to empty string
```

**Alternative fix**
- Install and run Azurite: `npm install -g azurite && azurite`

Reference:
https://learn.microsoft.com/azure/azure-functions/functions-develop-local

---

### Issue: "Port 7071 is unavailable"
**Likely cause**
- Previous `func start` still running

**What to try**
```bash
# Kill the process
kill -9 $(lsof -ti:7071)

# Or use different port
func start --port 7072
```

---

### Issue: "ModuleNotFoundError" for azure-eventhub or pytz
**Likely cause**
- Dependencies not installed

**Fix**
```bash
pip install -r requirements.txt

# Verify installation
pip list | grep -E 'azure-eventhub|pytz|requests'
```

---

### Issue: "OPENWEATHER_API_KEY environment variable not set"
**What to check**
- `local.settings.json` exists (not just the template)
- Variable name matches exactly in code and file
- JSON syntax is valid (no missing commas)
- Function runtime restarted after changing settings

**Test your settings file**
```bash
# Verify it's valid JSON
python3 -c "import json; print(json.load(open('local.settings.json')))"
```

---

### Issue: "AttributeError: type object 'datetime.datetime' has no attribute 'timezone'"
**Likely cause**
- Missing import

**Fix**
Add `timezone` to your imports:
```python
from datetime import datetime, timezone  # Add timezone here
import pytz
```

Then use:
```python
utc_time = datetime.now(timezone.utc)  # Not datetime.timezone.utc
```

---

### Issue: "401 Unauthorized" from OpenWeather API
**Likely causes**
- API key invalid or not activated
- Wrong parameter name (`api_key` vs `appid`)

**What to try**
- Test API key in browser first
- Check API key is active at openweathermap.org
- Verify parameter is named `appid` (not `api_key`)

**Test URL**
```
http://api.openweathermap.org/data/2.5/weather?lat=42.3601&lon=-71.0589&appid=YOUR_KEY&units=metric
```

---

### Issue: Function runs but returns None for API calls
**Likely causes**
- Exception caught but not logged clearly
- Network timeout

**What to add**
More detailed logging in your exception handler:
```python
except requests.exceptions.RequestException as e:
    logging.error(f"API Error: {type(e).__name__}: {str(e)}")
    logging.error(f"Response status: {e.response.status_code if e.response else 'N/A'}")
    return None
```

---

## Azure Deployment Issues

### Issue: Deployment succeeds but function not listed
**Likely causes**
- Wrong folder deployed
- Function not properly structured

**What to check**
- Deployed folder contains `host.json` at root
- `WeatherIngestion/function.json` exists
- Refresh Functions list in Portal

---

### Issue: Function fails immediately in Azure
**Most common cause**
- Environment variables not set in Azure

**Fix**
1. Function App → Settings → Environment variables
2. Add both:
   - `EVENT_HUB_CONNECTION_STRING`
   - `OPENWEATHER_API_KEY`
3. Click "Apply" then "Confirm"
4. Wait 30 seconds for restart

**Verify:** Code + Test → Run → Check logs for "environment variable not set"

---

### Issue: "ModuleNotFoundError" in Azure but works locally
**Likely causes**
- `requirements.txt` not deployed
- Python version mismatch

**What to check**
- `requirements.txt` exists at project root (next to `host.json`)
- Settings → Configuration → Python version matches local
- Redeploy and check Deployment Center logs

---

### Issue: Function shows "Stopped" status
**Cause**
- Function App was manually stopped

**Fix**
- Function App → Overview → Click "Start"

---

## Event Hub Validation Issues

### Issue: Function logs show success but Event Hub metrics show 0 messages
**Likely causes**
- Event Hub name mismatch in code
- Connection string points to wrong namespace

**What to check**
```python
# In send_to_eventhub() - must match exactly (case-sensitive)
eventhub_name = "weather-events"  
```
- Portal: Event Hub instance name is "weather-events"
- Connection string contains correct namespace name

**Debug test**
Add logging before send:
```python
logging.info(f"Sending to: {eventhub_name}")
logging.info(f"Connection string namespace: {connection_str.split(';')[0]}")
```

---

### Issue: Cannot see messages in Event Hub
**Where to look**
1. Event Hub → Process data → Enable real-time insights
2. Wait for next function execution
3. Query data to view messages
4. **Remember to disable when done** (costs money)

**Alternative:** Wait for HW4 when Stream Analytics will show messages

---

### Issue: Timestamp in message shows UTC (+00:00) instead of EST
**Likely cause**
- Did not convert timezone

**Fix in get_est_timestamp()**
```python
def get_est_timestamp():
    utc_time = datetime.now(timezone.utc)
    est_tz = pytz.timezone('America/New_York')
    est_time = utc_time.astimezone(est_tz)
    return est_time.isoformat()  # Returns with -05:00 or -04:00
```

---

### Issue: Message missing weather or pollution field
**Likely cause**
- Sending message even when one API fails

**Fix in main()**
```python
# Only send if BOTH succeeded
if weather_data and pollution_data:
    # combine and send
else:
    logging.error("One or both APIs failed - message not sent")
```

---

## Timer Trigger Issues

### Issue: Function does not run on schedule
**What to check**
- Function App is "Running" (not "Stopped")
- Monitor → Invocations shows execution attempts
- NCRONTAB expression is correct: `0 0 * * * *`

**If no invocations appear:**
- Trigger manually first (Code + Test → Run)
- Check for errors preventing startup
- Verify `function.json` has correct schedule

Reference:
https://learn.microsoft.com/azure/azure-functions/functions-bindings-timer

---

### Issue: Timer fires at wrong time
**Likely cause**
- NCRONTAB expression incorrect

**Correct format for hourly at :00**
```json
"schedule": "0 0 * * * *"
// Format: {second} {minute} {hour} {day} {month} {day-of-week}
```

**Test your expression:** Monitor page shows "Next 5 occurrences"

---

## Cost Management Issues

### Issue: Unexpected charges appearing
**What to check**
- Event Hubs namespace still active (Basic tier = ~$0.015/hour)
- Function App still running (Consumption usually free for low usage)
- Storage account costs (usually minimal)

**Immediate actions**
1. Stop Function App (Overview → Stop)
2. Delete Event Hub instance (keep namespace for HW4)
3. Check Cost Management for breakdown

---

### Issue: Cannot delete Event Hub instance
**Likely cause**
- Capture enabled or active connections

**What to try**
- Disable Capture first
- Wait 5 minutes for connections to close
- Try delete again

---

## Debugging Workflow (Follow This Order)

### When something doesn't work:

**1. Test APIs outside your code**
- Use browser or Postman
- Verify API key and parameters work

**2. Test function locally**
- Run `func start`
- Trigger manually
- Check logs for errors

**3. Check Azure configuration**
- Environment variables set?
- Function App running?
- Correct Python version?

**4. Verify Event Hubs**
- Check metrics for incoming messages
- Inspect sample message
- Verify schema matches requirements

---

## When Asking for Help, Include

**For local issues:**
- Complete error message from terminal
- Your `requirements.txt` contents
- Python version (`python3 --version`)
- Screenshot of `local.settings.json` structure (blur values)

**For Azure issues:**
- Screenshot of Monitor page showing failed execution
- Screenshot of Environment variables page (blur values)
- Copy of error from Logs tab
- Screenshot of Event Hub metrics

**For schema issues:**
- Sample message JSON
- Describe what's wrong vs what's expected

---

## Quick Reference: Common Error Messages

| Error Message | Most Likely Cause | Quick Fix |
|---------------|-------------------|-----------|
| Connection refused 127.0.0.1:10000 | Azurite not running | Set `AzureWebJobsStorage: ""` |
| Port 7071 unavailable | Previous func still running | `kill -9 $(lsof -ti:7071)` |
| OPENWEATHER_API_KEY not set | Missing env var | Check `local.settings.json` |
| 401 Unauthorized | Bad API key | Test in browser first |
| AttributeError: datetime.timezone | Missing import | Add `timezone` to imports |
| EVENT_HUB_CONNECTION_STRING not set | Env var not in Azure | Add in Portal settings |
| ModuleNotFoundError in Azure | Requirements not deployed | Verify `requirements.txt` at root |
| Metrics show 0 messages | Event Hub name mismatch | Check `eventhub_name = "weather-events"` |
| Timestamp shows UTC | Timezone not converted | Implement `get_est_timestamp()` |

---

## Still Stuck?

1. **Re-read the relevant section** in the main HW3 README
2. **Check the debugging hierarchy** in the assignment (Level 1-4)
3. **Review the solution structure** - is your code similar?
4. **Post on Piazza** with the information listed above

**Before posting:**
- Have you tested locally?
- Have you checked environment variables?
- Have you verified Event Hubs namespace is active?
- Can you access OpenWeather API in a browser?
