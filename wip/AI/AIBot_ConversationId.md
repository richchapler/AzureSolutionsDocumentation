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
- **Resource Group** `{prefix}`
- **OpenAI** `{prefix}oa`: with Deployment model `gpt-35-turbo`
- **Logic App** `{prefix}la`: Consumption with dependency **Log Analytics Workspace** `{prefix}law` and Diagnostic Setting
- **API Management** `{prefix}am`: Consumption with dependency **Log Analytics Workspace** `{prefix}law`
- **Bot** `{prefix}bot001`: Multi Tenant with dependency **App Registration**: `{prefix}bot001` and Secret
- **Bot Framework Emulator**: Download latest {e.g., https://github.com/Microsoft/BotFramework-Emulator/releases/tag/v4.15.1}

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Solution Instructions

### OpenAI: Confirm Prompt-ability

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
Invoke-RestMethod -Method Get -Uri "$((az cognitiveservices account show --resource-group {prefix} --name {prefix}oa --query properties.endpoint -o tsv).TrimEnd('/'))/openai/models?api-version=2023-10-01-preview" -Headers @{ 'api-key' = (az cognitiveservices account keys list --resource-group {prefix} --name {prefix}oa --query key1 -o tsv) }
```

**Expected Output**
```plaintext
data
----
{@{status=succeeded; capabilities=; lifecycle_status=generally-available; deprecation=; id=dall-e-3-3.0; created_at=1691712000; updated_at=1691712000; object=model}, @{…
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Logic App: Create Prompt Workflow

Navigate to Azure Portal >> Logic App `{prefix}la` >> **Development Tools** >> **Logic app designer**.

#### Add Trigger: Request

Click **Add a trigger**, search for and select **Request** >> **When an HTTP request is received**.

Complete the **When an HTTP request is received** popout form:

Setting | Value
:--- | :---
Method | POST
Request Body JSON Schema | `{"type":"object","properties":{"type":{"type":"string"},"text":{"type":"string"},"from":{"type":"object"},"conversation":{"type":"object"},"serviceUrl":{"type":"string"}},"required":["text"]}`

Click **Save** and then copy the **HTTP URL** value for later use.

<!-- ------------------------- -->

#### Add Action: Compose

Click **+** underneath the **When an HTTP request is received** trigger and then **Add an action**.

On the **Add an action** popout, search for and select **Data Operations** >> **Compose**.

On the **Compose** popout form, enter **Inputs** value: `@{triggerBody()?['text']}`.

Click **Save**.

<!-- ------------------------- -->

#### Add Action: HTTP 

Click **+** underneath the **Compose** trigger and then **Add an action**.

On the **Add an action** popout, search for and select **HTTP** >> **HTTP**.

On the **HTTP** popout, complete the form:
- URI: `https://westus.api.cognitive.microsoft.com/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15`
- Method: POST
- Headers: `Content-Type` : `application/json` and `api-key` : `[**REPLACE with OpenAI, KEY 1**]`
- Body: `{"messages":[{"role":"user","content":"@{outputs('Compose')}"}],"temperature":1,"top_p":1,"stream":false,"max_tokens":4096,"n":1}`

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

Click **Run** >> **Run with payload**, and on the resulting pane, paste **Body** value: `{"type":"message","text":"Hello World"}`

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
Invoke-RestMethod "[**REPLACE with Logic App, HTTP URL**]" -Method POST -Headers @{ "Content-Type" = "application/json" } -Body '{ "type":"message","text":"Hello World" }'
```

**Expected Output**

You should see a response like the following:

```plaintext
Hello! How can I assist you today?
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### API Management: Produce Clean URL

#### APIs

Navigate to Azure Portal >> Logic App `{prefix}am` >> **APIs** >> **APIs**, then click **+ Add API** >> **HTTP**.

Complete the **Create an HTTP API** popup form:

Setting | Value
:--- | :---
**Display name** and **Name** | `{prefix}bot-api`
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
        <set-backend-service base-url="[**REPLACE with Logic App, HTTP URL (through port)**]" />
        <rewrite-uri template="[**REPLACE with Logic App, HTTP URL (/workflows through invoke)**]" />
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
            <value>[**REPLACE with Logic App, Shared Access Signature**]</value>
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
        <set-backend-service base-url="[**REPLACE with Logic App, HTTP URL (through port)**]" />
        <rewrite-uri template="[**REPLACE with Logic App, HTTP URL (/workflows through invoke)**]" />
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
            <value>[**REPLACE with Logic App, Shared Access Signature**]</value>
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

Navigate to Azure Portal >> Logic App `{prefix}am` >> **APIs** >> **Products**, then click **+ Add**.

Complete the **Add product** form:

Setting | Value
:--- | :---
**Display name**, **Id**, and **Description** | `{prefix}bot-product`
**Published** | CHECKED
**Requires Subscription** | UNCHECKED

Click **Create**.

Navigate to Product `{prefix}bot-product` then click **+ Add API**.

On the **APIs** popout form, check `{prefix}bot-api` and then click **Select**.

<!-- ------------------------- -->

#### Confirm Success

##### ...with API Management interface

On the **Test** tab, enter **Request body**: `{ "type":"message","text":"Hello World" }`, then click **Send**.

**Expected Output**

You should see a response like the following:

```plaintext
...
Hello! How can I assist you today?
```

Your APIM instance now provides a clean URL (`https://{prefix}am.azure-api.net/api/messages`) that integrates seamlessly with Azure Bot Service.

##### ...with API call


Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

```powershell
Invoke-RestMethod -Method Post -Uri 'https://{prefix}am.azure-api.net/api/messages' -Headers @{ 'Content-Type' = 'application/json' } -Body '{ "type":"message","text":"Hello World" }'
```

**Expected Output**

You should see a response like the following:

```plaintext
Hello! How can I assist you today?
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Bot: Configure and Test

#### Configuration

Navigate to Azure Portal >> Bot `{prefix}bot001` >> **Settings** >> **Configuration** and complete the form.

Setting | Value
:--- | :---
**Messaging endpoint** | `https://{prefix}am.azure-api.net/api/messages`
**Bot Type** | `Multi Tenant`

Click **Apply**.

> BLOCKED BY 401 ISSUES... GOING TO WORK ON CONVERSATION ID FOR FUNCTIONAL ARCHITECTURE