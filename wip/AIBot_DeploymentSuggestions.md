# AI Bot: Deployment Suggestions

## Use Case

Customer is implementing an "AI Bot" solution that should:

* **Replace External AI Dependencies**: Eliminate reliance on external AI solutions {e.g., ChatGPT, Copilot} by delivering a fully internal, secured AI service that meets enterprise compliance and governance standards

* **Integrate with Microsoft Teams**: Provide a natural language prompt AI directly within Microsoft Teams, ensuring that all interactions occur in a secure, centralized environment for enhanced productivity

* **Ensure Enterprise-Grade Security**: Maintain rigorous internal security protocols to keep AI interactions, data storage, and search capabilities fully within the secured network, safeguarding sensitive information from exposure to external services

* **Scale for Broad User Adoption**: Transition from development / testing stage (supporting ~10 users), to a robust production environment capable of seamlessly serving approximately 2,000 users

<!-- ------------------------- ------------------------- -->

## Security

### Endpoints 
All resources connect via private endpoints within their respective subscriptions—including Application Gateway and API Management, which live in separate subscriptions peered via VWan.
- **RECOMMENDATION**: Validate that private endpoint DNS resolution and connectivity work correctly across subscriptions over the VWan peering 
- **RECOMMENDATION**: Ensure Network Security Groups and route tables are consistently applied in each subscription to enforce segmentation 
- **QUESTION**: How are private DNS zones and peering configurations managed to guarantee secure, reliable access between the peered subscriptions?

### Firewall 
We need to learn more about network access controls beyond the current setup.
- **QUESTION**: Is a hard firewall block against external AI solutions (e.g., ChatGPT, Copilot) necessary, or is guidance on using the internal solution sufficient? 
- **QUESTION**: How does the customer plan to handle access from outside the private network—should all services remain strictly behind a VPN/internal firewall?

### Identities 
All resources use managed identities (system- or user-assigned).
- **RECOMMENDATION**: Ensure robust, isolated Role-based Access Control policies are configured for all resources 
- **RECOMMENDATION**: Leverage system-assigned identities for standard components while using user-assigned identities for services requiring tighter control {e.g., the bot app and container apps}
- **QUESTION**: Does the customer have interest in integrating additional measures like Privileged Identity Management to further secure and simplify the management of elevated access?

### Authentication 
Authentication is handled by Entra ID.
- **RECOMMENDATION**: Incorporate multi-factor authentication to add another layer of protection 
- **RECOMMENDATION**: Explore integrating Privileged Identity Management to ease and secure the existing Role-based Access Control framework

### Access 
Usage must be secured to only specific, authorized users and identities.
- **RECOMMENDATION**: Restrict invocation of Prompt Flow container apps to the Bot App Service’s identity via RBAC and access policies
- **RECOMMENDATION**: Conduct thorough security reviews for the bot service to confirm that only authorized individuals gain access
- **RECOMMENDATION**: Consider enforcing further controls, such as strict IP whitelisting or advanced firewall rules

### Access Chain 
A clear mapping of which identities access which resources is needed to enforce least‑privilege across the entire prompt flow

- **RECOMMENDATION**: Document the full access chain—e.g. 
 - User (via Teams) → Bot App Service identity 
 - Bot App Service identity → Prompt Flow container app identity 
 - Prompt Flow container app identity → OpenAI (AI Services) and Azure Search identities 
 - AI/Search identities → Storage (if used for context or logs) 
- **RECOMMENDATION**: For each hop in the chain, enforce RBAC so that only the upstream identity can invoke the downstream resource (no direct user access to AI Services or Search) 
- **RECOMMENDATION**: Apply network restrictions (NSGs, private endpoints) to complement RBAC and prevent any unauthorized lateral access 
- **QUESTION**: Can the customer provide a detailed resource dependency diagram to validate and enforce these least‑privilege controls?

### Access Segmentation 
Clarity is needed on potential isolation and segmentation requirements beyond the current Role-based Access Control configuration.
- **QUESTION**: Are there specific isolation or segmentation requirements for different components (e.g., the bot service versus API endpoints) that exceed our current Role-based Access Control setup? 
- **QUESTION**: How will the customer validate that the current setup (private endpoints, VPN/internal firewall, and managed identities) meets their security expectations?

<!-- ------------------------- ------------------------- -->

