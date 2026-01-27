# Homework 1: End-to-End Data Ingestion

## Overview

Welcome to your first Homework in DS591. Out of the gate, please be patient, read the instructions fully, and use the help links provided. It would also make sense to go and check out Homework 0 to gain familiarity if you've never used tools like this. Remember, this homework gives you a basis for the project. If you run into problems, before going to and inundating the TAs post to Piazza and look at the troubleshooting page as it may provide your answers. Finally, as you get on in your career, you will find that the opportunity to learn new things provides job satisfaction, and you will never be given step by step instructions on how to use new tools. The more you embrace the process of learning the tools you are working with, the better you will be at your job.

In this assignment, you will build your **first complete Azure-based data pipeline**, starting from account setup and ending with building your first Azure Data Factory Pipeline.

You are **not expected to have prior Azure experience**. This assignment is intentionally written in a step-by-step, procedural manner. The goal is not speed or elegance, but understanding how modern data platforms are assembled and used in practice.

By the end of this homework, you will have:
- Ingested real-world data from an external API
- Stored that data using a Lakehouse-style (Medallion) folder structure
- Designed ingestion pipelines that handle real API constraints
- Identified patterns, relationships, and data quality issues

---

## Learning Objectives

After completing this assignment, you should be able to:
- Navigate the Azure Portal and understand core services
- Explain the purpose of Azure Data Factory and Data Lake Storage
- Build ingestion pipelines using REST/HTTP APIs

---

## Part 1: Azure Account and Resource Setup

### Objective

Create the Azure resources required for the rest of the assignment.

Take your time here. **Most issues later in the homework originate from mistakes in this section**, especially subscription selection and storage configuration.

---

### Step 1: Create an Azure Student Account

1. Go to:  
   https://azure.microsoft.com/free/students/
2. Click **Activate your student benefits**
3. Sign up using your ***university email***
4. When prompted, select **Azure for Students**

Do **not** select a university-managed or enterprise subscription such as "IS&T Engineering"

**Help**
- Azure Student FAQ:  
  https://learn.microsoft.com/azure/cost-management-billing/manage/azure-for-students-faq

---

### Step 2: Create a Resource Group

A **resource group** is a logical container that holds all related Azure resources.

1. In the Azure Portal, search for **Resource groups**
2. Click **Create**
3. Use the following naming convention:
   FirstName-LastName-RG
> **Note:** You will be graded on your ability to standardize on naming conventions. This is for several reasons, including readability, troubleshooting, and ease of grading. 
5. Region: 
    Select the region closest to you but be sure to use this region throughout the semester
6. Click **Review + Create**

**Help**
- Resource groups explained:  
https://learn.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal

---

### Step 3: Create an Azure Data Lake Storage Gen2 Account

This is where all ingested data will be stored.

1. Search for **Storage accounts**
2. Click **Create**
3. Select:
- Your subscription
- Your resource group
4. Storage account name:
- Must be lowercase
- Must be globally unique
5. Region: Select the region you selected for your Resource Group
6. Performance: **Standard**
7. Redundancy: **Locally-redundant storage (LRS)** - Saves Cost

**Important (Required)**  
Under **Advanced** settings:
- Enable **Hierarchical namespace**

This converts Blob Storage into **Data Lake Storage Gen2**, which is required for Spark and Lakehouse patterns.

**Help**
- Data Lake Gen2 overview:  
https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction

---

### Step 4: Create Medallion Architecture Folders

1. Open your storage account
2. Navigate to **Storage browser**
3. Create the following Containers (folders):
Bronze
Silver
Gold

For this assignment, **all ingested data remains in Bronze** (raw, unprocessed).

**Help**
- Storage browser overview:  
https://learn.microsoft.com/azure/storage/blobs/storage-blob-storage-browser

---

### Step 5: Create Azure Data Factory

Azure Data Factory (ADF) is used to **orchestrate and automate data movement**.

1. Search for **Data Factory**
2. Click **Create**
3. Select:
- Your resource group
- Region: Select the region you selected for your Resource Group
1. Click **Review + Create**

