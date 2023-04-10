# Surface Data: Cognitive Search (via Power Apps)
_Note: Cognitive Search is also known as "Search Services" and "Azure Search"_

<img src="https://user-images.githubusercontent.com/44923999/230908250-3724404b-8aff-4ce5-9e2d-315db10c27eb.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to share our secured Cognitive Search index with our communit of business users"
* "The Demo App only supports QueryKey-based security"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Power Apps**](https://powerapps.microsoft.com/en-us/)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Prepare Index
* Exercise 2: Lorem

_Note: Because I have covered creation and securing of an index in other documentation, you can expect a brief version in the first two exercises._

-----

## Exercise 1: Prepare Index
In this exercise, we will create an index using sample data, create a Demo App, and then configure index security.

### Step 1: Create Index

Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/230911383-a7135ef7-65e2-45e7-bb4f-38f5addaa8e9.png" width="800" title="Snipped: April 10, 2023" />

On the "**Connect your data**" tab, select "**Samples**" using the "**Data Source**" drop-down menu.<br>
Select "**realestate-us-sample**" from the resulting data source list and then, click "**Next: Add cognitive skills (Optional)**".

<img src="https://user-images.githubusercontent.com/44923999/230911764-3ed44b43-4673-416b-9882-87b809e957cd.png" width="800" title="Snipped: April 10, 2023" />

On the "**Add cognitive skills (Optional)**" tab, simply click "**Skip to: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/230911991-df3cabf9-8f28-48f1-95b7-41cb9c1458d5.png" width="800" title="Snipped: April 10, 2023" />

On the "**Customize target index**" tab, review default configuration, modify if desired, and then click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/230913029-a23d514c-5fb8-40e3-937c-7d1757b031a0.png" width="800" title="Snipped: April 10, 2023" />

On the "**Create an indexer**" tab, review default configuration, modify if desired, and then click "**Submit**".

<img src="https://user-images.githubusercontent.com/44923999/230914454-a2730070-4811-40f1-8005-4530a30bf561.png" width="800" title="Snipped: April 10, 2023" />

Allow time for processing and then click into the new "**realestate-us-sample-index**" index.

<img src="https://user-images.githubusercontent.com/44923999/230914830-9de5c8fe-8475-44d1-9b05-47ad258e6325.png" width="800" title="Snipped: April 10, 2023" />

On the "**Search explorer**" tab, click "**Search**" and review results.

### Step 2: Create Demo App

Click "**Create Demo App**".

<img src="https://user-images.githubusercontent.com/44923999/230915524-487c5670-165b-4487-89e5-0040c41210cf.png" width="800" title="Snipped: April 10, 2023" />

On the resulting "**Create Demo App**" pop-up, click "**Enable CORS and continue**".

<img src="https://user-images.githubusercontent.com/44923999/230916396-cdbc6bbf-5cd2-4a90-84ff-e6f1e307d704.png" width="800" title="Snipped: April 10, 2023" />

On the "**Individual result**" tab, review default configuration, modify if desired, and then click "**Create Demo App**".
_Note: Review "Sidebar" and "Suggestions" tab, as desired._

<img src="https://user-images.githubusercontent.com/44923999/230918267-a9df50e2-21f9-40ae-8d82-22293a2db82c.png" width="800" title="Snipped: April 10, 2023" />

On the resulting "**Your demo app is ready**" pop-up, click "**Download**".<br>
Use Windows Explorer to open your "**Downloads**" folder and then the new "**AzSearch.html**" file.

<img src="https://user-images.githubusercontent.com/44923999/230919150-61603545-44b7-41c9-8066-a1150bc9f455.png" width="800" title="Snipped: April 10, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Lorem
In this exercise, we will lorem.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Tutorial: Query a Cognitive Search index from Power Apps](https://learn.microsoft.com/en-us/azure/search/search-howto-powerapps)
