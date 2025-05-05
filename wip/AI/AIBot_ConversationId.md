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

<!-- ------------------------- ------------------------- -->

### Resource Group

#### List Available Subscriptions

```powershell
az account list --output table
``` 

Review the resulting list and copy the `SubscriptionId` value for the subscription you want to use.

<!-- ------------------------- -->

#### Set Active Subscription

```powershell
az account set --subscription "c4ea206c-9e4c-44a8-b6ee-756cea47f04e"
``` 

<!-- ------------------------- -->

#### Create Resource Group

```powershell
az group create --name "im" --location "westus"
```

<!-- ------------------------- ------------------------- -->

### Log Analytics

#### Instantiate Log Analytics
```powershell
az monitor log-analytics workspace create --resource-group "im" --workspace-name "imlaw" --location "westus" --sku "PerGB2018"
```

##### Expected Output
```plaintext
Resource provider 'Microsoft.OperationalInsights' used by this operation is not registered. We are registering for you.
Registration succeeded.
{
  "createdDate": "2025-05-05T14:08:58.8762176Z",
  "customerId": "c0fc52e0-3df6-42ef-a810-6f5d4a2d7087",
  "etag": "\"060308e9-0000-0700-0000-6818c67a0000\"",
  "features": {
    "enableLogAccessUsingOnlyResourcePermissions": true
  },
  "id": "/subscriptions/c4ea206c-9e4c-44a8-b6ee-756cea47f04e/resourceGroups/im/providers/Microsoft.OperationalInsights/workspaces/imlaw",
  "location": "westus",
  "modifiedDate": "2025-05-05T14:08:58.8762176Z",
  "name": "imlaw",
  "provisioningState": "Creating",
  "publicNetworkAccessForIngestion": "Enabled",
  "publicNetworkAccessForQuery": "Enabled",
  "resourceGroup": "im",
  "retentionInDays": 30,
  "sku": {
    "lastSkuUpdate": "2025-05-05T14:08:58.8762176Z",
    "name": "PerGB2018"
  },
  "type": "Microsoft.OperationalInsights/workspaces",
  "workspaceCapping": {
    "dailyQuotaGb": -1.0,
    "dataIngestionStatus": "RespectQuota",
    "quotaNextResetTime": "2025-05-06T12:00:00Z"
  }
}
```

<!-- ------------------------- ------------------------- -->

### OpenAI

#### Instantiate OpenAI
```powershell
az cognitiveservices account create --resource-group "im" --name "imoa" --kind OpenAI --sku S0 --location "westus"
``` 

