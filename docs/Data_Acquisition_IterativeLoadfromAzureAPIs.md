## Data Acquisition... Iterative Load from Azure APIs

![image](https://user-images.githubusercontent.com/44923999/190155800-820282af-0eb6-47ff-b392-ed1abf9783f9.png)

This use case considers requirement statements like:
* "We need to capture and analyze cost data from many subscriptions"
* "Our subscriptions have more than 1,000 resources and are hitting the Cost Management API's per-request limitation"
* "We want to pull historical data... 730 days {i.e., 2 years}"
* "We want to pull more Cost Management API columns than before, but we haven't settled on a final set {i.e., capture should be dynamic}"

The proposed solution described in this documentation will:
* Leverage Logic Apps' nested iteration capability with input parameters for Subscriptions and Start / End Dates 
* Leverage Data Explorer's dynamic data type to provide for potential schema drift
* Request data from the Cost Management API at the Resource Group rather than Subscription level

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**Application Registration**](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group
* [**Logic App**](Infrastructure_LogicApp.md)

### Step 2: Lorem Ipsum

![image](https://user-images.githubusercontent.com/44923999/190196827-51bef9f0-22dd-44c9-90d1-15ca74d4c139.png)

* Navigate to your Logic App


_Note: Scope ResourceGroup does not allow use of **BillingPeriod** and **ServiceTier** columns_

scope Resource Group {i.e., '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}'}, rather than scope Subscription {i.e., '/subscriptions/{subscriptionId}/'}

(ISO-8601 formatted strings per Logic App requirements)
