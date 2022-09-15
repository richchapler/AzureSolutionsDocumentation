## Data Acquisition... Iterative Load from Azure APIs

![image](https://user-images.githubusercontent.com/44923999/190155800-820282af-0eb6-47ff-b392-ed1abf9783f9.png)

This use case considers requirement statements like:
* "We need to capture and analyze cost data from many subscriptions"
* "Our subscriptions have more than 1,000 resources and are hitting the Cost Management API's per-request limitation"
* "We want to pull historical data... 730 days {i.e., 2 years}"
* "We want to pull more Cost Management API columns than before, but we haven't settled on a final set {i.e., schema **will** drift}"

<br>The solution described in this documentation will:
* Leverage Logic Apps' nested iteration capability with input parameters for Subscriptions and Start / End Dates 
* Leverage Data Explorer's dynamic data type to provide for potential schema drift
* Request data from the Cost Management API at the Resource Group level rather than Subscription level

_Note: If you have Resource Groups with more than 1,000 resources, you might have to choose a more granular scope or other strategy_

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**Application Registration**](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at Subscription or Resource Group)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group
* [**Logic App**](Infrastructure_LogicApp.md)

### Step 2: Prepare Workflow
In this step, we will create a workflow, initialize variables, and add parameters.

* Navigate to your Logic App

  <img src="https://user-images.githubusercontent.com/44923999/190197666-84f0e96f-72c3-4ab7-b527-890eeebc0c23.png" width="800" title="Snipped: September 15, 2022" />

#### Workflow

* Click on "**Create workflow >**" in the "**Create a workflow in Designer**" rectangle
* On the resulting page click "**+ Add**"

  <img src="https://user-images.githubusercontent.com/44923999/190224201-41c58642-dbb5-4636-8da4-d151b82bf838.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**New workflow**" pop-out form and then click **Create**

_Note: Choice of stateful or stateless will depend on your use case and environment... reference: https://docs.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare#stateful-stateless_

* Click on the link for your new workflow

  <img src="https://user-images.githubusercontent.com/44923999/190224716-25683448-8416-46ed-92ee-e284d7a326bf.png" width="800" title="Snipped: September 15, 2022" />

* Navigate to **Designer**

  <img src="https://user-images.githubusercontent.com/44923999/190228291-84e1f972-8d70-446d-8413-409c12ed2bc5.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting screen and "**Add a trigger**" pop-out, search for and then select "**Schedule**" (aka "Recurrence")

  <img src="https://user-images.githubusercontent.com/44923999/190257035-35c15279-4117-4e5b-8b7f-c4cfff15a386.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting pop-out form and then click **Save**

#### Variables

* Click the **+** icon and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190261952-61599cfe-a8f5-4bff-9f32-c6f75ec0535f.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190410009-bed8f453-2963-4912-8f34-c9fdf5df1df5.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Date" 
  **Type** | Select "String" 
  **Value** | {null}
  
  _Note: Logic Apps does not have a data type for DateTime, so we are using string and will handle usage in expressions_

  <img src="https://user-images.githubusercontent.com/44923999/190411759-3537a9d9-7b14-4775-a269-4488e71982e2.png" width="800" title="Snipped: September 15, 2022" />

* Click the **+** icon and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"
* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Scope" 
  **Type** | Select "String" 
  **Value** | {null}

#### Parameters

* Click **Parameters** in the menu bar
* On the resulting **Parameters** pop-out form, click "**+ Create parameter**"

  <img src="https://user-images.githubusercontent.com/44923999/190414578-e50034b4-3dcc-4459-bdd1-b2aeed81251a.png" width="800" title="Snipped: September 15, 2022" />

* Complete the pop-out form, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Subscriptions" 
  **Type** | Select "Array"
  **Value** | Your SubscriptionId values in form: `[ "{Subscription1_Id}","{Subscription2_Id}" ]`

* Click **X** to close the pop-out form

  <img src="https://user-images.githubusercontent.com/44923999/190417711-b98ea504-7f0d-4ab3-a4e3-79e03c47c724.png" width="800" title="Snipped: September 15, 2022" />

* Repeat for the **StartDate** parameter

  Prompt | Entry
  ------ | ------
  **Name** | Enter "StartDate" 
  **Type** | Select "String"
  **Value** | Enter `2022-01-01` (or a date value that is meaningful for you)

  _Note: Date values will be required in ISO-8601 formatted strings {e.g., 2022-09-01T00:00:00.0000000}; abbreviated versions work fine_

* Repeat for the **EndDate** parameter

  Prompt | Entry
  ------ | ------
  **Name** | Enter "EndDate" 
  **Type** | Select "String"
  **Value** | Enter `2022-08-31` (or a date value that is meaningful for you)
  
  _Note: Parameters will be alphabetized regardless of the order in which you create them_

* Click **X** to close the pop-out form

### Step 3: Get Bearer Token
In this step, we will request an access token from the Client Credentials Token URL and initialize a Token variable.

