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

## Information Flow 
• Teams sends message to Azure Bot Channels Registration 
• Bot Channels Registration forwards activity to App Service endpoint 
• Bot middleware extracts “X-Correlation-Id” header or generates a new GUID 
• Bot logs incoming request to Application Insights with operation_Id set to the correlation id 
• Bot issues HTTP request to Azure OpenAI, including the “X-Correlation-Id” header 
• Application Insights logs the dependency call with the same operation_Id 
• Bot receives response, logs response telemetry, then returns message to Teams