##### Expected Output
```plaintext
{
  "etag": "\"5b0132c6-0000-0700-0000-6818c6c60000\"",
  "id": "/subscriptions/c4ea206c-9e4c-44a8-b6ee-756cea47f04e/resourceGroups/im/providers/Microsoft.CognitiveServices/accounts/imoa",
  "identity": null,
  "kind": "OpenAI",
  "location": "westus",
  "name": "imoa",
  "properties": {
    "abusePenalty": null,
    "allowedFqdnList": null,
    "apiProperties": {
      "aadClientId": null,
      "aadTenantId": null,
      "additionalProperties": null,
      "eventHubConnectionString": null,
      "qnaAzureSearchEndpointId": null,
      "qnaAzureSearchEndpointKey": null,
      "qnaRuntimeEndpoint": null,
      "statisticsEnabled": null,
      "storageAccountConnectionString": null,
      "superUser": null,
      "websiteName": null
    },
    "callRateLimit": {
      "count": null,
      "renewalPeriod": null,
      "rules": [
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.dalle.post",
          "matchPatterns": [
            {
              "method": "POST",
              "path": "dalle/*"
            },
            {
              "method": "POST",
              "path": "openai/images/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.dalle.other",
          "matchPatterns": [
            {
              "method": "*",
              "path": "dalle/*"
            },
            {
              "method": "*",
              "path": "openai/operations/images/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 120.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.assistants.list",
          "matchPatterns": [
            {
              "method": "GET",
              "path": "openai/assistants"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 120.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.threads.list",
          "matchPatterns": [
            {
              "method": "GET",
              "path": "openai/threads"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 120.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.vectorstores.list",
          "matchPatterns": [
            {
              "method": "GET",
              "path": "openai/vector_stores"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 100000.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.assistants.default",
          "matchPatterns": [
            {
              "method": "*",
              "path": "openai/assistants"
            },
            {
              "method": "*",
              "path": "openai/assistants/*"
            },
            {
              "method": "*",
              "path": "openai/threads"
            },
            {
              "method": "*",
              "path": "openai/threads/*"
            },
            {
              "method": "*",
              "path": "openai/vector_stores"
            },
            {
              "method": "*",
              "path": "openai/vector_stores/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 100000.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.responses.default",
          "matchPatterns": [
            {
              "method": "*",
              "path": "openai/responses"
            },
            {
              "method": "*",
              "path": "openai/responses/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.batches.post",
          "matchPatterns": [
            {
              "method": "POST",
              "path": "openai/batches"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 500.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.batches.get",
          "matchPatterns": [
            {
              "method": "GET",
              "path": "openai/batches/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 100.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai.batches.list",
          "matchPatterns": [
            {
              "method": "GET",
              "path": "openai/batches"
            }
          ],
          "minCount": null,
          "renewalPeriod": 60.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "openai",
          "matchPatterns": [
            {
              "method": "*",
              "path": "openai/*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        },
        {
          "count": 30.0,
          "dynamicThrottlingEnabled": null,
          "key": "default",
          "matchPatterns": [
            {
              "method": "*",
              "path": "*"
            }
          ],
          "minCount": null,
          "renewalPeriod": 1.0
        }
      ]
    },
    "capabilities": [
      {
        "name": "VirtualNetworks",
        "value": null
      },
      {
        "name": "CustomerManagedKey",
        "value": null
      },
      {
        "name": "MaxFineTuneCount",
        "value": "500"
      },
      {
        "name": "MaxRunningFineTuneCount",
        "value": "3"
      },
      {
        "name": "MaxUserFileCount",
        "value": "100"
      },
      {
        "name": "MaxTrainingFileSize",
        "value": "512000000"
      },
      {
        "name": "MaxUserFileImportDurationInHours",
        "value": "1"
      },
      {
        "name": "MaxFineTuneJobDurationInHours",
        "value": "720"
      },
      {
        "name": "MaxEvaluationRunDurationInHours",
        "value": "5"
      },
      {
        "name": "MaxRunningEvaluationCount",
        "value": "5"
      },
      {
        "name": "TrustedServices",
        "value": "Microsoft.CognitiveServices,Microsoft.MachineLearningServices,Microsoft.Search,Microsoft.VideoIndexer"
      },
      {
        "name": "RaiMonitor",
        "value": null
      }
    ],
    "commitmentPlanAssociations": null,
    "customSubDomainName": null,
    "dateCreated": "2025-05-05T14:10:06.3708256Z",
    "deletionDate": null,
    "disableLocalAuth": null,
    "dynamicThrottlingEnabled": null,
    "encryption": null,
    "endpoint": "https://westus.api.cognitive.microsoft.com/",
    "endpoints": {
      "OpenAI Dall-E API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Language Model Instance API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Model Scaleset API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Moderations API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Realtime API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Sora API": "https://westus.api.cognitive.microsoft.com/",
      "OpenAI Whisper API": "https://westus.api.cognitive.microsoft.com/",
      "Token Service API": "https://westus.api.cognitive.microsoft.com/"
    },
    "internalId": "e3e7da52bab84a599e097ed33efc47a7",
    "isMigrated": false,
    "locations": null,
    "migrationToken": null,
    "networkAcls": null,
    "privateEndpointConnections": [],
    "provisioningState": "Succeeded",
    "publicNetworkAccess": "Enabled",
    "quotaLimit": null,
    "restore": null,
    "restrictOutboundNetworkAccess": null,
    "scheduledPurgeDate": null,
    "skuChangeInfo": null,
    "userOwnedStorage": null
  },
  "resourceGroup": "im",
  "sku": {
    "capacity": null,
    "family": null,
    "name": "S0",
    "size": null,
    "tier": null
  },
  "systemData": {
    "createdAt": "2025-05-05T14:10:05.794164+00:00",
    "createdBy": "admin@MngEnvMCAP500976.onmicrosoft.com",
    "createdByType": "User",
    "lastModifiedAt": "2025-05-05T14:10:05.794164+00:00",
    "lastModifiedBy": "admin@MngEnvMCAP500976.onmicrosoft.com",
    "lastModifiedByType": "User"
  },
  "tags": null,
  "type": "Microsoft.CognitiveServices/accounts"
}
```

