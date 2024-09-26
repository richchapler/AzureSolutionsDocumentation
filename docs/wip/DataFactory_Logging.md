# Data Factory:Logging

## Data Factory Studio

### Pipeline Runs

**1. Open Data Factory Studio**
   - Open your browser and navigate to the Azure portal.
   - Go to your Azure Data Factory resource.
   - Click on **Author & Monitor** to open the ADF Studio.

**2. Navigate to the Monitor Tab**
   - Once in ADF Studio, find the menu on the left-hand side.
   - Click on the **Monitor** tab to enter the monitoring interface.

**3. View Pipeline Runs**
   - In the Monitor tab, you'll see options for **Pipeline Runs**, **Trigger Runs**, **Integration Runtimes**, and more.
   - Select **Pipeline Runs** to see a list of all pipeline executions.

**4. Filter and Search Pipeline Runs**
   - Use the search bar or filter options to find specific pipeline runs based on criteria such as status, date range, or pipeline name.
   - This helps you quickly identify runs that need attention.

**5. Check Pipeline Run Status**
   - Each pipeline run will have a status, such as **In Progress**, **Succeeded**, **Failed**, or **Cancelled**.
   - Click on a pipeline run to view detailed information about its activities and their statuses.

**6. Investigate Pipeline Run Details**
   - Within the selected pipeline run, you can see a breakdown of each activity, its duration, and any errors encountered.
   - This detailed view helps you pinpoint where issues may have occurred.

**7. Take Action Based on Run Status**
   - If a pipeline run failed, investigate the error messages to understand what went wrong.
   - Use this information to debug and fix issues in your pipeline definition or data.

**8. Set Up Alerts for Pipeline Runs**
   - Go to the **Alerts and Metrics** section within the Monitor tab.
   - Create alerts to notify you via email, SMS, or other means when a pipeline run meets certain conditions, such as failure or success.

* Studio > Monitor: Pipeline Runs, Trigger Runs, Alerts & Metrics
* Portal > Monitoring: Diagnostic Settings, Alerts, Metrics, Logs
* Log Analytics: Sample queries {e.g., Pipeline Success Rate, Pipeline Duration, Activity Errors}
* Custom Logging: Pipeline / Activity failures and error handling

## Log Analytics: Configure

1. Instantiate Data Factory and Log Analytics
   
2. Navigate to Data Factory >> Monitoring >> Diagnostic Settings
   
3. Click on "+ Add diagnostic setting" and complete the resulting form:

   | Prompt | Entry |
   | ----- | ----- |
   | Logs | Check "**allLogs**" and "**allMetrics**"<br>...or those specific logs which make sense for your instance |
   | Destination Details | Check "**Send to Log Analytics workspace**" and then configure connection (including "Destination table") |

   ### Logs

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
   
   ### Destination Details
   
   * **Azure Diagnostics** - When you choose "Azure Diagnostics", **all your diagnostic logs from all resource types are sent to a single table called AzureDiagnostics**. This can make querying easier if you want to correlate events across different resource types, as you only need to query one table. However, the AzureDiagnostics table is a flat table, so complex properties are serialized into JSON strings which can make some data harder to work with.  
   * **Resource Specific** - When you choose "Resource Specific", your **logs are sent to a table that corresponds to the resource type**. For example, if you're working with a Storage Account, your logs might be sent to a table like StorageAccountLogs. These tables are designed to best fit the schema of the log data of the specific resource type, and complex properties are represented as distinct fields, which can make queries more intuitive and the data easier to work with. However, if you want to correlate events across different resource types, you would need to query multiple tables.  
   
4. Click "Save".  
   
Now, your Azure Data Factory logs will be sent to Log Analytics. You can run queries on these logs in Log Analytics to generate reports.  

## Sample #1: `AzureDiagnostics`

1. Navigate to Data Factory >> Monitoring >> Logs
  
2. Close the "Welcome to Log Analytics" or "Queries hub" popups  
   
3. Click the "Simple mode" dropdown and select "KQL mode"
   
4. In the new query window, paste the very simple starter query:  
   
```kql  
AzureDiagnostics   
```  
   
5. Click "Run" to execute the query.  

## Sample #2: PipelineRuns

Repeat with the following KQL

```kql  
AzureDiagnostics    
| where ResourceProvider == "MICROSOFT.DATAFACTORY" and Category == "PipelineRuns"    
| where TimeGenerated > ago(1d)    
| order by TimeGenerated desc       
```  
