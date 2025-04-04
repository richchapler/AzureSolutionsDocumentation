# Data Surface: Cognitive Search, PDF-based Thumbnails

<img src="https://user-images.githubusercontent.com/44923999/230174162-ec4fa1da-788e-40bc-a9b1-f4d1bed6e49c.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have PDF files in Azure Storage that we want to surface with Cognitive Search"
* "We want our demonstration app to include thumbnails for the PDFs"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search) (aka "Search Service")
  * Enable **System-Assigned Managed Identity**
  * Use a Region that supports Cognitive Search {e.g., West}

* Cognitive Services
  * [**Computer Vision**](https://learn.microsoft.com/en-us/azure/cognitive-services/computer-vision/overview) with System-Assigned Managed Identity

* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)

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

* [Generate Image Thumbnail using Azure Computer Vision and Logic Apps](https://www.youtube.com/watch?v=ubavaB2zYBg)
