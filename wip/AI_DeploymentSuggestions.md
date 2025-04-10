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
All services connect via private endpoints.
- RECOMMENDATION: Verify that each service also sits behind a VPN or internal firewall to enforce network segmentation  
- QUESTION: Will there be a hard block on external AI solutions (e.g., ChatGPT, Copilot)?

### Firewall 
We need to learn more about network access controls beyond the current setup.
- QUESTION: Is a hard firewall block against external AI solutions (e.g., ChatGPT, Copilot) necessary, or is guidance on using the internal solution sufficient?  
- QUESTION: How does the customer plan to handle access from outside the private network—should all services remain strictly behind a VPN/internal firewall?

### Identities  
All resources use managed identities (system- or user-assigned).
- RECOMMENDATION: Ensure robust, isolated Role-based Access Control policies are configured for all resources 
- RECOMMENDATION: Leverage system-assigned identities for standard components while using user-assigned identities for services requiring tighter control {e.g., the bot app and container apps}
- QUESTION: Does the customer have interest in integrating additional measures like Privileged Identity Management to further secure and simplify the management of elevated access?

### Authentication  
Authentication is handled by Entra ID.
- RECOMMENDATION: Incorporate multi-factor authentication to add another layer of protection 
- RECOMMENDATION: Explore integrating Privileged Identity Management to ease and secure the existing Role-based Access Control framework

### Access  
Usage must be secured to only specific, authorized users.
- RECOMMENDATION: Conduct thorough security reviews for the bot service to confirm that only authorized individuals gain access
- RECOMMENDATION: Consider enforcing further controls, such as strict IP whitelisting or advanced firewall rules

### Access & Segmentation 
Clarity is needed on potential isolation and segmentation requirements beyond the current Role-based Access Control configuration.
- QUESTION: Are there specific isolation or segmentation requirements for different components (e.g., the bot service versus API endpoints) that exceed our current Role-based Access Control setup?  
- QUESTION: How will the customer validate that the current setup (private endpoints, VPN/internal firewall, and managed identities) meets their security expectations?

<!-- ------------------------- ------------------------- -->

## Scalability
The current solution supports development and testing (with ~10 users) and must scale to support ~2,000 users.

### Resource Configuration
Critical components are configured for light workloads.

- RECOMMENDATION: Adjust SKU selections as needed (see Resource Inventory section below) 

  | **Type** | **SKU** | **Estimated Scale** | **Ready?** | **Recommendation** |
  | :--- | :--- | :--- | :--- | :--- |
  | AI Services | **S0 – AI Services** | Typically supports roughly 20–50 concurrent calls; suitable for light-to-moderate workloads, though heavier loads may require upgrades or sharding | ❓ | Upgrade to **S1 – AI Services** (not available in all regions) |
  | API Management | Premium – stv2 | Built for high throughput – can typically support thousands of requests per second subject to backend configuration and tuning | ✅ | N/A – Already production-ready |
  | App Service Plan | P0v3 (Premium) | A baseline instance may support approximately 250–500 concurrent connections; scale-out options are available for higher loads | ✅ | N/A – Already production-ready |
  | App Service Web App | **Unknown** | – | ❓ | Ensure the web app is provisioned on a **Standard or Premium tier** (aligned with the P0v3 plan) for production |
  | Application Gateway | WAF v2 | Enterprise-grade – designed to handle tens of thousands of requests per minute, with autoscaling available to adjust capacity dynamically | ✅ | N/A – Already production-ready |
  | Bot Service | **F0 (Free Tier)** | Free tier is very limited – generally suited for development/testing; roughly estimated at 10–20 concurrent requests before hitting limits in production | ❓ | Upgrade to the **S1 (Standard) tier** for Azure Bot Service to accommodate production demand |
  | Search Service | **Standard** | With a single replica, usually can handle hundreds of queries per second; additional replicas/partitions can boost capacity further | ✅ | N/A – Already production-ready (with additional scaling options available) |
  | Storage Account | ZRS, StorageV2 (Geo-redundant, V2) | Enterprise-grade throughput – supports thousands of IOPS and concurrent transactions; performance is subject to account limits and network conditions | ✅ | N/A – Already production-ready |