## Scalability
The current solution supports development and testing (with ~10 users) and must scale to support ~2,000 users.

### Resource Configuration
Critical components are configured (in general) for development / test workloads.

- **RECOMMENDATION**: Adjust selections as needed (see Resource Inventory section below) 

 | **Type** | **Tier** | **Estimated Scale** | **Ready?** | **Recommendation** |
 | :--- | :--- | :--- | :--- | :--- |
 | AI Services (OpenAI) | **GPT 4o "standard" deployment model** | Standard deployments support light-to-moderate workloads; provisioned capacity may be needed at higher loads | ❓ | Consider moving to a provisioned (dedicated capacity) model if load testing warrants it |
 | API Management | Premium – stv2 | Built for high throughput – can typically support thousands of requests per second | ✅ | Already production-ready; verify that scaling and SKU choices account for any cross‑subscription latency or peering constraints |
 | App Service Plan | P0v3 (Premium) | A baseline instance may support approximately 250–500 concurrent connections; scale-out is available | ✅ | N/A – Already production-ready |
 | App Service Web App | **Premium v3** (P1v3, P2v3) | Each P1v3 instance supports ~250–500 concurrent connections; autoscaling can support 2000+ users | ❓ | Use Premium v3 with autoscaling and confirm performance via load testing |
 | Application Gateway | WAF v2 | Enterprise-grade – designed to handle tens of thousands of requests per minute | ✅ | Already production-ready; verify that scaling and SKU selections consider cross‑subscription peering impacts |
 | Bot Service | **F0 (Free Tier)** | Supports only ~10–20 concurrent requests; suitable for testing | ❓ | Upgrade to the **S1 (Standard)** tier for production use |
| Container Apps (Prompt Flow) | Consumption-based (vCPU/memory) or Dedicated D1 | Each instance handles ~20–50 prompt executions; autoscaling adds instances as needed | ❓ | Define SKU, instance count, and resource allocation based on demand; enable KEDA autoscaling to support both horizontal and vertical scaling |
 | Search Service | **Standard** | A single replica usually handles hundreds of queries per second; replicas/partitions boost capacity further | ✅ | N/A – Already production-ready (with additional scaling options available) |
 | Storage Account | ZRS, StorageV2 (Geo-redundant, V2) | Enterprise-grade throughput supports thousands of IOPS and concurrent transactions, subject to limits | ✅ | N/A – Already production-ready |

- **QUESTION**: Have we verified that all subscription‑specific quotas and network egress limits in our peered resources are sufficient to support our expected load?

 > NOTE: During out work session, we confirmed that each subscription may enforce different limits. It’s essential to review these constraints (such as API throttling and network egress) to ensure they align with our performance goals.

### Auto‑Scaling 
Some resources support autoscaling, though full utilization across the solution is not yet confirmed. Note that API Management and Application Gateway autoscaling occur in separate subscriptions peered via VWan.

- **RECOMMENDATION**: Validate that autoscaling configurations for API Management and Application Gateway in their respective subscriptions align with production requirements, and ensure any cross‑subscription latency or policy differences are accounted for.
- **QUESTION**: How will autoscaling events and metrics from these peered subscriptions be aggregated and monitored in the central Log Analytics workspace?
- **QUESTION**: Have we verified that subscription‑level quotas (such as network egress limits) and policy differences will not impact autoscaling behavior?

 > **NOTE:** Our work session confirmed that each subscription may enforce different quotas and policies, so it’s essential to review these constraints to ensure they meet our performance goals.

### Landing Zone (RESUME HERE)
Most workloads are in one subscription (split across resource groups), but Application Gateway and API Management reside in separate subscriptions peered via VWan.
- **QUESTION**: How are resource groups and subscriptions organized (for example, which components live in which subscription and resource group), and how will that structure impact resource selection and scaling strategies? 
- **QUESTION**: What is the planned region and availability‑zone configuration (for example, single region with multi‑zone versus multi‑region deployment) across all subscriptions?
- **QUESTION**: Are there specific network or geographic redundancy or peering requirements that must be incorporated given the multi‑subscription VWan architecture?