* Navigate to **Designer**
* Click the **+** icon underneath "**Recurrence, Daily**"

  <img src="https://user-images.githubusercontent.com/44923999/190418481-06a71761-cec0-4f6b-b06f-20e7230c58d9.png" width="800" title="Snipped: September 15, 2022" />

* Click "**Add a parallel branch**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190419847-2bd758fe-fa7f-441e-a4fc-e8210ee9197e.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/190432016-ee55661b-22c0-4f08-bbf9-2c9e860c5107.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Method** | Select "POST" 
  **URI** | Add dynamic content: `https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token`
  **Headers** | Add dynamic content: `content-type` :: `application/x-www-form-urlencoded`
  **Body** | Add dynamic content:  `grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}&resource=https://management.azure.com/`

  _Note: If you expect processing to take longer than an hour {i.e., the lifespan of a token}, you might consider moving token logic to the nested iteration_

#### Initialize Token Variable

* Click the **+** icon under **HTTP** and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190437313-7be79dc5-9120-44e8-a69a-bef6fd290aeb.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Token" 
  **Type** | Select "String"
  **Value** | Add dynamic content `concat('Bearer ',body('HTTP,_Get_Token').access_token)`

* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**

  <img src="https://user-images.githubusercontent.com/44923999/190438832-fa58b059-953a-4597-bc20-c1c8c2e472d7.png" width="800" title="Snipped: September 15, 2022" />

* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190440627-6944ea79-cbdc-4479-8358-4af0df74e53c.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 4: Prepare Dates Array
In this step, we will iterate through dates between StartDate and EndDate and append to the Dates array.

* Navigate to **Designer**
* Click the **+** icon underneath "**Recurrence, Daily**" and then "**Add a parallel branch**" on the resulting pop-up menu

#### "Counter" Variable

* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190445905-5a3247ab-c764-4330-985f-dff913fa3348.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Counter" 
  **Type** | Select "String" 
  **Value** | Enter "0"

#### "Dates" Variable

* Click the **+** icon and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190446998-f40ffa00-2dae-43a9-8445-0ffd7db5d057.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Dates" 
  **Type** | Select "Array" 
  **Value** | {null}

#### Until Loop

* Click the **+** icon and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190447971-9107b07e-78a0-43b7-a9d7-134ec445a786.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select **Until**

  <img src="https://user-images.githubusercontent.com/44923999/190448952-3407b518-4eaa-4ef3-b095-12d4a82efe6a.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting **Until** pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Choose a value** | Add dynamic content: `addDays(parameters('StartDate'), variables('Counter'))`
  **Type** | Select "is greather than" 
  **Choose a value** | Add dynamic content: `addDays(parameters('EndDate'), 0)`

#### Until Loop, Append Date

* Click the **+** icon inside the "Do..Until" action and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190450048-918ca5c1-a382-44f2-8e20-6cc3053562b1.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Append to array variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190450440-bee99bd8-3889-40a0-9772-d19445c2f806.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Append to array variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Select "Dates" 
  **Value** | Add dynamic content: `addDays(parameters('StartDate'),variables('Counter'))`
  
#### Until Loop, Increment Counter

* Click the **+** icon inside the "Do..Until" action and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190453240-f8abf10c-cc9d-48f2-8b06-c69525b87ccb.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Increment variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190453412-c626ce60-6f2b-4577-8fa5-47063807ee3a.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Append to array variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Select "Counter" 
  **Value** | Enter "1"
  
* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**
* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190454850-6fbf4344-6e90-4249-aaa8-cd4b39ed212d.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 5: Iterate through Subscriptions
In this step, we will nest "For Each" actions for Subscriptions.

* Navigate to **Designer**
* Click the **+** icon at the bottom of the page and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190455774-3fc7ce49-37d8-40c0-bb4d-b0ba410fe0e9.png" width="800" title="Snipped: September 15, 2022" />

  _Note: Dependencies from all parallel branches will be added to the new action_

* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/190459574-150f4e41-8427-49de-b134-33bde1a9679b.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Select an output from previous steps** | Select "Subscriptions" 
  
* Click the **+** icon inside the "For Each, Subscriptions" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/190466131-a89cc632-8a24-4806-9368-87e5e591922e.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Method** | Select "GET" 
  **URI** | Add dynamic content: `concat('https://management.azure.com/subscriptions/',item(),'/resourcegroups?api-version=2021-04-01')`
  **Headers** | Add:<br>`authorization` :: dynamic content `variables('Token')`<br>`content-type` :: `application/json;charset=utf-8`

* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**
* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190466998-7779d281-3390-481c-b520-d7e995b30249.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 6: Iterate through Resource Groups
In this step, we will nest "For Each" actions for Resource Groups {aka Scopes}.

* Navigate to **Designer**
* Click the **+** icon inside the "For Each, Subscriptions" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/190470775-60ff85a3-4247-4a4f-a523-e4dff1fad22b.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Select an output from previous steps** | Enter dynamic content: `body('HTTP,_Get_Resource_Groups').value`

