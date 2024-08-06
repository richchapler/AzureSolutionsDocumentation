# Data Factory plus Log Analytics

## Configure

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
      * **SSIS**
         * **SSIS package event messages** - This log captures event messages from the execution of SQL Server Integration Services (SSIS) packages. SSIS packages are a type of ETL tool used to extract, transform, and load data. The event messages could include information about the start and end of a package run, errors encountered, or other significant events during execution.  
         * **SSIS package executable statistics** - This log captures detailed statistical data about the execution of SSIS packages. These statistics could include the start time, end time, duration, success or failure status, the number of rows processed, and more. This can help identify performance bottlenecks and troubleshoot issues.  
         * **SSIS package event message context** - This log captures context information about event messages during the execution of SSIS packages. This could include details about the data source, destination, or transformations applied, helping you understand the context in which each event occurred.  
         * **SSIS package execution component phases** - This log records information about the different phases of execution for components within an SSIS package. For example, it might record when a data extraction phase started and ended, followed by the transformation phase, and so on.  
         * **SSIS package execution data statistics** - This log captures data statistics from the execution of SSIS packages. These could include the number of rows extracted, transformed, and loaded, helping you monitor the effectiveness of your SSIS packages.  
         * **SSIS integration runtime logs** - This log records information about the runtime environment for SSIS packages. This can be useful for troubleshooting issues related to the environment in which your SSIS packages are running.  
      * **Airflow**
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

## Sample

To run the report, you need to use Azure Monitor's Log Analytics. Here is a step-by-step guide:  
   
1. Sign in to the Azure portal.  
   
2. In the left-hand menu, click on "Monitor" or search for "Monitor" in the search box.  
   
3. In the Monitor blade, under the "Insights" section, click on "Logs".  
   
4. In the new query window, paste the query:  
   
```kql  
AzureDiagnostics    
| where ResourceProvider == "MICROSOFT.DATAPROCESSING" and Category == "PipelineRuns"    
| where TimeGenerated > ago(7d)    
| project TimeGenerated, Resource, operationName, status_s, Duration_d    
| order by TimeGenerated desc    
```  
   
5. Click "Run" to execute the query.  
