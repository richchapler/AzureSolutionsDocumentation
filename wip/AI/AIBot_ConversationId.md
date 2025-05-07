# AI Bot: Conversation Identifier

## Use Case

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

## Required Resources

Instantiate the following Azure resources:
- **Resource Group** `im`
- **OpenAI** `imoa`: with Deployment model `gpt-35-turbo`
- **Logic App** `imla`: Consumption with dependency **Log Analytics Workspace** `imlaw` and Diagnostic Setting
- **API Management** `imam`: Consumption with dependency **Log Analytics Workspace** `imlaw`
- **Bot** `imbot001`: Multi Tenant with dependency **App Registration**: `imbot001` and Secret
- **Bot Framework Emulator**: Download latest {e.g., https://github.com/Microsoft/BotFramework-Emulator/releases/tag/v4.15.1}

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Solution Instructions

### OpenAI: Confirm Prompt-ability

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
Invoke-RestMethod -Method Get -Uri "$((az cognitiveservices account show --resource-group im --name imoa --query properties.endpoint -o tsv).TrimEnd('/'))/openai/models?api-version=2023-10-01-preview" -Headers @{ 'api-key' = (az cognitiveservices account keys list --resource-group im --name imoa --query key1 -o tsv) }
```

**Expected Output**
```plaintext
data
----
{@{status=succeeded; capabilities=; lifecycle_status=generally-available; deprecation=; id=dall-e-3-3.0; created_at=1691712000; updated_at=1691712000; object=model}, @{…
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Logic App: Create Prompt Workflow

Navigate to Azure Portal >> Logic App `imla` >> **Development Tools** >> **Logic app designer**.

#### Add Trigger

Click **Add a trigger**, search for and select **Request** >> **When an HTTP request is received**.

Complete the **When an HTTP request is received** popout form:

Setting | Value
:--- | :---
Method | POST
Request Body JSON Schema | `{"type":"object","properties":{"prompt":{"type":"string"}},"required":["prompt"]}`

Click **Save** and then copy the **HTTP URL** value for later use.

**Example HTTP URL**

```plaintext
https://prod-72.westus.logic.azure.com:443/workflows/6024321d5af541faa25473ee91e850c0/triggers/When_a_HTTP_request_is_received/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2FWhen_a_HTTP_request_is_received%2Frun&sv=1.0&sig=<sig>
```

<!-- ------------------------- -->

#### Add Action: HTTP 

Click **+** underneath the **When an HTTP request is received** trigger and then **Add an action**.

On the **Add an action** pane, search for and select **HTTP** >> **HTTP**.

On the **HTTP** pane, complete the form:
- URI: `https://westus.api.cognitive.microsoft.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15`
- Method: POST
- Headers: `Content-Type` : `application/json` and `api-key` : `[OpenAI KEY 1]`
- Body: `{"messages":[{"role":"user","content":"@{triggerBody()?['prompt']}"}],"temperature":1,"top_p":1,"stream":false,"max_tokens":4096,"n":1}`

Click **Save**.

<!-- ------------------------- -->

#### Add Action: Response

Click **+** underneath the **HTTP** action and then **Add an action**.

On the **Add an action** pane, search for and select **Request** >> **Response**.

On the **Response** pane, complete the form:
- Status Code: `200`
- Body: `@{body('HTTP')?['choices'][0]?['message']?['content']}`

Click **Save**.

<!-- ------------------------- ------------------------- -->

#### Confirm Success

##### ...with Logic App interface

Click **Run** >> **Run with payload**, and on the resulting pane, paste **Body** value: `{"prompt":"Hello World"}`

Click **Run**. Allow time for processing.

After it completes, navigate to **Overview** >> **Run history**, click to select the latest run, and confirm success.

**Expected Output**

You should see a response like the following:
```plaintext
Hello! How can I assist you today?
```

Once you see a successful run and valid response in the designer, you'll know the endpoint is published and handling requests correctly.

<!-- ------------------------- ------------------------- -->

##### ...with API call

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

```powershell
Invoke-RestMethod "https://prod-72.westus.logic.azure.com:443/workflows/6024321d5af541faa25473ee91e850c0/triggers/When_a_HTTP_request_is_received/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2FWhen_a_HTTP_request_is_received%2Frun&sv=1.0&sig=nWo7QV6M0OAu1QalS9KCFiofRp6VX0d42ld-kAv5U4U" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body '{"prompt":"Hello World"}'
```

**Expected Output**

You should see a response like the following:

```plaintext
Hello! How can I assist you today?
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### API Management: Produce Clean URL

#### APIs

Navigate to Azure Portal >> Logic App `imam` >> **APIs** >> **APIs**, then click **+ Add API** >> **HTTP**.

Complete the **Create an HTTP API** popup form:

Setting | Value
:--- | :---
**Display name** and **Name** | `imbot-api`
**Web service URL** and **API URL suffix** | blank (for now)

Click **Create**.

<!-- ------------------------- -->

##### Operation

Click **+ Add Operation** and then complete the **Frontend** form:

Setting | Value
:--- | :---
**Display name** | `Bot Messages`
**Name** | `bot-messages`
**URL** | POST and `/api/messages`

Click **Save**.

<!-- ------------------------- -->

###### Inbound Processing

Click **Inbound processing** >> **...** >> **Code editor**.

Replace the default code with:

```xml
<policies>
    <inbound>
        <base />
        <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-backend-service base-url="https://prod-72.westus.logic.azure.com:443" />
        <rewrite-uri template="/workflows/6024321d5af541faa25473ee91e850c0/triggers/When_a_HTTP_request_is_received/paths/invoke" />
        <set-query-parameter name="api-version" exists-action="override">
            <value>2016-10-01</value>
        </set-query-parameter>
        <set-query-parameter name="sp" exists-action="override">
            <value>/triggers/When_a_HTTP_request_is_received/run</value>
        </set-query-parameter>
        <set-query-parameter name="sv" exists-action="override">
            <value>1.0</value>
        </set-query-parameter>
        <set-query-parameter name="sig" exists-action="override">
            <value>nWo7QV6M0OAu1QalS9KCFiofRp6VX0d42ld-kAv5U4U</value>
        </set-query-parameter>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

Click **Save**.

<!-- ------------------------- -->

###### Backend

Replace the default code with:
```xml
<policies>
    <inbound>
        <base />
        <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
        </set-header>
        <set-header name="Authorization" exists-action="override">
            <value>@(context.Request.Headers.GetValueOrDefault("Authorization",""))</value>
        </set-header>
        <set-backend-service base-url="https://prod-72.westus.logic.azure.com" />
        <rewrite-uri template="/workflows/6024321d5af541faa25473ee91e850c0/triggers/When_a_HTTP_request_is_received/paths/invoke" />
        <set-query-parameter name="api-version" exists-action="override">
            <value>2016-10-01</value>
        </set-query-parameter>
        <set-query-parameter name="sp" exists-action="override">
            <value>/triggers/When_a_HTTP_request_is_received/run</value>
        </set-query-parameter>
        <set-query-parameter name="sv" exists-action="override">
            <value>1.0</value>
        </set-query-parameter>
        <set-query-parameter name="sig" exists-action="override">
            <value>nWo7QV6M0OAu1QalS9KCFiofRp6VX0d42ld-kAv5U4U</value>
        </set-query-parameter>
    </inbound>
    <backend>
        <forward-request />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

Click **Save**.

<!-- ------------------------- -->

###### Settings

Navigate to **Settings** tab >> **Subscription** section and uncheck **Subscription required**.

Click **Save**.

<!-- ------------------------- -->

#### Products

Navigate to Azure Portal >> Logic App `imam` >> **APIs** >> **Products**, then click **+ Add**.

Complete the **Add product** form:

Setting | Value
:--- | :---
**Display name**, **Id**, and **Description** | `imbot-product`
**Published** | CHECKED
**Requires Subscription** | UNCHECKED

Click **Create**.

Navigate to Product `imbot-product` then click **+ Add API**.

On the **APIs** popout form, check `imbot-api` and then click **Select**.

<!-- ------------------------- -->

#### Confirm Success

##### ...with API Management interface

On the **Test** tab, enter **Request body**: `{ "prompt": "Hello World" }`, then click **Send**.

**Expected Output**

You should see a response like the following:

```plaintext
HTTP/1.1 200 OK
cache-control: no-cache
content-encoding: gzip
content-type: text/plain; charset=utf-8
date: Tue, 06 May 2025 20:19:14 GMT
expires: -1
pragma: no-cache
strict-transport-security: max-age=31536000; includeSubDomains
transfer-encoding: chunked
vary: Accept-Encoding,Origin
x-ms-client-tracking-id: 08584550441310742100938252124CU40
x-ms-correlation-id: b34448a6-9482-4b46-9695-6f0d48289c7d
x-ms-execution-location: westus
x-ms-ratelimit-burst-remaining-workflow-writes: 89999
x-ms-ratelimit-remaining-workflow-download-contentsize: 3221225438
x-ms-ratelimit-remaining-workflow-upload-contentsize: 3221225445
x-ms-ratelimit-time-remaining-directapirequests: 299999494
x-ms-request-id: westus:b34448a6-9482-4b46-9695-6f0d48289c7d
x-ms-tracking-id: b34448a6-9482-4b46-9695-6f0d48289c7d
x-ms-trigger-history-name: 08584550441310742100938252124CU40
x-ms-workflow-id: 6024321d5af541faa25473ee91e850c0
x-ms-workflow-name: imla
x-ms-workflow-run-id: 08584550441310742100938252124CU40
x-ms-workflow-system-id: /locations/westus/scaleunits/prod-72/workflows/6024321d5af541faa25473ee91e850c0
x-ms-workflow-version: 08584550448306832703

Hello! How can I assist you today?
```

Your APIM instance now provides a clean URL (`https://imam.azure-api.net/api/messages`) that integrates seamlessly with Azure Bot Service.

##### ...with API call


Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

```powershell
Invoke-RestMethod -Method Post -Uri 'https://imam.azure-api.net/api/messages/When_a_HTTP_request_is_received/paths/invoke' -Headers @{ 'Content-Type' = 'application/json' } -Body '{ "prompt": "Hello World" }'
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Bot: Configure and Test

#### Configuration

Navigate to Azure Portal >> Bot `imbot001` >> **Settings** >> **Configuration** and complete the form.

Setting | Value
:--- | :---
**Messaging endpoint** | `https://imam.azure-api.net/api/messages`
**Bot Type** | `Multi Tenant`

Click **Apply**.

<!-- ------------------------- -->

#### Lorem

Navigate to Azure Portal >> Bot `imbot001` >> **Settings** >> **Configuration** and complete the form.


LOREM IPSUM







<!-- ------------------------- ------------------------- -->

#### Confirm Success

##### ...with Bot Service interface

Navigate to `imbot001` >> **Settings** >> **Test in Web Chat**.

In the **Type your message** box, try a prompt {e.g., `Hello World`}, then click the Send button.

<!-- ------------------------- ------------------------- -->

##### ...with API call

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

```powershell
Invoke-RestMethod "https://prod-162.westus.logic.azure.com:443/workflows/d61b0fc8cdab49208a7830aa521e3f0f/triggers/When_a_HTTP_request_is_received/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2FWhen_a_HTTP_request_is_received%2Frun&sv=1.0&sig=IunxdWoa_VdKPTY7uxLE3B66t4mMHdmYSR_NLfporCs" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body '{"prompt":"Hello World"}'
```

**Expected Output**

You should see a response like the following:

```plaintext
```







```powershell
$dlSecret='<secret>'; $token=(Invoke-RestMethod -Uri 'https://directline.botframework.com/v3/directline/tokens/generate' -Method POST -Headers @{Authorization="Bearer $dlSecret"}).token; $conv=(Invoke-RestMethod -Uri 'https://directline.botframework.com/v3/directline/conversations' -Method POST -Headers @{Authorization="Bearer $token"}); Invoke-RestMethod -Uri "https://directline.botframework.com/v3/directline/conversations/$($conv.conversationId)/activities" -Method POST -Headers @{Authorization="Bearer $token"} -Body (@{type='message';from=@{id='user1'};text='test-001: What’s a quick soup recipe?'}|ConvertTo-Json); Invoke-RestMethod -Uri "https://directline.botframework.com/v3/directline/conversations/$($conv.conversationId)/activities" -Headers @{Authorization="Bearer $token"}
```

If you see your bot’s reply in the activities list, your setup is confirmed.


<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Teams

Navigate to the Bot, then Settings >> **Channels**, then click to select **Microsoft Teams**.

Select **Microsoft Teams Commercial** and then click **Apply**.

Return to Settings >> **Channels** and you will see **Microsoft Teams** included in the "This bot is connected with..." list.


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

Once your **imbot001** is registered and pointed at your Logic App, you can verify end-to-end functionality directly in the Azure Portal:

##### Bot

Bot Diagnostic Settings capture only "Requests from the channels to the bot"

Logs of category "Requests from the channels to the bot" (aka `ChannelOperationalLogs`) are added any time a configured channel makes an HTTP POST to your bot’s messaging endpoint.

—whether that’s Teams, Direct Line, or the built-in Web Chat test pane in the Portal.

"Test in Web Chat" uses a debug channel that may not trigger full diagnostic logging, especially in newer Bot Framework versions where telemetry is optimized for production channels like Teams, Direct Line, or Web Chat embedded in a live app.

**Open the Bot resource** 
 - In the Azure Portal go to **Resource groups › im › imbot001**. 

**Use the built-in Web Chat tester** 
 - In the left menu select **Test in Web Chat**. 
 - In the chat pane type a sample prompt, for example: 
 ```
 Whats a quick soup recipe?
 ``` 
 - Press Enter and watch for your Logic App–driven response to appear. 


1. **Run your test** 
 - In **imbot001** → **Test in Web Chat**, send: 
 ```text
 test-001: What’s a quick soup recipe?
 ``` 

1. **Open your Log Analytics workspace** 

2. **Query the Bot’s diagnostic logs** 
 Paste and run a query like this: 
 ```kusto
 AzureDiagnostics
 | whise Resource == "imbot001" // your bot’s resource name
 | whise Category == "ChannelOperationalLogs" // the log category you enabled
 | sort by TimeGenerated desc
 | project TimeGenerated, OperationName, Level, Message
 ``` 

1. **Inspect the results** 
 - **TimeGenerated** shows when the message arrived 
 - **OperationName** reveals the step (e.g. "ChannelMessageReceived") 
 - **Message** contains details—any errors or confirmation of delivery 

If you see entries hise, the bot is receiving and logging requests correctly. 

---

**Next steps if you get no results**: 
- Double-check your diagnostic setting on **imbot001** includes **ChannelOperationalLogs**. 
- Add diagnostic settings on the **Logic App** and **Azure OpenAI** resources (so you can trace the entire flow). 
- Then re-run your test and query using a broader union: 
 ```kusto
 union AzureDiagnostics, WorkflowRuntime, CognitiveServicesRequests
 | sort by TimeGenerated desc
 ``` 
This will show you every hop—bot, logic app, and OpenAI—in one timeline.





**Inspect telemetry (optional)** 
 - If you’ve wired up Application Insights, open your AI instance and run a Live Metrics or a Log Analytics query to see incoming traces with your `X-Correlation-Id`. 

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Reference Articles
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