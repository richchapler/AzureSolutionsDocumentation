# Surface Data with Cognitive Search
_Note: Cognitive Search is also known as Search Services and Azure Search_

![image](https://user-images.githubusercontent.com/44923999/214127109-5208bf6e-f1d3-40b0-bcbc-a302b5834ab8.png)

## Use Case
This solution considers the following requirements:

* "We have archived a significant amount of rich, historical data and want to help business users search this data asset"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)

## Proposed Solution
This solution will address requirements in three exercises:

*	Exercise 1: Import Data
*	Exercise 2: Create Search Index
*	Exercise 3: Prepare Interface

## Exercise 1: Import Data
In this exercise, we will import built-in sample data.

* Open the Azure Portal and navigate to your Search Service instance

  <img src="https://user-images.githubusercontent.com/44923999/214132098-21179866-e85c-43f8-8737-7bf5efd0ebef.png" width="800" title="Snipped: January 23, 2023" />

* Click "**Import Data**"

  <img src="https://user-images.githubusercontent.com/44923999/214140498-3b1a486c-57bd-407a-b249-60a93e54602e.png" width="800" title="Snipped: January 23, 2023" />

* On the resulting "**Import Data**" page, "**Connect to your data**" tab, select **Samples** from the "**Existing data source**" drop-down menu
* Select the "**realestate-us-sample**" row
* Click "**Next: Add cognitive skills (Optional)**" tab

## Reference

* [What's Azure Cognitive Search?](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
* [Quickstart: Create an Azure Cognitive Search index in the Azure portal](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal)
