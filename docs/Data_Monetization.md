# Data Monetization
_aka "Analytics as a Service", "AaaS"_

![image](https://user-images.githubusercontent.com/44923999/211827398-33d50c55-7fad-42df-999a-78a7a6b18026.png)

## Use Case
This solution considers the following requirements:

* "We have customer-specific data that we want to share with customers securely"
* "We want to monetize the analysis and data shared with customers"

## Proposed Solution
This solution will address requirements in three steps:

*	Exercise 1: Surface data via API (Function App GET customers, products, etc.) 
*	Exercise 2: Surface API via APIM (calls from customers will come through API for security, organization, throttling, “subscription model”, etc.) … http://api.blah.com/customer?id=7
*	Exercise 3: APIM Monetization? … with Stripe?
*	Exercise 4: Subscribe to API in APIM (persona: customer)
*	Exercise 5: Power BI template that uses API

## Required Infrastructure
This solution requires the following resources:

* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured to use [**Application Insights**](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) for monitoring
* [Visual Studio](https://visualstudio.microsoft.com/) with **Azure development** workload

## Reference

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-the-sdk

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-kusto-data

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/connection-strings/kusto
