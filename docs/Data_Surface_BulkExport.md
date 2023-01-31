# Surface Data with Bulk Export

![image](https://user-images.githubusercontent.com/44923999/215786929-46b22f51-5b64-459a-8af5-493d09000f80.png)

## Use Case
This solution considers the following requirements:

* "We have hundreds of Data Explorer tables that we want to ready for use with Cognitive Search"
* "Individual export will take much more human effort than we want to invest"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md) ... MAY JUST USE DATA EXPLORER HELP DATABASE
* [**Storage Account**](Infrastructure_StorageAccount.md) with container "exports" (and related SAS token)

## Proposed Solution
This solution will address requirements in N exercises:

* Exercise 1: Figure out what tables are in Data Explorer
* Exercise 2: Iterate through list of tables and export to Storage Account

-----

## Exercise 1: Data Exploer, Export Data
In this exercise, we will discuss two ways of preparing Data Explorer-based data for use by Cognitive Search:

* Option #1: One-Time Export (to Blob Storage)... for data that will not change
* Option #2: Continuous Export (to Blob Storage)... for data that will change over time

_Note: Continuous Export does not provide for data headers and that complicates the import of data into Cognitive Search_

### Step 1: Perform One-Time Export (Option #1)
In this step, we will: 1) load sample data and 2) run a KQL query to perform an export to Blob Storage.
![image](https://user-images.githubusercontent.com/44923999/215788562-676e145f-5baf-4843-a74f-7256b341f53e.png)
