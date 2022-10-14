## Data Migration... Data Explorer to Data Explorer

![image](https://user-images.githubusercontent.com/44923999/195881302-076c87c3-3bd0-4495-ac0a-f111730bf670.png)

This use case considers requirement statements like:
* "We have instantiated a new instance of Data Explorer and need to migrate ~1.5 billion records from the old instance"
* "We have tried `set-or-append async`, but we are hitting Data Explorer's one-hour timeout maximum"
* "To overcome the timeout, we want to chunk and iteratively copy our data by date"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* Data Explorer, [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md) ... 2 instances
* [**Synapse**](Infrastructure_Synapse.md)

#### Prepare Data Explorer

* Navigate to Data Explorer Database, then Query in the Data grouping of the left-hand navigation
* Run the following KQL:

  ```
  .create table CostManagement (
      PreTaxCost: decimal
      , UsageDate: int
      , ResourceGroupName: string
      , ResourceType: string
      , ResourceId: string
      , ResourceLocation: string
      , MeterCategory: string
      , MeterSubCategory: string
      , Meter: string
      , ServiceName: string
      , PartNumber: string
      , PricingModel: string
      , ChargeType: string
      , ReservationName: string
      , Frequency: string
      , Currency: string
      )
  ```

### Step 2: Prepare Workflow
In this step, we will create a workflow, initialize variables, and add parameters.

### Reference
> https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
