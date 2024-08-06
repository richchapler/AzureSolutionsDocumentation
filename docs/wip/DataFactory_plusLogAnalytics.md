# Data Factory plus Log Analytics

## Connect Data Factory to Log Analytics

1. Instantiate Data Factory and Log Analytics
   a. Note the Log Analytics Workspace ID and Primary Key
   
2. Navigate to Data Factory >> Monitoring >> Diagnostic Settings
   
3. Click on "+ Add diagnostic setting" and complete the resulting form:

   
   
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
