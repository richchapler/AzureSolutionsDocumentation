# Data Factory: Debugging

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ead90204-81af-4428-a47c-803cae65a214" width="1000" />

## Use Case
* "We need to understand all of the ways that we can debug our Data Factory pipelines, data flows, etc."

## Studio

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/56315a73-0864-4319-a568-63bc84a01b40" width="800" title="Snipped January 21, 2025" />

### Pipelines

1. **Open Azure Data Factory Studio**:
   - Launch Azure Data Factory Studio.
   - Navigate to the **Author** tab.
   - Click the **+** button and select **Pipeline** to create a new pipeline.

2. **Add Activities to the Pipeline**:
   - Drag and drop the required activities from the **Activities pane** onto the pipeline canvas.
   - Example: Add a **Copy Data activity** to copy data from one source to another.

3. **Configure the "onFail" Feature**:
   - Select the activity you want to monitor for failures.
   - In the **Activities pane**, locate the **onFail** section and click **Add activity**.
   - Choose the activity to execute if the primary activity fails, such as a **Stored Procedure activity** to log failure details.

4. **Set Up the Stored Procedure Activity**:
   - Drag and drop a **Stored Procedure activity** onto the pipeline canvas.
   - Configure the activity to connect to your SQL Server database.
   - Specify the stored procedure to log failure details, ensuring it accepts parameters for:
     - **Error message**: A description of the failure.
     - **Activity name**: The name of the failed activity.
     - **Timestamp**: The time the failure occurred.

5. **Link Activities Using "onFail"**:
   - Link the primary activity to the Stored Procedure activity using an **onFail dependency**.
   - Ensure the Stored Procedure activity only executes when the primary activity fails.

6. **Publish and Test the Pipeline**:
   - Click **Publish** to save and publish your pipeline.
   - Trigger the pipeline to test its execution.
   - Verify that if the primary activity fails, the **onFail** feature triggers the Stored Procedure activity, logging failure details to your SQL Server table.

---

### Data Flows

#### Step 1: Prepare Example

Navigate to Data Factory Studio >> Author and add a new Data Flow.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/b9b25a94-93a1-42eb-9643-debdefa471da" width="800" title="Snipped January 21, 2025" />

Toggle "Data flow debug" and in the resulting pop-out, set "Debug time to live" to four hours.

2. **Add Source Dataset**  
   - In the data flow canvas, click **Add Source**  
   - Click **Source options** > **New dataset**  
     - Select **Azure SQL Database** as the source type  
     - Configure the dataset:  
       - Choose an **Azure SQL Database linked service** (or create a new one)  
       - Specify the server name, database name (e.g., `AdventureWorks`), and authentication credentials  
       - Select table `SalesLT.Address`

3. **Inspect Source Schema**  
   - Click the **Data Preview** tab  
   - Click **Refresh** to load a sample of the data from the **Address** table

4. **Add a Transformation Activity**  
   - Click + in the bottom right of the Source  
   - Search and select **Filter** transformation  
   - Configure the transformation: Add a filter condition `StateProvince == 'Arizona'`

5. **Add a Sink Dataset**  
   - Click + in the bottom right of the Filter  
   - Search and select **Sink** transformation  
   - Click **Sink options** > **New dataset**  
     - Select **Azure SQL Database** as the destination type  
     - Configure the dataset:  
       - Choose the same **Azure SQL Database linked service** as the source  
       - Create the table schema with:  
         ```sql
         SELECT * INTO [SalesLT].[Address2] FROM [SalesLT].[Address] WHERE 1 = 0
         ALTER TABLE [SalesLT].[Address2] DROP COLUMN AddressID
         ```  
       - Ensure the sink table schema matches the output from the transformation

6. **Add Data Flow to Pipeline**  
   - Navigate to **Author** > **+** > **Pipeline**  
   - Drag and drop the **Data Flow** activity into the pipeline canvas  
   - Configure the activity: Select the created data flow (e.g., `dataflow1`)

7. **Set the Logging Level**  
   - Open the **Settings** tab of the data flow activity in the pipeline  
   - Under **Monitoring**, set **Logging Level** to:  
     - **None**: No logs collected (default)  
     - **Basic**: Captures high-level details (start, end, row counts)  
     - **Verbose**: Captures detailed logs, including transformation steps and data lineage  
   - For this exercise, set the level to **Verbose**

8. **Preview Data**  
   - Open the **Data Preview** tab for each activity (source, transformation, and sink)  
   - Click **Refresh** to load sample data  
   - Verify:  
     - Source data is correctly loaded from the **Address** table  
     - The filter transformation produces only Arizona addresses  
     - The sink configuration aligns with the target table `Address2`

9. **Run the Data Flow in Debug Mode**  
   - Click **Debug** to execute the pipeline interactively  
   - Monitor execution in real-time via the **Output** pane

---

#### Step 2: Review

1. **Review Debug Results**  
   - Expand each activity in the debug results  
   - Examine details, including:  
     - Row counts at each step  
     - Errors during transformations or data writing  
     - Connectivity issues with the database  

2. **Logging Levels Overview**  
   - **None**: Minimal details (pipeline status only)  
   - **Basic**: High-level details (row counts, activity times)  
   - **Verbose**: Complete details (data lineage, transformation logs, error traces)  

