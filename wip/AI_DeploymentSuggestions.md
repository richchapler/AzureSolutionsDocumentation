# AI Bot: Deployment Suggestions

## Use Case

Customer is implementing an "AI Bot" solution that should:

* **Replace External AI Dependencies**: Eliminate reliance on external AI solutions {e.g., ChatGPT, Copilot} by delivering a fully internal, secured AI service that meets enterprise compliance and governance standards

* **Integrate with Microsoft Teams**: Provide a natural language prompt AI directly within Microsoft Teams, ensuring that all interactions occur in a secure, centralized environment for enhanced productivity

* **Ensure Enterprise-Grade Security**: Maintain rigorous internal security protocols to keep AI interactions, data storage, and search capabilities fully within the secured network, safeguarding sensitive information from exposure to external services

* **Scale for Broad User Adoption**: Transition from development / testing stage (supporting ~10 users), to a robust production environment capable of seamlessly serving approximately 2,000 users

## Security

> TO-DO: Chad Hage will explore this topic further

* **Network Security**: All services connect via **private endpoints** to ensure that no sensitive data or prompts leave the enterprise network 
 
* **Authentication**: Entra ID (Azure AD) is used for all authentication
 
* **Bot Security**: Primary pre-deployment objective... secure bot usage to individuals

## Scalability 

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

### Additional Recommendations

* **Compute Resources**: Verify that container apps and app services are configured for both horizontal (adding more instances) and vertical (increasing CPU/memory) scaling
* **Autoscaling**: Enable autoscaling features on resources that support it (such as API Management and container apps) to handle traffic spikes
* **Load Testing**: Schedule comprehensive load tests to simulate peak traffic conditions
* **Centralized Monitoring**: Activate log analytics and set up centralized logging to capture performance metrics and detect bottlenecks early

### Final Thoughts 
* Real-world capacity will vary based on the workload, application configuration, and dynamic scaling rules
* Be prepared to adjust resource allocation as actual usage data comes in, **optimizing performance continuously**

## Monitoring

> TO-DO: Adam Ray will activate Log Analytics for each resource

