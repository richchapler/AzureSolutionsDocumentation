## Data Acquisition... from Log Analytics, via REST API

![image](https://user-images.githubusercontent.com/44923999/186208783-6c8db61d-01ea-4d5a-9a39-ac02458ec463.png)

This use case considers requirement statements like:
* "We capture custom logs from various device and apps to Log Analytics"
* "We want to capture Log Analytics data and maintain an extended history (past the maximum retention period)"
* "We feel that Log Analytics has more functionality than we need and costs too much"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md)
* [Data Lake](Infrastructure_DataLake.md) with a [container](Infrastructure_DataLake_Container.md)
* [Log Analytics](Infrastructure_LogAnalytics.md)
* [Synapse](Infrastructure_Synapse.md)

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
  **Body** | Modify and enter:<br>`grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://api.loganalytics.io/`

* Click **Debug** and monitor to confirm success

#### Activity 2: Get Data
This activity will make a REST API call and capture the response as a delimited file in the Data Lake.

Complete the following steps:
* Expand "**Move & Transform**" in the **Activities** bar
* Drag-and-drop a "**Copy data**" component into the activity window
* Create a dependency from the "**Get Token**" component to the "**Get Data**" component
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
