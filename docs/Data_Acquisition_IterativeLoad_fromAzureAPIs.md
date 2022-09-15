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
* Request data from the Cost Management API at the Resource Group rather than Subscription level

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**Application Registration**](Infrastructure_ApplicationRegistration.md) with "Cost Management Reader" role assignment (granted at subscription)
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
* Click on the link for your new workflow

  <img src="https://user-images.githubusercontent.com/44923999/190224716-25683448-8416-46ed-92ee-e284d7a326bf.png" width="800" title="Snipped: September 15, 2022" />

* Click on **Designer** in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/190228291-84e1f972-8d70-446d-8413-409c12ed2bc5.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting screen and "**Add a trigger**" pop-out, search for and then select "**Schedule**" (aka "Recurrence")

  <img src="https://user-images.githubusercontent.com/44923999/190257035-35c15279-4117-4e5b-8b7f-c4cfff15a386.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting pop-out form and then click **Save**

#### Variables

* Click the **+** to insert a new step and then "**Add an action**" on the resulting pop-up menu

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

* Click **Save**

  <img src="https://user-images.githubusercontent.com/44923999/190411759-3537a9d9-7b14-4775-a269-4488e71982e2.png" width="800" title="Snipped: September 15, 2022" />

* Click the **+** to insert a new step and then "**Add an action**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"
* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Scope" 
  **Type** | Select "String" 
  **Value** | {null}

* Click **Save**

#### Parameters

* Click **Parameters** in the menu bar
* On the resulting **Parameters** pop-out form, click "**+ Create parameter**"

  <img src="https://user-images.githubusercontent.com/44923999/190414578-e50034b4-3dcc-4459-bdd1-b2aeed81251a.png" width="800" title="Snipped: September 15, 2022" />

* Complete the pop-out form, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Subscriptions" 
  **Type** | Select "Array"
  **Value** | Your SubscriptionId values in form:<br>`[ "{Subscription1_Id}","{Subscription2_Id}" ]`

* Click **X** to close the pop-out form and then click **Save**

  <img src="https://user-images.githubusercontent.com/44923999/190417711-b98ea504-7f0d-4ab3-a4e3-79e03c47c724.png" width="800" title="Snipped: September 15, 2022" />

* Repeat for the Start Date parameter

  Prompt | Entry
  ------ | ------
  **Name** | Enter "StartDate" 
  **Type** | Select "String"
  **Value** | `2022-01-01` (or a date value that is meaningful for you)

  _Note: Date values will be required in ISO-8601 formatted strings {e.g., 2022-09-01T00:00:00.0000000}; abbreviated versions work fine_

* Repeat for the End Date parameter

  Prompt | Entry
  ------ | ------
  **Name** | Enter "EndDate" 
  **Type** | Select "String"
  **Value** | `2022-08-31` (or a date value that is meaningful for you)
  
  _Note: Parameters will be alphabetized regardless of the order in which you create them_

* Click **X** to close the pop-out form and then click **Save**

### Step 3: Get Bearer Token
In this step, we will request an access token from the Client Credentials Token URL and initialize a Token variable.

* Click the **+** underneath "**Recurrence, Daily**"

  <img src="https://user-images.githubusercontent.com/44923999/190418481-06a71761-cec0-4f6b-b06f-20e7230c58d9.png" width="800" title="Snipped: September 15, 2022" />

* Click "**Add a parallel branch**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/190419847-2bd758fe-fa7f-441e-a4fc-e8210ee9197e.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/190432016-ee55661b-22c0-4f08-bbf9-2c9e860c5107.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Method** | Select "POST" 
  **URI** | `https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token`
  **Headers** | `content-type` :: `application/x-www-form-urlencoded`
  **Body** | `grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}&resource=https://management.azure.com/`

* Click **Save**

#### Initialize Token Variable

* Click the **+** under **HTTP** and then "**Add an action**" on the resulting pop-up menu

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

* Click the **+** underneath "**Recurrence, Daily**" and then "**Add a parallel branch**" on the resulting pop-up menu
* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/190444707-9b5f3484-fd83-47db-a724-3d8921ed9eca.png" width="800" title="Snipped: September 15, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/190445093-9afe8806-8102-4824-9d33-7c0481adf007.png" width="800" title="Snipped: September 15, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  Prompt | Entry
  ------ | ------
  **Name** | Enter "Counter" 
  **Type** | Select "String" 
  **Value** | Enter "0"

* Click **Save**








### Miscellaneous Notes (incorporate or delete before publishing)

_Note: Scope ResourceGroup does not allow use of **BillingPeriod** and **ServiceTier** columns_

scope Resource Group {i.e., '/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}'}, rather than scope Subscription {i.e., '/subscriptions/{subscriptionId}/'}

(ISO-8601 formatted strings per Logic App requirements)

