# Data Factory plus Log Analytics

## Connect Data Factory to Log Analytics

1. Instantiate Data Factory and Log Analytics
   a. Note the Log Analytics Workspace ID and Primary Key
   
2. Navigate to Data Factory >> Monitoring >> Diagnostic Settings
   
3. Click on "+ Add diagnostic setting" and complete the resulting form:

   | Prompt | Entry |
   | ----- | ----- |
   | Category Groups | Check "**allLogs**" and "**allMetrics**"<br>...or those specific logs which make sense for your instance |
   | Destination Details | Check "**Send to Log Analytics workspace**" |

### Category Groups

* **allLogs** - Choose to send all types of logs {e.g., pipeline runs, trigger runs, activity runs, and more}

### Categories

* **Pipeline activity runs log** - This log captures details about the execution of activities within a pipeline. For instance, if a pipeline includes activities to extract data from a source, transform it, and load it into a destination, this log would capture details about the execution of each of these activities.  
   
* **Pipeline runs log** - This log records information about each pipeline run, including its start time, end time, and status. For example, if a pipeline is scheduled to run once a day, this log would contain a record for each daily run.  
   
* **Trigger runs log** - This log captures information about trigger runs, which are instances where a trigger has fired to start a pipeline. This could include details about the time the trigger fired and the pipeline that was started.  
   
* **Sandbox Pipeline runs log** - This is a specialized log for capturing information about pipeline runs in a sandbox environment. A sandbox environment is a separate instance of your data factory used for testing and development. This log can help you monitor and troubleshoot your pipelines in the sandbox environment.  
   
* **Sandbox Activity runs log** - This is a specialized log for capturing information about activity runs in a sandbox environment. It provides similar benefits to the Sandbox Pipeline runs log but focuses on individual activities within the pipelines.  

* SSIS

   * **SSIS package event messages** - This log captures event messages from the execution of SQL Server Integration Services (SSIS) packages. SSIS packages are a type of ETL tool used to extract, transform, and load data. The event messages could include information about the start and end of a package run, errors encountered, or other significant events during execution.  
      
   * **SSIS package executable statistics** - This log captures detailed statistical data about the execution of SSIS packages. These statistics could include the start time, end time, duration, success or failure status, the number of rows processed, and more. This can help identify performance bottlenecks and troubleshoot issues.  
      
   * **SSIS package event message context** - This log captures context information about event messages during the execution of SSIS packages. This could include details about the data source, destination, or transformations applied, helping you understand the context in which each event occurred.  
      
   * **SSIS package execution component phases** - This log records information about the different phases of execution for components within an SSIS package. For example, it might record when a data extraction phase started and ended, followed by the transformation phase, and so on.  
      
   * **SSIS package execution data statistics** - This log captures data statistics from the execution of SSIS packages. These could include the number of rows extracted, transformed, and loaded, helping you monitor the effectiveness of your SSIS packages.  
      
   * **SSIS integration runtime logs** - This log records information about the runtime environment for SSIS packages. This can be useful for troubleshooting issues related to the environment in which your SSIS packages are running.  

* Airflow

   * **Airflow task execution logs** - This log captures details about the execution of tasks within an Apache Airflow workflow. For example, it might record the start time, end time, and status of each task within a workflow.  
      
   * **Airflow worker logs** - This log records information about the operation of Apache Airflow workers. Workers in Airflow are the processes that execute tasks. The logs could include details about task execution, worker status, errors, and more, which can be useful for troubleshooting and understanding worker performance.  
      
   * **Airflow dag processing logs** - This log captures information about the processing of Directed Acyclic Graphs (DAGs), which are the workflows defined in Apache Airflow. It can include information about DAG parsing, scheduling decisions, and execution status, which can be useful for understanding how your workflows are performing and identifying any issues.  
      
   * **Airflow scheduler logs** - This log records information about the operation of the Apache Airflow scheduler. The scheduler in Airflow is responsible for deciding when to run workflows based on their schedules. Logs could include details about scheduling decisions, errors, and more, which can help in troubleshooting and understanding scheduler performance.  
      
   * **Airflow web logs** - This log captures information about the operation of the Apache Airflow web interface. This could include details about user interactions, errors, and more. This can be useful for understanding how users are interacting with the web interface and identifying any issues.  
   
### Metrics

* **AllMetrics** - This represents all the metrics available in Log Analytics. If you select this, you are choosing to send all types of metrics to Log Analytics. This could include metrics related to pipeline runs, activity runs, trigger runs, and more, giving you a comprehensive view of your system's performance.

6. In the "Diagnostic settings" form, fill in the name for your settings, and under "Destination details", choose "Send to Log Analytics".  
   
7. In the "Log Analytics workspace" dropdown, select the workspace you created earlier.  
   
8. Under "Log", select the types of logs you want to send to Log Analytics, such as ActivityRun, PipelineRun, etc.  
   
9. Click "Save".  
   
Now, your Azure Data Factory logs will be sent to Log Analytics. You can run queries on these logs in Log Analytics to generate reports.  
   
A great report to run might be one that shows the status of all pipeline runs over the last 7 days. Here's an example of such a query:  
   
```KQL  
AzureDiagnostics  
| where ResourceProvider == "MICROSOFT.DATAPROCESSING" and Category == "PipelineRuns"  
| where TimeGenerated > ago(7d)  
| project TimeGenerated, Resource, operationName, status_s, Duration_d  
| order by TimeGenerated desc  
```  
   
This query filters logs to only show pipeline run logs from the last 7 days. It then projects the time the log was generated, the resource involved, the operation name, the status of the pipeline run, and its duration. The results are ordered by the time they were generated in descending order.  
   
This report can be useful for getting a quick overview of your pipeline runs, identifying any failures, and understanding how long your pipelines are taking to run.
