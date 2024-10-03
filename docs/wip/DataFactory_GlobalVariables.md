# Azure Data Factory: "Global Variables"

While Azure Data Factory (ADF) doesn't support global variables in the traditional sense, there are various work-arounds:

| Method | Comment | Feasibility |      
| :--- | :--- | :--- |      
| Pipeline / Data Flow | Unidirectional {i.e., parent >> child only} | 50% |      
| SQL Database | Additional latency / complexity | 100% |      
| Blob Storage | Additional latency / complexity | 100% |   
   
## ...via Pipeline

### Step 1: Create `GlobalVariable_Parent` Pipeline
  
* Navigate to Data Factory Studio >> Author
* Create a pipeline and name it `GlobalVariable_Parent`
* Add a new variable named `globalVariable` of type Array with default value `["Initial Value"]`

### Step 2: Create `GlobalVariable_Child` Pipeline
  
* Create another pipeline and name it `GlobalVariable_Child`
* Add a new parameter named `input` of type Array (with no default value)
* Add a new variable named `localVariable` of type Array (with no default value)
* Add a 'Set Variable' activity to the pipeline canvas with settings:
  * Name: `localVariable`
  * Value: `@pipeline().parameters.input`
* Add a 'Web' activity to the pipeline canvas, name it `Web_Original` and change default settings:
  * URL: `http://httpbin.org/post`
  * Method: POST
  * Body: `@variables('localVariable')`
* Create a success dependency from the 'Set Variable' activity to the 'Web' activity

### Step 3: `GlobalVariable_Parent` Pipeline + 'Execute Pipeline'

* Return to the `GlobalVariable_Parent` pipeline
* Add a 'Execute Pipeline' activity to the pipeline canvas with settings:
  * Invoked Pipeline: `GlobalVariable_Child`
  * Wait on Completion: checked
  * Parameters >> `input` >> Value: `@variables('globalVariable')`

### Step 4: `GlobalVariable_Child` Pipeline + `Append Variable`
  
* Return to the `GlobalVariable_Child` pipeline
* Add a 'Append Variable' activity to the pipeline canvas with settings:
  * Name: `localVariable`
  * Value: `New Value`
* Add a 'Web' activity to the pipeline canvas, name it `Web_Changed` and change default settings:
  * URL: `http://httpbin.org/post`
  * Method: POST
  * Body: `@variables('localVariable')`
* Create a success dependency from the 'Set Variable' activity to the 'Web' activity
  
### Step 5: `GlobalVariable_Parent` Pipeline >> `GlobalVariable_Child` Pipeline  
  
Unfortunately, Azure Data Factory does not natively support passing values from a child pipeline back to a parent pipeline using output parameters. The scope of variables and parameters is limited to the pipeline in which they are defined.

The common workaround for this limitation is to use an external service such as Azure Key Vault, Azure Blob Storage, or a database to temporarily store the values that need to be passed between pipelines, as I mentioned in the previous response.

Another possible workaround is to restructure your pipelines so that all the necessary data transformations are done within a single pipeline, thus avoiding the need to pass data between pipelines.

_Note: The same limitation applies to Data Flows. Data Flows do not support output parameters and cannot pass values back to the parent pipeline._

## ...via SQL Database  
   
### Step 1: Set Up a SQL Database  
   
* Navigate to SQL Database >> Query Editor
* Execute the following T-SQL query: `CREATE TABLE GlobalVariables ( Name NVARCHAR(64) PRIMARY KEY, Value NVARCHAR(MAX) );` 
   
### Step 2: Create `GlobalVariable` Pipeline  
   
* Navigate to Data Factory Studio >> Author 
* Create a pipeline and name it `GlobalVariable`
* Add a 'Lookup' activity to the pipeline canvas. Configure it to retrieve the value of the global variable from the SQL Database using a SQL script. For example, `SELECT Value FROM GlobalVariable WHERE Name = 'YourVariableName'`.  
* Add a 'Set Variable' activity to the pipeline canvas. Configure it to set the value of a pipeline variable to the value retrieved by the 'Lookup' activity.  
   
### Step 3: Update Global Variable in SQL Database  
   
* In the `GlobalVariable` pipeline, add a 'SQL Server Stored Procedure' activity to the pipeline canvas. Instead of calling a stored procedure, use a SQL script to update the value of the global variable in the SQL Database. For example, `UPDATE GlobalVariable SET Value = 'NewValue' WHERE Name = 'YourVariableName'`.  
   
### Step 4: `GlobalVariable` Pipeline + 'Lookup'  
   
* Return to the `GlobalVariable` pipeline.  
* Add another 'Lookup' activity to the pipeline canvas. Configure it to retrieve the updated value of the global variable from the SQL Database using a SQL script.  
   
With this setup, the `GlobalVariable` pipeline can retrieve the value of the global variable from the SQL Database, use it in the pipeline, update the value in the SQL Database, and then retrieve the updated value.