```
{
    "definition": {
        "$schema": https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#,
        "actions": {
            "For_Each_Subscription": {
                "actions": {
                    "For_Each_Resource_Group": {
                        "actions": {
                            "For_Each_Date": {
                                "actions": {
                                    "Get_Cost_Management_Data": {
                                        "inputs": {
                                            "body": {
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
                                            },
                                            "headers": {
                                                "authorization": "@variables('Token')",
                                                "content-type": "application/json;charset=utf-8"
                                            },
                                            "method": "POST",
                                            "uri": https://management.azure.com/@{variables('Scope')}/providers/Microsoft.CostManagement/query?api-version=2021-10-01
                                        },
                                        "runAfter": {
                                            "Set_Date": [
                                                "Succeeded"
                                            ]
                                        },
                                        "type": "Http"
                                    },
                                    "Send_Event": {
                                        "inputs": {
                                            "body": {
                                                "ContentData": "@{base64(body('Get_Cost_Management_Data'))}"
                                            },
                                            "host": {
                                                "connection": {
                                                    "referenceName": "eventhubs"
                                                }
                                            },
                                            "method": "post",
                                            "path": "/@{encodeURIComponent('rchaplereh')}/events"
                                        },
                                        "runAfter": {
                                            "Get_Cost_Management_Data": [
                                                "Succeeded"
                                            ]
                                        },
                                        "type": "ApiConnection"
                                    },
                                    "Set_Date": {
                                        "inputs": {
                                            "name": "Date",
                                            "value": "@{item()}"
                                        },
                                        "runAfter": {},
                                        "type": "SetVariable"
                                    }
                                },
                                "foreach": "@variables('Dates')",
                                "runAfter": {
                                    "Set_Scope": [
                                        "Succeeded"
                                    ]
                                },
                                "type": "Foreach"
                            },
                            "Set_Scope": {
                                "inputs": {
                                    "name": "Scope",
                                    "value": "@{item().id}"
                                },
                                "runAfter": {},
                                "type": "SetVariable"
                            }
                        },
                        "foreach": "@body('Get_Resource_Groups').value",
                        "runAfter": {
                            "Get_Resource_Groups": [
                                "Succeeded"
                            ]
                        },
                        "type": "Foreach"
                    },
                    "Get_Resource_Groups": {
                        "inputs": {
                            "headers": {
                                "authorization": "@variables('Token')",
                                "content-type": "application/json;charset=utf-8"
                            },
                            "method": "GET",
                            "uri": "@{concat('https://management.azure.com/subscriptions/',item(),'/resourcegroups?api-version=2021-04-01')}"
                        },
                        "runAfter": {},
                        "type": "Http"
                    }
                },
                "foreach": "@parameters('Subscriptions')",
                "runAfter": {
                    "Initialize_Date": [
                        "Succeeded"
                    ],
                    "Set_Token": [
                        "Succeeded"
                    ],
                    "Until": [
                        "Succeeded"
                    ]
                },
                "type": "Foreach"
            },
            "Get_Token": {
                "inputs": {
                    "body": "grant_type=client_credentials&client_id=75afc8e9-f297-4ba4-8b5b-5ce3495258a1&client_secret=BUo8Q~DLdAEJTlhn5sfPPUXwg_WAtH7hyXKfscX9&resource=https://management.azure.com/",
                    "headers": {
                        "content-type": "application/x-www-form-urlencoded"
                    },
                    "method": "POST",
                    "uri": https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token
                },
                "runAfter": {},
                "type": "Http"
            },
            "Initialize_Counter": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Counter",
                            "type": "integer",
                            "value": 0
                        }
                    ]
                },
                "runAfter": {},
                "type": "InitializeVariable"
            },
            "Initialize_Date": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Date",
                            "type": "string"
                        }
                    ]
                },
                "runAfter": {
                    "Initialize_Scope": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Initialize_Dates_Array": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Dates",
                            "type": "array"
                        }
                    ]
                },
                "runAfter": {
                    "Initialize_Counter": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Initialize_Scope": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Scope",
                            "type": "string"
                        }
                    ]
                },
                "runAfter": {},
                "type": "InitializeVariable"
            },
            "Set_Token": {
                "inputs": {
                    "variables": [
                        {
                            "name": "Token",
                            "type": "string",
                            "value": "@{concat('Bearer ',body('Get_Token').access_token)}"
                        }
                    ]
                },
                "runAfter": {
                    "Get_Token": [
                        "Succeeded"
                    ]
                },
                "type": "InitializeVariable"
            },
            "Until": {
                "actions": {
                    "Append_Date": {
                        "inputs": {
                            "name": "Dates",
                            "value": "@addDays(parameters('StartDate'),variables('Counter'))"
                        },
                        "runAfter": {},
                        "type": "AppendToArrayVariable"
                    },
                    "Increment_variable": {
                        "inputs": {
                            "name": "Counter",
                            "value": 1
                        },
                        "runAfter": {
                            "Append_Date": [
                                "Succeeded"
                            ]
                        },
                        "type": "IncrementVariable"
                    }
                },
                "expression": "@greater(addDays(parameters('StartDate'), variables('Counter')), addDays(parameters('EndDate'), 0))",
                "limit": {
                    "count": 60,
                    "timeout": "PT1H"
                },
                "runAfter": {
                    "Initialize_Dates_Array": [
                        "Succeeded"
                    ]
                },
                "type": "Until"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "triggers": {
            "Trigger_Daily": {
                "recurrence": {
                    "frequency": "Day",
                    "interval": 1
                },
                "type": "Recurrence"
            }
        }
    },
    "kind": "Stateful"
}
```

@range(0,div(div(div(div(div(div(sub(ticks(pipeline().parameters.EndDate),ticks(pipeline().parameters.StartDate)),10),1000),1000),60),60),24))

ADF equivalent of DATEDIFF

@addDays(pipeline().parameters.StartDate, item())

@concat('https://management.azure.com/subscriptions/'
    ,item()
    ,'/resourcegroups?api-version=2021-04-01'
    )


dateDifference(addToTime('1970-01-01', parameters('StartDate'),'second'),addToTime('1970-01-01', parameters('EndDate'),'second'))

addDays(parameters('StartDate'),Counter,'yyyy-MM-ddTHH:mm:ssZ')

formatDateTime(parameters('StartDate'),'yyyy-MM-ddTHH:mm:ssZ')

