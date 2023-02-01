# Surface Data with Bulk Export

![image](https://user-images.githubusercontent.com/44923999/215880260-d581ec97-3b29-4490-94ae-0f473fcc3612.png)

## Use Case
This solution considers the following requirements:

* "We have hundreds of Data Explorer tables that we want to prepare for use with Cognitive Search"
* "Manual {i.e., one-by-one} data export will take an unreasonable amount of human effort"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/)
  * [**Cluster**](Infrastructure_DataExplorer_Cluster.md)
  * [**Database**](Infrastructure_DataExplorer_Database.md) with **Viewer** permissions granted to Synapse managed identity
  * Three copies of [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard) {i.e., tables StormEvents1, StormEvents2, and StormEvents3}
* [**Storage Account**](Infrastructure_StorageAccount.md) with container "exports" (and related SAS token)
* [**Synapse**](Infrastructure_Synapse.md)
  * [Firewall Rule](Infrastructure_Synapse_FirewallRules.md) allowing your Client IP
  * [Linked Service](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services?tabs=data-factory) for your Data Explorer instance
  * [Dataset](https://learn.microsoft.com/en-us/azure/data-factory/concepts-datasets-linked-services?tabs=data-factory) for your Data Explorer instance

-----

## Exercise 1: Data Explorer, Export Data
In this exercise, we will create a Synapse pipeline that identifies and then iterates through a list of tables for export.

_Note: These instructions will not detail how to establish Synapse >> Data Explorer permissions, how to create a linked service, or dataset_

Navigate to Synapse Studio, Integrate and then click the **+** icon and then **Pipeline** in the resulting dropdown menu.

### Add Activity: Lookup

<img src="https://user-images.githubusercontent.com/44923999/216063424-12f1909e-26de-4572-8812-e85609a4cf16.png" width="800" title="Snipped: February 1, 2023" />

* Drag-and-drop a **Lookup** component from the **Activities** tree, **General** grouping
* On the **Settings** tab, select your Data Explorer dataset, uncheck "**First row only**" and then paste the following **Query**:

  `.show tables | project TableName`
  
_Note: Data Explorer datasets must specifically reference a table, but we do not actually use the referenced table with our ".show tables..." query_







