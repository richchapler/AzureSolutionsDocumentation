## Acquire Data from Purview via REST API

Requirement statements might include:

* "Our approved Data Governance solution (not Purview) lacks Azure connectors"
* "We want to use Purview to collect the best possible source metadata from Azure resources"

### Step 1: Prepare Resources

This solution requires the following resources:

* [Application Registration](Prepare_ApplicationRegistration.md) with Purview [collection role assignments](PrepareResources_Purview_CollectionRoleAssignment.md) for **Collection admins**, **Data source admins**, and **Data curators**
* [Data Lake](Prepare_DataLake.md) (with [container](PrepareResources_DataLake_Container.md))
* [Purview](PrepareResources_Purview.md) with an already-scanned data source (example: https://docs.microsoft.com/en-us/azure/purview/register-scan-azure-sql-database?tabs=managed-identity)
* [Synapse](PrepareResources_Synapse.md)

### Step 2: Create Linked Service

Complete the following steps:

* Open **Synapse Studio** and click the **Manage** navigation icon
* Click **Linked services** from the **External connections** grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/180606347-670321a8-896f-41fe-afe6-0dfdb7d87d61.png" width="800" title="Snipped: July 23, 2022" />

* Click **+ New**

  <img src="https://user-images.githubusercontent.com/44923999/180805015-db42aa59-bbef-4fa3-8651-7b55f2b4ea32.png" width="800" title="Snipped: July 25, 2022" />

* Search for and select "REST" on the **New linked service** pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/180804710-17c92274-44a9-4322-b2c3-bf03c5db0e26.png" width="800" title="Snipped: July 25, 2022" />

* Complete the resulting **New linked service** pop-out, including:

  Prompt | Entry
  ------ | ------
  **Base URL** | Modify and enter:<br>`https://{Purview Account Name}.purview.azure.com/catalog/api/search/query?api-version=2022-03-01-preview`  
  **Authentication Type** | Select **Anonymous**  

* Click **Test connection** to confirm successful connection and then click **Create**

### Step 3: Create Dataset

Complete the following steps:

* Open **Synapse Studio** and click the **Data** navigation icon
* Click **+** and then select **Integration dataset** from the **Linked** grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/180805485-9187f29f-0232-42ad-af43-c6fbc4ae8510.png" width="800" title="Snipped: July 25, 2022" />

* Search for and select **REST** on the **New linked service** pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/180805719-c13e9621-bdb3-4e46-88f5-c337c860a382.png" width="800" title="Snipped: July 25, 2022" />

* Complete the **Set properties** pop-out form and then click **OK**
* Click **Test connection** to confirm successful connection

### Step 4: Create Pipeline

Complete the following steps:
* Open **Synapse Studio** and click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token

This activity will make a REST API call to http://login.microsoftonline.com and get a bearer token.

Complete the following steps:
* Expand **General** in the **Activities** bar
* Drag-and-drop a **Web** component into the main window
* Complete the form on the **Settings** tab

  <img src="https://user-images.githubusercontent.com/44923999/180803926-1a18dab9-22d1-41c9-8896-13e3436d7772.png" width="800" title="Snipped: July 25, 2022" />

  Prompt | Entry
  ------ | ------
  **URL** | Modify and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
  **Method** | Select **POST**  
  **Headers** | Click **+ Add** and enter key-value pair: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | Modify and enter:<br>`grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://purview.azure.net`

* Click **Debug** and monitor to confirm success

#### Activity 2: Get Data

This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

Complete the following steps:
* Expand **Move & Transform** in the **Activities** bar
* Drag-and-drop a **Copy data** component into the activity window
* Create a dependency from the **Get Token** component to the **Get Data** component
* Enter values on the **Source** tab 

  <img src="https://user-images.githubusercontent.com/44923999/180803310-4399a334-7b54-4720-9766-4701bde373ba.png" width="800" title="Snipped: July 25, 2022" />

  Prompt | Entry
  ------ | ------
  **Source dataset** | Select your REST dataset
  **Request method** | Select **POST**
  **Request body** | Enter: `{ "limit": 10 }`
  **Additional headers** | Click **+ Add** and enter key-value pairs:<br>content-type: `application/json;charset=utf-8`<br>Authorization: `@concat('Bearer ',activity('Get Token').output.access_token)`

* Select the appropriate dataset (and configuration) on the **Sink** tab 

  <img src="https://user-images.githubusercontent.com/44923999/180802206-ac29e3f2-b0bd-4892-8c30-d78361e60cb9.png" width="800" title="Snipped: July 25, 2022" />

  _Note: for this exercise, I am using the default connection to Data Lake_

#### Confirm Success

Click **Debug** and monitor result to confirm successful progress.
Browse to your Data Lake and download the resulting file.
