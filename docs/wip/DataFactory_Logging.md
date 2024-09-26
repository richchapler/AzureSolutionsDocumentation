# Data Factory: Logging Options

## Studio >> Monitor: Pipeline and Trigger Runs

* Navigate to Azure Data Factory >> Studio >> Monitor >> Pipeline Runs
* Review filter settings {e.g., date, name, status, etc.}
* Click into a specific log entry and review
* Repeat for Trigger Runs... note that a Trigger Run might have "Succeeded" even though the corresponding Pipeline Run failed

## Studio >> Monitor: Alerts & Metrics
Create alerts to notify you via email, SMS, or other means when a pipeline run meets certain conditions, such as failure or success.

 * Navigate to Azure Data Factory >> Studio >> Monitor >> Alerts & Metrics
 * Click "New alert rule" and complete the resulting pop-out form, including:

   Label | Value
   :----- | :-----
   Severity | Sev0 is most severe... Sev5 is least severe
   Add criteria | Click then select the desired Metric on the resulting pop-out

### Metric Descriptions

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
   




**5. Define the Alert Condition**
   - In the **Condition** section, specify the criteria for triggering the alert.
   - Choose the signal type (e.g., Pipeline Runs, Trigger Runs, etc.).
   - Set the condition logic, such as "greater than," "less than," or "equal to" a certain value.

**6. Configure the Alert Details**
   - Provide a name and description for your alert rule.
   - Set the severity level of the alert (e.g., Sev 0 for critical, Sev 4 for informational).
   - Define the frequency of evaluations to determine how often the alert conditions should be checked.

**7. Set Up Action Groups**
   - Action groups determine who gets notified and how when an alert is triggered.
   - Click on **+ New Action Group** to create a new action group.
   - Specify the notification methods (e.g., email, SMS, push notifications).
   - Add the recipients who should be notified when the alert is triggered.

**8. Review and Create the Alert Rule**
   - Review all the configured settings for your alert rule.
   - Click on **Create Alert Rule** to save and activate the alert.

**9. Monitor Metrics and Alerts**
   - Back in the **Monitor** tab, you can now view and track the metrics related to your pipelines and triggers.
   - Use the provided charts and graphs to analyze performance over time.
   - Check the **Alert History** to see any triggered alerts and their statuses.

**10. Respond to Alerts**
   - When an alert is triggered, review the details and take necessary actions to address the issue.
   - Use the information provided by the alert to troubleshoot and resolve any problems in your data workflows.


## Portal > Monitoring: Diagnostic Settings, Alerts, Metrics, Logs

## Log Analytics: Configure
Sample queries {e.g., Pipeline Success Rate, Pipeline Duration, Activity Errors}

1. Instantiate Data Factory and Log Analytics
   
2. Navigate to Data Factory >> Monitoring >> Diagnostic Settings
   