| **Type** | **Categories** | **Documentation** |
| :--- | :--- | :--- |
| AI Services | Audit<br>AzureOpenAIRequestUsage<br>RequestResponse<br>Trace | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-cognitiveservices-accounts-logs) |
| API Management | DeveloperPortalAuditLogs<br>GatewayLogs<br>WebSocketConnectionLogs<br>(Plus: GatewayLlmLogs for AI usage) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-apimanagement-service-logs#:~:text=Portal%20usage%20APIMDevPortalAuditDiagnosticLog) |
| App Service Plan | AppServiceHTTPLogs<br>AppServiceAppLogs<br>AppServiceAuditLogs<br>AppServiceConsoleLogs<br>AppServiceFileAuditLogs<br>AppServicePlatformLogs<br>AppServiceAntivirusScanAuditLogs<br>AppServiceIPSecAuditLogs<br>AppServiceAuthenticationLogs (Preview) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-web-sites-logs) |
| Application Gateway | ApplicationGatewayAccessLog<br>ApplicationGatewayFirewallLog<br>ApplicationGatewayPerformanceLog | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-network-applicationgateways-logs#:~:text=AzureDiagnostics) |
| Bot Service | BotRequest (requests from channels to the bot and vice versa) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-botservice-botservices-logs#:~:text=ABSBotRequests) |
| Search Service | OperationLogs | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-search-searchservices-logs#:~:text=Operation%20Logs%20AzureDiagnostics) |
| Storage Account | StorageRead, StorageWrite, StorageDelete<br>(for Blob, File, Queue, Table services) | [🔗](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-storage-storageaccounts-blobservices-logs#:~:text=StorageBlobLogs) |

### Sample Log Queries

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

### Sample Alerts

> Need to review these with Adam Ray on the next call

1.	Unhandled Exceptions (any and all hits from the App Insights exceptions table)
2.	HTTP 5xx responses (5xx hits from AppServiceHTTPLogs)
3.	Container Restarts / App Service Restarts (What's the best source for these logs?)
4.	Latency / response time thresholds (I'm thinking 2-3 seconds for initial response, does MS have recommended threshholds? )
5.	Abrupt increase in message count / spikes. (ideally this would indicate spam or misuse, but maybe this is being overly protective? would this be valuable info to alert on? )
6.	CPU/Memory threshold (80% unless MS disagrees)
7.	MIssing JWT token (this would indicate the message didnt originate from teams / related to detecting spam or unexpected sources of ingress)

### Cost Management

* **Cost Tracking:** 
 - Implement monitoring to track usage and costs at the individual user level.
* **Budget Forecasting:** 
 - Use the collected data to forecast expenses accurately as the user base expands.

## Load Testing

### Phase 1: Unit and Component-Level Validation

Focus on establishing that each key resource functions correctly under baseline loads. These tests isolate individual components so you know that each behaves as expected before combining them.

#### Web App (Hosting the AI Chat Bot)
* **Objective:** Validate that the App Service hosting the chat bot can handle baseline operations.
* **Tests:**
  * **Input Handling:** Verify that messages from Teams are processed correctly.
  * **Basic Throughput:** Simulate a modest number of requests (e.g., 50–200 concurrent users) to ensure the web app responds within acceptable latency.
* **Outcome:** Confirms that input validation and core functionality are stable at low load.

#### Application Gateway
* **Objective:** Confirm that the gateway routes HTTP/S traffic accurately and enforces WAF rules.
* **Tests:**
  * **Routing Validation:** Generate synthetic requests matching common access patterns.
  * **Security Checks:** Ensure that the WAF properly blocks known malicious patterns.
* **Outcome:** Verifies that the gateway can process baseline traffic and protect the backend without high latency.

#### API Management
* **Objective:** Validate that API calls (from bot or other services) are handled correctly.
* **Tests:**
  * **Simple API Calls:** Simulate moderate API traffic with realistic payloads.
  * **Error Propagation:** Ensure that any failures are properly handled or throttled.
* **Outcome:** Confirms that API Management can relay and process requests from underlying services reliably.

#### Bot Service
* **Objective:** Check that the bot service correctly processes chat interactions.
* **Tests:**
  * **Message Handling:** Simulate low-load chat interactions (basic text exchanges).
  * **Error Logging:** Verify that errors are logged, and unhandled exceptions trigger appropriate fallback mechanisms.
* **Outcome:** Ensures that the Bot Service operates reliably under controlled conditions.

#### AI Services
* **Objective:** Ensure that the AI processing layer responds within acceptable latency under low load.
* **Tests:**
  * **Basic Query Testing:** Send a set of typical prompts and record average response times.
  * **Error and Throttling:** Verify that the service handles occasional network issues gracefully.
* **Outcome:** Validates that AI processing is performant and its error handling mechanisms are in place.

#### Search Service
* **Objective:** Confirm that indexing and search operations function correctly.
* **Tests:**
  * **Simulated Searches:** Run queries against the index and verify response accuracy and latency.
  * **Indexing Latency:** Check that new content (e.g., from file uploads) appears in search results within acceptable time frames.
* **Outcome:** Ensures that search is responsive and indexing is up-to-date.

#### Storage Account
* **Objective:** Ensure that file uploads, downloads, and transactions perform reliably.
* **Tests:**
  * **I/O Simulation:** Perform concurrent file uploads/downloads at a low load.
  * **Transaction Monitoring:** Check for any errors or unexpected latencies.
* **Outcome:** Verifies that the storage system can handle typical file operations and logs metrics accurately.

### Phase 2: Component-Level Load Testing

Gradually increase the load on each resource in isolation, targeting the level expected in production and ensuring auto-scaling rules engage.

* **Ramp-Up Testing:**  
  For each resource (Web App, API Management, Bot Service, etc.), begin at a lower load and incrementally simulate larger numbers of concurrent requests up to and above 2,000 users (per resource component).  
  * **Metrics to Monitor:** Response times, error counts, CPU/memory usage, auto-scaling events.
* **Individual Resource Benchmarks:**  
  Each test will yield performance benchmarks that confirm the resource’s ability to handle load. For example:
  * **Web App:** Should maintain <500ms response times at peak load.
  * **Bot Service:** Should manage thousands of parallel chat sessions with minimal errors.
  * **Cognitive Services:** Should handle high query frequency without triggering severe throttling.

### Phase 3: Integrated End-to-End Testing

Run comprehensive tests that simulate the full user journey, verifying that all components interact smoothly under load.

* **Scenario Simulations:**  
  Create end-to-end scenarios where a user:
  1. Logs into Teams and starts a chat with the bot.
  2. Interacts with the bot (sending a prompt, receiving a response).
  3. Uploads a file, which is processed and indexed by Azure Search.
  4. Receives a follow-up response leveraging AI services.
* **Distributed Load:**  
  Simulate the entire process with a distributed load test tool (e.g., Azure Load Testing). Start with a smaller batch (e.g., 200 users), monitor performance, then gradually increase to 2,000 concurrent users.
* **Monitoring:**  
  Use Azure Monitor and Log Analytics dashboards to track metrics across all services, ensuring that integrated operations (e.g., API call chains) maintain low latency.
  
**Outcome:**  
When the integrated test shows consistent performance across the entire workflow with acceptable response times and robust auto-scaling behaviors, you have strong evidence that your entire architecture can support production-level traffic.

### Phase 4: Stress, Spike, and Soak Testing

Finally, test the resilience of the system by pushing it beyond the expected load and maintaining the load over an extended time period.

* **Stress Testing:**  
  Increase the load further to identify the breaking points. Observe how the system degrades and recovers.
* **Spike Testing:**  
  Simulate sudden surges in traffic to see how quickly the system adapts. Check for increased error rates or delayed auto-scaling responses.
* **Soak Testing:**  
  Run sustained tests at peak load (2,000 users or slightly higher) for several hours to identify any issues with resource exhaustion, memory leaks, or cumulative errors.
  
**Outcome:**  
The system should handle stress gracefully, recover from spikes, and remain stable over prolonged periods, giving you confidence in its long-term reliability.

## Post-Deployment

* **User Interface**: Future iterations will explore enhancements such as visualizing parts of uploaded PDFs and embedding response form (perhaps using adaptive cards to improve UI interactions)
 
* **Containers**: Localize processing with Azure Containers