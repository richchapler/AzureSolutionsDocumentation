# AI Bot: Correlation Identifier

## Sandbox Architecture
The following minimum resources are required to test this concept:

| Resource | Why it’s needed to demonstrate correlation identifier |
| :--- | :--- |
| Azure Bot Channels Registration | entry point where the correlation id is generated |
| App Service (Web App) | hosts the bot/Web API code that emits, forwards and logs the same id |
| Function App | reads the propagated id, logs it and invokes the Azure OpenAI call |
| Application Insights | captures and links operation_Id / W3C trace context across every service hop |
| Azure OpenAI resource | external dependency used to validate end-to-end propagation of the correlation id |

## Expected Flow 
- Teams sends message to Azure Bot Channels Registration 
- Bot Channels Registration forwards activity to App Service endpoint 
- Bot middleware extracts “X-Correlation-Id” header or generates a new GUID 
- Bot logs incoming request to Application Insights with operation_Id set to the correlation id 
- Bot issues HTTP request to Azure OpenAI, including the “X-Correlation-Id” header 
- Application Insights logs the dependency call with the same operation_Id 
- Bot receives response, logs response telemetry, then returns message to Teams

## Prepare Environment

### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

### Set Active Subscription

```powershell
az account set --subscription "ed7eaf77-d411-484b-92e6-5cba0b6d8098"
``` 

### Create Resource Group

```powershell
az group create --name "im" --location "westus"
```

<!-- ------------------------- -->

### Instantiate OpenAI

```powershell
az cognitiveservices account create --resource-group "im" --name "imoa" --kind OpenAI --sku S0 --location "westus"
``` 

### Create Deployment

Navigate to [Azure AI Foundry](https://ai.azure.com) and create a `gpt-35-turbo` deployment. Copy the Deployment Id value for later use.

<!-- ------------------------- -->

### Configure Azure CLI
...for automatic extension installs

```powershell
az config set extension.use_dynamic_install=yes_without_prompt
```

### Install Logic App Extension
...for Azure CLI

```powershell
az extension add --name logic
``` 

### Instantiate Logic App

```powershell
az logic workflow create --resource-group "im" --name "imla" --definition '{"definition":{"$schema":"https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json#","contentVersion":"1.0.0.0","triggers":{},"actions":{},"outputs":{}},"parameters":{}}' --location "westus"
```

<!-- ------------------------- ------------------------- -->

## Setup Workflow

Navigate to Azure Portal >> Logic App `imla` >> Logic App Designer.

### Add Trigger

Click **Add a trigger**, search for and select "HTTP", then Request >> "When an HTTP request is received".

### Configure the trigger’s Request Body JSON Schema  

On the **When an HTTP request is received** pane, in the **Request Body JSON Schema** textbox, paste:  
```json
{
  "type": "object",
  "properties": {
    "prompt":       { "type": "string" },
    "correlationId":{ "type": "string" }
  },
  "required": [ "prompt" ]
}
```  

Click **Save** and then copy the **HTTP URL** value for later use.

### Add Action: OpenAI  

Click **+** underneath the **When an HTTP request is received** pane and then **Add an action**.

On the **Add an action** pane, search for and select **Azure OpenAI** >> **Creates a completion for the chat message**.

On the **Create connection** pane, complete the form:
- Connection Name: `imoa`
- Azure OpenAI Resource Name: `imoa`
- Azure OpenAI API Key: `[OpenAI, KEY 1]`

Leave the **Azure Cognitive Search**... fields blank. Click **Create new**.

On the **Creates a completion for the chat message** pane, complete the form:
- Deployment ID Of The Deployed Model: `gpt-35-turbo`
- API Version: `2023-05-15`
- Messages: click **+ Add new item** and then enter **Role**: `user` and **Content**: `triggerBody()?['prompt']` 

Click **Save**.

### Add Action: Response

Click **+** underneath the **Creates a completion for the chat message** pane and then **Add an action**.

On the **Response** pane, complete the form:
- Status Code: `200`
- Body: `body('Creates_a_completion_for_the_chat_message')['choices'][0]?['message']?['content']'

Click **Save**.
