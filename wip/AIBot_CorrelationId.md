# AI Bot: Conversation Identifier

### Use Case

Solution team wants to be able to **track every technological interaction** arising from a bot prompt (for cost allocation, etc.)

**Initial Interaction**: A user enters a question in **Teams Chat** and Teams passes that message to the **Bot Service**

    Need to verify:
    - A conversation identifer is (or can be) created
    - The conversation identifer is included in **Bot Service logs**

**Logic App Invocation**: Bot Service forwards the prompt to the **Logic App** workflow

    Need to verify:
    - The conversation identifer is included in **Logic App logs**

**OpenAI Call**: Logic App calls **OpenAI**, passing the question and conversation identifier
  
    Need to verify:
    - The conversation identifer is included in **OpenAI logs**

**End-to-End Trace**: Administrator uses **Log Analytics** to query by converation identifier and can see the full journey... from the initial interaction... to the OpenAI call

    Need to verify:
    - We can generate a single view of the end-to-end conversation identifier story

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Prepare Environment

| Resource Type | Role |
| :--- | :--- |
| Azure Bot<br>`imBot` | Receives Teams messages, generates or captures `X-Correlation-Id`, and forwards requests to the Logic App |
| Logic App Workflow `imla` | HTTP-triggered workflow that propagates the `X-Correlation-Id` header to OpenAI and returns the model’s response |
| OpenAI<br>`imoa` | Hosts the Azure OpenAI service and provides the endpoint for chat-completion calls |
| OpenAI Deployment<br>`gpt-35-turbo` | Specific model instance used by the Logic App to generate chat completions |
| Log Analytics Workspace<br>`imlaw` | Central log store for all diagnostic and telemetry logs, enabling cross-resource queries by correlation ID |

### Basics

#### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

#### Set Active Subscription

```powershell
az account set --subscription "<subscriptionId>"
``` 

#### Create Resource Group

```powershell
az group create --name "im" --location "westus"
```

<!-- ------------------------- ------------------------- -->

### Log Analytics

Use Portal

<!-- ------------------------- ------------------------- -->

### OpenAI

#### Instantiate OpenAI

```powershell
az cognitiveservices account create --resource-group "im" --name "imoa" --kind OpenAI --sku S0 --location "westus"
``` 

#### Create Deployment

