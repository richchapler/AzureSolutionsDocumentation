# Data Factory: Debugging

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ead90204-81af-4428-a47c-803cae65a214" width="1000" />

## Studio

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/56315a73-0864-4319-a568-63bc84a01b40" width="800" title="Snipped January 21, 2025" />

### Pipelines

1. Open Azure Data Factory Studio:
   - Launch Azure Data Factory Studio.
   - Navigate to the Author tab.
   - Click the + button and select Pipeline to create a new pipeline.

2. Add Activities to the Pipeline:
   - Drag and drop the required activities from the Activities pane onto the pipeline canvas.
   - Example: Add a Copy Data activity to copy data from one source to another.

3. Configure the "onFail" Feature:
   - Select the activity you want to monitor for failures.
   - In the Activities pane, locate the onFail section and click Add activity.
   - Choose the activity to execute if the primary activity fails, such as a Stored Procedure activity to log failure details.

4. Set Up the Stored Procedure Activity:
   - Drag and drop a Stored Procedure activity onto the pipeline canvas.
   - Configure the activity to connect to your SQL Server database.
   - Specify the stored procedure to log failure details, ensuring it accepts parameters for:
     - Error message: A description of the failure.
     - Activity name: The name of the failed activity.
     - Timestamp: The time the failure occurred.

5. Link Activities Using "onFail":
   - Link the primary activity to the Stored Procedure activity using an onFail dependency.
   - Ensure the Stored Procedure activity only executes when the primary activity fails.

6. Publish and Test the Pipeline:
   - Click Publish to save and publish your pipeline.
   - Trigger the pipeline to test its execution.
   - Verify that if the primary activity fails, the onFail feature triggers the Stored Procedure activity, logging failure details to your SQL Server table.

---

### Data Flows

#### Step 1: Prepare Example

Navigate to Data Factory Studio >> Author and add a new Data Flow.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/b9b25a94-93a1-42eb-9643-debdefa471da" width="800" title="Snipped January 21, 2025" />

Toggle "Data flow debug" and in the resulting pop-out, set "Debug time to live" to four hours.

##### Add Source

Click "Add Source".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a080ebb9-def5-485a-b932-514a75c0dd6c" width="800" title="Snipped January 21, 2025" />

Select "Dataset" and confirm successful connection. For this example, I am using `SalesLT.Address` in an AdventureWorks sample database.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a080ebb9-def5-485a-b932-514a75c0dd6c" width="800" title="Snipped January 21, 2025" />

Navigate to the "Projection" tab and click "Import projection" to import schema.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/892d51b7-3ad5-470a-9f04-68e4df88f943" width="800" title="Snipped January 21, 2025" />

Navigate to the "Data preview" tab and click "Refresh" to see source data.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/49575d15-8760-4649-a9df-ffda4dd774de" width="800" title="Snipped January 21, 2025" />

_Note: The interface may not gracefully flex to a source with many columns (even showing columns with a single letter or nothing at all)_

##### Add Transformation

Click the "+" icon in the bottom right of the Source arrow and on the resulting dropdown, search for and select "Filter"

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/da2d10de-f631-4c08-8642-0cce2a49b9f7" width="800" title="Snipped January 21, 2025" />

On the "Filter settings" tab, enter "Filter on" expression `StateProvince == 'Arizona'`

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2bfd255b-9681-4597-987f-406d383e5667" width="800" title="Snipped January 21, 2025" />

Navigate to the "Data preview" tab and click "Refresh" to see source data.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/d1e876e8-0580-42d5-9e38-deba4043ecc9" width="800" title="Snipped January 21, 2025" />

_Note: The "INSERT" value has dropped to 13_

##### Add Sink

Click the "+" icon in the bottom right of the Source arrow and on the resulting dropdown, search for and select "Sink"

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/9073854c-5d28-455b-9bc9-488e7ca8124d" width="800" title="Snipped January 22, 2025" />

On the "Settings" tab, click "+ New" to create a new dataset pointing at your sink. For this example, I am creating `SalesLT.Address2` in the AdventureWorks sample database.

```sql
SELECT * INTO [SalesLT].[Address2] FROM [SalesLT].[Address] WHERE 1 = 0
ALTER TABLE [SalesLT].[Address2] DROP COLUMN AddressID
```

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/aa06786e-83e7-4644-baa0-a47fd98f3a2e" width="800" title="Snipped January 22, 2025" />

_Note: Once the Sink has been added you can "Publish all" to save progress.

##### Add Pipeline

Add a new pipeline with a "Data flow" activity. Set "Logging level" to "None".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/9ef1fab8-120a-47b4-b647-4f980fe2a925" width="800" title="Snipped January 22, 2025" />

_Note: Logging level primarily enhances result statistics_

Click "Debug" to execute the pipeline interactively and then monitor execution in real-time on the "Output" tab.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/4ecddf3d-f7b0-4a4d-afd5-c121836fb7cf" width="800" title="Snipped January 22, 2025" />

#### Step 2: Review Results

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/13455cfd-c95c-4908-97e6-3f1276c36cce" width="800" title="Snipped January 22, 2025" />

Once execution is complete, roll-over the output row to see icons otherwise hidden {e.g., Input, Output, Data Flow Details}.

##### Input

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/760e765b-a4dd-450a-8b91-10ad4da9d595" width="800" title="Snipped January 22, 2025" />

```json
{
    "dataflow": {
        "referenceName": "dataflow1",
        "type": "DataFlowReference",
        "parameters": {},
        "datasetParameters": {
            "source1": {},
            "sink1": {}
        }
    },
    "staging": {},
    "compute": {
        "coreCount": 8,
        "computeType": "General"
    },
    "traceLevel": "None",
    "cacheSinks": {
        "firstRowOnly": true
    },
    "dataFlowDebugSessionId": "2cba1f56-bb3e-48b6-8bd3-becd0416441f",
    "continuationSettings": {
        "customizedCheckpointKey": "pipeline1-Data flow1-4615720a-b514-4354-b4d7-12e184c20fd6"
    }
}
```

##### Output

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a5949165-baaa-45d1-ab9b-8381721eb2c6" width="800" title="Snipped January 22, 2025" />