### Performance Thresholds 
Key performance triggers and cost strategies for scaling remain to be defined; ensure cross‑subscription components (API Management, Application Gateway) are included.
- **RECOMMENDATION**: Define performance metrics for each critical component—including peered services like API Management and Application Gateway (for example, response times, concurrent connection limits, error thresholds) 
- **RECOMMENDATION**: Develop a cost management plan that factors in network egress and VWan peering charges between subscriptions 
- **QUESTION**: How will performance data from peered subscriptions be aggregated in the central workspace and used to trigger scaling actions? 
- **QUESTION**: Should cost forecasts include inter‑subscription data transfer and peering costs when projecting expenses?

<!-- ------------------------- ------------------------- -->

## Monitoring

### What we know...

**Log Analytics**: Centralized Log Analytics workspace and Diagnostic Settings auto-deployed via Azure Policy

- **QUESTION**: What subscription is the centralized Log Analytics in and do all stakeholders have access?
- **QUESTION**: Do Diagnostic Settings for all resources point to the central Log Analytics workspace?

**Cost Management**: Exclusively per-subscription use of Azure Portal > Cost Management + Billing interface

- **RECOMMENDATION**: Prepare solution for centralized cost management across subscriptions 
- **RECOMMENDATION**: Confirm consistent tagging across all subscriptions {e.g., `CostCenter`} 

**Budget Forecasting**: No proactive strategy for forecasting expenses

- **RECOMMENDATION**: Prepare solution for centralized budget forecasting across subscriptions 
- **RECOMMENDATION**: Establish recurring process for reviewing spending trends and adjusting resource allocation based on real-world usage 

#### Requirements

The following reporting requirements were shared:

