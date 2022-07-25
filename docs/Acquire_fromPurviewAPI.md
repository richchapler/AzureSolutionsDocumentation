## Acquire Data from Purview via REST API

Requirement statements might include:

* "We have a corporate-approved governance solution, but it doesn't have connectors for Azure resources"
* "We want to use Purview to collect the best possible source metadata from Azure resources"

### Step 1: Prepare Resources

This solution requires the following resources:

* [Application Registration](PrepareResources_ApplicationRegistration.md) with Purview [collection role assignments](PrepareResources_Purview_CollectionRoleAssignment.md) for **Collection admins**, **Data source admins**, and **Data curators**
* [Data Lake](PrepareResources_DataLake.md) (with [container](PrepareResources_DataLake_Container.md))
* [Purview](PrepareResources_Purview.md) with an already-scanned data source (example: https://docs.microsoft.com/en-us/azure/purview/register-scan-azure-sql-database?tabs=managed-identity)
* [Synapse](PrepareResources_Synapse.md)

### Step 2: Create Linked Service

*	Open **Synapse Studio** and click the **Manage** navigation icon
*	Click **Linked services** from the **External connections** grouping in the resulting navigation

    <img src="https://user-images.githubusercontent.com/44923999/180606347-670321a8-896f-41fe-afe6-0dfdb7d87d61.png" width="800" title="Snipped: July 23, 2022" />

*	Click **+ New**

    <img src="https://user-images.githubusercontent.com/44923999/180606430-36d57546-9a80-463a-9977-0d6875fa4d3a.png" width="800" title="Snipped: July 23, 2022" />

* Search for and select "REST" in the first page of the **New linked service** pop-out and then click **Continue**

    <img src="https://user-images.githubusercontent.com/44923999/180672477-d7d16dea-9a5a-4d6d-8099-4bacda77d3e3.png" width="800" title="Snipped: July 23, 2022" />

* Complete the resulting **New linked service…**” pop-out, including:

# RESUME HERE!

### Step 3: Create Pipeline

Complete the following steps:
* Navigate to **Synapse Studio**
* Click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token

This activity will make a REST API call to http://login.microsoftonline.com and get a bearer token.

Complete the following steps:
* Expand **General** in the **Activities** bar
* Drag-and-drop a **Web** component into the main window
* Complete the form on the **Settings** tab

  <img src="https://user-images.githubusercontent.com/44923999/179229885-810ac78b-b59c-4ce6-a2c5-6e12047011b7.png" width="800" title="Snipped: July 15, 2022" />

  Prompt | Entry
  ------ | ------
  **URL** | Modify and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
  **Method** | Select **POST**  
  **Headers** | Click **+ Add** and enter key-value pair: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | ...for Cost Management, modify and enter:<br>`grant_type=client_credentials&client_id={ClientId}&client_secret={ClientSecret}&resource=https://management.azure.com/`<br><br>...for Log Analytics, modify and enter:<br>`grant_type=client_credentials&client_id={ClientId}&client_secret={ClientSecret}&resource=https://api.loganalytics.io/`<br><br>...for Purview, modify and enter:<br>`grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://purview.azure.net`

* Click **Debug** and monitor to confirm success

#### Activity 2: Get Data

This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

Complete the following steps:
* Expand **Move & Transform** in the **Activities** bar
*	Drag-and-drop a **Copy data** component into the activity window
*	Create a dependency from the **Get Token** component to the **Copy data** component
*	Complete the form on the **Source** tab

    <img src="https://user-images.githubusercontent.com/44923999/179236666-66456de7-73f3-4867-967e-c04289bff466.png" width="800" title="Snipped: July 15, 2022" />

    Prompt | Entry
    ------ | ------
    **Source Dataset** | Select your REST dataset
    **Request Method** | ...for Cost Management, select **POST**<br>...for Log Analytics,  select **POST**<br>...for Purview, select **GET**
    **Request Body** | ...for Cost Management, enter:<br>
`{
  "type": "Usage",
  "timeframe": "TheLastMonth",
  "dataset": {
    "granularity": "None",
    "aggregation": {
      "totalCost": {
        "name": "PreTaxCost",
        "function": "Sum"
      }
    },
    "grouping": [
      {
        "type": "Dimension",
        "name": "ResourceGroup"
      }
    ]
  }
}`
<br><br>...for Log Analytics, enter:<br>`{ "query": "Sample_CL | take 10" }`
  
  
  and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
