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
- **Regional quotas**: Synapse Spark pools support up to 200 nodes per pool, with a minimum of 3 nodes for dedicated pools in many regions. Check your region’s limits under "Apache Spark pool service limits" in the Synapse docs.

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
	"id": "/subscriptions/<subscriptionId>/resourceGroups/prefix/providers/Microsoft.Synapse/workspaces/prefixsaw/bigDataPools/prefixasp",
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

- Bypass the Livy REST endpoint to submit jobs directly to the Synapse Spark Pool
  - Quote from meeting: "So how about if we just bypass the Livy endpoint and then send the job to the Spark?"
- Leverage the Spark driver’s native submission API within the Synapse Spark Pool
  - Quote from meeting: "Look at the driver has its own endpoint, right? We can call the API for the driver, not the DB."
- Eliminate notebook startup and Livy overhead to reduce cold-start latency in the Spark Pool
  - Quote from meeting: "We can send the jobs directly to the Spark without the Livy."

<!-- ------------------------- ------------------------- -->

### Proposed Solution (WORK IN PROGRESS)

#### Required Resources

* **Resource Group** `cnb`

* **Synapse Workspace** `cnbsaw`
  * **Managed Resource Group** `cnbmrg`
  * **Data Lake Storage Gen2** `cnbdls`
  * Data Lake Storage Gen2, **File System** `cnbdlsfs`
  * Assigned-to-self **Storage Blob Contributor** role
  * Authentication method: **Use both local and Microsoft Entra ID authentication** with admin `sqladminuser`
  * Firewall rules: **Allow connections from all IP addresses**

* **Apache Spark Pool** `cnbasp`
  * Node size: Small (4 vCores / 32 GB)
  * Automatic pausing: Enabled

* **Application Registration** `cnbar` and secret
  * Assigned **Contributor** role on Apache Spark Pool `cnbasp`

<!-- ------------------------- ------------------------- -->

#### Step 1

##### Prepare `sample.py`

Use a text editor to create a file with the following Python:
```python
from pyspark.sql import SparkSession
count = SparkSession.builder.getOrCreate().sparkContext.parallelize([1]).count()
print(count)
```

Navigate to storage account `cnbdls` >> **Storage browser** >> **Blob containers** and click **+ Add container**.

On the **New container** popout, enter name `scripts` and click **Create**.

Navigate to the `scripts` container, then upload `sample.py`.

<!-- ------------------------- -->

##### Get Token

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

```powershell
$token=(Invoke-RestMethod `
  -Method Post `
  -Uri "https://login.microsoftonline.com/<tenantId>/oauth2/v2.0/token" `
  -ContentType "application/x-www-form-urlencoded" `
  -Body "grant_type=client_credentials&client_id=<clientId>&client_secret=<clientSecret>&scope=https%3A%2F%2Fdev.azuresynapse.net%2F.default"
).access_token
```

<!-- ------------------------- -->

##### Warm Pool

In Synapse Studio, open any notebook, attach it to the `cnbasp` pool, and run a trivial command (e.g., `sc.parallelize([1]).count()`).

Wait for the pool to fully spin up (you should see a 3–5 minute cold-start).

<!-- ------------------------- -->

##### Submit Job

```powershell
$response=Invoke-RestMethod -Method Post -Uri "https://cnbsaw.dev.azuresynapse.net/sparkPools/cnbasp/driver/submissions?api-version=2021-06-01" -Headers @{Authorization="Bearer $token"} -ContentType "application/json" -Body (@{file="abfss://scripts@cnbdls.dfs.core.windows.net/KeepAlive.py";args=@();conf=@{"spark.dynamicAllocation.enabled"="true";"spark.dynamicAllocation.minExecutors"="1";"spark.dynamicAllocation.maxExecutors"="10"}}|ConvertTo-Json -Depth 5); $submissionId=$response.submissionId
```

<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->

##### Poll for Completion

```powershell
do {
  Start-Sleep -Seconds 15
  $status = Invoke-RestMethod -Method Get `
    -Uri "https://<ws>.dev.azuresynapse.net/sparkPools/<pool>/driver/submissions/$submissionId?api-version=2021-06-01" `
    -Headers @{ Authorization = "Bearer $token" }
} while ($status.code -notin @("FINISHED","ERROR"))

$status.code
```

##### (Optional) Pre-warm the Pool

Run a trivial submission to keep executors hot, for example:

```powershell
Invoke-RestMethod -Method Post `
  -Uri "https://<ws>.dev.azuresynapse.net/sparkPools/<pool>/driver/submissions?api-version=2021-06-01" `
  -Headers @{ Authorization = "Bearer $token" } `
  -Body @{
    file = "abfss://scripts@<storage>.dfs.core.windows.net/KeepAlive.py"
    args = @()
  } | Out-Null
```

Schedule this with Azure Automation or a Logic App if desired.

##### Monitor and Tune

* View driver and executor metrics in Synapse Studio → **Monitor** → **Apache Spark applications**
* Adjust pool node counts and dynamic-allocation settings based on observed utilization
* Tag each submission in your payload (add a `"name": "MyJobName"` field) for cost tracking and auditing

