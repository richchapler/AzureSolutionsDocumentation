# Synapse: Spark Pool

## "Keep Alive"

<!-- ------------------------- ------------------------- -->

### Use Case

- Scheduled pipelines run Spark notebooks between 8am and 12pm daily (keeping clusters active during the ~4h window)
- End users launch ad-hoc Spark notebooks between 8am and 6pm on weekdays and expect jobs to start without delay  
- The Synapse Spark Pool's 10-minute idle timeout triggers a full cold start (5–7 minutes) whenever no one runs a job for more than 10 minutes
- Management decided on the 10-minute timeout after observing the cost associated with running the cluster with a 60-minute timeout

<!-- ------------------------- ------------------------- -->

### Proposed Solution

- Create a `Keep Alive` pipeline ("a 1 + 1 script") scheduled to run every 10 minutes from 12pm to 6pm on weekdays
<br>  (existing scheduled pipelines already keep the pool alive from 8am to 12pm)  
- When the pipeline is re-activated by the pipeline, it will, by default, be set to minimum configuration
- This leaves the Spark Pool active during business hours with minimal cost

<!-- ------------------------- ------------------------- -->

### Fundamentals

#### Spark

- **Executor**: a worker process in Spark that runs parts of your job and keeps data in memory while it’s working  
- **Dynamic Executor Allocation**: lets a job add / remove Executors as needed

#### Pool-Level Controls

- **Auto-scale**: a setting that automatically increases or decreases the number of machines in your Spark pool based on how busy it is  
- **Auto-pause**: a setting that shuts down your Spark pool after it’s been idle for a set time, so you don’t pay for unused machines  
- **Regional quotas**: Synapse Spark pools support up to 200 nodes per pool, with a minimum of 3 nodes for dedicated pools in many regions. Check your region’s limits under “Apache Spark pool service limits” in the Synapse docs.

#### Pipeline Constructs

- **Spark job definition**: a preset recipe in Synapse that tells Spark which code to run and with what settings, without opening a notebook  
- **Web activity**: a pipeline step that calls an Azure REST API (for example to change Spark pool settings) using your workspace’s identity  
- **Schedule trigger**: a pipeline scheduler that runs your pipeline on a set timetable (for example every 10 minutes between 8 AM and 6 PM)

<!-- ------------------------- ------------------------- -->

### Walk-Through

-------------------------

#### Create Apache Spark Pool

Navigate to **Synapse Studio** >> **Manage** >> **Analytics pools** >> **Apache Spark pools**.

Click **+ New** and complete the resulting **New Apache Spark pool** popout form:

Setting | Value | Reason
:--- | :--- | :---
Apache Spark pool name | `prefixasp` | Naming convention
Node size family | Memory Optimized | Prioritizes memory for in-memory operations
Node size | Small (4 vCores / 32 GB) | Balances compute performance with cost efficiency
Autoscale | Enabled | Automatically adjusts node count to match workload demand
Number of nodes | 3 and 10 | Ensures baseline capacity (3) and allows scaling to handle peak loads
Dynamically allocate executors | Enabled | Allocates only needed executors at runtime to minimize resource waste

Click **Review + create** and then **Create**.

##### Add Role Assignment

Navigate to Azure Portal >> Apache Spark pool `prefixasp` >> **Access control (IAM)**.

Click **+ Add** >> **Add role assignment**, select role **Contributor**, then click **Next**.

Select **Managed identity** `prefixsaw`, then click **Review + assign** (twice).

-------------------------

#### Create Notebook

Navigate to **Synapse Studio** >> **Develop** and create a new Notebook named `KeepAlive` 

Paste the following logic:
```python
sc.parallelize([1]).count()
```

> This logic counts the single element, forcing the cluster to spin up executors and stay active

Click **Publish**.

Attach to the `prefixasp` Spark Pool and click **Run** to confirm functionality; expect output like:

``` plaintext
Apache Spark session started in 3 min 54 sec 102 ms. Command executed in 16 sec 854 ms by rchapler on 7:12:44 AM, 5/01/25
```

-------------------------

#### Create Pipeline

