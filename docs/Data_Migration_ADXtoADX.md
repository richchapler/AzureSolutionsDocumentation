## Data Migration... Data Explorer 1 to Data Explorer 2

![image](https://user-images.githubusercontent.com/44923999/192547308-676da706-ba85-49cc-9c6e-d8bbe997fa62.png)

This use case considers requirement statements like:
* "We need to capture and analyze cost data from many subscriptions"
* "Our subscriptions have more than 1,000 resources and are hitting the Cost Management API's per-request limitation"
* "We want to pull historical data... 730 days {i.e., 2 years}"

<br>The solution described in this documentation will:
* Leverage Logic Apps' nested iteration capability with input parameters for Subscriptions and Start / End Dates 
* Request data from the Cost Management API at the Resource Group level rather than Subscription level

_Note: If you have Resource Groups with more than 1,000 resources, you might have to choose a more granular scope or other strategy_

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**Application Registration**](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at Subscription or Resource Group)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Logic App**](Infrastructure_LogicApp.md)

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


  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
