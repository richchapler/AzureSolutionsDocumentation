# Data Factory: Debugging

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ead90204-81af-4428-a47c-803cae65a214" width="1000" />

## Studio

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

### Cluster Failures

Clusters fail when they become unavailable, overloaded, or are misconfigured.

In the examples below, we will not try to overload the cluster (which is difficult because Azure is robust).
Instead, we will simulate failure with simple incorrect configurations, such as referencing nonexistent resources, using invalid connection strings, or misconfiguring integration runtimes.

#### Simulate Failure (Debug)

To simulate cluster failure, click "Debug" on the pipeline, then immediately flip over to the dataflow and switch "Data flow debug" to off.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/caf0e349-8fa3-4a3e-aad2-ad0cf9e0e110" width="800" title="Snipped February 10, 2025" />

Flip back to the pipeline and watch the debug run until it fails.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2bdefa42-fd5a-4f5d-a974-10dec7ba2a9c" width="800" title="Snipped February 10, 2025" />

Click the "Error" icon next to the "Failed" message in the Activity grid.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/9a78b826-0977-4b76-a329-2312607262d4" width="800" title="Snipped February 10, 2025" />

The error message: `Internal Server Error: An error occurred while executing the data flow activity. Please try again later.` is underwhelming.

"Debug" produces logs: `**ADFSandboxActivityRun**` and `ADFSandboxPipelineRun`. Activity logs include error messages.

Navigate to Data Factory >> Logs >> New Query >> "KQL mode" (in the portal).

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/3ac9bc90-104c-49cb-841d-2a224ea9f601" width="800" title="Snipped February 10, 2025" />

##### `ADFSandboxActivityRun`
...resulting from "Debug" >> "Use activity runtime" and "Use data flow debug session"

```kql
ADFSandboxActivityRun
| order by TimeGenerated desc
| take 1
| extend TimeGenerated_Pacific = TimeGenerated - 8h
| project TenantId, SourceSystem, TimeGenerated, TimeGenerated_Pacific, ResourceId, OperationName, Category, CorrelationId, Level, Location, Tags, Status, UserProperties, Annotations, EventMessage, Start, ActivityName, ActivityRunId, PipelineRunId, EffectiveIntegrationRuntime, ActivityType, ActivityIterationCount, LinkedServiceName, End, FailureType, PipelineName, Input, Output, ErrorCode, ErrorMessage, Error, Type, _ResourceId
| project TimeGenerated_Pacific, Output = todynamic(pack_all())
```

###### Results
(manually pivoted for readability)

```json
{
  "TenantId": "696b4f93-a588-41cd-a945-76bf106b8cde",
  "SourceSystem": "Azure",
  "TimeGenerated": "2025-02-11T14:36:02.7883900Z",
  "TimeGenerated_Pacific": "2025-02-11T06:36:02.7883900Z",
  "ResourceId": "/SUBSCRIPTIONS/ED7EAF77-D411-484B-92E6-5CBA0B6D8098/RESOURCEGROUPS/UBSAG/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/UBSAGDATAFACTORY",
  "OperationName": "customer_sql_to_storage - Failed",
  "Category": "SandboxActivityRuns",
  "CorrelationId": "ef259719-6310-4bf3-84c8-322e11fa5c85",
  "Level": "Error",
  "Location": "westus",
  "Tags": "{}",
  "Status": "Failed",
  "UserProperties": "",
  "Annotations": "",
  "EventMessage": "",
  "Start": "2025-02-11T14:35:40.0000000Z",
  "ActivityName": "customer_sql_to_storage",
  "ActivityRunId": "fdd9938a-8c33-457f-8d56-2a9567e0602b",
  "PipelineRunId": "ef259719-6310-4bf3-84c8-322e11fa5c85",
  "EffectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (West US)",
  "ActivityType": "ExecuteDataFlow",
  "ActivityIterationCount": 1,
  "LinkedServiceName": "",
  "End": "2025-02-11T14:36:02.0000000Z",
  "FailureType": "SystemError",
  "PipelineName": "customer_sql_to_storage",
  "Input": "",
  "Output": "",
  "ErrorCode": "",
  "ErrorMessage": "",
  "Error": "",
  "Type": "ADFSandboxActivityRun",
  "_ResourceId": "/subscriptions/ed7eaf77-d411-484b-92e6-5cba0b6d8098/resourcegroups/ubsag/providers/microsoft.datafactory/factories/ubsagdatafactory"
}
```

