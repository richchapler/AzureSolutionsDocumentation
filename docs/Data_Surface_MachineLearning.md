# Surface Data with Machine Learning

![image](https://user-images.githubusercontent.com/44923999/215880260-d581ec97-3b29-4490-94ae-0f473fcc3612.png)

## Use Case
This solution considers the following requirements:

* "We want to surface data from Data Explorer in Machine Learning"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/)
  * [**Cluster**](Infrastructure_DataExplorer_Cluster.md)
  * [**Database**](Infrastructure_DataExplorer_Database.md) with **Viewer** permissions granted to Synapse managed identity
  * Three copies of [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard) {i.e., tables StormEvents1, StormEvents2, and StormEvents3}

-----

Steps
* Create Compute (should this be in Infrastructure?)
* Connect ADX

```
# Step 1: Install the package
!pip install azure-kusto-data
 
# Step 2: Import necessary modules
from azure.kusto.data import KustoClient, KustoConnectionStringBuilder
 
# Step 3: Build the connection string
connection_string = KustoConnectionStringBuilder.with_aad_device_authentication(https://rchaplerdec.kusto.windows.net).with_database("rchaplerded").build()
 
# Step 4: Create the Kusto client
client = KustoClient(connection_string)
 
# Step 5: Query data
query = "StormEvents | count"
results = client.execute("rchaplerded", query)
 
# Step 6: Print the results
print(results.primary_results[0])
```

## Exercise 1: Data Explorer, Export Data
In this exercise, we will create a Synapse pipeline that identifies and then iterates through a list of tables for export.

_Note: These instructions will not detail how to establish Synapse >> Data Explorer permissions, how to create a linked service, or dataset_

Navigate to Synapse Studio, Integrate and then click the **+** icon and then **Pipeline** in the resulting dropdown menu.

### Step 1: Add Activity: Lookup

<img src="https://user-images.githubusercontent.com/44923999/216063424-12f1909e-26de-4572-8812-e85609a4cf16.png" width="800" title="Snipped: February 1, 2023" />

Drag-and-drop a **Lookup** component from the **Activities** tree, **General** grouping.<br>
