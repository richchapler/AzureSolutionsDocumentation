# AI Bot: Correlation Identifier

## Need to Review and Incorporate These...

https://microsoft.github.io/code-with-engineering-playbook/observability/correlation-id/
https://learn.microsoft.com/en-us/javascript/api/botbuilder-applicationinsights/?view=botbuilder-ts-latest
https://learn.microsoft.com/en-us/azure/bot-service/bot-builder-telemetry?view=azure-bot-service-4.0&tabs=csharp

## Delete Me

### Sandbox Architecture
The following minimum resources are required to test this concept:

| Resource | Why it’s needed to demonstrate correlation identifier |
| :--- | :--- |
| Azure Bot Channels Registration | entry point where the correlation id is generated |
| App Service (Web App) | hosts the bot/Web API code that emits, forwards and logs the same id |
| Function App | reads the propagated id, logs it and invokes the Azure OpenAI call |
| Application Insights | captures and links operation_Id / W3C trace context across every service hop |
| Azure OpenAI resource | external dependency used to validate end-to-end propagation of the correlation id |

### Expected Flow 
- Teams sends message to Azure Bot Channels Registration 
- Bot Channels Registration forwards activity to App Service endpoint 
- Bot middleware extracts “X-Correlation-Id” header or generates a new GUID 
- Bot logs incoming request to Application Insights with operation_Id set to the correlation id 
- Bot issues HTTP request to Azure OpenAI, including the “X-Correlation-Id” header 
- Application Insights logs the dependency call with the same operation_Id 
- Bot receives response, logs response telemetry, then returns message to Teams

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Prepare Environment

### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

### Set Active Subscription

```powershell
az account set --subscription "<subscriptionId>"
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

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

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
 "prompt": { "type": "string" },
 "correlationId":{ "type": "string" }
 },
 "required": [ "prompt" ]
}
``` 

Click **Save** and then copy the **HTTP URL** value for later use.

<!-- ------------------------- ------------------------- -->

### Add Action: HTTP 

Click **+** underneath the **When an HTTP request is received** pane and then **Add an action**.

On the **Add an action** pane, search for and select **HTTP**.

On the **HTTP** pane, complete the form:
- URI: `https://westus.api.cognitive.microsoft.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15`
- Method: POST
- Headers: `Content-Type` : `application/json`, `api-key` : `[OpenAI KEY 1]`, and `X-Correlation-Id` : `@{triggerBody()?['correlationId']}` 
- Body: 
 ```json
 {
 "messages": [
 {
 "role": "user",
 "content": "@triggerBody()?['prompt']"
 }
 ],
 "temperature": 1,
 "top_p": 1,
 "stream": false,
 "max_tokens": 4096,
 "n": 1
 }
 ``` 

Click **Save**.

<!-- ------------------------- ------------------------- -->

### Add Action: Response

Click **+** underneath the **HTTP** pane and then **Add an action**.

On the **Response** pane, complete the form:
- Status Code: `200`
- Body: `body('HTTP')?['choices'][0]?['message']?['content']'

Click **Save**.

<!-- ------------------------- ------------------------- -->

### Confirm Success

#### Test Open AI

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
Invoke-RestMethod "https://westus.api.cognitive.microsoft.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15" -Method POST -Headers @{"api-key"="<apiKey>";"Content-Type"="application/json";"X-Correlation-Id"="test-123"} -Body '{"messages":[{"role":"user","content":"Whats a quick soup recipe?"}],"temperature":1,"top_p":1,"stream":false,"max_tokens":4096,"n":1}'
```

##### Expected Response

You should see a response like the following:
```plaintext
choices : {@{finish_reason=stop; index=0; message=}}
created : 1745428938
id  : chatcmpl-BPY5a9XvXiwLvQGh86BR4r7keGNTm
model : gpt-3.5-turbo-0125
object : chat.completion
system_fingerprint : fp_0165350fbb
usage : @{completion_tokens=183; prompt_tokens=13; total_tokens=196}
```

<!-- ------------------------- -->

#### Run with payload

Click the **Run** dropdown button and select **Run with payload** from the resulting menu.

On the **Run with payload** pane, paste **Body** value:
```json
{
 "prompt": "Whats a quick soup recipe?",
 "correlationId":"test-001"
}
```

Click **Run**. Allow time for processing.

After it completes, navigate to **Overview** >> **Run history**, click to select the latest run, and confirm success.

##### Expected Response

You should see a response like the following:
```plaintext
One quick and easy soup recipe is tomato soup. Here's a simple recipe:

Ingredients:
- 1 can of diced tomatoes
- 1 cup of vegetable broth
- 1/2 cup of milk
- 1/2 teaspoon of garlic powder
- Salt and pepper to taste
- Fresh basil leaves for garnish

Instructions:
1. In a saucepan, combine the diced tomatoes (undrained) and vegetable broth. Bring to a boil, then reduce heat to simmer for 10 minutes.
2. Remove from heat and use an immersion blender to blend the tomato mixture until smooth.
3. Add milk, garlic powder, salt, and pepper. Stir to combine and heat through.
4. Serve the soup hot, garnished with fresh basil leaves.

Enjoy your quick and delicious tomato soup!
```

Once you see a successful run and valid response in the designer, you’ll know the endpoint is published and handling requests correctly.

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Application Registration

<!-- ------------------------- -->

### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

<!-- ------------------------- -->

### Set Active Subscription

```powershell
az account set --subscription "<subscriptionId>"
```

<!-- ------------------------- -->

### Create Application

```powershell
$appId = az ad app create --display-name "imbs" --query appId -o tsv
```

An application defines the identity, permissions and configuration of your bot in the directory.

<!-- ------------------------- -->

### Create Service Principal

```powershell
az ad sp create --id $appId
```

A Service Principal is the tenant-specific instance of that application that holds credentials (secrets or certificates) and can be assigned roles.

<!-- ------------------------- -->

### Create Client Secret

```powershell
$clientSecret = az ad app credential reset --id $appId --append --end-date "2025-05-01T00:00:00Z" --query password -o tsv
``` 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Bot Service

### Create Bot

```powershell
```

<!-- ------------------------- -->

### Register Bot

```powershell
az bot create --resource-group "im" --name "imbs" --kind registration --app-type SingleTenant --sku S1 --appid $appId --password $clientSecret --endpoint "<endpointURL>"
``` 

<!-- ------------------------- -->





### Enable Microsoft Teams channel 
```powershell
az bot msteams create \
 --resource-group "im" \
 --name "imbs"
``` 

Your bot is now registered, pointed at your Logic App, and enabled for Teams.