Navigate to **Synapse Studio** >> **Integrate**, and then add a new Pipeline named `KeepAlive`.

##### Add Parameters

Parameter | Type | Value  
:--- | :--- | :---  
subscriptionId | String | `<Your Subscription ID>`
resourceGroupName | String | `prefix`
workspaceName | String | `prefixsaw`
sparkPoolName | String | `prefixasp`
minNodes | Int | `1`
maxNodes | Int | `10`
autoPauseDelay | Int | `10`

-------------------------

##### Add Activity: Check Cluster Status

> This activity is informational only... it shows that **current status (e.g., paused or running) cannot be derived programmatically**

Add and configure a **Web** activity.

Setting | Value | Reason
:--- | :--- | :---
Name | `Check Cluster Status` | Identifies this activity in the pipeline
URL | `@concat('https://management.azure.com/subscriptions/',pipeline().parameters.subscriptionId,'/resourceGroups/',pipeline().parameters.resourceGroupName,'/providers/Microsoft.Synapse/workspaces/',pipeline().parameters.workspaceName,'/bigDataPools/',pipeline().parameters.sparkPoolName,'?api-version=2021-06-01')`
Method | GET | Updates existing Spark pool properties
Authentication | System-assigned managed identity | Instructs the pool to autoscale within your min/max and keeps auto-pause enabled with a delay that avoids shutdown between keep-alive runs
Resource | `https://management.azure.com/` | Specifies the audience for token issuance when using Managed Identity
Headers | `Content-Type` :: `application/json` | Declares that the request body is JSON and requests JSON back from the ARM API

###### Validate Progress

Click **Debug** and confirm successful execution.

Click **Output** and you should see something like:
```plaintext
{
	"properties": {
		"creationDate": "2025-05-01T13:04:51.9766667Z",
		"sparkVersion": "3.4",
		"nodeCount": 10,
		"nodeSize": "Small",
		"nodeSizeFamily": "MemoryOptimized",
		"autoScale": {
			"enabled": true,
			"minNodeCount": 3,
			"maxNodeCount": 10
		},
		"autoPause": {
			"enabled": true,
			"delayInMinutes": 15
		},
		"isComputeIsolationEnabled": false,
		"sessionLevelPackagesEnabled": false,
		"cacheSize": 50,
		"dynamicExecutorAllocation": {
			"enabled": false
		},
		"lastSucceededTimestamp": "2025-05-01T13:04:58.9266667Z",
		"isAutotuneEnabled": false,
		"provisioningState": "Succeeded"
	},
	"id": "/subscriptions/ed7eaf77-d411-484b-92e6-5cba0b6d8098/resourceGroups/prefix/providers/Microsoft.Synapse/workspaces/prefixsaw/bigDataPools/prefixasp",
	"name": "prefixasp",
	"type": "Microsoft.Synapse/workspaces/bigDataPools",
	"location": "westus",
	"tags": {},
	"ADFWebActivityResponseHeaders": {
		"Pragma": "no-cache",
		"Strict-Transport-Security": "max-age=31536000; includeSubDomains",
		"x-ms-request-id": "2b773fb4-7222-47db-a517-e79901d20b4a",
		"x-ms-ratelimit-remaining-subscription-reads": "249",
		"x-ms-ratelimit-remaining-subscription-global-reads": "3749",
		"x-ms-correlation-request-id": "408f77af-e36b-4f30-a1dc-a3a50dc63892",
		"x-ms-routing-request-id": "WESTUS:20250501T143306Z:408f77af-e36b-4f30-a1dc-a3a50dc63892",
		"X-Content-Type-Options": "nosniff",
		"X-Cache": "CONFIG_NOCACHE",
		"X-MSEdge-Ref": "Ref A: 65FDF7B822C94CBCA87EDF33DEE456F0 Ref B: SJC211051205045 Ref C: 2025-05-01T14:33:05Z",
		"Cache-Control": "no-cache",
		"Date": "Thu, 01 May 2025 14:33:05 GMT",
		"Content-Length": "763",
		"Content-Type": "application/json; charset=utf-8",
		"Expires": "-1"
	},
	"effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (West US)",
	"executionDuration": 1,
	"durationInQueue": {
		"integrationRuntimeQueue": 0
	},
	"billingReference": {
		"activityType": "ExternalActivity",
		"billableDuration": [
			{
				"meterType": "AzureIR",
				"duration": 0.016666666666666666,
				"unit": "Hours"
			}
		]
	}
}
```

