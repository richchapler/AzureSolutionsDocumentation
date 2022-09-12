## Data Acquisition... from Cost Management API, using Iteration

![image](https://user-images.githubusercontent.com/44923999/188199195-34c228d5-37e8-4c06-8d7d-88b0e8d2a3ec.png)

_Note: These instructions build on the following documentation:_
* _[Data Acquisition... from Cost Management, via REST API](Data_Acquisition_fromCostManagementAPI.md)_
* _[Deployment of Synapse using Parameterized Linked Services](Deployment_Synapse_ParameterizedLinkedServices.md)_

This use case considers requirement statements like:
* "Our subscription has more than 1,000 resources... so, when we try to use the documented solution for pulling data from the Cost Management API, we hit the limitation"
* "We want to pull historical data... at least 730 days {i.e., 2 years}"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [Synapse](Infrastructure_Synapse.md) with a Data Explorer [Integration Dataset](Infrastructure_Synapse_Dataset.md) and Data Explorer, "**AllDatabasesAdmin**" permissions for the Synapse, System-Assigned Managed Identity
