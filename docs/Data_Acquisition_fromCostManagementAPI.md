## Data Acquisition... from Cost Management, via REST API

![image](https://user-images.githubusercontent.com/44923999/186208783-6c8db61d-01ea-4d5a-9a39-ac02458ec463.png)

This use case considers requirement statements like:
* "Power BI, Cost Management connector requires billing account permissions we cannot get"
* "We want to view costs across many subscriptions"
* "An understanding of resources and costs would really help us get a handle of inherited subscriptions"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md)
* [Data Explorer](Infrastructure_ApplicationRegistration.md)
* [Synapse](Infrastructure_Synapse.md) with an [Integration Dataset](Infrastructure_Synapse_Dataset.md) for Data Explorer

### Step 2: Create Linked Service
In this step, we will create the Linked Service we will use to get our Bearer Token.

Complete the following steps:

* Open **Synapse Studio** and click the **Manage** navigation icon
* Click "**Linked services**" from the "**External connections**" grouping in the resulting navigation
* Click "**+ New**"

  <img src="https://user-images.githubusercontent.com/44923999/186210475-7a993950-f5ba-4775-bb55-0dcb165a57a2.png" width="800" title="Snipped: August 23, 2022" />

* Search for and select "REST" on the "**New linked service**" pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/186212084-c41badaf-a6c3-4602-8dd2-937cbbe3e44e.png" width="800" title="Snipped: August 23, 2022" />

* Complete the resulting "**New linked service**" pop-out, including:

  Prompt | Entry
  ------ | ------
  "**Base URL**" | Modify and enter:<br>`https://api.loganalytics.io/v1/workspaces/{LogAnalyticsWorkspaceId}/query`  
  "**Authentication Type**" | Select **Anonymous**  

* Click "**Test connection**" to confirm successful connection and then click **Create**

### Step 3: Create Dataset
In this step, we will create the Dataset we will use to get our Bearer Token.

Complete the following steps:

* Open **Synapse Studio** and click the **Data** navigation icon
* Click **+** and then select "**Integration dataset**" from the **Linked** grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/186212524-f226d182-cc9e-4964-bbdc-c7ee68f0f467.png" width="800" title="Snipped: August 23, 2022" />

* Search for and select **REST** on the "**New linked service**" pop-out and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/186212779-0ec84877-1ca9-482f-aa22-c38f24993174.png" width="800" title="Snipped: August 23, 2022" />

* Complete the "**Set properties**" pop-out form and then click **OK**
* Click "**Test connection**" to confirm successful connection

### Step 4: Create Pipeline
In this step, we will create a Pipeline and add Activities.

Complete the following steps:
* Open "**Synapse Studio**" and click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token
This activity will make a REST API call and get a bearer token.

Complete the following steps:
* Expand **General** in the **Activities** bar
* Drag-and-drop a **Web** component into the main window
* Complete the form on the **Settings** tab

  <img src="https://user-images.githubusercontent.com/44923999/186214719-30d3b72a-65dd-41e5-8128-472d593b0a52.png" width="800" title="Snipped: August 23, 2022" />

  Prompt | Entry
  ------ | ------
  **URL** | Modify and enter:`https://login.microsoftonline.com/{TenantId}/oauth2/token`  
  **Method** | Select **POST**  
  **Headers** | Click "**+ Add**" and enter key-value pair: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | Modify and enter:<br> `grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://api.loganalytics.io/`

* Click **Debug** and monitor to confirm success

#### Activity 2: Get Data
This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

Complete the following steps:
* Expand "**Move & Transform**" in the **Activities** bar
* Drag-and-drop a "**Copy data**" component into the activity window
* Create a dependency from the "**Get Token**" component to the "**Get Data**" component
* Enter values on the **Source** tab 

  <img src="https://user-images.githubusercontent.com/44923999/186216401-ec555ffc-190b-4ea2-9be8-d59b0b30e1f0.png" width="800" title="Snipped: August 23, 2022" />

  Prompt | Entry
  ------ | ------
  **Source dataset** | Select your REST dataset
  **Request method** | Select **POST**
  **Request body** | Modify and enter:<br>`{ "query": "Sample_CL" }`
  **Additional headers** | Click **+ Add** and enter key-value pairs:<br>`content-type` :: `application/json;charset=utf-8`<br>`Authorization` :: `@concat('Bearer ',activity('Get Token').output.access_token)`

* Select the appropriate dataset (and configuration) on the **Sink** tab 

  <img src="https://user-images.githubusercontent.com/44923999/186217944-78b9131f-846b-4ba7-9d21-10e18dde9fa7.png" width="800" title="Snipped: August 23, 2022" />

#### Confirm Success

Click **Debug** and monitor result to confirm successful progress.
Browse to your Data Lake and download the resulting file.

<img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