You will notice that the logged data for this failure is limited:

* `"OperationName": "customer_sql_to_storage - Failed"`
* `"Level": "Error"`
* `"Status": "Failed"`
* `"FailureType": "SystemError"`
* `"ErrorCode": ""`
* `"ErrorMessage": ""`
* `"Error": ""`

#### Simulate Failure (Trigger)

Next, we want to repeat with "Trigger Now" to see logs: `**ADFActivityRun**` and `ADFPipelineRun`, but trigger is harder.

If debug mode is off when a pipeline with a data flow is triggered, Azure Data Factory (ADF) will restart a new Spark cluster to execute the data flow.

##### How Data Flow Clusters Work

1. Debug Mode On:  
   - When you enable Data Flow Debug, ADF pre-warms a Spark cluster.
   - The cluster stays alive for up to 4 hours for interactive development and testing.
   - When you run a data flow while debug mode is on, it reuses the existing cluster (faster execution).

2. Debug Mode Off:  
   - When you trigger a pipeline execution (via "Trigger Now" or scheduled runs) and debug mode is off, ADF does not use the debug cluster.
   - Instead, ADF provisions a new Spark cluster to execute the data flow.
   - This startup process introduces cold start latency, adding several minutes to the execution time.

###### Impact on Cluster Behavior
- If debug mode was previously on, but you turn it off before triggering, the debug cluster remains but is not used.
- A new cluster is created for the execution, and once the pipeline finishes, the cluster is deallocated.
- Each new pipeline execution (with debug off) repeats the cluster provisioning process unless using Azure IR with TTL (Time-to-Live).

Would you like a query to check cluster restarts from logs?

##### `ADFActivityRun`
...resulting from "Add trigger" >> "Trigger now"

```kql
ADFActivityRun
| order by TimeGenerated desc
| take 1
| extend TimeGenerated_Pacific = TimeGenerated - 8h
| project TenantId, SourceSystem, TimeGenerated, TimeGenerated_Pacific, ResourceId, OperationName, Category, CorrelationId, Level, Location, Tags, Status, UserProperties, Annotations, EventMessage, Start, ActivityName, ActivityRunId, PipelineRunId, EffectiveIntegrationRuntime, ActivityType, ActivityIterationCount, LinkedServiceName, End, FailureType, PipelineName, Input, Output, ErrorCode, ErrorMessage, Error, Type, _ResourceId
| project TimeGenerated_Pacific, Output = pack_all()
```

_Notes: 1) `project` includes all possible columns (not shown otherwise) and 2) `output` logic included to output JSON rather than columnar data_

###### Results
(manually pivoted for readability)