3. **Areas Affected by Logging Levels**  
   - **Pipeline Runs Monitoring**: Shows pipeline-level or activity-level details  
   - **Activity Runs**: Provides row counts and processing times  
   - **Debug Mode**: Enables intermediate metrics and data lineage  
   - **Output Pane During Execution**: Displays activity status, row counts, and errors  
   - **Sink-Level Details**: Shows row counts and errors  
   - **Data Preview**: Uses debug session to load sample data  
   - **Log Analytics Integration**: Sends detailed logs when logging is set to **Basic** or **Verbose**

4. **Examine Detailed Logs**  
   - Open the **Monitor** tab in Azure Data Factory  
   - Navigate to **Pipeline Runs** > **Data Flow Debug Runs**  
   - Click on the specific activity to view:  
     - Error messages  
     - Stack traces  
     - Detailed execution steps  

---

#### Step 3: Iterate, Refine, and Optimize

1. **Adjust Configurations**  
   - Refine transformation logic, schemas, or sink settings based on error analysis  

2. **Re-run the Debug Session**  
   - Validate changes by executing the pipeline again  

3. **Reduce Logging Overhead**  
   - Switch to **Basic** or **None** logging for production to improve performance  

4. **Test with Real Data**  
   - Run the data flow with the full dataset and monitor execution metrics  

---

#### Step 3: Explore Additional Error Detection Methods

1. **Use Alert Rules**  
   - Navigate to **Monitor** > **Alerts & Metrics**  
   - Create alerts for data flow or pipeline failures  

2. **Enable Diagnostic Settings**  
   - Configure logs to be sent to:  
     - **Log Analytics** for querying  
     - **Blob Storage** for archival  
     - **Event Hubs** for third-party integration  

3. **Query Logs in Log Analytics**  
   - Use the following KQL for errors:  
     ```kql
     AzureDiagnostics
     | where ResourceProvider == "MICROSOFT.DATAFACTORY"
     | where Category == "DataFlowActivityRuns"
     | where Status == "Failed"
     | order by TimeGenerated desc
     ```

---

This structure collapses **Debug** into **Step 1**, moves **Review Debug Results** into **Step 2**, and cleanly separates stages while retaining all relevant content. Let me know if further refinements are needed!

---

### Monitor

#### Runs >> "Pipeline runs" and "Trigger runs"

* Navigate to Azure Data Factory >> Studio >> Monitor >> Pipeline Runs
* Review filter settings {e.g., date, name, status, etc.}
* Click into a specific log entry and review
* Repeat for Trigger Runs... note that a Trigger Run might have "Succeeded" even though the corresponding Pipeline Run failed

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
   
   * **Azure Diagnostics** - When you choose "Azure Diagnostics", **all your diagnostic logs from all resource types are sent to a single table called AzureDiagnostics**. This can make querying easier if you want to correlate events across different resource types, as you only need to query one table. However, the AzureDiagnostics table is a flat table, so complex properties are serialized into JSON strings which can make some data harder to work with.  
   * **Resource Specific** - When you choose "Resource Specific", your **logs are sent to a table that corresponds to the resource type**. For example, if you're working with a Storage Account, your logs might be sent to a table like StorageAccountLogs. These tables are designed to best fit the schema of the log data of the specific resource type, and complex properties are represented as distinct fields, which can make queries more intuitive and the data easier to work with. However, if you want to correlate events across different resource types, you would need to query multiple tables.  

Be sure to click "Save".

---

#### Logs

##### Sample #1: `AzureDiagnostics`

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

#### Alerts

* Navigate to Azure Data Factory >> Monitoring >> Alerts
* Click "Create" and select "Alert rule" from the resulting dropdown
* Complete the resulting "Create an alert rule" page, including:

   Label | Value
   :----- | :-----
   Signal name | Select and configure the desired Signal<br><sub>Signal descriptions included in Appendix</sub>

* Review additional tabs {e.g., Scope, Actions, Details, Tags}, then "Review + create"

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

### Diagnostic Setting >> Log Categories

| Category | Description |  
| --- | --- |  
| allLogs | Choose to send all types of logs {e.g., pipeline runs, trigger runs, activity runs, and more} |  
| Pipeline activity runs log | This log captures details about the execution of activities within a pipeline. |  
| Pipeline runs log | This log records information about each pipeline run, including its start time, end time, and status. |  
| Trigger runs log | This log captures information about trigger runs, which are instances where a trigger has fired to start a pipeline. |  
| Sandbox Pipeline runs log | This is a specialized log for capturing information about pipeline runs in a sandbox environment. |  
| Sandbox Activity runs log | This is a specialized log for capturing information about activity runs in a sandbox environment. |  
| SSIS package event messages | This log captures event messages from the execution of SQL Server Integration Services (SSIS) packages. |  
| SSIS package executable statistics | This log captures detailed statistical data about the execution of SSIS packages. |  
| SSIS package event message context | This log captures context information about event messages during the execution of SSIS packages. |  
| SSIS package execution component phases | This log records information about the different phases of execution for components within an SSIS package. |  
| SSIS package execution data statistics | This log captures data statistics from the execution of SSIS packages. |  
| SSIS integration runtime logs | This log records information about the runtime environment for SSIS packages. |  
| Airflow task execution logs | This log captures details about the execution of tasks within an Apache Airflow workflow. |  
| Airflow worker logs | This log records information about the operation of Apache Airflow workers. |  
| Airflow dag processing logs | This log captures information about the processing of Directed Acyclic Graphs (DAGs), which are the workflows defined in Apache Airflow. |  
| Airflow scheduler logs | This log records information about the operation of the Apache Airflow scheduler. |  
| Airflow web logs | This log captures information about the operation of the Apache Airflow web interface. | 

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