**Help**
- Azure Data Factory introduction:  
https://learn.microsoft.com/azure/data-factory/introduction

---

### Step 6: Grant Instructor Access (Required for Grading)

1. Open your **Resource Group**
2. Click **Access control (IAM)**
3. Click **Add role assignment**
4. Click **Privileged Administrator Roles**
5. Role: **Contributor**
6. Assign to instructor email(s)
   - seferlis@bu.edu
   - z2004@bu.edu
   - nc0926@bu.edu
7. Ensure assignment is:
- **Active**
- **Permanent**

Submissions without access **cannot be graded**.

**Help**
- Azure IAM basics:  
https://learn.microsoft.com/azure/role-based-access-control/role-assignments-portal

---

## Part 2: API Data Ingestion with Azure Data Factory

### Objective

Use Azure Data Factory to ingest **air pollution** data from the OpenWeather API into your Data Lake.

This section introduces your first real **engineering challenge**: APIs do not always allow unlimited data access, so your pipelines must adapt to constraints.

>**Note:** Azure Data Factory is an online only experience. If you are working in the studio, you don't save changes unless you connect to your GitHub account. The alternative is to Publish your work each time, but assets cannot be published unless there are no errors. Plan carefully as you work on your pipelines because simply closing your laptop and walking away from the interface can result in lost work
>**Note:** ***DO NOT USE THE "DEBUG" OPTION. THIS WILL DEPLETE YOUR RESOURCES QUICKLY***

---

### Step 1: Create an OpenWeather API Account

1. Go to:  
https://openweathermap.org/our-initiatives
2. Create a free student account
3. Generate an **API key**
4. Save the key securely (you will use it in ADF)

**Help**
- OpenWeather API documentation:  
https://openweathermap.org/api

---

## Postman Mini-Tutorial (Recommended Before Building ADF Pipelines)

### Why use Postman here?

Before you automate API calls in Azure Data Factory, it helps to **manually call the API once** so you can:
- confirm your API key works
- see the exact JSON shape you will ingest
- understand which parameters are required vs optional
- detect API limits and error messages early

### Step 1: Install or Access Postman
- Download Postman: https://www.postman.com/downloads/
- Or use Postman Web: https://www.postman.com/

**Help**
- Postman “Send your first request”:  
https://learning.postman.com/docs/getting-started/sending-the-first-request/

### Step 2: Create a New Request
1. Open Postman
2. Click **New** → **HTTP Request**
3. Set the method to **GET**

### Step 3: Set Up Environment Variables (Recommended)

Create variables such as:
- `base_url` = `https://api.openweathermap.org`
- `appid` = `<your_api_key>`
- `lat` = `42.3601`
- `lon` = `-71.0589`

**Help**
- Postman variables:  
https://learning.postman.com/docs/sending-requests/variables/

### Step 4: Test a Simple Current Weather Call

**Request**
- GET `{{base_url}}/data/2.5/weather`

**Query Params**
- `lat` = `{{lat}}`
- `lon` = `{{lon}}`
- `appid` = `{{appid}}`
- `units` = `metric`

You should receive a `200 OK` response with JSON output.

---

### Step 2: Air Pollution Pipeline (Single API Call Pattern)

#### Goal
Ingest approximately **one year of hourly air pollution data (01/01/2025 to 01/01/2026)** for Boston.

#### Required Design
1. Open **Azure Data Factory Studio**
2. Create a new **Pipeline**
3. Add a **Copy Data** activity
4. Configure the **Source**:
- Connector: HTTP
- Base URL: `http://api.openweathermap.org`
- Use dataset parameters for:
  - Latitude
  - Longitude
  - Start timestamp
  - End timestamp
  - API key
5. Configure the **Sink**:
- Azure Data Lake Storage
- Output folder:
  ```
  /bronze/air_pollution/
  ```

  You have now created your first Azure Pipeline. Submit that you are complete in Gradescope and the TAs will grade your assignment.
