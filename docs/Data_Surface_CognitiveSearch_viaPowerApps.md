# Surface Data: Cognitive Search (via Power Apps)
_Note: Cognitive Search is also known as "Search Services" and "Azure Search"_

<img src="https://user-images.githubusercontent.com/44923999/230908250-3724404b-8aff-4ce5-9e2d-315db10c27eb.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to share our secured Cognitive Search index with our community of business users"
* "Since the Demo App only supports QueryKey-based security, we need an alternate solution"

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

### Step 3: Secure Index

The default Demo App {i.e., **AzSearch.html**} includes the index "**queryKey**" directly in the auto-generated HTML:

```
var automagic = new AzSearch.Automagic({ index: "{INDEX_NAME}", queryKey: "{QUERY_KEY}", service: "{SEARCH_SERVICE_NAME}", dnsSuffix:"search.windows.net" });
```

We cannot use the Demo App to surface Cognitive Search index and data because:
* Any user with AzSearch.html (unmodified) will be able to access search results
* The Demo App only works with queryKey (not RBAC)

We must do two things to switch to use of Role-Based Access Control {i.e., Azure Active Directory} for index security:
1. Modify API Access Control
2. Programmatically set index permissions

#### Step 3a: Modify API Access Control

Navigate to Cognitive Search, and then "**Keys**" in the "**Settings**" grouping of the left-hand navigation.

<img src="https://user-images.githubusercontent.com/44923999/230929406-ae8bc92b-109b-48ff-9932-40202c027840.png" width="800" title="Snipped: April 10, 2023" />

Under the "**API access control**" header, click to select the "**Role-based access control**" radio button.


-----
-----


### Step 3: Programmatically set index permissions

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

<img src="https://user-images.githubusercontent.com/44923999/230094993-e818f45d-85b3-4e6b-ab6e-c106aeb9de25.png" width="800" title="Snipped: April 5, 2023" />

Modify, copy / paste, and then run the following command:

```
PowerShell
New-AzRoleAssignment -ObjectId {OBJECT_ID}
  -RoleDefinitionName "Search Index Data Reader"
  -Scope "/subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Search/searchServices/{SERVICE_NAME}/indexes/stormevents-index"
```

You can expect a result like:

```
RoleAssignmentName : 89f76855-6dbc-470b-aae2-177e7a203c27
RoleAssignmentId   : /subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Search/searchServices/{SERVICE_NAME}/indexes/stormevents-index/providers/Microsoft.Authorization/roleAssignments/89f76855-6dbc-470b-aae2-177e7a203c27
Scope              : /subscriptions/{SUBSCRIPTION_ID}/resourceGroups/{RESOURCE_GROUP}/providers/Microsoft.Search/searchServices/{SERVICE_NAME}/indexes/stormevents-index
DisplayName        : {USER_NAME}
SignInName         : {USER_SIGNIN}
RoleDefinitionName : Search Index Data Reader
RoleDefinitionId   : 1407120a-92aa-4202-b7e9-c0e197c71c8f
ObjectId           : {OBJECT_ID}
ObjectType         : User
CanDelegate        : False
Description        : 
ConditionVersion   : 
Condition          : 
```

#### Confirm Success

Navigate to the User in Azure Active Directory.

<img src="https://user-images.githubusercontent.com/44923999/230096444-63f2c93d-5b7e-4c8a-8aa6-32141c1be662.png" width="800" title="Snipped: April 5, 2023" />

Click "**Azure role assignments**" in the left-hand navigation.

<img src="https://user-images.githubusercontent.com/44923999/230096988-81b6e89a-8715-4719-be15-5e0c4e31461b.png" width="800" title="Snipped: April 5, 2023" />

Confirm success using Search Explorer.

<img src="https://user-images.githubusercontent.com/44923999/230675015-6164dc3d-1f25-49b8-9433-8eceaa89f54c.png" width="800" title="Snipped: April 7, 2023" />

_Note: You will not be able to confirm index security settings using the Demo App (which uses only queryKey). You will have to create an app that can leverage the Cognitive Search API in order to use RBAC._

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