- RECOMMENDATION: Configure container apps and app services for both horizontal scaling (adding more instances) and vertical scaling (increasing CPU/memory)

### Auto-Scaling
Some resources support autoscaling, though full utilization across the solution is not yet confirmed.
- RECOMMENDATION: Enable autoscaling on all resources that support it to manage traffic spikes and dynamic workloads  
- QUESTION: What specific autoscaling policies will be implemented and how will their effectiveness be validated?

### Landing Zone
The planned landing zone configuration is not fully defined and requires further clarification.
- QUESTION: What is the planned landing zone configuration (for example, single region with multi-zone versus multi-region deployment) and how will it impact resource selection and scaling strategies? 
- QUESTION: Are there specific network or geographic redundancy requirements that must be incorporated?

### Performance Thresholds
Key performance triggers and cost strategies for scaling remain to be defined.
- RECOMMENDATION: Define performance metrics {e.g., response times, concurrent connection limits, error thresholds} that will trigger scaling adjustments  
- RECOMMENDATION: Develop a cost management plan to forecast and monitor the financial impact of scaling  
- QUESTION: Are there predetermined performance thresholds that will mandate SKU upgrades or autoscaling adjustments?
- QUESTION: How will the customer manage and forecast the financial impact as traffic increases?

<!-- ------------------------- ------------------------- -->

## Monitoring

### Resources  
Current resource setup is unclear... Diagnostic Settings are deployed via Azure Policy, but there is not a known, central instance of Application Insights or Log Analytics.
- RECOMMENDATION: Deploy centralized logging and log analytics (for example, using Application Insights) to capture real-time performance metrics  
- QUESTION: What is the customer's timeline and strategy for deploying Application Insights and establishing proactive monitoring dashboards?

### Logs, Metrics & Alerts
Current metric capture is unclear.
- RECOMMENDATION: Prepare pre-deployment, **baseline** metrics ("green state") for critical components  
- RECOMMENDATION: Identify key performance indicators (e.g., response times, error rates, throughput) to form the basis for monitoring  
- RECOMMENDATION: Configure automated alerts to quickly identify and remediate deviations from the established baseline  
- RECOMMENDATION: nsure that alerts are integrated with existing operational procedures to enable prompt response to issues  

#### Log Categories by Resource Type

| **Type** | **Categories** | **Documentation** |
| :--- | :--- | :--- |
| AI Services | Audit<br>AzureOpenAIRequestUsage<br>RequestResponse<br>Trace | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-cognitiveservices-accounts-logs) |
| API Management | DeveloperPortalAuditLogs<br>GatewayLogs<br>WebSocketConnectionLogs<br>(Plus: GatewayLlmLogs for AI usage) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-apimanagement-service-logs#:~:text=Portal%20usage%20APIMDevPortalAuditDiagnosticLog) |
| App Service Plan | AppServiceHTTPLogs<br>AppServiceAppLogs<br>AppServiceAuditLogs<br>AppServiceConsoleLogs<br>AppServiceFileAuditLogs<br>AppServicePlatformLogs<br>AppServiceAntivirusScanAuditLogs<br>AppServiceIPSecAuditLogs<br>AppServiceAuthenticationLogs (Preview) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-web-sites-logs) |
| Application Gateway | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog<br>ApplicationGatewayPerformanceLog | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-network-applicationgateways-logs#:~:text=AzureDiagnostics) |
| Bot Service | BotRequest (requests from channels to the bot and vice versa) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-botservice-botservices-logs#:~:text=ABSBotRequests) |
| Search Service | OperationLogs | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-search-searchservices-logs#:~:text=Operation%20Logs%20AzureDiagnostics) |
| Storage Account | StorageRead, StorageWrite, StorageDelete<br>(for Blob, File, Queue, Table services) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-storage-storageaccounts-blobservices-logs#:~:text=StorageBlobLogs) |

