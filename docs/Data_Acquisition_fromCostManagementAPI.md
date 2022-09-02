## Data Acquisition... from Cost Management, via REST API

![image](https://user-images.githubusercontent.com/44923999/188199195-34c228d5-37e8-4c06-8d7d-88b0e8d2a3ec.png)

This use case considers requirement statements like:
* "Power BI, Cost Management connector requires billing account permissions we cannot get"
* "We want to view costs across many subscriptions"
* "An understanding of resources and costs would really help us get a handle of inherited subscriptions"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [Synapse](Infrastructure_Synapse.md) with an [Integration Dataset](Infrastructure_Synapse_Dataset.md) for Data Explorer

### Step 2: Create Destination Table
In this step, we will create the Data Explorer table which will serve as destination for our Synapse Pipeline

* Navigate to Data Explorer and then select Query from the navigation

  <img src="https://user-images.githubusercontent.com/44923999/188202710-018bd814-90f5-4283-a70c-29904f47512e.png" width="800" title="Snipped: September 2, 2022" />

* **Run** the following KQL:

  ```
  .create table CostManagement (
      Currency: string
      , PreTaxCost: real
      , ResourceType: string
      , UsageDate: string
      )
  ```
 
_Note: This initial version of the table includes only a few of the columns that might be included as you evolve your query logic_

### Step 3: Create Linked Service
In this step, we will create the Linked Service we will use to get our Bearer Token.

* Open **Synapse Studio** and click the **Manage** navigation icon
* Click "**Linked services**" from the "**External connections**" grouping in the resulting navigation
* Click "**+ New**"

  <img src="https://user-images.githubusercontent.com/44923999/188169292-9886c049-e9cb-4d11-b6d9-a7ffe7e285d9.png" width="800" title="Snipped: September 2, 2022" />

* Search for and select "REST" on the "**New linked service**" pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/188169910-55df4aad-6c9f-434a-8fe9-f0b321aee9af.png" width="800" title="Snipped: September 2, 2022" />

* Complete the resulting "**New linked service**" pop-out, including:

  Prompt | Entry
  ------ | ------
  **Base URL** | Modify and enter:<br>`https://management.azure.com/subscriptions/{Subscription Id}/providers/Microsoft.CostManagement/query?api-version=2021-10-01`
  **Authentication Type** | Select **Anonymous**  

* Click "**Test connection**" to confirm successful connection and then click **Create**

### Step 4: Create Dataset
In this step, we will create the Dataset we will use to get our Bearer Token.

* Open **Synapse Studio** and click the **Data** navigation icon
* Click **+** and then select "**Integration dataset**" from the **Linked** grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/188195944-563f1dae-23eb-487d-b2d0-1747a6058ee1.png" width="800" title="Snipped: September 2, 2022" />

* Search for and select **REST** on the "**New linked service**" pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/188196140-3ac10408-7991-4702-8d64-568f74b281c5.png" width="800" title="Snipped: September 2, 2022" />

* Complete the "**Set properties**" pop-out form and then click **OK**
* Click "**Test connection**" to confirm successful connection
* Click "**Publish all**", review changes on the resulting "**Publish all**" pop-out and then click **Publish**

### Step 5: Create Pipeline
In this step, we will create a Pipeline and add Activities.

* Open "**Synapse Studio**" and click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token
This activity will make a REST API call and get a bearer token.

* Expand **General** in the **Activities** bar
* Drag-and-drop a **Web** component into the main window
* Complete the form on the **Settings** tab

  <img src="https://user-images.githubusercontent.com/44923999/188197200-3f5083fd-fcc3-4a92-bbfa-bde3e01e77dc.png" width="800" title="Snipped: September 2, 2022" />

  Prompt | Entry
  ------ | ------
  **URL** | Modify and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
  **Method** | Select **POST**  
  **Headers** | Click "**+ Add**" and enter key-value pair: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | Modify and enter:<br> `grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://api.loganalytics.io/`

* Click **Debug** and confirm success
* Click "**Publish all**", review changes on the resulting "**Publish all**" pop-out and then click **Publish**









#### Activity 2: Copy Data
This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

* Expand "**Move & Transform**" in the **Activities** bar
* Drag-and-drop a "**Copy data**" component into the activity window
* Create a dependency from the "**Get Token**" component to the "**Get Data**" component
* Enter values on the **Source** tab 

  <img src="https://user-images.githubusercontent.com/44923999/186216401-ec555ffc-190b-4ea2-9be8-d59b0b30e1f0.png" width="800" title="Snipped: September 2, 2022" />

  Prompt | Entry
  ------ | ------
  **Source dataset** | Select your REST dataset
  **Request method** | Select **POST**
  **Request body** | Modify and enter:<br>`{ "query": "Sample_CL" }`
  **Additional headers** | Click **+ Add** and enter key-value pairs:<br>`content-type` :: `application/json;charset=utf-8`<br>`Authorization` :: `@concat('Bearer ',activity('Get Token').output.access_token)`

* Select the appropriate dataset (and configuration) on the **Sink** tab 

  <img src="https://user-images.githubusercontent.com/44923999/186217944-78b9131f-846b-4ba7-9d21-10e18dde9fa7.png" width="800" title="Snipped: September 2, 2022" />

#### Confirm Success

* Click **Debug** and confirm success
* Browse to your Data Lake and download the resulting file.

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
