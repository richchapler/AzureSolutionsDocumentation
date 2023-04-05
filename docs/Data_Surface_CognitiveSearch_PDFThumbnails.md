# Data Surface: Cognitive Search, Multi-Source Index

<img src="https://user-images.githubusercontent.com/44923999/226446292-5131f0ab-9d2c-45aa-80f2-20ddd0224458.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have structured data {e.g., SQL Database} and unstructured data {e.g., CAD drawings} that want to search"
* "We want to have a single search experience, not one for each source"
* "We are eager to embrace Azure OpenAI when it becomes available"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search) (aka "Search Service")
  * Enable **System-Assigned Managed Identity**
  * Use a Region that supports Cognitive Search {e.g., West}

* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)

* [**SQL**](https://learn.microsoft.com/en-us/azure/azure-sql) Server and Database
  * Include sample data
  * Enable "**Allow Azure services and resources to access this server**"
  * Grant Role Assignment "Reader" role for Search Service, System-Assigned Managed Identity
  * Grant access to Cognitive Search using the following T-SQL:
    ```
    CREATE USER [rchaplerss] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [rchaplerss];
    ```
    
* [**Storage Account**](Infrastructure_StorageAccount.md)
  * Create container named "drawings" (with sample files)
  * Grant Role Assignment "Storage Blob Data Reader" role for Search Service, System-Assigned Managed Identity

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Cognitive Search + SQL Database
* Exercise 2: Cognitive Search + Blob Storage
* Exercise 3: Manually Create Multi-Source Index

-----

## Exercise 1: Cognitive Search + SQL Database
In this exercise, we will create and learn about Cognitive Search index functionality for SQL Server.

### Step 1: Import Data
Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/226375829-57106809-9582-46b5-ba64-638d3348e36b.png" width="800" title="Snipped: March 20, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Export data to storage](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/export-data-to-storage)
