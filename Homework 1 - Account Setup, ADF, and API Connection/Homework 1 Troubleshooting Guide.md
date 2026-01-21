# Homework 1 – Student Troubleshooting Guide
*(Scoped to Azure setup and Air Pollution API ingestion only)*

This guide covers the most common issues students encounter while completing Homework 1

Review this document before asking for help.

---

## Azure Account and Subscription Issues

### Issue: I can’t create resources or I don’t see them
**Likely cause**
- Wrong subscription selected

**How to fix**
1. Click your profile in the Azure Portal (top right)
2. Select **Switch directory**
3. Confirm **Azure for Students** is selected

Reference:  
https://learn.microsoft.com/azure/cost-management-billing/manage/azure-for-students-faq

---

### Issue: Resource creation fails immediately
**Likely causes**
- Incorrect region selected
- Azure credits exhausted

**How to fix**
- Use **Region Discussed in Resource Group Setup**
- Check **Cost Management → Credits**

---

## Storage Account and Data Lake Issues

### Issue: I can’t create folders inside folders
**Likely cause**
- Hierarchical namespace was not enabled

**How to fix**
- You must recreate the storage account with hierarchical namespace enabled

Reference:  
https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction

---

### Issue: Bronze folder exists but is empty
**Likely causes**
- Pipeline not executed
- Sink path incorrect

**How to fix**
- Confirm pipeline run status = **Succeeded**
- Verify sink path starts with `/bronze/air_pollution/`

---

## Azure Data Factory – Air Pollution Pipeline

### Issue: HTTP 401 or 403 error
**Likely cause**
- API key missing or incorrect

**How to fix**
- Verify `appid` query parameter exists
- Re-copy API key from OpenWeather
- Test API call in Postman

Reference:  
https://learn.microsoft.com/azure/data-factory/connector-http

---

### Issue: Pipeline succeeds but output JSON is empty or very small
**Likely causes**
- Invalid timestamps
- Incorrect latitude / longitude

**How to fix**
- Verify timestamps are valid UNIX seconds
- Confirm Boston coordinates
- Test the exact API URL in Postman

---

### Issue: Pipeline was saved but never run
**How to fix**
- Click **Debug** or **Trigger now**
- Confirm run status = **Succeeded**

---

## When Asking for Help, Include
- Screenshot of the error
- Screenshot of pipeline run output
- What you expected vs what happened