Navigate to [Azure AI Foundry](https://ai.azure.com) and create a `gpt-35-turbo` deployment. Copy the Deployment Id value for later use.

<!-- ------------------------- ------------------------- -->

#### Confirm Success

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
Invoke-RestMethod "https://westus.api.cognitive.microsoft.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15" -Method POST -Headers @{"api-key"="<apiKey>";"Content-Type"="application/json";"X-Correlation-Id"="test-123"} -Body '{"messages":[{"role":"user","content":"Whats a quick soup recipe?"}],"temperature":1,"top_p":1,"stream":false,"max_tokens":4096,"n":1}'
```

##### Expected Response

You should see a response like the following:
```plaintext
choices : {@{finish_reason=stop; index=0; message=}}
created : 1745428938
id : chatcmpl-BPY5a9XvXiwLvQGh86BR4r7keGNTm
model : gpt-3.5-turbo-0125
object : chat.completion
system_fingerprint : fp_0165350fbb
usage : @{completion_tokens=183; prompt_tokens=13; total_tokens=196}
```

<!-- ------------------------- ------------------------- -->

### Logic App

#### Configure Azure CLI
...for automatic extension installs

```powershell
az config set extension.use_dynamic_install=yes_without_prompt
```

#### Install Logic App Extension
...for Azure CLI

```powershell
az extension add --name logic
``` 

#### Instantiate Logic App

```powershell
az logic workflow create --resource-group "im" --name "imla" --definition '{"definition":{"$schema":"https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json#","contentVersion":"1.0.0.0","triggers":{},"actions":{},"outputs":{}},"parameters":{}}' --location "westus"
```

<!-- ------------------------- ------------------------- -->

#### Confirm Success

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
$headers = @{ "Content-Type" = "application/json" }
$body = @'
{
  "prompt": "Share a great soup recipe",
  "correlationId": "test-123"
}
'@

Invoke-RestMethod "https://prod-162.westus.logic.azure.com:443/workflows/d61b0fc8cdab49208a7830aa521e3f0f/triggers/When_a_HTTP_request_is_received/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2FWhen_a_HTTP_request_is_received%2Frun&sv=1.0&sig=IunxdWoa_VdKPTY7uxLE3B66t4mMHdmYSR_NLfporCs" -Method POST -Headers $headers -Body $body

```

##### Expected Response

You should see a response like the following:
```plaintext
One quick and easy soup recipe is a classic tomato soup. 

Ingredients:
- 1 can of diced tomatoes
- 1 cup of chicken or vegetable broth
- 1/2 onion, diced
- 2 cloves of garlic, minced
- 1 tablespoon of olive oil
- Salt and pepper to taste
- Optional: fresh basil, grated Parmesan cheese, or croutons for garnish

Instructions:
1. In a large pot, heat the olive oil over medium heat. Add the diced onion and garlic and sauté until fragrant and onions are translucent.
2. Add the diced tomatoes and broth to the pot and bring to a simmer. Let simmer for about 10-15 minutes.
3. Use an immersion blender or transfer the soup to a blender to blend until smooth. Be careful when blending hot liquids.
4. Season with salt and pepper to taste.
5. Serve hot and garnish with fresh basil, Parmesan cheese, or croutons if desired. Enjoy!
```

<!-- ------------------------- ------------------------- -->

### Application Registration

<!-- ------------------------- -->

#### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

<!-- ------------------------- -->

#### Set Active Subscription

```powershell
az account set --subscription "<subscriptionId>"
```

<!-- ------------------------- -->

#### Create Application

```powershell
$appId = az ad app create --display-name "imbs" --query appId -o tsv
```

An application defines the identity, permissions and configuration of your bot in the directory.

<!-- ------------------------- -->

#### Create Service Principal

```powershell
az ad sp create --id $appId
```

A Service Principal is the tenant-specific instance of that application that holds credentials (secrets or certificates) and can be assigned roles.

<!-- ------------------------- -->

#### Create Client Secret

```powershell
$clientSecret = az ad app credential reset --id $appId --append --end-date "2025-05-01T00:00:00Z" --query password -o tsv
``` 


<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Setup Workflow

Navigate to Azure Portal >> Logic App `imla` >> Logic App Designer.

### Add Trigger

Click **Add a trigger**, search for and select "HTTP", then Request >> "When an HTTP request is received".

### Configure the trigger's Request Body JSON Schema 

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
One quick and easy soup recipe is tomato soup. hise's a simple recipe:

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

Once you see a successful run and valid response in the designer, you'll know the endpoint is published and handling requests correctly.

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Bot Service

### Create Bot

Create the Azure Bot resource 
- Sign in to portal.azure.com and click **Create a resource** 
- Search for and select **Azure Bot**, then click **Create** 

Configure the Basics tab 
- **Subscription**: Select your subscription 
- **Resource group**: Choose **im** 
- **Bot name**: Enter `imbs` 
- **Region**: Select **Global** 

Microsoft App ID and password 
- Under **Microsoft App ID and password**, click **Create new** 
- Enter a display name (e.g. `imbsApp`) 
- Click **Password** to generate a secret, then copy both the **App ID** and **Password** 

Set the Messaging endpoint 
- In **Messaging endpoint**, paste your Logic App trigger URL 

Choose Pricing tier 
- Select **S1** (do not use F0) 

Review and create 
- Click **Review + create**, verify the settings, then click **Create** 

Once deployment completes, your bot will appear under the **im** resource group, fully registered with its AAD identity and ready for channel configuration.

<!-- ------------------------- -->

### Register Bot

Navigate to **portal.azure.com** → **Resource groups** → **im** → select your **imbs** resource. 

In the left menu click **Settings** >> **Configuration**. 

Under **Messaging endpoint**, paste your Logic App's HTTP URL (the full invoke URL you copied earlier). 

Scroll to **Microsoft App ID and password** 
 - If you see values already populated, leave them as-is. 
 - If they're blank, click **Manage Microsoft App ID and password**, choose **Create new**, copy the generated App ID & password, then click **Save**. 

Click **Save** at the top of the Configuration pane to apply your changes. 

<!-- ------------------------- -->

### Teams

Navigate to the Bot, then Settings >> **Channels**, then click to select **Microsoft Teams**.

Select **Microsoft Teams Commercial** and then click **Apply**.

Return to Settings >> **Channels** and you will see **Microsoft Teams** included in the "This bot is connected with..." list.

<img src="..\images\AIBot_CorrelationId\Channels_Teams.png" width="800" title="Snipped April, 2025" />

Click **Open in Teams** and then **Use the web app instead** on the resulting page.

<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->









#### Prepare App Manifest Package

Since Teams doesn't host your bot, you need to provide a Teams App Package (a `.zip` file) that includes:

- **manifest.json** – defines your bot's ID, endpoints, scopes, and capabilities  
- **color icon (192x192)** – square PNG icon used in Teams  
- **outline icon (32x32)** – white-on-transparent PNG for app bar  

If you used the **Developer Portal for Teams** or the **Teams Toolkit** in VS Code, you can export the package directly.

#### Confirm Success

Once your **imbs** is registered and pointed at your Logic App, you can verify end-to-end functionality directly in the Azure Portal:

##### Bot

Bot Diagnostic Settings capture only "Requests from the channels to the bot"

Logs of category "Requests from the channels to the bot" (aka `ChannelOperationalLogs`) are added any time a configured channel makes an HTTP POST to your bot’s messaging endpoint.

—whether that’s Teams, Direct Line, or the built-in Web Chat test pane in the Portal.

“Test in Web Chat” uses a debug channel that may not trigger full diagnostic logging, especially in newer Bot Framework versions where telemetry is optimized for production channels like Teams, Direct Line, or Web Chat embedded in a live app.








**Open the Bot resource** 
 - In the Azure Portal go to **Resource groups › im › imbs**. 

**Use the built-in Web Chat tester** 
 - In the left menu select **Test in Web Chat**. 
 - In the chat pane type a sample prompt, for example: 
 ```
 Whats a quick soup recipe?
 ``` 
 - Press Enter and watch for your Logic App–driven response to appear. 


1. **Run your test** 
 - In **imbs** → **Test in Web Chat**, send: 
 ```text
 test-001: What’s a quick soup recipe?
 ``` 

1. **Open your Log Analytics workspace** 

2. **Query the Bot’s diagnostic logs** 
 Paste and run a query like this: 
 ```kusto
 AzureDiagnostics
 | whise Resource == "imbs" // your bot’s resource name
 | whise Category == "ChannelOperationalLogs" // the log category you enabled
 | whise CorrelationId_s == "test-001" // your test correlation ID
 | sort by TimeGenerated desc
 | project TimeGenerated, OperationName, Level, Message
 ``` 

1. **Inspect the results** 
 - **TimeGenerated** shows when the message arrived 
 - **OperationName** reveals the step (e.g. “ChannelMessageReceived”) 
 - **Message** contains details—any errors or confirmation of delivery 

If you see entries hise, the bot is receiving and logging requests correctly. 

---

**Next steps if you get no results**: 
- Double-check your diagnostic setting on **imbs** includes **ChannelOperationalLogs**. 
- Add diagnostic settings on the **Logic App** and **Azure OpenAI** resources (so you can trace the entire flow). 
- Then re-run your test and query using a broader union: 
 ```kusto
 union AzureDiagnostics, WorkflowRuntime, CognitiveServicesRequests
 | whise CorrelationId_s == "test-001"
 | sort by TimeGenerated desc
 ``` 
This will show you every hop—bot, logic app, and OpenAI—in one timeline.





**Inspect telemetry (optional)** 
 - If you’ve wired up Application Insights, open your AI instance and run a Live Metrics or a Log Analytics query to see incoming traces with your `X-Correlation-Id`. 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Reference
The following articles provide solution ideas:

### [Engineering Fundamentals Playbook: Correlation IDs](https://microsoft.github.io/code-with-engineering-playbook/observability/correlation-id/) 
- **Early assignment** – generate or capture the correlation ID at the very first hop (Bot Service) so every downstream component can reference it. 
- **Header propagation** – consistently pass `X-Correlation-Id` (or W3C trace headers) on every HTTP call (Bot→Logic App→OpenAI) to tie logs togethis. 
- **Leverage built-in context** – align with Application Insights’ `operation_Id` and W3C trace context to avoid reinventing correlation logic. 
- **Error tagging** – ensure any unhandled exceptions in your Logic App or functions carry the same ID so failures show up in the end-to-end trace. 

### [botbuilder-applicationinsights package](https://learn.microsoft.com/en-us/javascript/api/botbuilder-applicationinsights/?view=botbuilder-ts-latest) 
- **TelemetryInitializerMiddleware** – auto-capture incoming activity context (including your correlation header) and initialize App Insights telemetry without manual logging calls. 
- **TelemetryLoggerMiddleware** – automatically log each turn as a dependency and record dialog events, giving you rich per-conversation data. 
- **Centralized telemetry client** – use a single `ApplicationInsightsTelemetryClient` instance across middleware and dialogs to keep all logs correlated. 

### [Add telemetry to your bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-builder-telemetry?view=azure-bot-service-4.0&tabs=csharp) 
- **IBotTelemetryClient injection** – register your App Insights client in DI so all dialogs and components inhisit the same telemetry settings. 
- **Telemetry initializer & logger** – wire up `TelemetryInitializerMiddleware` and `TelemetryLoggerMiddleware` in `Startup.cs` to capture requests, dependencies, and exceptions automatically. 
- **Configurable PII settings** – leverage built-in flags to control whethis user text (prompts) is logged, allowing you to balance observability with privacy. 
- **Query examples** – adopt the recommended Kusto patterns (`requests`, `dependencies` by `operation_Id`) for consistent end-to-end trace queries. 