```json
{
  "runStatus": {
    "ClusterId": "azuresolutionsdatafactory.AutoResolveIntegrationRuntime.2",
    "sparkVersion": "3.3",
    "computeAcquisitionDuration": 4563,
    "version": "20241104.1",
    "profile": {
      "source1": { "computed": [], "lineage": {}, "dropped": 0, "drifted": 0, "newer": 9, "total": 9, "updated": 0 },
      "filter1": { "computed": [], "lineage": {}, "dropped": 0, "drifted": 0, "newer": 0, "total": 9, "updated": 0 },
      "sink1": {
        "computed": [{ "source": "source1", "columns": ["StateProvince"] }],
        "lineage": {
          "ModifiedDate": { "mapped": true, "from": [{ "source": "source1", "columns": ["ModifiedDate"] }] },
          "CountryRegion": { "mapped": true, "from": [{ "source": "source1", "columns": ["CountryRegion"] }] },
          "rowguid": { "mapped": true, "from": [{ "source": "source1", "columns": ["rowguid"] }] },
          "StateProvince": { "mapped": true, "from": [{ "source": "source1", "columns": ["StateProvince"] }] },
          "AddressLine1": { "mapped": true, "from": [{ "source": "source1", "columns": ["AddressLine1"] }] },
          "City": { "mapped": true, "from": [{ "source": "source1", "columns": ["City"] }] },
          "AddressLine2": { "mapped": true, "from": [{ "source": "source1", "columns": ["AddressLine2"] }] },
          "PostalCode": { "mapped": true, "from": [{ "source": "source1", "columns": ["PostalCode"] }] }
        },
        "dropped": 0,
        "drifted": 0,
        "newer": 0,
        "total": 9,
        "updated": 8
      }
    },
    "metrics": {
      "sink1": {
        "format": "",
        "stages": [{
          "stage": 0,
          "partitionTimes": [],
          "recordsWritten": 0,
          "lastUpdateTime": "2025-01-22 13:18:06.497",
          "bytesWritten": 0,
          "recordsRead": 13,
          "bytesRead": 0,
          "partitionStatus": "Success",
          "streams": {},
          "target": "sink1",
          "time": 12057,
          "progressState": "Completed"
        }],
        "sinkPostProcessingTime": 0,
        "store": "",
        "rowsWritten": 0,
        "details": {
          "preSQLDuration": [0],
          "tempTable": ["##T_8327_00c4524df3a24d83b71cbef69d2597ee"],
          "tableOperationSQLDuration": [17],
          "postSQLDuration": [0]
        },
        "progressState": "Completed",
        "sources": {},
        "sinkProcessingTime": 16028
      }
    },
    "clusterComputeId": "ffdf999b-3127-4db8-abb1-e40f55965261",
    "dsl": "source() ~> source1\nsource1 filter() ~> filter1\nfilter1 sink() ~> sink1"
  },
  "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (West US)",
  "billingReference": {
    "activityType": "executedataflow",
    "billableDuration": [{ "meterType": "Data Flow", "duration": 0.08176134266666667, "unit": "coreHour", "sessionType": "JobCluster" }]
  },
  "reportLineageToPurview": { "status": "NotReported" }
}
```

_Note: I've compacted the JSON above for presentation_

##### Data Flow Details

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/db449c99-4d5f-41fd-ac17-38d3158ee0c0" width="800" title="Snipped January 22, 2025" />

###### Stages

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/d0e85c47-d469-4a79-aef4-7c128eb8745b" width="800" title="Snipped January 22, 2025" />

Click into "Stages" and note that since Logging was set to "None", very little data is presented.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a3271f93-ceac-4c9d-80b6-f0c963fd7988" width="800" title="Snipped January 22, 2025" />

Return to the pipeline, change the data flow "Logging level" setting to "Verbose", then re-execute with "Debug".

_Note: While it might seem logical to always set "verbose", there is a performance cost that some customers want to avoid_

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/3127c591-9184-49ff-a7f0-8aa8c16e13e8" width="800" title="Snipped January 22, 2025" />

"Verbose" setting presents a little more Stages data.

###### Lineage

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/e20e9b99-c424-42d6-8409-64f2594c3ef9" width="800" title="Snipped January 22, 2025" />

"Lineage" doesn't vary based on logging level setting.

###### Activities

Click on the first activity arrow.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2f1e925b-85ea-4016-8eb4-679c32d02dfd" width="800" title="Snipped January 22, 2025" />

Repeat for each activity arrow to see presented information {e.g., row count, processing times, errors, etc.}.

---

### Monitor

#### Runs >> "Pipeline runs" and "Trigger runs"

* Navigate to Azure Data Factory >> Studio >> Monitor >> Pipeline Runs
* Review filter settings {e.g., date, name, status, etc.}
* Click into a specific log entry and review
* Repeat for Trigger Runs... note that a Trigger Run might have "Succeeded" even though the corresponding Pipeline Run failed

#### Runs >> "Pipeline runs" and "Trigger runs"

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/20701095-b85b-4bfa-8010-ec89bba8cae8" width="800" title="Snipped January 22, 2025" />