```json
{
{
  "TenantId": "696b4f93-a588-41cd-a945-76bf106b8cde",
  "SourceSystem": "Azure",
  "TimeGenerated": "2025-02-10T16:34:40.1810160Z",
  "ResourceId": "/SUBSCRIPTIONS/ED7EAF77-D411-484B-92E6-5CBA0B6D8098/RESOURCEGROUPS/{prefix}/PROVIDERS/MICROSOFT.DATAFACTORY/FACTORIES/{prefix}DATAFACTORY",
  "OperationName": "customer_sql_to_storage - Succeeded",
  "Category": "ActivityRuns",
  "CorrelationId": "04901bdc-f92f-46a7-98e6-2037581006a2",
  "Level": "Informational",
  "Location": "westus",
  "Tags": {},
  "Status": "Succeeded",
  "UserProperties": {},
  "Annotations": [],
  "EventMessage": "",
  "Start": "2025-02-10T16:31:34.0000000Z",
  "End": "2025-02-10T16:34:40.0000000Z",
  "Activity": {
    "Name": "customer_sql_to_storage",
    "RunId": "f3c4a3de-bc18-43dc-880b-b88679678118",
    "Type": "ExecuteDataFlow",
    "IterationCount": 1,
    "LinkedServiceName": ""
  },
  "Pipeline": {
    "Name": "customer_sql_to_storage",
    "RunId": "04901bdc-f92f-46a7-98e6-2037581006a2"
  },
  "EffectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (West US)",
  "Failure": {
    "Type": "",
    "ErrorCode": "",
    "ErrorMessage": "",
    "ErrorDetails": {
      "errorCode": "",
      "message": "",
      "failureType": "",
      "target": "customer_sql_to_storage",
      "details": ""
    }
  },
  "Input": {
    "Dataflow": {
      "ReferenceName": "customer_sql_to_storage",
      "Type": "DataFlowReference",
      "Parameters": {},
      "DatasetParameters": {
        "{prefix}sqldatabase": {},
        "{prefix}storagecontainer": {}
      }
    },
    "Staging": {},
    "Compute": {
      "CoreCount": 8,
      "ComputeType": "General"
    },
    "TraceLevel": "Fine",
    "DataFlowETag": "390225c1-0000-0700-0000-67a620e70000"
  },
  "Output": {
    "RunStatus": {
      "IdleTimeBeforeCurrentJob": 0,
      "SparkVersion": "3.3",
      "ComputeAcquisitionDuration": 145868,
      "Version": "20241104.1",
      "Metrics": {
        "{prefix}storagecontainer": {
          "Format": "json",
          "Stages": [
            {
              "Stage": 0,
              "PartitionTimes": [7821],
              "RecordsWritten": 847,
              "LastUpdateTime": "2025-02-10 16:34:36.253",
              "BytesWritten": 359128,
              "RecordsRead": 847,
              "BytesRead": 0,
              "PartitionStatus": "Success",
              "Streams": {
                "{prefix}storagecontainer": {
                  "Count": 847,
                  "Cached": false,
                  "TotalPartitions": 1,
                  "PartitionStatus": "Success",
                  "PartitionCounts": [847],
                  "Type": "sink"
                },
                "{prefix}sqldatabase": {
                  "Count": 847,
                  "Cached": false,
                  "TotalPartitions": 1,
                  "PartitionStatus": "Success",
                  "PartitionCounts": [847],
                  "Type": "source"
                }
              },
              "Target": "{prefix}storagecontainer",
              "Time": 23667,
              "ProgressState": "Completed"
            }
          ],
          "SinkPostProcessingTime": 0,
          "Store": "adlsgen2",
          "RowsWritten": 847,
          "Details": {
            "PreCommandsDuration": [0],
            "PostCommandsDuration": [0]
          },
          "ProgressState": "Completed",
          "Sources": {
            "{prefix}sqldatabase": {
              "RowsRead": 847,
              "Store": "sqlserver",
              "Details": {},
              "Format": "table"
            }
          },
          "SinkProcessingTime": 26306
        }
      },
      "ClusterComputeId": "f3c4a3de-bc18-43dc-880b-b88679678118",
      "DSL": "\nsource() ~> {prefix}sqldatabase\n\n{prefix}sqldatabase sink() ~> {prefix}storagecontainer",
      "IntegrationRuntimeName": "General-8-1-15a24942-8030-4b79-b812-6132bd83edad",
      "SparkRunId": "0"
    },
    "EffectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (West US)",
    "BillingReference": {
      "ActivityType": "executedataflow",
      "BillableDuration": [
        {
          "MeterType": "Data Flow",
          "Duration": 0.38505121488888888,
          "Unit": "coreHour",
          "SessionType": "JobCluster"
        }
      ]
    },
    "ReportLineageToPurview": {
      "Status": "NotReported"
    }
  },
  "Type": "ADFActivityRun",
  "_ResourceId": "/subscriptions/ed7eaf77-d411-484b-92e6-5cba0b6d8098/resourcegroups/{prefix}/providers/microsoft.datafactory/factories/{prefix}datafactory"
}
```

### Integration Runtime Failures

Integration runtime failures occur when the compute resource used to process pipeline activities becomes unavailable.

We have been using the default `AutoResolveIntegrationRuntime` integration runtime, but will create a new one over which we have greater control for this exercise.

Navigate to "Manage" >> "Integration runtimes", then click "+ New"

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/3b1f3870-3277-4cee-8944-40c1047c3503" width="800" title="Snipped February 11, 2025" />

On the "Integration runtime setup" popout, select "Azure, Self-Hosted" and then click "Continue".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/98d96e7a-e17d-4ab7-bc00-0896ef5d93d8" width="800" title="Snipped February 11, 2025" />

