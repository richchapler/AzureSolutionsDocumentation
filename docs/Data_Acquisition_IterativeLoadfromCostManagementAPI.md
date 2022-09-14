## Data Acquisition... Iterative Load from Cost Management API

![image](https://user-images.githubusercontent.com/44923999/190155800-820282af-0eb6-47ff-b392-ed1abf9783f9.png)

_Note: These instructions build on the following documentation:_
* _[Data Acquisition... from Cost Management, via REST API](Data_Acquisition_fromCostManagementAPI.md)_
* _[Deployment of Synapse using Parameterized Linked Services](Deployment_Synapse_ParameterizedLinkedServices.md)_

_The instructions below will only briefly cover any repetitive topics._

This use case considers requirement statements like:
* "Our subscriptions have more than 1,000 resources... so, they are hitting the Cost Management API's per-request limitation"
* "We want to pull more Cost Management API columns than before, but we haven't settled on a final set {i.e., capture should be dynamic}"
* "We need to change scope from '/subscriptions/{subscriptionId}/' to '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}'"
* "We want to pull historical data... 730 days {i.e., 2 years}"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* Event Hub
* Logic App

### Step 2: Lorem Ipsum

_Note: Scope ResourceGroup does not allow use of **BillingPeriod** and **ServiceTier** columns_