-------------------------

##### Add Activity: Notebook

> This step will run the `KeepAlive` notebook... if the cluster is paused it will start it with minimum settings (no need for additional reconfiguration)

Add and configure a **Notebook** activity.

Setting | Value | Reason  
:--- | :--- | :---  
Name | `KeepAlive` | Identifies this activity in the pipeline  
Notebook | `KeepAlive` | Points to the pre-published keep-alive notebook  
Spark pool | `prefixasp` | Targets the correct Spark pool for execution  
Executor size | `Small (4 vCores / 32 GB)` | Minimizes cost for the trivial notebook job  
Dynamically allocate executors | `Enabled` | Ensures only needed executors spin up and idle ones release  
Min executors | `1` | Guarantees a single executor is available  
Max executors | `2` | Limits executor scale-out to control cost  
Driver size | `Small (4 vCores / 32 GB)` | Matches executor size for consistency  
Authentication | `System-assigned managed identity` | Uses the workspace identity for secure, password-free authentication

###### Validate Progress

Click **Debug** and allow time for processing.

Click **Open notebook snapshot** and review.

-------------------------

##### Schedule

Final step, not documented here... schedule this for "every 10 minutes"

------------------------- ------------------------- ------------------------- -------------------------

## Direct Submission

<!-- ------------------------- ------------------------- -->

### Use Case

- Today, every Spark workload—both scheduled pipelines and ad-hoc user jobs—is run through an interactive notebook, which incurs a 5–7 minute cold-start delay whenever a new Livy session spins up.  
- Notebooks carry additional overhead (UI, notebook context initialization) beyond the pure Spark execution.  
- Users need rapid, repeatable batch runs of PySpark code without waiting on notebook startup.  
- Management is open to script-based submissions if they can shave off latency while still fitting into existing Synapse pipelines.

<!-- ------------------------- ------------------------- -->

### Proposed Solution

- **Convert Notebooks to Scripts**  
  Extract your core PySpark logic from notebooks into standalone `.py` files (for example, `process_data.py`) and store them in your workspace’s linked storage.  

- **Define Spark Job Definitions**  
  In Synapse Studio’s **Manage → Apache Spark → Job definitions**, create one definition per script, specifying the file path, main class (for JVM jobs), parameters, and resource settings (dynamic allocation, driver/executor sizes).  

- **Submit via Pipeline Activity**  
  Replace your Notebook activities with **Spark Job Definition** activities in Synapse pipelines—pointing at your new definitions. This invokes the Livy batch endpoint directly, skipping notebook context setup.  

- **Or Submit via CLI/REST**  
  Use the Azure CLI command  
  ```bash
  az synapse spark job submit \
    --workspace-name <ws> \
    --name process-data-job \
    --file abfss://scripts@<storage>.dfs.core.windows.net/process_data.py \
    --livy-only \
    --arguments "--input","/data/in","--output","/data/out"
  ```  
  or call the equivalent REST endpoint (`POST /sparkJobDefinitions/{jobName}/submit`) from a Web activity—again bypassing the notebook layer.  

- **Leverage Dynamic Allocation**  
  In each job definition, enable  
  ```text
  spark.dynamicAllocation.enabled=true  
  spark.dynamicAllocation.minExecutors=1  
  spark.dynamicAllocation.maxExecutors=10
  ```  
  so your batch jobs only consume needed executors and release them when finished.  

- **Integrate with CI/CD**  
  Store your `.py` scripts in a Git repo and automate deployment of job definitions via ARM templates or the Synapse CLI, ensuring script and definition versioning.  

By submitting pure batch jobs instead of interactive notebooks, you eliminate notebook-startup overhead and shorten end-to-end latency—while still running on the same Synapse Spark pool and leveraging its autoscale, auto-pause, and dynamic-allocation features.