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

### Step 1: Add Activity: Lookup

<img src="https://user-images.githubusercontent.com/44923999/216063424-12f1909e-26de-4572-8812-e85609a4cf16.png" width="800" title="Snipped: February 1, 2023" />

* Drag-and-drop a **Lookup** component from the **Activities** tree, **General** grouping
* On the **Settings** tab, select your Data Explorer dataset, uncheck "**First row only**" and then paste the following **Query**:

  `.show tables | project TableName`
  
  _Note: Data Explorer datasets must specifically reference a table, but we do not actually use the referenced table with our ".show tables..." query_

### Step 2: Add Activity: ForEach

<img src="https://user-images.githubusercontent.com/44923999/216064663-066d56f8-e775-43e4-933c-ab4a9b78905c.png" width="800" title="Snipped: February 1, 2023" />

* Drag-and-drop a **ForEach** component from the **Activities** tree, "**Iteration & conditionals**" grouping
* Complete the form on the **Settings** tab

  Prompt | Entry
  ------ | ------
  **Sequential** | Checked
  **Items** | Paste expression `@activity('Lookup').output.value`

### Step 3: Add Sub-Activity: Copy Data

<img src="https://user-images.githubusercontent.com/44923999/216067419-dc310846-709a-45b8-baee-c3a0e4a87ab9.png" width="800" title="Snipped: February 1, 2023" />

* Click the **+** button in the **Activities** area of the **ForEach** component
* Select "Azure Data Explorer Command" from the "Azure Data Explorer" grouping of the resulting drop-down menu
* Select your Data Explorer dataset on the **Connection** tab
* On the Command tab, update and then paste the following logic into the Command textbox:

  ```
  @concat('.export to csv ( "https://rchaplers.blob.core.windows.net/exports;STORAGEACCOUNT_ACCESSKEY" ) with ( includeHeaders = "all", namePrefix ="',item().TableName,'") <| ',item().TableName )
  ```

### Step 4: Confirm Success

<img src="https://user-images.githubusercontent.com/44923999/216067715-573fef58-9b80-43bb-a301-fe710d0719cc.png" width="800" title="Snipped: February 1, 2023" />

* Click **Debug** and confirm successful pipeline processing

<img src="https://user-images.githubusercontent.com/44923999/216069273-a28c64dd-cd9c-42dc-820e-8a21428e84fa.png" width="800" title="Snipped: February 1, 2023" />

* Navigate to the **exports** container in your Storage Account instance and confirm proper export

## Congratulations
You have successfully completed this exercise.
