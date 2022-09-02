## Data Acquisition... from Cost Management, via REST API

![image](https://user-images.githubusercontent.com/44923999/188199195-34c228d5-37e8-4c06-8d7d-88b0e8d2a3ec.png)

This use case considers requirement statements like:
* "We want to analyze Azure resource costs across many subscriptions"
* "An understanding of resources and costs would really help us get a handle of inherited subscriptions"
* "Power BI, Cost Management connector requires billing account permissions we cannot get"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [Synapse](Infrastructure_Synapse.md) with a Data Explorer [Integration Dataset](Infrastructure_Synapse_Dataset.md) and Data Explorer, "**AllDatabasesAdmin**" permissions for the Synapse, System-Assigned Managed Identity

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

  <img src="https://user-images.githubusercontent.com/44923999/188210464-6d5a3462-9777-4523-a29c-9b239592ad60.png" width="800" title="Snipped: September 2, 2022" />

* On the resulting **Output**, mouse over the "**Get Token**" row and click on the **Output** icon

  <img src="https://user-images.githubusercontent.com/44923999/188210549-725e73ff-5c3b-4890-90ec-6cbf87b7e25d.png" width="800" title="Snipped: September 2, 2022" />

* Copy the "access token" value for later use... in the example below, everything from `eyJ0...` to `...1mqg`

  ```
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjJaUXBKM1VwYmpBWVhZR2FYRUpsOGxWMFRPSSIsImtpZCI6IjJaUXBKM1VwYmpBWVhZR2FYRUpsOGxWMFRPSSJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuYXp1cmUuY29tLyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzE2YjNjMDEzLWQzMDAtNDY4ZC1hYzY0LTdlZGEwODIwYjZkMy8iLCJpYXQiOjE2NjIxMzU3MzgsIm5iZiI6MTY2MjEzNTczOCwiZXhwIjoxNjYyMTM5NjM4LCJhaW8iOiJFMlpnWURnZXE4L2x6ZGlTWmFNdmNQTDVtUXZyQUE9PSIsImFwcGlkIjoiNzVhZmM4ZTktZjI5Ny00YmE0LThiNWItNWNlMzQ5NTI1OGExIiwiYXBwaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMTZiM2MwMTMtZDMwMC00NjhkLWFjNjQtN2VkYTA4MjBiNmQzLyIsImlkdHlwIjoiYXBwIiwib2lkIjoiZmY1Njc1NzQtMzY2Ny00ZDJiLWIxNmUtMmMyOTJjOTZkYzBkIiwicmgiOiIwLkFVWUFFOEN6RmdEVGpVYXNaSDdhQ0NDMjAwWklmM2tBdXRkUHVrUGF3ZmoyTUJPQUFBQS4iLCJzdWIiOiJmZjU2NzU3NC0zNjY3LTRkMmItYjE2ZS0yYzI5MmM5NmRjMGQiLCJ0aWQiOiIxNmIzYzAxMy1kMzAwLTQ2OGQtYWM2NC03ZWRhMDgyMGI2ZDMiLCJ1dGkiOiJ3QklycG9nYzIwZThxdXZ0NjhLT0FBIiwidmVyIjoiMS4wIiwieG1zX3RjZHQiOjE2NDUxMzcyMjh9.JSSyRQzqkcDatL9d8Vlbd8pr6_FXPmYJdf5fiZI1zWbQalH48pIVF57l4ZiWEwQLGdzsfZEwZQN9-ujSUWPSXGIUw3iTGqtAl8sI48G4hD8rPB9YwCt8sSKWVd1vx30_f7Cm4CyDiY7qNqkOfNdADzfpBj-CwB2U4zhmrVzbFk57qIpAuXUDDlAVen6JKXokC931mmUpZ_fYDBHiE14tXgPalPklP2bxaJKTlahObvfuLworr-A70zJi2Pdp5ckmMR3GqySM34Hz9uMbKtnEs8fJNJRYaeVGSHIjbY0Zp0wXgwmcaK31TXsNOl1_HXoMwvwLPicqR1y28Mq0ef1mqg"
  ```

_Note: If your "Preview Data" fails and you get an "The access token is invalid" error, it is likely that the token expired... simply Debug to get a new token_

#### Activity 2: Copy Data
This activity will make a REST API call, parse the response and write the data to Data Explorer.

* Expand "**Move & Transform**" in the **Activities** bar
* Drag-and-drop a "**Copy Data**" component into the activity window
* Create a dependency from the "**Get Token**" component to the "**Copy Data**" component

  <img src="https://user-images.githubusercontent.com/44923999/188206412-e90cef93-615e-403f-8e66-64d46fb9af86.png" width="800" title="Snipped: September 2, 2022" />

* Enter values on the **Source** tab 

  Prompt | Entry
  ------ | ------
  **Source dataset** | Select your REST dataset
  **Request method** | Select **POST**
  **Additional headers** | Click **+ Add** and enter key-value pairs:<br>`content-type` :: `application/json;charset=utf-8`<br>`authorization` :: `@concat('Bearer ',activity('Get Token').output.access_token)`

  Finally, paste the following JSON into '**Request body**':

  ```
  {
      "type": "Usage",
      "timeframe": "Custom",
      "timePeriod": {
          "from": "2022-08-29T00:00:00Z",
          "to": "2022-08-30T00:00:00Z"
      },
      "dataset": {
          "granularity": "Daily",
          "aggregation": {
              "totalCost": {
                  "name": "PreTaxCost",
                  "function": "Sum"
              }
          },
          "grouping": [
              {
                  "type": "Dimension",
                  "name": "ResourceType"
              }
          ]
      }
  }
  ```
  
  _Note: See links in Reference (below) for additional help with preparation of Request Body JSON {i.e., the query}_
  
* Click "Preview data"  

  <img src="https://user-images.githubusercontent.com/44923999/188211809-54c79983-ef20-47ef-b754-4909164d1cf5.png" width="800" title="Snipped: September 2, 2022" />

* On the resulting "**Please provide actual value of the parameters to preview data**" pop-out, paste the previously-copied Bearer Token value into the **Value** textbox and then click **OK** 

  <img src="https://user-images.githubusercontent.com/44923999/188220207-9c8c7891-868a-4d99-883d-bada3de2996c.png" width="800" title="Snipped: September 2, 2022" />

* Review the response and then close the "**Preview data**" window 





  <img src="https://user-images.githubusercontent.com/44923999/188209702-056fff32-982b-4f9f-8d71-d0f5928db823.png" width="800" title="Snipped: September 2, 2022" />

* Click on the  **Sink** tab, and then select your Data Explorer dataset

* Click on the  **Mapping** tab


* Click "**Publish all**", review changes on the resulting "**Publish all**" pop-out and then click **Publish**

#### Confirm Success

* Click **Debug** and confirm success
* Browse to your Data Lake and download the resulting file.

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />

### Reference
[Query - Usage - REST API (Azure Cost Management)](https://docs.microsoft.com/en-us/rest/api/cost-management/query/usage)<br>
[Dimensions - List - REST API (Azure Cost Management)](https://docs.microsoft.com/en-us/rest/api/cost-management/dimensions/list)
