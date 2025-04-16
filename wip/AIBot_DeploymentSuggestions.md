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

### Resources  
A centralized Log Analytics workspace exists in the primary subscription (diagnostic settings auto‑deployed via Azure Policy), but Application Gateway and API Management reside in separate subscriptions.
- **RECOMMENDATION**: Validate that diagnostic settings for Application Gateway and API Management subscriptions point to the central Log Analytics workspace  
- **RECOMMENDATION**: Confirm that those subscriptions have the necessary role assignments or policy exemptions to send logs across subscription boundaries  
- **QUESTION**: Are there any additional permissions, private link configurations, or DNS considerations required for cross‑subscription log ingestion?

### Logs, Metrics & Alerts  
Logs and metrics are captured centrally, but cross‑subscription resources (API Management, Application Gateway) must be included.
- **RECOMMENDATION**: Ensure diagnostic settings in the API Management and Application Gateway subscriptions forward logs and metrics to the central Log Analytics workspace  
- **RECOMMENDATION**: Create Azure Monitor alert rules that span subscriptions, incorporating metrics and logs from both the primary and peered subscriptions  
- **QUESTION**: Which team will own the configuration and ongoing maintenance of these cross‑subscription alert rules and dashboards?

#### Log Categories by Resource Type

| **Type** | **Categories** | **Documentation** |
| :--- | :--- | :--- |
| AI Services | Audit<br>AzureOpenAIRequestUsage<br>RequestResponse<br>Trace | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-cognitiveservices-accounts-logs) |
| API Management | DeveloperPortalAuditLogs<br>GatewayLogs<br>WebSocketConnectionLogs<br>(Plus: GatewayLlmLogs for AI usage) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-apimanagement-service-logs#:~:text=Portal%20usage%20APIMDevPortalAuditDiagnosticLog) |
| App Service Plan | AppServiceHTTPLogs<br>AppServiceAppLogs<br>AppServiceAuditLogs<br>AppServiceConsoleLogs<br>AppServiceFileAuditLogs<br>AppServicePlatformLogs<br>AppServiceAntivirusScanAuditLogs<br>AppServiceIPSecAuditLogs<br>AppServiceAuthenticationLogs (Preview) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-web-sites-logs) |
| Application Gateway | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog<br>ApplicationGatewayPerformanceLog | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-network-applicationgateways-logs#:~:text=AzureDiagnostics) |
| Bot Service | BotRequest (requests from channels to the bot and vice versa) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-botservice-botservices-logs#:~:text=ABSBotRequests) |
| Container Apps (Prompt Flow)   | ContainerAppConsoleLogs<br>ContainerAppSystemLogs             | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-app-managedenvironments-logs) |
| Search Service | OperationLogs | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-search-searchservices-logs#:~:text=Operation%20Logs%20AzureDiagnostics) |
| Storage Account | StorageRead, StorageWrite, StorageDelete<br>(for Blob, File, Queue, Table services) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-storage-storageaccounts-blobservices-logs#:~:text=StorageBlobLogs) |

<!-- ------------------------- ------------------------- -->

#### Required Logs

##### "Unhandled Exceptions"

- Intent: "Catch-all" monitoring of unexpected errors across all resources







Category values:

• **Audit** – This category logs administrative, security, and configuration events. It’s generally used for tracking changes or policy actions rather than reporting service exceptions.
• **AzureOpenAIRequestUsage** – This records usage metrics (like token consumption) and is not focused on errors or exceptions.
• **RequestResponse** – This category logs full HTTP transactions, including the status codes returned. If an API call fails—such as receiving a 401, 404, or 500 error—it will show up here. It’s your best candidate for finding exceptions or error responses from the OpenAI service.
• **Trace** – This provides detailed diagnostic information and can include error messages and stack traces when the system logs an unexpected event. It’s also useful for troubleshooting issues, as it may capture more granular exception details.