3. Click on "+ Add diagnostic setting" and complete the resulting form:

   | Prompt | Entry |
   | ----- | ----- |
   | Logs | Check "**allLogs**" and "**allMetrics**"<br>...or those specific logs which make sense for your instance |
   | Destination Details | Check "**Send to Log Analytics workspace**" and then configure connection (including "Destination table") |

   #### Logs

   * **Category Groups**
      * **allLogs** - Choose to send all types of logs {e.g., pipeline runs, trigger runs, activity runs, and more}

   * **Categories**
      * **Pipeline activity runs log** - This log captures details about the execution of activities within a pipeline. For instance, if a pipeline includes activities to extract data from a source, transform it, and load it into a destination, this log would capture details about the execution of each of these activities.  
      * **Pipeline runs log** - This log records information about each pipeline run, including its start time, end time, and status. For example, if a pipeline is scheduled to run once a day, this log would contain a record for each daily run.  
      * **Trigger runs log** - This log captures information about trigger runs, which are instances where a trigger has fired to start a pipeline. This could include details about the time the trigger fired and the pipeline that was started.  
      * **Sandbox Pipeline runs log** - This is a specialized log for capturing information about pipeline runs in a sandbox environment. A sandbox environment is a separate instance of your data factory used for testing and development. This log can help you monitor and troubleshoot your pipelines in the sandbox environment.  
      * **Sandbox Activity runs log** - This is a specialized log for capturing information about activity runs in a sandbox environment. It provides similar benefits to the Sandbox Pipeline runs log but focuses on individual activities within the pipelines.
 
      * **SSIS package event messages** - This log captures event messages from the execution of SQL Server Integration Services (SSIS) packages. SSIS packages are a type of ETL tool used to extract, transform, and load data. The event messages could include information about the start and end of a package run, errors encountered, or other significant events during execution.  
      * **SSIS package executable statistics** - This log captures detailed statistical data about the execution of SSIS packages. These statistics could include the start time, end time, duration, success or failure status, the number of rows processed, and more. This can help identify performance bottlenecks and troubleshoot issues.  
      * **SSIS package event message context** - This log captures context information about event messages during the execution of SSIS packages. This could include details about the data source, destination, or transformations applied, helping you understand the context in which each event occurred.  
      * **SSIS package execution component phases** - This log records information about the different phases of execution for components within an SSIS package. For example, it might record when a data extraction phase started and ended, followed by the transformation phase, and so on.  
      * **SSIS package execution data statistics** - This log captures data statistics from the execution of SSIS packages. These could include the number of rows extracted, transformed, and loaded, helping you monitor the effectiveness of your SSIS packages.  
      * **SSIS integration runtime logs** - This log records information about the runtime environment for SSIS packages. This can be useful for troubleshooting issues related to the environment in which your SSIS packages are running.

      * **Airflow task execution logs** - This log captures details about the execution of tasks within an Apache Airflow workflow. For example, it might record the start time, end time, and status of each task within a workflow.  
      * **Airflow worker logs** - This log records information about the operation of Apache Airflow workers. Workers in Airflow are the processes that execute tasks. The logs could include details about task execution, worker status, errors, and more, which can be useful for troubleshooting and understanding worker performance.  
      * **Airflow dag processing logs** - This log captures information about the processing of Directed Acyclic Graphs (DAGs), which are the workflows defined in Apache Airflow. It can include information about DAG parsing, scheduling decisions, and execution status, which can be useful for understanding how your workflows are performing and identifying any issues.
      * **Airflow scheduler logs** - This log records information about the operation of the Apache Airflow scheduler. The scheduler in Airflow is responsible for deciding when to run workflows based on their schedules. Logs could include details about scheduling decisions, errors, and more, which can help in troubleshooting and understanding scheduler performance.  
      * **Airflow web logs** - This log captures information about the operation of the Apache Airflow web interface. This could include details about user interactions, errors, and more. This can be useful for understanding how users are interacting with the web interface and identifying any issues.

   * **Metrics**   
      * **AllMetrics** - This represents all the metrics available in Log Analytics. If you select this, you are choosing to send all types of metrics to Log Analytics. This could include metrics related to pipeline runs, activity runs, trigger runs, and more, giving you a comprehensive view of your system's performance.
   
   #### Destination Details
   
   * **Azure Diagnostics** - When you choose "Azure Diagnostics", **all your diagnostic logs from all resource types are sent to a single table called AzureDiagnostics**. This can make querying easier if you want to correlate events across different resource types, as you only need to query one table. However, the AzureDiagnostics table is a flat table, so complex properties are serialized into JSON strings which can make some data harder to work with.  
   * **Resource Specific** - When you choose "Resource Specific", your **logs are sent to a table that corresponds to the resource type**. For example, if you're working with a Storage Account, your logs might be sent to a table like StorageAccountLogs. These tables are designed to best fit the schema of the log data of the specific resource type, and complex properties are represented as distinct fields, which can make queries more intuitive and the data easier to work with. However, if you want to correlate events across different resource types, you would need to query multiple tables.  
   
4. Click "Save".  
   
Now, your Azure Data Factory logs will be sent to Log Analytics. You can run queries on these logs in Log Analytics to generate reports.  

### Sample #1: `AzureDiagnostics`

1. Navigate to Data Factory >> Monitoring >> Logs
  
2. Close the "Welcome to Log Analytics" or "Queries hub" popups  
   
3. Click the "Simple mode" dropdown and select "KQL mode"
   
4. In the new query window, paste the very simple starter query:  
   
```kql  
AzureDiagnostics   
```  
   
5. Click "Run" to execute the query.  

### Sample #2: PipelineRuns

Repeat with the following KQL

```kql  
AzureDiagnostics    
| where ResourceProvider == "MICROSOFT.DATAFACTORY" and Category == "PipelineRuns"    
| where TimeGenerated > ago(1d)    
| order by TimeGenerated desc       
```  
## Custom Logging: Pipeline / Activity failures and error handling
