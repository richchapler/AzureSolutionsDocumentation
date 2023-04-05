# Data Surface: Cognitive Search, Thumbnail Generation

<img src="https://user-images.githubusercontent.com/44923999/226446292-5131f0ab-9d2c-45aa-80f2-20ddd0224458.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have PDF files in Azure Storage that we want to surface with Cognitive Search"
* "We want our demonstration app to include thumbnails for the images"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search) (aka "Search Service")
  * Enable **System-Assigned Managed Identity**
  * Use a Region that supports Cognitive Search {e.g., West}

* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)

* Logic Apps

* [**Storage Account**](Infrastructure_StorageAccount.md)
  * Create container named "drawings" (with sample files)
  * Grant Role Assignment "Storage Blob Data Reader" role for Search Service, System-Assigned Managed Identity

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Generate Thumbnail Images
* Exercise 2: Bake Thumbnails into Cognitive Search Index and Demonstration App

-----

## Exercise 1: Generate Thumbnail Images
In this exercise, we will generate thumbnail images with Logic Apps and Cognitive Services.

### Step 1: Lorem Ipsum
Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/226375829-57106809-9582-46b5-ba64-638d3348e36b.png" width="800" title="Snipped: March 20, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Export data to storage](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/export-data-to-storage)