<!-- ------------------------- ------------------------- -->

### Generative Pre-trained Transformer (GPT)

#### Create Deployment
```powershell
az cognitiveservices account deployment create --resource-group "im" --name "imoa" --deployment-name "gpt-35-turbo" --model-name "gpt-35-turbo" --model-version "0125" --model-format OpenAI --sku-name Standard --sku-capacity 240
``` 

##### Expected Output
```plaintext
{
  "etag": "\"2e1a44b8-45a5-455c-81d8-7794510e182b\"",
  "id": "/subscriptions/c4ea206c-9e4c-44a8-b6ee-756cea47f04e/resourceGroups/im/providers/Microsoft.CognitiveServices/accounts/imoa/deployments/gpt-35-turbo",
  "name": "gpt-35-turbo",
  "properties": {
    "callRateLimit": null,
    "capabilities": {
      "assistants": "true",
      "chatCompletion": "true",
      "maxContextToken": "16385",
      "maxOutputToken": "4096"
    },
    "model": {
      "callRateLimit": null,
      "format": "OpenAI",
      "name": "gpt-35-turbo",
      "source": null,
      "version": "0125"
    },
    "provisioningState": "Succeeded",
    "raiPolicyName": null,
    "rateLimits": [
      {
        "count": 240.0,
        "dynamicThrottlingEnabled": null,
        "key": "request",
        "matchPatterns": null,
        "minCount": null,
        "renewalPeriod": 10.0
      },
      {
        "count": 240000.0,
        "dynamicThrottlingEnabled": null,
        "key": "token",
        "matchPatterns": null,
        "minCount": null,
        "renewalPeriod": 60.0
      }
    ],
    "scaleSettings": null,
    "versionUpgradeOption": "OnceNewDefaultVersionAvailable"
  },
  "resourceGroup": "im",
  "sku": {
    "capacity": 240,
    "family": null,
    "name": "Standard",
    "size": null,
    "tier": null
  },
  "systemData": {
    "createdAt": "2025-05-05T14:44:40.881375+00:00",
    "createdBy": "admin@MngEnvMCAP500976.onmicrosoft.com",
    "createdByType": "User",
    "lastModifiedAt": "2025-05-05T14:44:40.881375+00:00",
    "lastModifiedBy": "admin@MngEnvMCAP500976.onmicrosoft.com",
    "lastModifiedByType": "User"
  },
  "type": "Microsoft.CognitiveServices/accounts/deployments"
}
```

<!-- ------------------------- -->

#### Confirm Success

Invoke OpenAI via REST call to confirm successful chat completion:
```powershell
Invoke-RestMethod -Method Get -Uri "$((az cognitiveservices account show --resource-group im --name imoa --query properties.endpoint -o tsv).TrimEnd('/'))/openai/models?api-version=2023-10-01-preview" -Headers @{ 'api-key' = (az cognitiveservices account keys list --resource-group im --name imoa --query key1 -o tsv) }
```

##### Expected Response
```plaintext
data
----
{@{status=succeeded; capabilities=; lifecycle_status=generally-available; deprecation=; id=dall-e-3-3.0; created_at=1691712000; updated_at=1691712000; object=model}, @{…
```

<!-- ------------------------- ------------------------- -->

### Logic App

#### Configure Azure CLI for automatic extension installs
```powershell
az config set extension.use_dynamic_install=yes_without_prompt
```

##### Expected Response
```plaintext
Command group 'config' is experimental and under development. Reference and support levels: https://aka.ms/CLI_refstatus
```

<!-- ------------------------- -->

#### Install Logic App Extension
```powershell
az extension add --name logic
```

<!-- ------------------------- -->

#### Instantiate Logic App
```powershell
az logic workflow create --resource-group "im" --name "imla" --definition '{"definition":{"$schema":"https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json#","contentVersion":"1.0.0.0","triggers":{},"actions":{},"outputs":{}},"parameters":{}}' --location "westus"
```