##### General Application Error Tracking
1. App Insights alerting on unhandled exception
2. What are the most common exceptions and what is the source?
3. Does the bot app service / orchestration container app restart unexpectedly (# of restarts over the last X days, separate counts for bot / orchestrator)

##### Performance
1. What is the average message response time from bot to user?
2. Which specific operations are introducing the most latency (is this possible? can we differentiate between different aspects like file uploads, querying the LLM, general network latency response times, etc? If so, how complex is this to log?)
3. How often do users experience timeouts when asking a **question** (when timeouts occur, is there some common pattern? i.e. file attached, common source, etc)
4. What % of messages are augmented with retrieval and what % need no additional context? (this would give great insight on how the bot's being used and how to prioritize service/features. We probably need to come up with a time span upon which to base this query)

##### Security
1. What % of bot queries come from unauthorized sources? (Find common times of day / sources)
2. How often are users being rate limited? (if at all? are rate limits originating from common sources or resulting from common query scenarios)
3. How often (if ever) do queries hit the container app from sources other than the bot? (Should be never, but would like to make sure that’s the case)

##### Alerts

1. Unhandled Exceptions (any and all hits from the App Insights exceptions table)
2. HTTP 5xx responses (5xx hits from AppServiceHTTPLogs)
3. Container Restarts / App Service Restarts (What's the best source for these logs?)
4. Latency / response time thresholds (I'm thinking 2-3 seconds for initial response, does MS have recommended threshholds? )
5. Abrupt increase in message count / spikes. (ideally this would indicate spam or misuse, but maybe this is being overly protective? would this be valuable info to alert on? )
6. CPU/Memory threshold (80% unless MS disagrees)
7. MIssing JWT token (this would indicate the message didnt originate from teams / related to detecting spam or unexpected sources of ingress)

<!-- ------------------------- ------------------------- -->

### What we've learned...

#### Tables and Categories by Resource Type

| **Resource Type** | **Table** | **Categories** |
| :--- | :--- | :--- |
| **AI Services (Open AI)** | AzureDiagnostics | Audit<br>AzureOpenAIRequestUsage<br>RequestResponse<br>Trace |
| **API Management** | AzureDiagnostics | DeveloperPortalAuditLogs<br>GatewayLogs<br>WebSocketConnectionLogs<br>GatewayLlmLogs |
| **App Service (Web App)** | AppServiceHTTPLogs<br>AppServiceAppLogs<br>… | **Not Applicable** |
| **Application Gateway** | AzureDiagnostics | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog<br>ApplicationGatewayPerformanceLog |
| **Bot Service** | WebAppConsoleLogs?* | BotRequests |
| **Container Apps** | ContainerAppConsoleLogs<br>ContainerAppSystemLogs | **Not Applicable** |
| **Search Service** | AzureDiagnostics | OperationLogs |
| **Storage Account** | AzureDiagnostics | StorageRead<br>StorageWrite<br>StorageDelete |

<!-- ------------------------- ------------------------- -->

### Exercise: "Catch All" Log

Creation of necessary reporting will take many iterations. The exercise below demonstrates a repeatable process:

* Confirm Diagnostic Settings
* Force Error (to create reportable data)
* Review Logs

We will do this first for OpenAI, then for the Web App, and then produce a combined report.

#### OpenAI

##### Confirm Diagnostic Settings

Open Azure Portal >> Cloud Shell and switch to PowerShell.

Set identifier values for OpenAI and Log Analytics:
```powershell
$resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{prefix}/providers/Microsoft.CognitiveServices/accounts/{prefix}oa"
$workspaceId = "/subscriptions/{subscriptionId}/resourceGroups/{prefix}/providers/Microsoft.OperationalInsights/workspaces/{prefix}law"
```

Check whether Diagnostic Setting has been added on OpenAI:
```powershell
Get-AzDiagnosticSetting -ResourceId $resourceId
```

If nothing is returned, add Diagnostic Setting:
```
New-AzDiagnosticSetting -Name "{prefix}oads" -ResourceId $resourceId -WorkspaceId $workspaceId `
 -Log @(
 @{ Category = "Audit"; Enabled = $true },
 @{ Category = "AzureOpenAIRequestUsage"; Enabled = $true },
 @{ Category = "RequestResponse"; Enabled = $true },
 @{ Category = "Trace"; Enabled = $true }
 ) | Out-Null
```

Use the portal to confirm that the Diagnostic Setting was correctly created.

##### Force Error

Open Azure Portal >> Cloud Shell and switch to PowerShell.

Define an intentionally-misnamed endpoint:
```powershell
$endpoint = "https://{prefix}oa.openai.azure.com/openai/deployments/gpt-foobar/completions?api-version=2022-12-01"
```

Set an HTTP header that contains the OpenAI, "KEY 1" value:
```powershell
$headers = @{ "api-key" = "{OpenAI_KEY1}"; "Content-Type" = "application/json" }
```

Create a tiny payload:
```powershell
$body = @{ prompt = "hi"; max_tokens = 1 } | ConvertTo-Json
```

Invoke the call and capture the expected failure:
```powershell
try { Invoke-WebRequest -Uri $endpoint -Method Post -Headers $headers -Body $body -ErrorAction Stop }
catch { $_.Exception.Response.StatusCode.value__ }
```

The expected output: `404` is typically logged to Log Analytics within minutes.

##### Review Logs

Open Azure Portal >> Log Analytics >> Logs and switch to "KQL Mode".

Run the following KQL:
```kql
AzureDiagnostics
| project TimeGenerated, ResourceId = tolower(ResourceId)
| where ResourceId contains "{prefix}oa"
| order by TimeGenerated desc
```

Validate results. Try running just `AzureDiagnostics` to see the variety of columns that are available.

<!-- ------------------------- ------------------------- -->

#### Web App

##### Confirm Diagnostic Settings

Set identifier values for the Web App and Log Analytics workspace:
```powershell
$resourceId = "/subscriptions/{subscriptionId}/resourceGroups/{prefix}/providers/Microsoft.Web/sites/{prefix}wa"
$workspaceId = "/subscriptions/{subscriptionId}/resourceGroups/{prefix}/providers/Microsoft.OperationalInsights/workspaces/{prefix}law"
```

Check whether Diagnostic Setting has been added on Web App:
```powershell
Get-AzDiagnosticSetting -ResourceId $resourceId
```

If nothing is returned, add Diagnostic Setting:
```powershell
New-AzDiagnosticSetting -Name "{prefix}wads" -ResourceId $resourceId -WorkspaceId $workspaceId -Log @(@{Category="AppServiceHTTPLogs";Enabled=$true}) | Out-Null
```

Use the portal to confirm that the Diagnostic Setting was correctly created.

##### Force Error 

Open Azure Portal >> Cloud Shell and switch to PowerShell.

Define a URL that you know does not exist on the Web App:
```powershell
$endpoint = "https://{prefix}wa-gagqhndmemfeanex.westus-01.azurewebsites.net/this-path-does-not-exist"
```

Trigger the request and surface the status code:
```powershell
try { Invoke-WebRequest -Uri $endpoint -Method Head -TimeoutSec 5 -UseBasicParsing -ErrorAction Stop } catch { $_.Exception.Response.StatusCode.value__ }
```

The expected output `404` is typically logged to Log Analytics within minutes.

##### Review Logs 

Open Azure Portal » Log Analytics » Logs and switch to “KQL Mode”.

Run the following KQL:
```kql
AppServiceHTTPLogs
| project TimeGenerated, ResourceId = tolower(_ResourceId)
| where ResourceId contains "{prefix}wa"
| order by TimeGenerated desc
```

> Note: We use the AppServiceHTTPLogs table because Web Apps do not log to AzureDiagnostics

Validate results. Try running just `AppServiceHTTPLogs` to see the variety of columns that are available.

<!-- ------------------------- ------------------------- -->

#### Combined “All Errors” Query

Open Azure Portal » Log Analytics » Logs and switch to “KQL Mode”.

Run the following KQL:
```kql
AzureDiagnostics
| project TimeGenerated, ResourceId = tolower(ResourceId)
| where ResourceId contains "{prefix}oa"
| take 3
| union (
    AppServiceHTTPLogs
    | project TimeGenerated, ResourceId = tolower(_ResourceId)
    | where ResourceId contains "{prefix}wa"
    | take 3
)
| order by TimeGenerated desc
```

> <u>Notes</u>
> * `take 3` to limit union'd results for presentation
> * `project` columns are severely limited for presentation... a mapping exercise will be required to map columns across log tables

<!-- ------------------------- ------------------------- -->

## Load Testing

The current solution has been validated at pilot loads (~10 users) but must undergo comprehensive stress testing to simulate production-level conditions (~2,000 users).

- **QUESTION:** Are there any specific traffic patterns or workloads expected for the AI Bot (e.g., bursty usage, high concurrency, file uploads)?
- **QUESTION:** Are there latency/throughput targets or SLAs already defined (e.g., max 2‑second response time)?
- **QUESTION:** What’s the target concurrency (e.g., 200 concurrent users)? And what’s the expected RPS (requests per second)?
- **QUESTION:** Are there environments we can safely run stress and soak tests against?
- **QUESTION:** Are all application dependencies (e.g., Azure OpenAI, storage, search) ready for load testing?
- **QUESTION:** Do we have budgetary or time constraints that affect how long or how often we can run tests?

### Load Testing Tool Options


| Tool | Pros | Cons | Ideal Use Cases |
| :--- | :--- | :--- | :--- |
| **Azure Load Testing** | - Fully managed and natively integrated with Azure services and monitoring.<br>- Simplifies setup with visual test editors and seamless integration with Azure Monitor.<br>- Lower operational overhead, with centralized logging and alerting out‑of‑the‑box. | - May have higher costs for extended or very high‑load tests.<br>- Custom scripting may be more limited compared to open‑source solutions. | - Team has familiarity with JMeter or existing JMeter scripts.<br>- Quick integration in Azure DevOps pipelines using the Azure Load Testing extension.<br>- Scenarios requiring tight coupling with Azure Monitor and centralized logging. |
| **K6** | - Highly flexible and scriptable, allowing detailed user journey simulations and custom spike patterns.<br>- Open‑source with strong community support; can be run in containers.<br>- Easily incorporated into Azure DevOps pipelines via self‑hosted agents or Docker tasks. | - Requires management of the test execution environment, including scaling for large tests.<br>- More development effort to set up centralized reporting and integrate with Azure Monitor. | - Complex test scenarios that demand custom scripting and fine‑grained control.<br>- Situations where cost control and custom environment setups (e.g., running tests in existing containers) are prioritized. |

### Load Testing types

| Test Type | Purpose | Recommended Now? | Notes |
| :--- | :--- | :--- | :--- |
| Component-Level | Validate individual services or APIs under expected load | Yes | Low-cost, fast to run. Helps catch regressions early in isolated components. |
| End-to-End (E2E) | Simulate real user journeys across the full system | Yes | Critical for validating performance SLAs (e.g., ≤5s) under realistic usage. |
| Stress Testing | Push system beyond expected limits to identify bottlenecks/failure points | Yes (Periodically) | Useful after baseline/E2E stability. Can be run ad hoc or pre-release. |
| Spike Testing | Simulate sudden surge in traffic to test auto-scaling and recovery | Maybe Later | Important but lower priority initially; do after stress testing is stable. |
| Soak Testing | Run extended tests to detect slow memory leaks or performance degradation | Maybe Later | Valuable before production go-live. Schedule during off-peak hours. |

### Component-Level Testing

- **OBJECTIVE:** Validate that individual resources (web app, container apps, API endpoints) function correctly under baseline (low‑load) conditions. Can be used as smoke test after production deployments.
- **RECOMMENDATIONS:** 
 - **Baseline Validation:** Create isolated tests that send representative requests to each component to confirm proper input handling, error management, and throughput under low concurrency. 
 - **Monitoring:** Use Application Insights and Log Analytics to record latency, response codes, and resource usage. Define baseline metrics (“green state”) for key performance indicators such as average response time and error rates. 

### End-to-End Testing

- **OBJECTIVE:** Simulate distributed, realistic user journeys to validate the complete system from login to chat interactions and file uploads.
- **RECOMMENDATIONS:** 
 - **User Journey Simulation:** Develop end-to-end test scenarios that mimic full user sessions including authentication, sending chat requests, handling file uploads, and response processing. 
 - **Cross-Component Interactions:** Ensure tests cover interactions between the App Service, Container Apps, and any external dependencies (e.g., Azure Cognitive Search or OpenAI API). 

### Stress Testing

- **OBJECTIVE:** Identify breaking points and monitor performance degradation/recovery under sustained high load.
- **RECOMMENDATIONS:** 
 - **Sustained Load Scenarios:** Use either tool to inject load that significantly exceeds baseline usage (e.g., ramp up to high levels of concurrent sessions) to monitor system behavior, error rates, recovery times, and rollback of autoscaling. 
 - **Metrics & Alerting:** Monitor metrics such as CPU, memory utilization, error rates, queue lengths, and response times. Define explicit failure criteria (e.g., >5% of requests exceeding 5 seconds, excessive 5xx errors). 

### Spike Testing

- **OBJECTIVE:** Validate resilience under sudden, unexpected traffic surges.
- **RECOMMENDATIONS:** 
 - **Surge Scenarios:** Configure tests to simulate abrupt increases in concurrent sessions (spike patterns) for short durations. For example, simulate a sudden jump to 500–1,000 concurrent sessions over a short span (e.g., 1–2 minutes). 
 - **Autoscaling Response:** Measure how quickly autoscaling rules are triggered, the time taken to add or remove resources, and the error rate during the surge.

- **QUESTION:** What spike patterns (magnitude and duration) should we test to validate resilience under burst traffic?

### Soak Testing

- **OBJECTIVE:** Evaluate system stability, detect memory leaks, resource exhaustion, and cumulative errors over extended periods.
- **RECOMMENDATIONS:** 
 - **Test Duration:** Run soak tests at or near peak load conditions for extended periods (recommend 4–8 hours) to mimic prolonged real‑world usage.
- **Resource‑Health Metrics:** Track long‑term metrics such as memory and CPU trends, number of container/app restarts, log anomalies, and gradual degradation of throughput or response times. 

### Performance Monitoring

- **OBJECTIVE:** Establish a comprehensive monitoring strategy for validating stress test results and guiding scaling decisions.
- **RECOMMENDATIONS:** 
 - **Centralized Logging:** Aggregate logs and metrics in a central Log Analytics workspace. Ensure App Service, Container Apps, and any API endpoints send logs and performance metrics. 
 - **Key Metrics:** Use response time percentiles (P95, P99), throughput, CPU and memory utilization, instance count, and error rates as primary performance indicators. 
 - **Integration:** Set up automated alerts (via Azure Monitor or Application Insights) for deviations from defined baselines. 

<!-- ------------------------- ------------------------- -->

## Post-Deployment

After the initial launch, focus on:

- **User Interface Enhancements**: Plan future iterations to incorporate visualizations of uploaded PDF content and embed response forms (for example, using adaptive cards) to improve user experience

- **Operational Maintenance**: Define a clear plan for ongoing system maintenance, including regular performance reviews, incident management, and patch updates

- **CI/CD Integration**: Implement continuous integration and deployment pipelines to ensure smooth updates and rapid recovery from any issues

- **Ongoing Monitoring & Feedback**: Establish processes for continuous monitoring of system performance, reliability, and cost metrics