* Click the **+** icon inside the "For Each, Resource Groups" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190469079-06379573-b7e9-46db-bb0a-49c3e05c8f35.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Set variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Select "Scope" 
  **Value** | Add dynamic content: `item().id`

* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**
* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190471493-79690223-8abd-46f0-b496-01d775c8e544.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 7: Iterate through Dates
In this step, we will nest "For Each" actions for Dates.

* Navigate to **Designer**
* Click the **+** icon inside the "For Each, Resource Groups" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/190472425-05d5589f-29bb-49d4-9f56-836a3a2fae6d.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Select an output from previous steps** | Enter dynamic content: `variables('Dates')`

* Click the **+** icon inside the "For Each, Dates" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190473339-570664df-77a3-4af7-a0f1-3febdc8cd08d.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Set variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Select "Date" 
  **Value** | Add dynamic content: `item()`

* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**
* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190473630-b2905f7a-1af8-4d00-a0b4-dd24ce08699b.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 8: Get Costs
In this step, we will send a request to the Cost Management API using iterative variables.

* Navigate to **Designer**
* Click the **+** icon inside the "For Each, Dates" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/190474598-47ff99ed-7339-43af-8d5d-d52bec1b25ff.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Method** | Select "POST" 
  **URI** | Add dynamic content:<br>`[concat('https://management.azure.com/subscriptions/',item(),'/resourcegroups?api-version=2021-04-01')](https://management.azure.com/@{variables('Scope')}/providers/Microsoft.CostManagement/query?api-version=2021-10-01)`
  **Headers** | Add:<br>`authorization` :: dynamic content `variables('Token')`<br>`content-type` :: `application/json;charset=utf-8`

*  Finally, paste the following in **Body**:
  ```
  {
    "dataset": {
      "aggregation": {
        "totalCost": {
          "function": "Sum",
          "name": "PreTaxCost"
        }
      },
      "granularity": "Daily",
      "grouping": [
        {
          "name": "ResourceGroupName",
          "type": "Dimension"
        },
        {
          "name": "ResourceType",
          "type": "Dimension"
        },
        {
          "name": "ResourceId",
          "type": "Dimension"
        },
        {
          "name": "ResourceLocation",
          "type": "Dimension"
        },
        {
          "name": "MeterCategory",
          "type": "Dimension"
        },
        {
          "name": "MeterSubCategory",
          "type": "Dimension"
        },
        {
          "name": "Meter",
          "type": "Dimension"
        },
        {
          "name": "ServiceName",
          "type": "Dimension"
        },
        {
          "name": "PartNumber",
          "type": "Dimension"
        },
        {
          "name": "PricingModel",
          "type": "Dimension"
        },
        {
          "name": "ChargeType",
          "type": "Dimension"
        },
        {
          "name": "ReservationName",
          "type": "Dimension"
        },
        {
          "name": "Frequency",
          "type": "Dimension"
        }
      ]
    },
    "timePeriod": {
      "from": "@{variables('Date')}",
      "to": "@{variables('Date')}"
    },
    "timeframe": "Custom",
    "type": "Usage"
  }
  ```

  _Note: Scope ResourceGroup does not allow use of **BillingPeriod** and **ServiceTier** columns_

* Click **Save**

#### Confirm Success
Before we move on to the next step, let's confirm that what we have created (so far) is functional.

* Navigate to **Overview**
* Click "**Run Trigger**" and then **Run** in the resulting dropdown menu
* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/190477511-052af683-c8d6-4c79-be7e-62612c493e27.png" width="800" title="Snipped: September 15, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 9: Send to Event Hub
In this step, we will send the Cost Management API response to the Event Hub (for downstream ingestion in Data Explorer).

* Navigate to **Designer**
* Click the **+** icon inside the "For Each, Dates" action and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Send Event**"

  <img src="https://user-images.githubusercontent.com/44923999/190478428-ca6f598d-cf32-44dd-a440-a9a791d7a2e8.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Send Event**" pop-out form, **Create Connection** tab, including:

  Prompt | Entry
  ------ | ------
  **Authentication Type** | Select "**Connection String**" 
  **Connection String** | Paste the "**Connection string-primary ke**y" value copied from your Event Hub Namespace >> **Shared access policies**

* Click **Create**

  <img src="https://user-images.githubusercontent.com/44923999/190478665-5e5dd77c-c211-4a31-8f14-284297005854.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Send Event**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Add new parameter** | Check "Content" 

  <img src="https://user-images.githubusercontent.com/44923999/190479224-43f1e5bf-a4b4-45d7-9f3b-c09358a40325.png" width="800" title="Snipped: September 15, 2022" />

* Click into the resulting "**Content of the event**" textbox, and in the resulting pop-up, select **Body** from "**HTTP, Get Costs**"

* Click **Save**

### Step 10: Ingest Data
Follow the instructions at [One-Click Ingestion](Data_Acquisition_OneClickIngestion.md) to setup the ingestion from Event Hub to Data Explorer.

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