#### Notifications >> "Alerts & metrics"
Create alerts to notify you via email, SMS, or other means when a pipeline run meets certain conditions, such as failure or success.

 * Navigate to Azure Data Factory >> Studio >> Monitor >> Alerts & Metrics
 * Click "New alert rule" and complete the resulting pop-out form, including:

   Label | Value
   :----- | :-----
   Severity | Sev0 = most severe... Sev5 = least severe
   Add criteria | Select and configure the desired Metric on the resulting pop-out<br><sub>Metric descriptions included in Appendix</sub>
   Configure Notification | Define "action group" {i.e., notification action and recipient

---

## Portal

### Monitoring

#### Diagnostic Settings

* Navigate to Azure Data Factory >> Monitoring >> Diagnostic Settings
* Click "+ Add diagnostic setting" and complete the resulting screen, including:

   Label | Value
   :----- | :-----
   Logs | Select and configure the desired Log Categories<br><sub>Category descriptions included in Appendix</sub>
   Destination Details | Select and configure the desired Log Categories<br><sub>Destination descriptions included in Appendix</sub><br><sub>Note that destination selection expands the interface and necessitates additional entries</sub>

##### Destination: Log Analytics

Selection of "Send to Log Analytics workspace" necessitates selection of "Destination table":
   
   * Azure Diagnostics - When you choose "Azure Diagnostics", all your diagnostic logs from all resource types are sent to a single table called AzureDiagnostics. This can make querying easier if you want to correlate events across different resource types, as you only need to query one table. However, the AzureDiagnostics table is a flat table, so complex properties are serialized into JSON strings which can make some data harder to work with.  
   * Resource Specific - When you choose "Resource Specific", your logs are sent to a table that corresponds to the resource type. For example, if you're working with a Storage Account, your logs might be sent to a table like StorageAccountLogs. These tables are designed to best fit the schema of the log data of the specific resource type, and complex properties are represented as distinct fields, which can make queries more intuitive and the data easier to work with. However, if you want to correlate events across different resource types, you would need to query multiple tables.  

Be sure to click "Save".

#### Logs

#### Sample #1: `AzureDiagnostics`

* Navigate to Azure Data Factory >> Monitoring >> Logs
* Close the "Welcome to Log Analytics" and "Queries hub" popups  
* In the query window, paste this very simple starter KQL query:  
   
   ```kql  
   AzureDiagnostics   
   ```  
   
* Click "Run" to execute the query.  

##### Sample #2: `PipelineRuns`

* Repeat with the following KQL:

   ```kql  
   AzureDiagnostics    
   | where ResourceProvider == "MICROSOFT.DATAFACTORY" and Category == "PipelineRuns"    
   | where TimeGenerated > ago(1d)    
   | order by TimeGenerated desc       
   ```

_Note: Error details (when applicable) are captured in ADFActivityRuns, not in ADFPipelineRuns_

#### Alerts

* Navigate to Azure Data Factory >> Monitoring >> Alerts
* Click "Create" and select "Alert rule" from the resulting dropdown
* Complete the resulting "Create an alert rule" page, including:

   Label | Value
   :----- | :-----
   Signal name | Select and configure the desired Signal<br><sub>Signal descriptions included in Appendix</sub>

* Review additional tabs {e.g., Scope, Actions, Details, Tags}, then "Review + create"

---

## Analyzing Failures

### Starter Configuration

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/76e6ff4d-3be6-469a-9b22-a927c17a7f20" width="800" title="Snipped February 10, 2025" />

Begin with an existing pipeline / dataflow configuration. If an existing configuration is unavailable, you can create a simple example with the following items:

- Data Flow Name: `customer_sql_to_storage`  
  - Reads from a source dataset (`{prefix}sqldatabase_customer`) pointing to table SalesLT.Customer in an Azure SQL Database.  
  - Writes to a sink dataset (`{prefix}storagecontainer`) referencing a JSON file in an Azure Blob Storage container.

- Pipeline Name: `customer_sql_to_storage`  
  - Contains a single Execute Data Flow activity, also named `customer_sql_to_storage`.  
  - The activity points to the data flow of the same name and uses 8 General cores (compute) with `traceLevel` set to Fine.

- Source Dataset: `{prefix}sqldatabase_customer` (Type: AzureSqlTable)  
  - References a linked service named `{prefix}sqldatabase` (Azure SQL DB).  
  - Table properties specify SalesLT.Customer and columns like `CustomerID`, `FirstName`, `LastName`, etc.

- Sink Dataset: `{prefix}storagecontainer` (Type: Json)  
  - References a linked service named `{prefix}storage` (Azure Blob Storage).  
  - Configured to write JSON files to a container named {prefix}storagecontainer.

- Data Flow Script:  
  - Declares the source output columns (`CustomerID`, `NameStyle`, `Title`, etc.).  
  - Specifies `allowSchemaDrift` and `validateSchema` options.  
  - Directly maps source data into the sink (no intermediate transformations).

Run the starter configuration to confirm that is functional before we begin simulating failures.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/df16c7ad-fcac-4aea-833e-7481d61b3d23" width="800" title="Snipped February 10, 2025" />

### Cluster Failure

Clusters fail when they become unavailable, overloaded, or are misconfigured.

In the examples below, we will not try to overload the cluster (which is difficult because Azure is robust)
Instead, we will simulate failure with simple incorrect configurations, such as referencing nonexistent resources, using invalid connection strings, or misconfiguring integration runtimes.

#### Simulate Failure

To similulate cluster failure, click "Debug" on the pipeline, then immediately flip over to the dataflow and switch "Data flow debug" to off.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/caf0e349-8fa3-4a3e-aad2-ad0cf9e0e110" width="800" title="Snipped February 10, 2025" />

Flip back to the pipeline and watch the debug run until it fails.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2bdefa42-fd5a-4f5d-a974-10dec7ba2a9c" width="800" title="Snipped February 10, 2025" />









#### Querying Logs
```kql
AzureDiagnostics
| where Category == "ActivityRuns"
| where Status == "Failed"
| where ErrorMessage contains "Dataset does not exist" or ErrorMessage contains "Connection failed"
| project TimeGenerated, Resource, ActivityName, Status, ErrorMessage
| order by TimeGenerated desc
```

---

### 2. Integration Runtime Failures

Integration runtime failures occur when the compute resource used to process pipeline activities becomes unavailable.

#### **Simulating Failure**
- **Azure Integration Runtime**: Set the **linked service** to an **invalid integration runtime**.
- **Self-Hosted Integration Runtime**: Stop the service on the machine:
  ```powershell
  Stop-Service -Name "DIAHostService"
  ```
- Modify the linked service to use an **incorrect authentication method**.

#### **Querying Logs**
```kql
AzureDiagnostics
| where Category == "ActivityRuns"
| where Status == "Failed"
| where ErrorMessage contains "Integration runtime not found" or ErrorMessage contains "Authentication failed"
| project TimeGenerated, Resource, ActivityName, Status, ErrorMessage
| order by TimeGenerated desc
```

---

### 3. Pipeline Activity Failures

Pipeline activities can fail due to invalid configurations or missing dependencies.

#### **Simulating Failure**
- In a **Copy Data** activity, change the destination dataset to a **nonexistent table**.
- In a **Stored Procedure** activity, reference a **nonexistent stored procedure**:
  ```sql
  EXEC NonExistentProcedure
  ```
- In a **Web Activity**, set the API URL to an **invalid endpoint**.

#### **Querying Logs**
```kql
AzureDiagnostics
| where Category == "PipelineRuns"
| where Status == "Failed"
| project TimeGenerated, Resource, PipelineName, Status, ErrorMessage
| order by TimeGenerated desc
```

---

### 4. Permissions Failures in Linked Services

Permissions issues often cause authentication failures when accessing external resources.

#### **Simulating Failure**
- Remove the **Storage Blob Data Reader** role for a managed identity accessing **Azure Blob Storage**.
- Modify the **linked service authentication** to use an **incorrect database user**.
- Disable **Azure SQL Database firewall rules**, blocking pipeline access.

#### **Querying Logs**
```kql
AzureDiagnostics
| where Category == "ActivityRuns"
| where Status == "Failed"
| where ErrorMessage contains "Permission denied" or ErrorMessage contains "Authentication failed"
| project TimeGenerated, Resource, ActivityName, Status, ErrorMessage
| order by TimeGenerated desc
```

---

### 5. Resource Failures

Resource failures occur when a required service or storage account becomes unavailable.

#### **Simulating Failure**
- **Azure SQL Database**: Temporarily **disable the database**:
  ```sql
  ALTER DATABASE [DatabaseName] SET OFFLINE WITH ROLLBACK IMMEDIATE;
  ```
- **Blob Storage**: Change the storage **account key** in the linked service to an **invalid value**.
- **Key Vault**: Remove access permissions for **Data Factory**.

#### **Querying Logs**
```kql
AzureDiagnostics
| where Category == "ActivityRuns"
| where Status == "Failed"
| where ErrorMessage contains "Resource not found" or ErrorMessage contains "Storage account unavailable"
| project TimeGenerated, Resource, ActivityName, Status, ErrorMessage
| order by TimeGenerated desc
```

---

## Appendix

### Alert Metrics

| Metric | Description |  
| --- | --- |  
| Airflow Integration Runtime Celery Task Timeout Error | Number of Celery tasks that have timed out. |  
| Airflow Integration Runtime Collect DB Dags | Number of DAGs collected from the database. |  
| Airflow Integration Runtime Cpu Percentage | CPU usage percentage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Cpu Usage | CPU usage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Dag Bag Size | Size of the DAG bag in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Dag Callback Exceptions | Number of exceptions occurred during DAG callbacks. |  
| Airflow Integration Runtime DAG File Refresh Error | Number of errors occurred during DAG file refresh. |  
| Airflow Integration Runtime DAG Processing Import Errors | Number of import errors occurred during DAG processing. |  
| Airflow Integration Runtime DAG Processing Last Duration | Duration of the last DAG processing. |  
| Airflow Integration Runtime DAG Processing Last Run Seconds Ago | Time in seconds since the last DAG processing run. |  
| Airflow Integration Runtime DAG Processing Processes | Number of processes used for DAG processing. |  
| Airflow Integration Runtime DAG Processing Processor Timeouts | Number of processor timeouts during DAG processing. |  
| Airflow Integration Runtime DAG Processing Total Parse Time | Total parse time for DAG processing. |  
| Airflow Integration Runtime DAG ProcessingManager Stalls | Number of stalls in the DAG ProcessingManager. |  
| Airflow Integration Runtime DAG Run Dependency Check | Number of dependency checks run for DAGs. |  
| Airflow Integration Runtime DAG Run Duration Failed | Duration of failed DAG runs. |  
| Airflow Integration Runtime DAG Run Duration Success | Duration of successful DAG runs. |  
| Airflow Integration Runtime DAG Run First Task Scheduling Delay | Delay in scheduling the first task of a DAG run. |  
| Airflow Integration Runtime DAG Run Schedule Delay | Delay in scheduling a DAG run. |  
| Airflow Integration Runtime Executor Open Slots | Number of open slots in the executor. |  
| Airflow Integration Runtime Executor Queued Tasks | Number of tasks queued in the executor. |  
| Airflow Integration Runtime Executor Running Tasks | Number of tasks running in the executor. |  
| Airflow Integration Runtime Heartbeat Failure | Number of heartbeat failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Job End | Number of jobs ended in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Job Start | Number of jobs started in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Memory Percentage | Memory usage percentage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Memory Usage | Memory usage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Node Count | Number of nodes in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Operator Failures | Number of operator failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Operator Successes | Number of operator successes in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Pool Open Slots | Number of open slots in the pool. |  
| Airflow Integration Runtime Pool Queued Slots | Number of queued slots in the pool. |  
| Airflow Integration Runtime Pool Running Slots | Number of running slots in the pool. |  
| Airflow Integration Runtime Pool Starving Tasks | Number of starving tasks in the pool. |  
| Airflow Integration Runtime Scheduler Critical Section Busy | Number of times the scheduler critical section was busy. |  
| Airflow Integration Runtime Scheduler Critical Section Duration | Duration of the scheduler critical section. |  
| Airflow Integration Runtime Scheduler Failed SLA Email Attempts | Number of failed attempts to send SLA emails by the scheduler. |  
| Airflow Integration Runtime Scheduler Heartbeats | Number of heartbeats from the scheduler. |  
| Airflow Integration Runtime Scheduler Orphaned Tasks Adopted | Number of orphaned tasks adopted by the scheduler. |  
| Airflow Integration Runtime Scheduler Orphaned Tasks Cleared | Number of orphaned tasks cleared by the scheduler. |  
| Airflow Integration Runtime Scheduler Tasks Executable | Number of executable tasks in the scheduler. |  
| Airflow Integration Runtime Scheduler Tasks Killed Externally | Number of tasks killed externally in the scheduler. |  
| Airflow Integration Runtime Scheduler Tasks Running | Number of running tasks in the scheduler. |  
| Airflow Integration Runtime Scheduler Tasks Starving | Number of starving tasks in the scheduler. |  
| Airflow Integration Runtime Started Task Instances | Number of task instances started in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Created Using Operator | Number of task instances created using an operator. |  
| Airflow Integration Runtime Task Instance Duration | Duration of task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Failures | Number of task instance failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Finished | Number of finished task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Previously Succeeded | Number of task instances that previously succeeded. |  
| Airflow Integration Runtime Task Instance Successes | Number of successful task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Removed From DAG | Number of tasks removed from a DAG. |  
| Airflow Integration Runtime Task Restored To DAG | Number of tasks restored to a DAG. |  
| Airflow Integration Runtime Triggers Blocked Main Thread | Number of triggers that blocked the main thread. |  
| Airflow Integration Runtime Triggers Failed | Number of failed triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Running | Number of running triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Succeeded | Number of successful triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Zombie Tasks Killed | Number of zombie tasks killed in the Airflow Integration Runtime. |  
| Cancelled activity runs metrics | Number of cancelled activity runs. |  
| Cancelled pipeline runs metrics | Number of cancelled pipeline runs. |  
| Cancelled SSIS integration runtime start metrics | Number of cancelled SSIS integration runtime starts. |  
| Cancelled SSIS package execution metrics | Number of cancelled SSIS package executions. |  
| Cancelled trigger runs metrics | Number of cancelled trigger runs. |  
| Copy available capacity percentage of MVNet integration runtime | Percentage of available capacity for copy operations in the MVNet integration runtime. |  
| Copy capacity utilization of MVNet integration runtime | Capacity utilization for copy operations in the MVNet integration runtime. |  
| Copy waiting queue length of MVNet integration runtime | Length of the waiting queue for copy operations in the MVNet integration runtime. |  
| Elapsed Time Pipeline Runs Metrics | Elapsed time for pipeline runs. |  
| External available capacity percentage of MVNet integration runtime | Percentage of available capacity for external operations in the MVNet integration runtime. |  
| External capacity utilization of MVNet integration runtime | Capacity utilization for external operations in the MVNet integration runtime. |  
| External waiting queue length of MVNet integration runtime | Length of the waiting queue for external operations in the MVNet integration runtime. |  
| Failed activity runs metrics | Number of failed activity runs. |  
| Failed pipeline runs metrics | Number of failed pipeline runs. |  
| Failed SSIS integration runtime start metrics | Number of failed SSIS integration runtime starts. |  
| Failed SSIS package execution metrics | Number of failed SSIS package executions. |  
| Failed trigger runs metrics | Number of failed trigger runs. |  
| Integration runtime available memory | Available memory in the integration runtime. |  
| Integration runtime available node count | Number of available nodes in the integration runtime. |  
| Integration runtime CPU utilization | CPU utilization in the integration runtime. |  
| Integration runtime queue duration | Duration of the queue in the integration runtime. |  
| Integration runtime queue length | Length of the queue in the integration runtime. |  
| Maximum allowed entities count | Maximum number of entities allowed. |  
| Maximum allowed factory size (GB unit) | Maximum size of the factory allowed in GB. |  
| Pipeline available capacity percentage of MVNet integration runtime | Percentage of available capacity for pipeline operations in the MVNet integration runtime. |  
| Pipeline capacity utilization of MVNet integration runtime | Capacity utilization for pipeline operations in the MVNet integration runtime. |  
| Pipeline waiting queue length of MVNet integration runtime | Length of the waiting queue for pipeline operations in the MVNet integration runtime. |  
| Stuck SSIS integration runtime stop metrics | Number of stuck SSIS integration runtime stops. |  
| Succeeded activity runs metrics | Number of successful activity runs. |  
| Succeeded pipeline runs metrics | Number of successful pipeline runs. |  
| Succeeded SSIS integration runtime start metrics | Number of successful SSIS integration runtime starts. |  
| Succeeded SSIS integration runtime stop metrics | Number of successful SSIS integration runtime stops. |  
| Succeeded SSIS package execution metrics | Number of successful SSIS package executions. |  
| Succeeded trigger runs metrics | Number of successful trigger runs. |  
| Total entities count | Total number of entities. |  
| Total factory size (GB unit) | Total size of the factory in GB. |  

---

### Logs

#### Category Groups

| <sub>Category Group</sub> | <sub>Description</sub> | <sub>Data Factory</sub> | <sub>Synapse</sub> |
:----- | :----- | :----- | :-----
| <sub>`allLogs`</sub> | <sub>Logs all available events, including audit logs and operational logs (e.g., pipeline runs, activity execution, SQL requests, Spark metrics)</sub> | ✅ | ✅ |
| <sub>`audit`</sub> | <sub>Logs security and access control events, including role assignments, security audits, and authentication failures</sub> | ❌ | ✅ |

#### Categories

| <sub>Service</sub> | <sub>Category</sub> | <sub>Description</sub> | <sub>Included in `audit`</sub> | <sub>Included in `allLogs`</sub> |
:----- | :----- | :----- | :----- | :-----
| <sub>Data Factory</sub> | <sub>`Airflow scheduler logs`</sub> | <sub>Logs scheduler operations for Apache Airflow</sub> | ❌ | ✅ |
| <sub>Data Factory</sub> | <sub>`Pipeline activity runs log`</sub> | <sub>Captures details about the execution of activities within a pipeline</sub> | ❌ | ✅ |
| <sub>Data Factory</sub> | <sub>`Pipeline runs log`</sub> | <sub>Records information about each pipeline run, including start time, end time, and status</sub> | ❌ | ✅ |
| <sub>Data Factory</sub> | <sub>`SSIS package execution data statistics`</sub> | <sub>Captures data statistics from SSIS package execution</sub> | ❌ | ✅ |
| <sub>Data Factory</sub> | <sub>`Trigger runs log`</sub> | <sub>Captures information about trigger runs that fired to start a pipeline</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Built-in SQL Pool Requests Ended`</sub> | <sub>Logs requests that have completed in the built-in SQL pool</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Integration Activity Runs`</sub> | <sub>Captures information about activities executed in Synapse pipelines</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Integration Pipeline Runs`</sub> | <sub>Logs details of pipeline executions inside Synapse</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Integration Trigger Runs`</sub> | <sub>Logs details about triggers that fired to start Synapse pipelines</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`SQL Security Audit Event`</sub> | <sub>Captures security-related SQL events (e.g., authentication failures, permission changes)</sub> | ✅ | ✅ |
| <sub>Synapse</sub> | <sub>`Synapse Gateway API Requests`</sub> | <sub>Captures API request logs for Synapse Gateway</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Synapse Link Event`</sub> | <sub>Captures logs related to Synapse Link operations</sub> | ❌ | ✅ |
| <sub>Synapse</sub> | <sub>`Synapse RBAC Operations`</sub> | <sub>Logs Synapse role-based access control (RBAC) operations</sub> | ✅ | ✅ |

### Tables

| <sub>Service</sub> | <sub>Category</sub> | <sub>Table</sub> | <sub>Description</sub> |
:----- | :----- | :----- | :-----
| <sub>Data Factory</sub> | <sub>`Airflow scheduler logs`</sub> | <sub>`ADFAirflowSchedulerLogs`</sub> | <sub>Logs scheduler operations for Apache Airflow.</sub> |
| <sub>Data Factory</sub> | <sub>`Pipeline activity runs log`</sub> | <sub>`ADFActivityRun`</sub> | <sub>Logs activity executions within pipelines (non-debug).</sub> |
| <sub>Data Factory</sub> | <sub>`Pipeline runs log`</sub> | <sub>`ADFPipelineRun`</sub> | <sub>Logs triggered pipeline executions (non-debug).</sub> |
| <sub>Data Factory</sub> | <sub>`Trigger runs log`</sub> | <sub>`ADFTriggerRun`</sub> | <sub>Logs triggers that fired pipeline executions.</sub> |
| <sub>Data Factory</sub> | <sub>`Sandbox activity runs log`</sub> | <sub>`ADFSandboxActivityRun`</sub> | <sub>Logs debug activity runs in Data Factory.</sub> |
| <sub>Data Factory</sub> | <sub>`Sandbox pipeline runs log`</sub> | <sub>`ADFSandboxPipelineRun`</sub> | <sub>Logs debug pipeline runs in Data Factory.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS package event messages`</sub> | <sub>`ADFSSISPackageEventMessages`</sub> | <sub>Logs events from SSIS package executions.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS package executable statistics`</sub> | <sub>`ADFSSISPackageExecutableStatistics`</sub> | <sub>Logs executable statistics from SSIS packages.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS package event message context`</sub> | <sub>`ADFSSISPackageEventMessageContext`</sub> | <sub>Provides context for SSIS package event messages.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS package execution component phases`</sub> | <sub>`ADFSSISPackageExecutionComponentPhases`</sub> | <sub>Logs component phase details of SSIS package executions.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS package execution data statistics`</sub> | <sub>`ADFSSISPackageExecutionDataStatistics`</sub> | <sub>Logs data statistics from SSIS package executions.</sub> |
| <sub>Data Factory</sub> | <sub>`SSIS integration runtime logs`</sub> | <sub>`ADFSSISIntegrationRuntimeLogs`</sub> | <sub>Logs related to SSIS integration runtime operations.</sub> |
| <sub>Synapse</sub> | <sub>`Integration Pipeline Runs`</sub> | <sub>`IntegrationPipelineRuns`</sub> | <sub>Logs triggered pipeline executions in Synapse.</sub> |
| <sub>Synapse</sub> | <sub>`Integration Activity Runs`</sub> | <sub>`IntegrationActivityRuns`</sub> | <sub>Logs activity executions within pipelines in Synapse.</sub> |
| <sub>Synapse</sub> | <sub>`Integration Trigger Runs`</sub> | <sub>`IntegrationTriggerRuns`</sub> | <sub>Logs triggers that fired pipeline executions in Synapse.</sub> |
| <sub>Synapse</sub> | <sub>`Synapse Gateway API Requests`</sub> | <sub>`SynapseGatewayApiRequests`</sub> | <sub>Logs API requests for Synapse Gateway.</sub> |
| <sub>Synapse</sub> | <sub>`Synapse RBAC Operations`</sub> | <sub>`SynapseRBACOperations`</sub> | <sub>Logs role-based access control (RBAC) changes.</sub> |
| <sub>Synapse</sub> | <sub>`SQL Security Audit Event`</sub> | <sub>`SQLSecurityAuditEvent`</sub> | <sub>Logs SQL authentication failures and permission changes.</sub> |
| <sub>Synapse</sub> | <sub>`Built-in SQL Pool Requests Ended`</sub> | <sub>`SynapseBuiltinSqlPoolRequestsEnded`</sub> | <sub>Logs completed requests in the built-in SQL pool.</sub> |
| <sub>Synapse</sub> | <sub>`Synapse Link Event`</sub> | <sub>`SynapseLinkEvent`</sub> | <sub>Logs events related to Synapse Link operations.</sub> |
| <sub>Synapse</sub> | <sub>`Apache Spark Applications`</sub> | <sub>`SynapseBigDataPoolApplicationsEnded`</sub> | <sub>Logs information about ended Apache Spark applications.</sub> |

#### Data Factory | Logs | Table: `ADFSandboxPipelineRun`

| <sub>Column Name</sub> | <sub>Description</sub> |
:----- | :-----
| <sub>`TenantId`</sub> | <sub>Unique identifier for the Azure tenant</sub> |
| <sub>`SourceSystem`</sub> | <sub>Indicates the log source, typically Azure</sub> |
| <sub>`TimeGenerated [UTC]`</sub> | <sub>Timestamp of the log event</sub> |
| <sub>`ResourceId`</sub> | <sub>Fully qualified Azure Resource ID of the Data Factory</sub> |
| <sub>`OperationName`</sub> | <sub>Current state of the pipeline (Queued, InProgress, Succeeded, etc.)</sub> |
| <sub>`Category`</sub> | <sub>Log category, which is `SandboxPipelineRuns` for debug pipelines</sub> |
| <sub>`CorrelationId`</sub> | <sub>Identifier for correlating related log entries</sub> |
| <sub>`Level`</sub> | <sub>Severity level of the log entry (e.g., Informational)</sub> |
| <sub>`Location`</sub> | <sub>Azure region where the pipeline execution occurred</sub> |
| <sub>`Tags`</sub> | <sub>Metadata tags associated with the pipeline run</sub> |
| <sub>`Start [UTC]`</sub> | <sub>Start time of the pipeline execution</sub> |
| <sub>`End [UTC]`</sub> | <sub>End time of the pipeline execution</sub> |
| <sub>`FailureType`</sub> | <sub>Indicates failure type if the pipeline execution failed</sub> |
| <sub>`PipelineName`</sub> | <sub>Name of the executed pipeline</sub> |
| <sub>`RunId`</sub> | <sub>Unique identifier for the pipeline execution</sub> |
| <sub>`Predecessors`</sub> | <sub>Links to any preceding pipeline runs</sub> |
| <sub>`Parameters`</sub> | <sub>Input parameters for the pipeline execution</sub> |
| <sub>`SystemParameters`</sub> | <sub>System-defined parameters for execution</sub> |
| <sub>`Type`</sub> | <sub>Log table type, which is `ADFSandboxPipelineRun`</sub> |
| <sub>`_ResourceId`</sub> | <sub>Normalized Azure resource ID</sub> |

#### Data Factory | Logs | Table: `ADFSandboxActivityRun`

| <sub>Column Name</sub> | <sub>Description</sub> |
:----- | :-----
| <sub>`TenantId`</sub> | <sub>Unique identifier for the Azure tenant</sub> |
| <sub>`SourceSystem`</sub> | <sub>Indicates the log source, typically Azure</sub> |
| <sub>`TimeGenerated [UTC]`</sub> | <sub>Timestamp of the log event</sub> |
| <sub>`ResourceId`</sub> | <sub>Fully qualified Azure Resource ID of the Data Factory</sub> |
| <sub>`OperationName`</sub> | <sub>Current state of the activity (Queued, InProgress, Succeeded, etc.)</sub> |
| <sub>`Category`</sub> | <sub>Log category, which is `SandboxActivityRuns` for debug activities</sub> |
| <sub>`CorrelationId`</sub> | <sub>Identifier for correlating related log entries</sub> |
| <sub>`Level`</sub> | <sub>Severity level of the log entry (e.g., Informational)</sub> |
| <sub>`Location`</sub> | <sub>Azure region where the activity execution occurred</sub> |
| <sub>`Tags`</sub> | <sub>Metadata tags associated with the activity run</sub> |
| <sub>`Start [UTC]`</sub> | <sub>Start time of the activity execution</sub> |
| <sub>`End [UTC]`</sub> | <sub>End time of the activity execution</sub> |
| <sub>`FailureType`</sub> | <sub>Indicates failure type if the activity execution failed</sub> |
| <sub>`PipelineName`</sub> | <sub>Name of the pipeline containing the executed activity</sub> |
| <sub>`Input`</sub> | <sub>Serialized input data provided to the activity</sub> |
| <sub>`Output`</sub> | <sub>Serialized output data from the activity</sub> |
| <sub>`ErrorCode`</sub> | <sub>Error code if the activity failed</sub> |
| <sub>`ErrorMessage`</sub> | <sub>Error message details if the activity failed</sub> |
| <sub>`Error`</sub> | <sub>Additional error information</sub> |
| <sub>`Type`</sub> | <sub>Log table type, which is `ADFSandboxActivityRun`</sub> |
| <sub>`_ResourceId`</sub> | <sub>Normalized Azure resource ID</sub> |

---

### Diagnostic Setting >> Destinations

| Destination | Description |  
| --- | --- |  
| Send to Log Analytics workspace | This option allows you to send the logs to a Log Analytics workspace. This is a cloud-based service for log data ingestion and storage. It provides real-time and historical analysis, visualizations, and insights for the data. |  
| Archive to a storage account | This option allows you to archive the logs to a storage account. This is useful for long-term storage and for compliance purposes. The logs can be retrieved and analyzed at a later date if needed. |  
| Stream to an event hub | This option allows you to stream the logs to an event hub. Event Hubs is a real-time data ingestion service that is capable of receiving and processing millions of events per second. You can use this option to integrate your logs with third-party services or custom analytics systems. |  
| Send to partner solution | This option allows you to send the logs to a partner solution. Azure has a number of partner solutions that provide advanced analytics and visualization capabilities. This option requires you to have a subscription with the partner solution. |  

### Alert Rules >> Signals

| Signal Name | Signal Type | Description |  
| --- | --- | --- |  
| Custom log search | Log Analytics | Allows you to perform a custom search on the logs. |  
| Activity Runs Availability | Log Analytics | Provides information about the availability of activity runs. |  
| PipelineRuns Availability | Log Analytics | Provides information about the availability of pipeline runs. |  
| TriggerRuns Availability | Log Analytics | Provides information about the availability of trigger runs. |  
| Resource health | Resource health | Provides information about the health of the resource. |  
| Airflow Integration Runtime Celery Task Timeout Error | Platform metrics | Number of Celery tasks that have timed out in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Collect DB Dags | Platform metrics | Number of DAGs collected from the database in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Cpu Percentage | Platform metrics | CPU usage percentage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Cpu Usage | Platform metrics | CPU usage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Dag Bag Size | Platform metrics | Size of the DAG bag in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Dag Callback Exceptions | Platform metrics | Number of exceptions occurred during DAG callbacks in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG File Refresh Error | Platform metrics | Number of errors occurred during DAG file refresh in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Import Errors | Platform metrics | Number of import errors occurred during DAG processing in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Last Duration | Platform metrics | Duration of the last DAG processing in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Last Run Seconds Ago | Platform metrics | Time in seconds since the last DAG processing run in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Processes | Platform metrics | Number of processes used for DAG processing in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Processor Timeouts | Platform metrics | Number of processor timeouts during DAG processing in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Processing Total Parse Time | Platform metrics | Total parse time for DAG processing in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG ProcessingManager Stalls | Platform metrics | Number of stalls in the DAG ProcessingManager in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Run Dependency Check | Platform metrics | Number of dependency checks run for DAGs in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Run Duration Failed | Platform metrics | Duration of failed DAG runs in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Run Duration Success | Platform metrics | Duration of successful DAG runs in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Run First Task Scheduling Delay | Platform metrics | Delay in scheduling the first task of a DAG run in the Airflow Integration Runtime. |  
| Airflow Integration Runtime DAG Run Schedule Delay | Platform metrics | Delay in scheduling a DAG run in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Executor Open Slots | Platform metrics | Number of open slots in the executor in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Executor Queued Tasks | Platform metrics | Number of tasks queued in the executor in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Executor Running Tasks | Platform metrics | Number of tasks running in the executor in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Heartbeat Failure | Platform metrics | Number of heartbeat failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Job End | Platform metrics | Number of jobs ended in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Job Start | Platform metrics | Number of jobs started in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Memory Percentage | Platform metrics | Memory usage percentage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Memory Usage | Platform metrics | Memory usage of the Airflow Integration Runtime. |  
| Airflow Integration Runtime Node Count | Platform metrics | Number of nodes in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Operator Failures | Platform metrics | Number of operator failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Operator Successes | Platform metrics | Number of operator successes in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Pool Open Slots | Platform metrics | Number of open slots in the pool in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Pool Queued Slots | Platform metrics | Number of queued slots in the pool in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Pool Running Slots | Platform metrics | Number of running slots in the pool in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Pool Starving Tasks | Platform metrics | Number of starving tasks in the pool in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Critical Section Busy | Platform metrics | Number of times the scheduler critical section was busy in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Critical Section Duration | Platform metrics | Duration of the scheduler critical section in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Failed SLA Email Attempts | Platform metrics | Number of failed attempts to send SLA emails by the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Heartbeats | Platform metrics | Number of heartbeats from the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Orphaned Tasks Adopted | Platform metrics | Number of orphaned tasks adopted by the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Orphaned Tasks Cleared | Platform metrics | Number of orphaned tasks cleared by the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Tasks Executable | Platform metrics | Number of executable tasks in the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Tasks Killed Externally | Platform metrics | Number of tasks killed externally in the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Tasks Running | Platform metrics | Number of running tasks in the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Scheduler Tasks Starving | Platform metrics | Number of starving tasks in the scheduler in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Started Task Instances | Platform metrics | Number of task instances started in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Created Using Operator | Platform metrics | Number of task instances created using an operator in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Duration | Platform metrics | Duration of task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Failures | Platform metrics | Number of task instance failures in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Finished | Platform metrics | Number of finished task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Previously Succeeded | Platform metrics | Number of task instances that previously succeeded in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Instance Successes | Platform metrics | Number of successful task instances in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Removed From DAG | Platform metrics | Number of tasks removed from a DAG in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Task Restored To DAG | Platform metrics | Number of tasks restored to a DAG in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Blocked Main Thread | Platform metrics | Number of triggers that blocked the main thread in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Failed | Platform metrics | Number of failed triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Running | Platform metrics | Number of running triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Triggers Succeeded | Platform metrics | Number of successful triggers in the Airflow Integration Runtime. |  
| Airflow Integration Runtime Zombie Tasks Killed | Platform metrics | Number of zombie tasks killed in the Airflow Integration Runtime. |  
| Cancelled activity runs metrics | Platform metrics | Number of cancelled activity runs. |  
| Cancelled pipeline runs metrics | Platform metrics | Number of cancelled pipeline runs. |  
| Cancelled SSIS integration runtime start metrics | Platform metrics | Number of cancelled SSIS integration runtime starts. |  
| Cancelled SSIS package execution metrics | Platform metrics | Number of cancelled SSIS package executions. |  
| Cancelled trigger runs metrics | Platform metrics | Number of cancelled trigger runs. |  
| Copy available capacity percentage of MVNet integration runtime | Platform metrics | Percentage of available capacity for copy operations in the MVNet integration runtime. |  
| Copy capacity utilization of MVNet integration runtime | Platform metrics | Capacity utilization for copy operations in the MVNet integration runtime. |  
| Copy waiting queue length of MVNet integration runtime | Platform metrics | Length of the waiting queue for copy operations in the MVNet integration runtime. |  
| Elapsed Time Pipeline Runs Metrics | Platform metrics | Elapsed time for pipeline runs. |  
| External available capacity percentage of MVNet integration runtime | Platform metrics | Percentage of available capacity for external operations in the MVNet integration runtime. |  
| External capacity utilization of MVNet integration runtime | Platform metrics | Capacity utilization for external operations in the MVNet integration runtime. |  
| External waiting queue length of MVNet integration runtime | Platform metrics | Length of the waiting queue for external operations in the MVNet integration runtime. |  
| Failed activity runs metrics | Platform metrics | Number of failed activity runs. |  
| Failed pipeline runs metrics | Platform metrics | Number of failed pipeline runs. |  
| Failed SSIS integration runtime start metrics | Platform metrics | Number of failed SSIS integration runtime starts. |  
| Failed SSIS package execution metrics | Platform metrics | Number of failed SSIS package executions. |  
| Failed trigger runs metrics | Platform metrics | Number of failed trigger runs. |  
| Integration runtime available memory | Platform metrics | Available memory in the integration runtime. |  
| Integration runtime available node count | Platform metrics | Number of available nodes in the integration runtime. |  
| Integration runtime CPU utilization | Platform metrics | CPU utilization in the integration runtime. |  
| Integration runtime queue duration | Platform metrics | Duration of the queue in the integration runtime. |  
| Integration runtime queue length | Platform metrics | Length of the queue in the integration runtime. |
| Maximum allowed entities count | Platform metrics | Maximum number of entities allowed. |  
| Maximum allowed factory size (GB unit) | Platform metrics | Maximum size of the factory allowed in GB. |  
| Pipeline available capacity percentage of MVNet integration runtime | Platform metrics | Percentage of available capacity for pipeline operations in the MVNet integration runtime. |  
| Pipeline capacity utilization of MVNet integration runtime | Platform metrics | Capacity utilization for pipeline operations in the MVNet integration runtime. |  
| Pipeline waiting queue length of MVNet integration runtime | Platform metrics | Length of the waiting queue for pipeline operations in the MVNet integration runtime. |  
| Stuck SSIS integration runtime stop metrics | Platform metrics | Number of stuck SSIS integration runtime stops. |  
| Succeeded activity runs metrics | Platform metrics | Number of successful activity runs. |  
| Succeeded pipeline runs metrics | Platform metrics | Number of successful pipeline runs. |  
| Succeeded SSIS integration runtime start metrics | Platform metrics | Number of successful SSIS integration runtime starts. |  
| Succeeded SSIS integration runtime stop metrics | Platform metrics | Number of successful SSIS integration runtime stops. |  
| Succeeded SSIS package execution metrics | Platform metrics | Number of successful SSIS package executions. |  
| Succeeded trigger runs metrics | Platform metrics | Number of successful trigger runs. |  
| Total entities count | Platform metrics | Total number of entities. |  
| Total factory size (GB unit) | Platform metrics | Total size of the factory in GB. |