On the "Integration runtime setup" >> "Network environment:" popout, select "Self-Hosted" and then click "Continue".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/17bcd4a6-ebd0-48e4-8be8-27ea850b4d08" width="800" title="Snipped February 11, 2025" />

Name the integration runtime and then click "Create".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/8c82c43a-8ac0-4345-a73c-16bd012292f4" width="800" title="Snipped February 11, 2025" />

Follow the instructions to install "Microsoft Integration Runtime" (latest version) on your computer.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/blah" width="800" title="Snipped February 11, 2025" />
![Uploading image.png…]()

Navigate to "Manage" >> "Linked services" and click "+New". On the "New linked service" popout, select "SQL server" and then click "Continue".











to use the new Integration Runtime / on-prem data source {e.g., [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)}.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/3b952b11-49b5-4d28-acb3-5f5d0cc3f23c" width="800" title="Snipped February 11, 2025" />

Once complete, click "Publish all".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a8a27f74-745b-4e1b-9ea1-56ce482b9dea" width="800" title="Snipped February 11, 2025" />

#### Simulate Failure

Simulate an Integration Runtime failure by deleting the integration runtime immediately after triggering the pipeline.

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

| <sub>Metric</sub> | <sub>Description</sub> |  
:----- | :-----
| <sub>Airflow Integration Runtime Celery Task Timeout Error</sub> | <sub>Number of Celery tasks that have timed out.</sub> |  
| <sub>Airflow Integration Runtime Collect DB Dags</sub> | <sub>Number of DAGs collected from the database.</sub> |  
| <sub>Airflow Integration Runtime Cpu Percentage</sub> | <sub>CPU usage percentage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Cpu Usage</sub> | <sub>CPU usage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Dag Bag Size</sub> | <sub>Size of the DAG bag in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Dag Callback Exceptions</sub> | <sub>Number of exceptions occurred during DAG callbacks.</sub> |  
| <sub>Airflow Integration Runtime DAG File Refresh Error</sub> | <sub>Number of errors occurred during DAG file refresh.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Import Errors</sub> | <sub>Number of import errors occurred during DAG processing.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Last Duration</sub> | <sub>Duration of the last DAG processing.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Last Run Seconds Ago</sub> | <sub>Time in seconds since the last DAG processing run.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Processes</sub> | <sub>Number of processes used for DAG processing.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Processor Timeouts</sub> | <sub>Number of processor timeouts during DAG processing.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Total Parse Time</sub> | <sub>Total parse time for DAG processing.</sub> |  
| <sub>Airflow Integration Runtime DAG ProcessingManager Stalls</sub> | <sub>Number of stalls in the DAG ProcessingManager.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Dependency Check</sub> | <sub>Number of dependency checks run for DAGs.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Duration Failed</sub> | <sub>Duration of failed DAG runs.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Duration Success</sub> | <sub>Duration of successful DAG runs.</sub> |  
| <sub>Airflow Integration Runtime DAG Run First Task Scheduling Delay</sub> | <sub>Delay in scheduling the first task of a DAG run.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Schedule Delay</sub> | <sub>Delay in scheduling a DAG run.</sub> |  
| <sub>Airflow Integration Runtime Executor Open Slots</sub> | <sub>Number of open slots in the executor.</sub> |  
| <sub>Airflow Integration Runtime Executor Queued Tasks</sub> | <sub>Number of tasks queued in the executor.</sub> |  
| <sub>Airflow Integration Runtime Executor Running Tasks</sub> | <sub>Number of tasks running in the executor.</sub> |  
| <sub>Airflow Integration Runtime Heartbeat Failure</sub> | <sub>Number of heartbeat failures in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Job End</sub> | <sub>Number of jobs ended in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Job Start</sub> | <sub>Number of jobs started in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Memory Percentage</sub> | <sub>Memory usage percentage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Memory Usage</sub> | <sub>Memory usage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Node Count</sub> | <sub>Number of nodes in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Operator Failures</sub> | <sub>Number of operator failures in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Operator Successes</sub> | <sub>Number of operator successes in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Pool Open Slots</sub> | <sub>Number of open slots in the pool.</sub> |  
| <sub>Airflow Integration Runtime Pool Queued Slots</sub> | <sub>Number of queued slots in the pool.</sub> |  
| <sub>Airflow Integration Runtime Pool Running Slots</sub> | <sub>Number of running slots in the pool.</sub> |  
| <sub>Airflow Integration Runtime Pool Starving Tasks</sub> | <sub>Number of starving tasks in the pool.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Critical Section Busy</sub> | <sub>Number of times the scheduler critical section was busy.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Critical Section Duration</sub> | <sub>Duration of the scheduler critical section.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Failed SLA Email Attempts</sub> | <sub>Number of failed attempts to send SLA emails by the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Heartbeats</sub> | <sub>Number of heartbeats from the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Orphaned Tasks Adopted</sub> | <sub>Number of orphaned tasks adopted by the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Orphaned Tasks Cleared</sub> | <sub>Number of orphaned tasks cleared by the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Tasks Executable</sub> | <sub>Number of executable tasks in the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Tasks Killed Externally</sub> | <sub>Number of tasks killed externally in the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Tasks Running</sub> | <sub>Number of running tasks in the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Tasks Starving</sub> | <sub>Number of starving tasks in the scheduler.</sub> |  
| <sub>Airflow Integration Runtime Started Task Instances</sub> | <sub>Number of task instances started in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Created Using Operator</sub> | <sub>Number of task instances created using an operator.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Duration</sub> | <sub>Duration of task instances in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Failures</sub> | <sub>Number of task instance failures in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Finished</sub> | <sub>Number of finished task instances in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Previously Succeeded</sub> | <sub>Number of task instances that previously succeeded.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Successes</sub> | <sub>Number of successful task instances in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Removed From DAG</sub> | <sub>Number of tasks removed from a DAG.</sub> |  
| <sub>Airflow Integration Runtime Task Restored To DAG</sub> | <sub>Number of tasks restored to a DAG.</sub> |  
| <sub>Airflow Integration Runtime Triggers Blocked Main Thread</sub> | <sub>Number of triggers that blocked the main thread.</sub> |  
| <sub>Airflow Integration Runtime Triggers Failed</sub> | <sub>Number of failed triggers in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Triggers Running</sub> | <sub>Number of running triggers in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Triggers Succeeded</sub> | <sub>Number of successful triggers in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Zombie Tasks Killed</sub> | <sub>Number of zombie tasks killed in the Airflow Integration Runtime.</sub> |  

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