##### Expected Response
```plaintext
Resource provider 'Microsoft.Logic' used by this operation is not registered. We are registering for you.
Registration succeeded.
{
  "accessEndpoint": "https://prod-113.westus.logic.azure.com:443/workflows/152752f42bab4073b11fb453a3d9558d",
  "changedTime": "2025-05-05T15:40:09.3727798Z",
  "createdTime": "2025-05-05T15:40:09.375635Z",
  "definition": {
    "$schema": "https://schema.management.azure.com/schemas/2016-06-01/Microsoft.Logic.json#",
    "actions": {},
    "contentVersion": "1.0.0.0",
    "outputs": {},
    "triggers": {}
  },
  "endpointsConfiguration": {
    "connector": {
      "outgoingIpAddresses": [
        {
          "address": "<abridged>"
        }
      ]
    },
    "workflow": {
      "accessEndpointIpAddresses": [
        {
          "address": "<abridged>"
        }
      ],
      "outgoingIpAddresses": [
        {
          "address": "<abridged>"
        }
      ]
    }
  },
  "id": "/subscriptions/c4ea206c-9e4c-44a8-b6ee-756cea47f04e/resourceGroups/im/providers/Microsoft.Logic/workflows/imla",
  "location": "westus",
  "name": "imla",
  "parameters": {},
  "provisioningState": "Succeeded",
  "resourceGroup": "im",
  "state": "Enabled",
  "type": "Microsoft.Logic/workflows",
  "version": "08584551472761213419"
}
```

<!-- ------------------------- ------------------------- -->

### Application Registration

The application registration serves as the bot's identity and credentials:

- **Bot Authentication**: The client identifier and secret become the “Microsoft App ID and password” you enter on the Bot Service; Teams and the Bot Framework use those values to authenticate incoming requests and validate that calls really are coming from your bot

- **Token Acquisition**: When your bot needs to call other services (e.g., pull information via Microsoft Graph or invoke Logic App), it uses its service principal to acquire tokens; assigning RBAC roles to that service principal allows control of exactly which resources the bot may access

<!-- ------------------------- -->

#### Create Application

Applications define the identity, permissions and configuration of your bot in the directory.

```powershell
$appId = az ad app create --display-name "imbs" --query appId -o tsv
```

<!-- ------------------------- -->

#### Create Service Principal

Service Principals are the tenant-specific instance of that application that holds credentials (secrets or certificates) and can be assigned roles.

```powershell
az ad sp create --id $appId
```

##### Expected Output
```plaintext
{      
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#servicePrincipals/$entity",
  "accountEnabled": true,
  "addIns": [],
  "alternativeNames": [],
  "appDescription": null,
  "appDisplayName": "imbs",
  "appId": "8b7df299-eeee-462b-83b0-22fda06843b4",
  "appOwnerOrganizationId": "4a863f70-7a4a-42de-886b-220bf0e2abb2",
  "appRoleAssignmentRequired": false,
  "appRoles": [],
  "applicationTemplateId": null,
  "createdDateTime": null,
  "deletedDateTime": null,
  "description": null,
  "disabledByMicrosoftStatus": null,
  "displayName": "imbs",
  "homepage": null,
  "id": "7b4ce6dd-b055-46aa-94ca-5628bb95dc5d",
  "info": {
    "logoUrl": null,
    "marketingUrl": null,
    "privacyStatementUrl": null,
    "supportUrl": null,
    "termsOfServiceUrl": null
  },
  "keyCredentials": [],
  "loginUrl": null,
  "logoutUrl": null,
  "notes": null,
  "notificationEmailAddresses": [],
  "oauth2PermissionScopes": [],
  "passwordCredentials": [],
  "preferredSingleSignOnMode": null,
  "preferredTokenSigningKeyThumbprint": null,
  "replyUrls": [],
  "resourceSpecificApplicationPermissions": [],
  "samlSingleSignOnSettings": null,
  "servicePrincipalNames": [
    "8b7df299-eeee-462b-83b0-22fda06843b4"
  ],
  "servicePrincipalType": "Application",
  "signInAudience": "AzureADMyOrg",
  "tags": [],
  "tokenEncryptionKeyId": null,
  "verifiedPublisher": {
    "addedDateTime": null,
    "displayName": null,
    "verifiedPublisherId": null
  }
}
```

<!-- ------------------------- -->

#### Create Client Secret

```powershell
$clientSecret = az ad app credential reset --id $appId --append --end-date "2025-05-10T00:00:00Z" --query password -o tsv
``` 

##### Expected Output
```plaintext
WARNING: The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli
```

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Setup Workflow

Navigate to Azure Portal >> Logic App `imla` >> Development Tools >> Logic App Designer.

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