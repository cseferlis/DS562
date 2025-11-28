# HW1: Azure Account Setup & API Connection

## Part 1: Azure Account Setup
**Objective:**
Set up an Azure account and the foundational resources necessary for the subsequent homework assignments. This involves creating a Resource Group, Storage Account, Blob Storage container, Azure Data Factory, and creating an OpenWeather account for an API key.
**Deliverables:**
- Air Pollution Data containing past year's of data
- Medallion Architecture setup for Blob Storage Directory (Bronze, Silver, Gold)
- Historical Weather Data containing past 52 weeks of data
- Azure Data Factory Creation
- ADF Pipeline creation for Air Pollution
- ADF Pipeline creation for Historical Weather utilizing For Each
- Correct Source and Sink Datasets
#### PLEASE USE YOUR FIRST AND LAST NAME FOR THE RESOURCE GROUP SO WE CAN EASILY IDENTIFY THE CORRECT HW!
#### PLEASE ALSO MAKE SURE TO GIVE INSTRUCTORS "contributor access to ACTIVE, PERMANENT"
#### 1. Sign Up for an Azure Student Account
1. Visit the [Azure Student Free Account website](https://azure.microsoft.com/en-us/free/students) and click on “Activate your student benefits.”
2. Follow the prompts to sign up for a new Azure student account. You might need to verify your student status through your educational institution’s email. Note that Azure offers *$100* in credits for 12 months and a range of free services.
3. Make sure to select the **Azure Student Subscription** (and not the BU IS&T subscription)
Using your existing Data Factory, you will:

#### 2. Set Up a Resource Group within your Subscription
> 💡 
[Resource groups](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources) are logical containers where you can deploy and manage Azure resources like virtual machines, web apps, databases, and storage accounts. Similar to the folder system inside your laptop/personal computer, it helps organize related Azure resources  that work together to support a specific application or service.
>
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create%20a%20resource%20group%20part%201.png" alt="Part 1" style="width: 30%; max-width: 400px; height: auto;">
  <img src="images/create%20a%20resource%20group%20part%202.png" alt="Part 2" style="width: 30%; max-width: 400px; height: auto;">
  <img src="images/create%20a%20resource%20group%20part%203.png" alt="Part 3" style="width: 30%; max-width: 400px; height: auto;">
</div>

#### 3. Create a Data Lake Storage Gen 2 Account
> 💡
[Data Lake Storage Gen 2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction) is a set of capabilities dedicated to big data analytics built on top of [Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-overview). It provides file system semantics, file-level security, and scale on top of Azure Blob Storage’s low-cost, tiered storage, with high availability/disaster recovery capabilities.
*A good analogy is to think of Blob Storage as a pile of books, whereas Data Lake Storage Gen 2 is putting that pile into a library, giving order/hierarchy to the unstructured pile of data files.*
>
Configuration Settings:
1. **Region**: ‘(US) East US 2’
2. **Performance**: Standard (for general-purpose storage)
3. **Redundancy**: Locally redundant storage (LRS) is sufficient for this homework
4. **Advanced**: Enable "Hierarchical namespace", which enables Data Lake Storage Gen 2 features on top of your Blob Storage.
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create storage account step 1.png" alt="Part 1" style="width: 30%; max-width: 400px; height: auto;">
  <img src="images/create storage account step 2.png" alt="Part 2" style="width: 30%; max-width: 400px; height: auto;">
</div>
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create storage account step 3.png" alt="Part 3" style="width: 30%; max-width: 400px; height: auto;">
  <img src="images/create storage account step 4.png" alt="Part 3" style="width: 30%; max-width: 400px; height: auto;">
</div>

[Data redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy) is the practice of storing multiple copies of the same data in different locations or systems. While having multiple copies of data might seem inefficient, it ensures that the data remains available and reliable across different regions and in the case of database failure.
<u>Designing Data-Intensive Applications</u> by Martin Kleppmann describes the reasoning behind data redundancy well:
*"Replication is used to keep a copy of the same data on multiple machines, which can serve several purposes: to increase **availability** (allowing the system to continue working even if some parts of it are down), to increase **read throughput** (by load balancing reads across replicas), and to **reduce latency** (by keeping data geographically close to users)…”*
In the context of our Azure use case,  we will be using the cheapest redundancy, **LRS**, which maintains three synchronous copies within a single data center. It does not replicate across other data centers/regions, and is the most at-risk in terms of data unavailability events.
>
#### 4. Create a Blob Storage Container within your Storage Account
This can be done by navigating to the storage browser within the Azure Blob Storage sidebar menu, and creating a container. If hierarchal namespace (Data Lake V2 Feature) wasn’t enabled, you would have noticed the ability to create a flat container, but not a container stored within a container (nested containers).
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create storage account step 4.png" alt="Part 3" style="width: 50%; height: auto;">
</div>

#### 5. Create a Azure Data Factory within your Resource Group
> 💡
Azure Data Factory(ADF) is a cloud-based data integration service that is designed to orchestrate and automate data movement and transformation across various data sources and destinations. It is cost effective (pay as you use) and scalable for enterprise data needs.
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create an adf.png" alt="Part 3" style="width: 50%; height: auto;">
</div>

#### 6. (Optional) Create and Connect a Git Repository
[Github](https://learn.microsoft.com/en-us/azure/data-factory/connector-github?tabs=data-factory) can be connected directly with your ADF environment to automate the build, test, and deployment process of pipelines. For homework purposes, this step is not necessary, but is an option you can consider for your future projects.
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create github connection to adf.png" style="width: 90%; height: auto;">
</div>

#### 7. Sign Up for OpenWeatherMap Free Access
We will be pulling from the OpenWeatherMap API for both stream data and batch processed data. This API provides access to current weather data, forecasts, and historical weather data for any location worldwide. 
1. **Create an OpenWeatherMap Account**:
    - Visit the [OpenWeatherMap for Education website](https://openweathermap.org/our-initiatives) and follow the steps to sign up for an account.
    - Provide the necessary information to verify your student status and gain free access to their services.
<div style="display: flex; justify-content: center; align-items: center; gap: 10px; flex-wrap: wrap;">
  <img src="images/create an open weather account.png" style="width: 80%; height: auto;">
</div>

#### 8. Grant Instructors Access to Resource Group
In order for your assignments to be graded, the instructors require access to view your resource group containing your assignment work. Below is a series of screenshots detailing how to properly grant instructors access.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\grant instructors part 1.png" alt="Grant Instructors Part 1" style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >Within the resource group menu, there is a submenu called Access Control, where you can add/remove role assignments to the overall resource.</i>
  <img src="images\grant instructors part 2.png" alt="Grant Instructors Part 1" style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >Amongst the roles you can assign, we want to assign instructors “Contributor” over the entire resource group. This allows instructors to view any resource created within the resource group.</i>
  <img src="images\grant instructors part 3.png" alt="Grant Instructors Part 3" style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >We then want to assign access to the relevant instructors.</i>
<img src="images\grant instructors part 4.png" alt="Grant Instructors Part 3" style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >We then ensure that the assignment type is both “Active” and “Permanent” </i>
</div>   

## Part 2: Connecting to the API via the ADF
**Objective:**
Ingest weekly historical weather and air pollution data from the OpenWeather APIs into Azure Data Lake Storage using Azure Data Factory. We want at least a **year’s worth of this weekly data**, and will have to model our pipelines to account for the API call restrictions. This involves using the API key within ADF, ADF linked services, building & running pipelines, monitoring the data ingestion process, and pushing configurations to a GitHub repository. 

*Before we start, we need to first learn about Data Lakehouse, as well as the Medallion Lakehouse Architecture.*

> 💡 
A [Data Lakehouse](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/) is a data management system that combines the best features of data lakes (scalability, flexibility) and data warehouses (structure, performance) all under a single architecture. 
>

> 💡
[Medallion Lakehouse Architecture](https://learn.microsoft.com/en-us/azure/databricks/lakehouse/medallion) is a data management and analytics architecture pattern used in modern Data Lakehouse environments. This architecture is designed to organize and manage data efficiently as it flows through different stages, typically referred to as Bronze, Silver, and Gold layers. Refer to the documentation on the specific differences of the layers. We will be using this to organize our data as we ingest and process it.
>

> 💡 
Originally, the various assignments utilized the Azure Key Vault service to securely contain API keys. Having unencrypted API keys within pipelines/code is a security issue, but for our homework purposes we want to avoid complicating processes. Identity management and permission sets will be discussed in class, but not required for the homework.
>
#### 1. Create an ADF Pipeline for Air Pollution Data Ingestion
We have to create pipelines to collect the historical data into our ADLS storage. The pipeline must ingest the following historical weather & air pollution data:
- Location: Boston *(this can be done by identifying the longitude and latitude coordinates of the Boston area)*
- Frequency: Hourly
- Time Frame: Data from approximately **one year ago to yesterday**, ensuring coverage for roughly 11 months.

#### (Optional) Testing the API with Postman
Before we schedule data factory activities within our data factory, we can first use Postman to test what our API calls should return. After registering a free account on Postman, you are able to make calls to better understand the API output.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\postman http.png"  style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >We first want to create an HTTP request connecting to the OpenWeather API (<a>https://openweathermap.org/history</a>).</i> 
  </div>
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\postman api.png"  style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >We then specify the historical weather API (https://api.openweathermap.org/data/2.5/weather). We then specify the above parameters for the API call. Ensure that the start, end unix timestamps only range for around a week long.</i> 
  </div>
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\postman output.png"  style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >You can then explore the output if correctly called.</i> 
  </div>
  
For better organization and maintenance, we want to create **two separate pipelines** for weather data and air pollution data ingestion. We are going to start with the *Air Pollution Pipeline* as its simpler and more straightforward.
##### Creating a Copy Data Activity 
We will create a [Copy Data](https://learn.microsoft.com/en-us/azure/data-factory/quickstart-hello-world-copy-data-tool) activity in our azure data factory, which will allow us to move data from Point A (source) to Point B (sink). For air pollution, our source is the online API, and our sink is Azure blob storage within your created storage account.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\air pollution activity.png" style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" ></i>
  </div>

##### Linked Service Setup
After creating a new Activity in the pipeline orchestration menu, you can click “+New” in the Linked Service menu under “Source” and “Sink”.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\linked service +new.png"  style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" ></i>
  </div>
  
>💡[Linked services](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services?tabs=data-factory) refer to connections to external resources/services, enabling the platform to interact with those sources. The true power of linked services comes from their *reusability in different pipelines/dataflows.*
For example, if you are copying data from an Azure SQL Database to an Azure Blob Storage, linked services must be first defined for the SQL Database and for the Azure Blob Storage. After creating these services, if you need to reference those same datasets for different transformations/dataflows, you can just reference the created linked services instead of making the connections from scratch again.
We will be creating ADF linked services for all resources we will be using. 

>💡[**REST vs HTTP:**](https://learn.microsoft.com/en-us/azure/data-factory/connector-http?tabs=data-factory)
You have two choices when creating linked services connecting to the OpenWeather API:
[REST](https://learn.microsoft.com/en-us/azure/data-factory/connector-rest?tabs=data-factory) connector specifically support copying data from RESTful APIs, following the REST architectural principles. 
[HTTP](https://learn.microsoft.com/en-us/azure/data-factory/connector-http?tabs=data-factory) connector is generic to retrieve data from any HTTP endpoint, e.g. to download file. It’s requests are unstructured compared to REST, and requires specific mapping to adhere to API requests.
Before REST connector becomes available for an API, you may use the HTTP connector to copy data from RESTful APIs, which is supported but less functional compared to REST connectors. For the purpose of the homework, we will be using the **HTTP** connector. The only difference being that with a REST dataset, you wouldn’t initially specify the datatype whereas with HTTP you would (JSON).

>💡
[Anonymous authentication](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/security/authentication/anonymousauthentication) allows users to access resources or applications without providing any identity verification (e.g., username or password). It is typically used for public-facing applications or websites where user identity is not necessary for basic access.

##### Copy Data Activity Setup
1. We must first create a source dataset connection which either uses REST or HTTP. The base URL will be referencing the openweathermap api (http://api.openweathermap.org/ for air pollution, http://history.openweathermap.org/ for historical weather).
2. We then use a relative URL, which defines resource paths without including the full URL, simplifying code and configurations:
```
data/2.5/air_pollution/history?lat=@{dataset().lat}&lon=@{dataset().lon}&start=@{dataset().start}&end=@{dataset().end}&appid=@{dataset().appid}
```
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\openweather relative url.png"  style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" ></i>
  </div>
>💡 The above screenshot shows an example Historical Weather API call. You would need to modify the base & relative URL for the historical weather API connection to match this API call.

>💡
Notice the similarities between the API documentation and the Azure API call.
The `@{dataset()}` function in Azure Data Factory (ADF) is used within a dataset to access parameters defined in that dataset. This allows you to create dynamic datasets that can change based on the input parameters passed to them from the pipeline. 
`@{dataset().lat}`: Accesses the `lat` parameter value passed to the dataset.
`@{dataset().lon}`: Accesses the `lon` parameter value passed to the dataset.
`@{dataset().dataType}`: Accesses the `dataType` parameter value passed to the dataset.
...and so on for the rest of the parameters…
3. After defining the relative URL, we have to define the dataset parameters. Latitude and Longitude will be Boston specific, and the start and end times need to be UNIX timestamps ranging from current time to a year ago.
The appid parameter will be your API key, which we are pasting directly without encrpytion into the dataset parameters.
4. Lastly, we sink our data as a JSON format into our azure blob storage account. This should be organized according to the medallion architecture, and should be separate from the historical weather data we will ingest.
5. Trigger the pipeline.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\image.png"  style="max-width: 75%; height: auto;">
  <i style ="max-width: 50%; height: auto;" ></i>
  </div>

#### 2. Create an ADF Pipeline for Historical Weather Data Ingestion
Historical Weather has some different parameters and API call restrictions when compared to historical weather. We will be creating another pipeline to ingest historical weather data.
##### For Each Activity
We are going to start with setting up the mechanisms to make multiple calls to the historical weather API. Due to the API restrictions on historical weather (there were no restrictions on Air Pollution calls), we have a maximum amount of data we can get per call. Thus, our pipeline will have to make multiple API calls through [Copy Data](https://learn.microsoft.com/en-us/azure/data-factory/quickstart-hello-world-copy-data-tool) activities within a [ForEach](https://learn.microsoft.com/en-us/azure/data-factory/control-flow-for-each-activity) loop, where ea                                                                                                                                                                                                                                                                   ch call sources around a week's worth of data from the API and sinks it into Azure blob storage.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\foreach activity.png" alt="Grant Instructors Part 1" style="max-width: 50%; height: auto;">
  <i style ="max-width: 60%; height: auto;" ></i>
  </div>

To make **multiple API calls** within ADF, we need to utilize the [**ForEach**](https://learn.microsoft.com/en-us/azure/data-factory/control-flow-for-each-activity) activity flow, which will repeat the **‘Copy’** activity for the specified dates. This activity is used to iterate over a collection and executes specified activities in a loop. The loop implementation of this activity is similar to ForEach looping structure in programming languages. Ensure  the **Sequential** option is checked if you want the iterations to run one after another. If you want them to run in parallel, leave it unchecked.

The **Items**   property (within the ForEach loop) is used in Azure Data Factory or Azure Synapse Pipelines for iterating over a collection of values. It defines the list of values or objects that a loop will iterate over.
>💡
The main [difference](https://learn.microsoft.com/en-us/azure/data-factory/concepts-parameters-variables) is that pipeline parameters cannot be modified during a pipeline run, whereas pipeline variables are values that can be set and modified during a pipeline run via “set variable activity”. For the purpose of this assignment, you will need a variable/parameter to define the range in which the ForEach loop should run. 
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\parameter vs variable.png" alt="Grant Instructors Part 1" style="max-width: 50%; height: auto;">
  <i style ="max-width: 50%; height: auto;" >You can access Parameters of a pipeline by simply clicking an empty area in the pipeline window, and the menu would appear.</i>
  </div>

##### Historical Weather Linked Service Specifics
**Source**
Compared to the air pollution relative url, the linked service for historical weather will have another parameter, *datatype*. We want to ingest hourly data for each API call, so you will need to specify this in the dataset parameters.
```data/2.5/history/city?lat=@{dataset().lat}&lon=@{dataset().lon}&type=@{dataset().dataType}&start=@{dataset().start}&end=@{dataset().end}&appid=@{dataset().appid}```
Every time the Copy Data activity within the ForEach loop is ran, it makes a GET request to the OpenWeather API. We want to collect at least a week’s worth of data per API call, and get the past year’s (52 weeks) historical weather data starting from today. To do this, we need to not only set dynamic expressions for the “start” and “end” date parameters (which the API requires) to collect the right range of information, but also match the formatting to the API’s UNIX time formatting. **You are tasked with creating the dynamic expressions to backtrack from the current time to 52 weeks ago, collecting a week's worth of data each API call.** Here are some useful functions from the ADF expression language that you might find yourself using for the **start, end** dataset parameters.

- *addDays(startDate, daysToAdd, format?) -* adds (or subtracts) a specified number of days to a given date or timestamp. You can use this function to dynamically set date ranges
- *utcNow() -* returns the current date and time in UTC format. You will need to convert this into the OpenWeather API time format within your expression.
- *item()* - returns the current value in a loop within a ForEach activity in Azure Data Factory.
- *ticks()* - converts a date into the number of "ticks" since specified epoch time in .NET ticks (1 tick = 100 nanoseconds). You will need this function to format the UNIX expression into the OpenWeather format, which is *ticks now - ticks since '1970-01-01T00:00:00Z’* (UNIX Epoch time). Then you would still need to convert these *ticks* into UTC format (divide by 10000000)

**Sink**
Since we are using a ForEach loop, there is some nuance when it comes to writing to the sink. Here is a breakdown of some of the sink configurations and their effects:
- **Flatten Hierarchy**: All files from the source in each iteration are written directly to the sink folder without retaining their original directory structure. If multiple iterations write files with the same name, subsequent iterations might overwrite the earlier files unless unique naming is applied.
*Example usecase*: You process daily logs in a loop and write all logs for a month into a single destination folder for reporting.
- **Merge Files**: Data from the source files processed during each iteration is combined into a single file per iteration in the destination. Each iteration generates one merged file.
*Example usecase*: Each iteration processes data for a specific region, and you create a single merged file per region.
- **Preserve Hierarchy**: The source folder structure from each iteration is replicated at the destination. Each iteration preserves the hierarchy of the processed files and directories.
*Example usecase*: Iterations handle data for different years, and you want to retain year-based subfolders in the sink for organization.

We want separate files for historical weather that look similar to the below screenshot. Feel free to try to [dynamically name](https://www.youtube.com/watch?v=f_9LjNXWYSc&ab_channel=MitchellPearson) the files, but that is not a requirement.
<div style="display: flex; flex-direction: column; align-items: center; gap: 10px; flex-wrap: wrap; text-align: center;">
  <img src="images\sink.png" alt="Grant Instructors Part 1" style="max-width: 50%; height: auto;">
  <i style ="max-width: 75%; height: auto;" ></i>
  </div>



After triggering the pipeline, we can then view the output in our Azure blob storage account using the storage browser and navigating to the correct directory. That concludes Homework 1! 

Instructors will access your blob storage account and data factory to verify completion. Please ensure you adhere to an appopriate and consistent naming convention for pipelines, folders, services, etc.