#### Sample Log Queries (KPI Candidates)

> Need to review these with Adam Ray on the next call

#### General Application Error Tracking
1.	App Insights alerting on unhandled exception
2.	What are the most common exceptions and what is the source?
3.	Does the bot app service / orchestration container app restart unexpectedly (# of restarts over the last X days, separate counts for bot / orchestrator)

#### Performance
1.	What is the average message response time from bot to user?
2.	Which specific operations are introducing the most latency (is this possible? can we differentiate between different aspects like file uploads, querying the LLM, general network latency response times, etc? If so, how complex is this to log?)
3.	How often do users experience timeouts when asking a question (when timeouts occur, is there some common pattern? i.e. file attached, common source, etc)
4.	What % of messages are augmented with retrieval and what % need no additional context? (this would give great insight on how the bot's being used and how to prioritize service/features. We probably need to come up with a time span upon which to base this query)

#### Security
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

### Monitoring
No cost management reporting / strategy exists beyond use of standard Azure Portal Cost Management.
- RECOMMENDATION: Implement centralized, multi-subscription cost management to track usage and costs at the individual user and resource levels  
- RECOMMENDATION: Develop a centralized cost dashboard that aggregates cost data across all subscriptions for detailed visibility  
- QUESTION: What mechanisms will be used to integrate detailed cost tracking with overall operational monitoring?

### Budget Forecasting  
No proactive strategy for forecasting expenses **based on scaling** is defined.
- RECOMMENDATION: Use the collected cost data to accurately forecast expenses as the user base expands  
- RECOMMENDATION: Establish a process for reviewing spending trends and adjusting resource allocation based on usage  
- QUESTION: How will the customer integrate budget forecasting into scaling and resource planning decisions?

<!-- ------------------------- ------------------------- -->

## Stress Testing  
The current solution has been validated at pilot loads (~10 users) but must undergo comprehensive stress testing to simulate production-level conditions (~2,000 users).

#### Component-Level Testing  
- RECOMMENDATION: Validate that each key resource (e.g., web app, container apps, API endpoints) functions correctly under baseline loads  
- RECOMMENDATION: Execute targeted tests to confirm input handling and basic throughput at low load  
- QUESTION: Have all individual components been thoroughly tested at baseline loads

#### Integrated End-to-End Testing  
- RECOMMENDATION: Simulate full user journeys (from login through chat interactions to file uploads) to confirm complete system functionality  
- RECOMMENDATION: Conduct tests that cover interactions across multiple resources and services  
- QUESTION: What is the customer’s approach for integrated, end-to-end testing as the system scales to 2,000 users?

#### Stress, Spike, and Soak Testing  
- RECOMMENDATION: Perform stress tests to identify system breaking points under maximum load and evaluate degradation and recovery  
- RECOMMENDATION: Execute spike tests to assess system resilience during sudden traffic surges and monitor auto-scaling responses  
- RECOMMENDATION: Conduct soak tests to monitor for issues such as resource exhaustion, memory leaks, or cumulative errors over extended periods  
- QUESTION: Are there predefined performance thresholds that will trigger alerts or remedial actions during these tests?

#### Performance Monitoring  
- RECOMMENDATION: Activate centralized logging and log analytics to capture performance metrics during stress tests  
- RECOMMENDATION: Establish clear baseline metrics (a "green state") for critical components to quickly detect deviations  
- RECOMMENDATION: Leverage performance data to identify and address bottlenecks early in the testing process  
- QUESTION: Which performance metrics will be used to validate stress test results and guide scaling decisions?

<!-- ------------------------- ------------------------- -->

## Post-Deployment

* **User Interface**: Future iterations will explore enhancements such as visualizing parts of uploaded PDFs and embedding response form (perhaps using adaptive cards to improve UI interactions)
 
* **Containers**: Localize processing with Azure Containers