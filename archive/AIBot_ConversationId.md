# AI Bot: Conversation Identifier

> NOTE: SUPPORT ENGINEER SAYS BOT SERVICE >> LOGIC APP IS NOT A SUPPORTED SCENARIO!!!

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

- **Function App** `{prefix}fa`: Consumption, Runtime stack `.NET 8`, Version `(LTS), in-process model`, "Enable public access", with dependencies:
  - **Storage Account** `{prefix}sa` (v1)
  - **Application Insights** `{prefix}ai`

- **API Management** `{prefix}am`: Consumption with dependency **Log Analytics Workspace** `{prefix}law`
- **Bot** `{prefix}bot001`: Multi Tenant with dependency **App Registration**: `{prefix}bot001` and Secret
- **Bot Framework Emulator**: Download latest {e.g., https://github.com/Microsoft/BotFramework-Emulator/releases/tag/v4.15.1}

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Fundamentals

Every message between a channel (or user) and your bot is sent as a JSON Activity object over HTTPS

### Inbound
Messages **from Azure Bot** to logic processing resource {e.g., Function App} must be in the following JSON format:

```json
{
  "type": "message",
  "id": "bf3cc9a2f5de...",                      
  "timestamp": "2016-10-19T20:17:52.2891902Z",
  "serviceUrl": "https://smba.trafficmanager.net/teams/",
  "channelId": "msteams",
  "from": {
    "id": "1234abcd",
    "name": "User Name"
  },
  "conversation": {
    "id": "abcd1234",
    "name": "Conversation Name"
  },
  "recipient": {
    "id": "12345678",
    "name": "Bot Name"
  },
  "text": "Hello World"
}
```

### Outbound
Responses from logic processing resource {e.g., Function App} **to Azure Bot** must be in the following JSON format:

```json
{
  "type": "message",
  "from": {
    "id": "12345678",
    "name": "Bot Name"
  },
  "conversation": {
    "id": "abcd1234",
    "name": "Conversation Name"
  },
  "recipient": {
    "id": "1234abcd",
    "name": "User Name"
  },
  "text": "Hello! How can I assist you today?",
  "replyToId": "bf3cc9a2f5de..."
}
```

[API reference for the Bot Framework Connector service](https://docs.azure.cn/en-us/bot-service/rest-api/bot-framework-rest-connector-api-reference?view=azure-bot-service-4.0)

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

### Function App: Create Function

<!-- ------------------------- ------------------------- -->

#### Environment Variables

Navigate to Azure Portal >> Function App `{prefix}fa` >> **Environment Variables** and iteratively add the following variables:

Name | Value
:--- | :---
`OPENAI_API_KEY` | <OpenAI_Key1>
`OPENAI_ENDPOINT` | <OpenAI_Key1> (no trailing `/`)

Click **Apply** and then **Confirm**.

<!-- ------------------------- ------------------------- -->

#### Create Function

Navigate to Azure Portal >> Function App `{prefix}fa` >> **Create in Azure portal** and click **Create Function**.

On the **Create function** popout >> **Select a template** tab, select **HTTP trigger** and then click **Next**.

On the ...**Template details** tab, enter **Function name** `PromptFunction`, confirm **Authorization level** `Function` and then click **Create**.

<!-- ------------------------- -->

#### Code + Test: `function.json`

Open `PromptFunction` and select path `{prefix}fa/PromptFunction/function.json`.

Replace default logic with:

```json
{
  "bindings": [
    {
      "authLevel": "function",
      "type":      "httpTrigger",
      "direction": "in",
      "name":      "req",
      "route":     "api/messages",
      "methods":   [ "post" ]
    },
    {
      "type":      "http",
      "direction": "out",
      "name":      "$return"
    }
  ]
}
```

Click **Save**.

<!-- ------------------------- -->

#### Code + Test: `run.csx`

Open `PromptFunction` and select path `{prefix}fa/PromptFunction/run.csx`.

Replace default logic with:

```csharp
#r "Newtonsoft.Json"
using System.Net;
using System.Net.Http;
using System.Text;
using Newtonsoft.Json.Linq;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log)
{
    var raw        = await req.Content.ReadAsStringAsync();
    var input      = JObject.Parse(raw);
    var text       = input["text"]?.ToString()                       ?? "";
    var from       = input["from"]        as JObject                ?? new JObject();
    var to         = input["recipient"]   as JObject                ?? new JObject();
    var conv       = input["conversation"]as JObject                ?? new JObject(new JProperty("id","conv1"));
    var replyToId  = input["id"]          ?.ToString()              ?? "";
    var serviceUrl = input["serviceUrl"]  ?.ToString()              ?? "";

    using var client = new HttpClient();
    client.DefaultRequestHeaders.Add("api-key", Environment.GetEnvironmentVariable("OPENAI_KEY"));
    var endpoint = Environment.GetEnvironmentVariable("OPENAI_ENDPOINT");
    var url      = $"{endpoint}/openai/deployments/gpt-35-turbo/chat/completions?api-version=2023-05-15";

    var payload = new JObject(
      new JProperty("messages", new JArray(
        new JObject(
          new JProperty("role",    "user"),
          new JProperty("content", text)
        )
    )));

    var resp  = await client.PostAsync(
                    url,
                    new StringContent(payload.ToString(), Encoding.UTF8, "application/json"));
    var reply = JObject.Parse(await resp.Content.ReadAsStringAsync())
                     ["choices"]?[0]?["message"]?["content"]
                     .ToString() ?? "";

    // now include 'type' and 'serviceUrl' as part of the reply Activity
    var result = new JObject(
      new JProperty("type",         "message"),
      new JProperty("serviceUrl",   serviceUrl),
      new JProperty("from",         from),
      new JProperty("recipient",    to),
      new JProperty("conversation", conv),
      new JProperty("replyToId",    replyToId),
      new JProperty("text",         reply)
    );

    return new HttpResponseMessage(HttpStatusCode.OK)
    {
        Content = new StringContent(result.ToString(), Encoding.UTF8, "application/json")
    };
}
```

Click **Save**.

<!-- ------------------------- -->

#### Code + Test: Test/Run

Default **Test/Run** functionality posts to `/api/<FunctionName>` and ignores the custom `/api/messages/` route.

To test the endpoint we must call it like any other API call:

```powershell
Invoke-RestMethod `
  "https://imfa.azurewebsites.net/api/messages" `
  -Method POST `
  -Headers @{ "Content-Type"="application/json" } `
  -Body '{
    "type":"message",
    "id":"12345",
    "conversation":{ "id":"conv1" },
    "serviceUrl":"https://example.org/",
    "from":      { "id":"user1","name":"User" },
    "recipient": { "id":"bot","name":"Bot" },
    "text":"Hello world"
  }'
```

Click **Test/Run** and on the resulting popout >> **Input** tab, enter **Body** `{ "text": "Hello World" }`.

**Expected HTTP Response Content**

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