| <sub>Destination</sub> | <sub>Description</sub> |  
:----- | :-----
| <sub>Send to Log Analytics workspace</sub> | <sub>This option allows you to send the logs to a Log Analytics workspace. This is a cloud-based service for log data ingestion and storage. It provides real-time and historical analysis, visualizations, and insights for the data.</sub> |  
| <sub>Archive to a storage account</sub> | <sub>This option allows you to archive the logs to a storage account. This is useful for long-term storage and for compliance purposes. The logs can be retrieved and analyzed at a later date if needed.</sub> |  
| <sub>Stream to an event hub</sub> | <sub>This option allows you to stream the logs to an event hub. Event Hubs is a real-time data ingestion service that is capable of receiving and processing millions of events per second. You can use this option to integrate your logs with third-party services or custom analytics systems.</sub> |  
| <sub>Send to partner solution</sub> | <sub>This option allows you to send the logs to a partner solution. Azure has a number of partner solutions that provide advanced analytics and visualization capabilities. This option requires you to have a subscription with the partner solution.</sub> |  

### Alert Rules >> Signals

| <sub>Signal Name</sub> | <sub>Signal Type</sub> | <sub>Description</sub> |  
:----- | :----- | :-----
| <sub>Custom log search</sub> | <sub>Log Analytics</sub> | <sub>Allows you to perform a custom search on the logs.</sub> |  
| <sub>Activity Runs Availability</sub> | <sub>Log Analytics</sub> | <sub>Provides information about the availability of activity runs.</sub> |  
| <sub>PipelineRuns Availability</sub> | <sub>Log Analytics</sub> | <sub>Provides information about the availability of pipeline runs.</sub> |  
| <sub>TriggerRuns Availability</sub> | <sub>Log Analytics</sub> | <sub>Provides information about the availability of trigger runs.</sub> |  
| <sub>Resource health</sub> | <sub>Resource health</sub> | <sub>Provides information about the health of the resource.</sub> |  
| <sub>Airflow Integration Runtime Celery Task Timeout Error</sub> | <sub>Platform</sub> | <sub>Number of Celery tasks that have timed out in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Collect DB Dags</sub> | <sub>Platform</sub> | <sub>Number of DAGs collected from the database in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Cpu Percentage</sub> | <sub>Platform</sub> | <sub>CPU usage percentage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Cpu Usage</sub> | <sub>Platform</sub> | <sub>CPU usage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Dag Bag Size</sub> | <sub>Platform</sub> | <sub>Size of the DAG bag in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Dag Callback Exceptions</sub> | <sub>Platform</sub> | <sub>Number of exceptions occurred during DAG callbacks in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG File Refresh Error</sub> | <sub>Platform</sub> | <sub>Number of errors occurred during DAG file refresh in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Import Errors</sub> | <sub>Platform</sub> | <sub>Number of import errors occurred during DAG processing in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Last Duration</sub> | <sub>Platform</sub> | <sub>Duration of the last DAG processing in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Last Run Seconds Ago</sub> | <sub>Platform</sub> | <sub>Time in seconds since the last DAG processing run in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Processes</sub> | <sub>Platform</sub> | <sub>Number of processes used for DAG processing in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Processor Timeouts</sub> | <sub>Platform</sub> | <sub>Number of processor timeouts during DAG processing in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Processing Total Parse Time</sub> | <sub>Platform</sub> | <sub>Total parse time for DAG processing in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG ProcessingManager Stalls</sub> | <sub>Platform</sub> | <sub>Number of stalls in the DAG ProcessingManager in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Dependency Check</sub> | <sub>Platform</sub> | <sub>Number of dependency checks run for DAGs in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Duration Failed</sub> | <sub>Platform</sub> | <sub>Duration of failed DAG runs in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Duration Success</sub> | <sub>Platform</sub> | <sub>Duration of successful DAG runs in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Run First Task Scheduling Delay</sub> | <sub>Platform</sub> | <sub>Delay in scheduling the first task of a DAG run in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime DAG Run Schedule Delay</sub> | <sub>Platform</sub> | <sub>Delay in scheduling a DAG run in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Executor Open Slots</sub> | <sub>Platform</sub> | <sub>Number of open slots in the executor in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Executor Queued Tasks</sub> | <sub>Platform</sub> | <sub>Number of tasks queued in the executor in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Executor Running Tasks</sub> | <sub>Platform</sub> | <sub>Number of tasks running in the executor in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Heartbeat Failure</sub> | <sub>Platform</sub> | <sub>Number of heartbeat failures in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Job End</sub> | <sub>Platform</sub> | <sub>Number of jobs ended in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Job Start</sub> | <sub>Platform</sub> | <sub>Number of jobs started in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Memory Percentage</sub> | <sub>Platform</sub> | <sub>Memory usage percentage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Memory Usage</sub> | <sub>Platform</sub> | <sub>Memory usage of the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Node Count</sub> | <sub>Platform</sub> | <sub>Number of nodes in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Operator Failures</sub> | <sub>Platform</sub> | <sub>Number of operator failures in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Operator Successes</sub> | <sub>Platform</sub> | <sub>Number of operator successes in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Pool Open Slots</sub> | <sub>Platform</sub> | <sub>Number of open slots in the pool in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Pool Queued Slots</sub> | <sub>Platform</sub> | <sub>Number of queued slots in the pool in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Pool Running Slots</sub> | <sub>Platform</sub> | <sub>Number of running slots in the pool in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Pool Starving Tasks</sub> | <sub>Platform</sub> | <sub>Number of starving tasks in the pool in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Heartbeats</sub> | <sub>Platform</sub> | <sub>Number of heartbeats from the scheduler in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Scheduler Tasks Running</sub> | <sub>Platform</sub> | <sub>Number of running tasks in the scheduler in the Airflow Integration Runtime.</sub> |  
| <sub>Airflow Integration Runtime Task Instance Successes</sub> | <sub>Platform</sub> | <sub>Number of successful task instances in the Airflow Integration Runtime.</sub> |  