##### General Application Error Tracking
1.	App Insights alerting on unhandled exception
2.	What are the most common exceptions and what is the source?
3.	Does the bot app service / orchestration container app restart unexpectedly (# of restarts over the last X days, separate counts for bot / orchestrator)

##### Performance
1.	What is the average message response time from bot to user?
2.	Which specific operations are introducing the most latency (is this possible? can we differentiate between different aspects like file uploads, querying the LLM, general network latency response times, etc? If so, how complex is this to log?)
3.	How often do users experience timeouts when asking a **question** (when timeouts occur, is there some common pattern? i.e. file attached, common source, etc)
4.	What % of messages are augmented with retrieval and what % need no additional context? (this would give great insight on how the bot's being used and how to prioritize service/features. We probably need to come up with a time span upon which to base this query)

##### Security
1.	What % of bot queries come from unauthorized sources? (Find common times of day / sources)
2.	How often are users being rate limited? (if at all? are rate limits originating from common sources or resulting from common query scenarios)
3.	How often (if ever) do queries hit the container app from sources other than the bot? (Should be never, but would like to make sure that’s the case) 

#### Sample Alerts

> Need to review these with Adam Ray on the next call

1.	Unhandled Exceptions (any and all hits from the App Insights exceptions table)
2.	HTTP 5xx responses (5xx hits from AppServiceHTTPLogs)
3.	Container Restarts / App Service Restarts (What's the best source for these logs?)
4.	Latency / response time thresholds (I'm thinking 2-3 seconds for initial response, does MS have recommended threshholds? )
5.	Abrupt increase in message count / spikes. (ideally this would indicate spam or misuse, but maybe this is being overly protective? would this be valuable info to alert on? )
6.	CPU/Memory threshold (80% unless MS disagrees)
7.	MIssing JWT token (this would indicate the message didnt originate from teams / related to detecting spam or unexpected sources of ingress)

<!-- ------------------------- ------------------------- -->

## Cost Management

### Cost Management → Monitoring  
Cost tracking currently focuses on the primary subscription; peered subscriptions (hosting Application Gateway and API Management) are not yet included.
- **RECOMMENDATION**: Expand the centralized cost dashboard to ingest and display costs from the peered subscriptions alongside the primary subscription  
- **RECOMMENDATION**: Apply consistent tagging across all subscriptions (including the Application Gateway and API Management resource groups) to enable unified cost reporting  
- **QUESTION**: Will the peered subscriptions be consolidated under the same billing scope, and how should budgets and alerts be configured to cover cross‑subscription expenses?

### Budget Forecasting  
No proactive strategy is currently defined for forecasting expenses in line with **scaling demands**.

- **RECOMMENDATION**: Use collected cost data to accurately forecast expenses as the user base expands  
- **RECOMMENDATION**: Establish a recurring process for reviewing spending trends and adjusting resource allocation based on real-world usage  
- **QUESTION**: How will the customer integrate budget forecasting into scaling and resource planning decisions?

<!-- ------------------------- ------------------------- -->

## Stress Testing  
The current solution has been validated at pilot loads (~10 users) but must undergo comprehensive stress testing to simulate production-level conditions (~2,000 users).

#### Component-Level Testing  
- **RECOMMENDATION**: Validate that each key resource (e.g., web app, container apps, API endpoints) functions correctly under baseline loads  
- **RECOMMENDATION**: Execute targeted tests to confirm input handling and basic throughput at low load  
- **QUESTION**: Have all individual components been thoroughly tested at baseline loads

#### Integrated End-to-End Testing 
- **RECOMMENDATION**: Use Azure Load Testing to simulate distributed user activity and validate end-to-end system performance under realistic, production-like conditions
- **RECOMMENDATION**: Simulate full user journeys (from login through chat interactions to file uploads) to confirm complete system functionality  
- **RECOMMENDATION**: Conduct tests that cover interactions across multiple resources and services  
- **QUESTION**: What is the customer’s approach for integrated, end-to-end testing as the system scales to 2,000 users?

#### Stress Testing  
- **RECOMMENDATION**: Use Chaos Studio to inject sustained, high‑load scenarios against all components (including cross‑subscription services) to identify breaking points and observe degradation/recovery  
- **QUESTION**: What sustained load levels (e.g., concurrent sessions) should define our stress tests, and what failure/recovery criteria will we monitor?

#### Spike Testing  
- **RECOMMENDATION**: Use Chaos Studio or Azure Load Testing to simulate sudden traffic surges and measure auto‑scaling response times and error rates across both primary and peered subscriptions  
- **QUESTION**: What spike patterns (magnitude and duration) should we test to validate resilience under burst traffic?

#### Soak Testing  
- **RECOMMENDATION**: Run long‑duration tests at or near peak load (e.g., 2,000 users for several hours) to detect resource exhaustion, memory leaks, and cumulative errors across all subscriptions  
- **QUESTION**: How long should soak tests run, and what resource‑health metrics will signal the need for remediation?

#### Performance Monitoring  
- **RECOMMENDATION**: Activate centralized logging and log analytics to capture performance metrics during stress tests  
- **RECOMMENDATION**: Establish clear baseline metrics (a "green state") for critical components to quickly detect deviations  
- **RECOMMENDATION**: Leverage performance data to identify and address bottlenecks early in the testing process  
- **QUESTION**: Which performance metrics will be used to validate stress test results and guide scaling decisions?

<!-- ------------------------- ------------------------- -->

## Post-Deployment

After the initial launch, focus on:

- **User Interface Enhancements**: Plan future iterations to incorporate visualizations of uploaded PDF content and embed response forms (for example, using adaptive cards) to improve user experience

- **Operational Maintenance**: Define a clear plan for ongoing system maintenance, including regular performance reviews, incident management, and patch updates

- **CI/CD Integration**: Implement continuous integration and deployment pipelines to ensure smooth updates and rapid recovery from any issues

- **Ongoing Monitoring & Feedback**: Establish processes for continuous monitoring of system performance, reliability, and cost metrics

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

## Probably "Overkill"...
The following topics were suggested after a "deep research" review:

### Security
- Add explicit mention of **DDoS Protection Standard** at the network level to cover Layer 3/4 attacks
- Clarify **encryption at rest and in transit** is enforced across services (AI data, chat logs, files)
- Mention use of **Azure Key Vault** for any future secrets or certificates (if needed for bot registration or integrations)
- Explicitly describe the **Teams-to-Bot traffic path**, including Application Gateway’s public IP and its protection (e.g., IP filtering, WAF rules)
- Ensure all **private DNS zone** configurations are defined (centralized or per-subscription) for private endpoint resolution
- Consolidate repeated **PIM (Privileged Identity Management)** references into a single, focused **recommendation**

### Scalability
- Add **recommendation** to **define region and availability zone strategy** (e.g., single-region/multi-zone vs. multi-region)
- Add a callout to **add replicas/partitions for Azure Search** to meet SLA and support higher QPS
- Include note to **validate service-level quotas and subscription limits** for scaling (e.g., vCPU limits, OpenAI throughput)
- Define rough estimates for **expected peak throughput/concurrent sessions** to inform SKU choices
- Reference potential **sharding or load balancing** for AI/Search if request volume is very high

### Monitoring
- Explicitly state that **Application Insights SDK** should be enabled in App Service, Bot Service, and Prompt Flow container apps
- Add **recommendation** to **set log retention periods** explicitly and review storage/cost tradeoffs
- Suggest **limiting Log Analytics access via RBAC** and possibly integrate with **Microsoft Sentinel** for security monitoring
- Recommend **testing alert rules and diagnostics post-deployment** to validate coverage

### Cost Management
- Add **recommendation** to **set up cost alerts** (e.g., if usage exceeds budget)
- Suggest evaluating **reserved instances or savings plans** after usage patterns stabilize
- Reconfirm all subscriptions are **under the same billing scope** for unified tracking

### Stress Testing
- Rename subsection “Stress Testing” to **“Sustained Load Testing”** to avoid redundancy with section title
- Add **recommendation** to **test in a production-parity environment** to avoid polluting production data
- Suggest defining **clear performance pass/fail thresholds** before testing (e.g., response time targets)
- Ensure test datasets (e.g., for Search, AI) **match production data volume and complexity**

### Post-Deployment
- Add **Disaster Recovery/Failover strategy** section (even if it's “not in scope for launch”)
- Recommend **periodic security assessments or pen tests** after go-live
- Add note about **user training or internal documentation** for bot usage
- Clarify ownership of **maintenance, CI/CD, and monitoring responsibilities**