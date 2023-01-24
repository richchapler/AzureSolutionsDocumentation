# Surface Data with Cognitive Search
_Note: Cognitive Search is also known as "Search Services" and "Azure Search"_

![image](https://user-images.githubusercontent.com/44923999/214127109-5208bf6e-f1d3-40b0-bcbc-a302b5834ab8.png)

## Use Case
This solution considers the following requirements:

* "We have archived a significant amount of rich, historical data and want to help business users search this data asset"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)
* [**Storage Account**](Infrastructure_StorageAccount.md)

## Proposed Solution
This solution will address requirements in three exercises:

*	Exercise 1: Import Data
*	Exercise 2: Create Search Index
*	Exercise 3: Prepare Interface

## Exercise 1: Import Sample Data
In this exercise, we will import built-in sample data.

<img src="https://user-images.githubusercontent.com/44923999/214132098-21179866-e85c-43f8-8737-7bf5efd0ebef.png" width="800" title="Snipped: January 23, 2023" />

Navigate to your instance and then click "**Import Data**"

### Step 1: Connect Data
On the "**Connect to your data**" tab, select **Samples** from the "**Existing data source**" drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/214140498-3b1a486c-57bd-407a-b249-60a93e54602e.png" width="800" title="Snipped: January 23, 2023" />

* Select the "**realestate-us-sample**" row
* Click "**Next: Add cognitive skills (Optional)**"

### Step 2: Add Cognitive Skills
On the "**Add cognitive skills (Optional)**" tab, expand "**Attach Cognitive Services**"

#### Step 2a: Attach Cognitive Services
<img src="https://user-images.githubusercontent.com/44923999/214144851-f5be1f08-374d-4b8d-8fed-b56e6a7980a7.png" width="800" title="Snipped: January 23, 2023" />

* Select your instance of Cognitive Services
* Collapse "**Attach Cognitive Services**" and then expand "**Add enrichments**"

#### Step 2b: Add Enrichments
<img src="https://user-images.githubusercontent.com/44923999/214149017-338ba6c7-3281-40ee-92ca-e8b099b5430f.png" width="800" title="Snipped: January 23, 2023" />

* Add appropriate enrichments<br>
  _Note: Even familiar data sources can be hard to configure the first time through {e.g., "will the Description field have "people names"?}. I lean towards experimentation in this situation... we can always re-configure once the initial Import Data exercise is complete._

* Collapse "**Add enrichments**" and then expand "**Save enrichments to a knowledge store**"

#### Step 2c: Save Enrichments
<img src="https://user-images.githubusercontent.com/44923999/214153287-beed5d62-200a-48ec-b852-578f38621af6.png" width="800" title="Snipped: January 23, 2023" />

* Configure knowledge store options and then click "**Next: Customize target index**"

### Step 3: Customize Index

<img src="https://user-images.githubusercontent.com/44923999/214154309-fea824d2-d3bf-435b-bf85-500dd28be462.png" width="800" title="Snipped: January 23, 2023" />

* On the "**Customize target index**" tab, configure index options
  _Note: Once again, experimentation and rework is the path I take when reviewing the many options._

* Click "**Next: Create an indexer**"

  <img src="https://user-images.githubusercontent.com/44923999/214154948-102c8185-cbdc-44f9-b21f-12bb9a22a43b.png" width="800" title="Snipped: January 23, 2023" />

* On the "**Create an indexer**" tab, configure and then click **Submit**

### Step 5: Confirm Success

Lorem Ipsum

-----------------------

## Reference

* [What's Azure Cognitive Search?](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
* [Quickstart: Create an Azure Cognitive Search index in the Azure portal](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal)
