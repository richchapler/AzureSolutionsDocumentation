## Data Acquisition... from Cost Management API, using Iteration

![image](https://user-images.githubusercontent.com/44923999/188199195-34c228d5-37e8-4c06-8d7d-88b0e8d2a3ec.png)

_Note: These instructions build on the following documentation:_
* _[Data Acquisition... from Cost Management, via REST API](Data_Acquisition_fromCostManagementAPI.md)_
* _[Deployment of Synapse using Parameterized Linked Services](Deployment_Synapse_ParameterizedLinkedServices.md)_

This use case considers requirement statements like:
* "Our ingestion is hitting the Cost Management API 1,000 resource limitation... we need to change scope from subscription to resource group"
* "We want to pull historical data... at least 730 days {i.e., 2 years}"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [Synapse](Infrastructure_Synapse.md) with a Data Explorer [Integration Dataset](Infrastructure_Synapse_Dataset.md) and Data Explorer, "**AllDatabasesAdmin**" permissions for the Synapse, System-Assigned Managed Identity








The scope associated with query and export operations. This includes 
'/subscriptions/{subscriptionId}/' for subscription scope, 
'/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}' for resourceGroup scope, 
'/providers/Microsoft.Billing/billingAccounts/{billingAccountId}' for Billing Account scope and '/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/departments/{departmentId}' for Department scope, '/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/enrollmentAccounts/{enrollmentAccountId}' for EnrollmentAccount scope, '/providers/Microsoft.Management/managementGroups/{managementGroupId}' for Management Group scope, '/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}' for billingProfile scope, '/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/invoiceSections/{invoiceSectionId}' for invoiceSection scope, and '/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/customers/{customerId}' specific for partners.

