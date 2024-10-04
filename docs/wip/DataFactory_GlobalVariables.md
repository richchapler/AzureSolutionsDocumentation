# Azure Data Factory: "Global Variables"

While Azure Data Factory (ADF) doesn't support global variables in the traditional sense, there are various work-arounds:

| Method | Comment | Feasibility | Documented? |      
| :--- | :--- | :--- | :--- |     
| Data Factory, Pipeline | Unidirectional {i.e., parent >> child only} | 50% | Yes |      
| Data Factory, Data Flow | Might be possible using "Set Variable" > Variable Type: "Pipeline Return Value" | TBD | In Progress |     
| SQL Database | Additional latency / complexity | 100% | Yes |
| Blob Storage | Additional latency / complexity | 100% | Not Yet |

-----

## ...via Pipelines

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
* Add an 'Append Variable' activity to the pipeline canvas with settings:
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

-----

## ...via Data Flows

### Step 1: Create a Data Flow
* Navigate to Data Factory Studio >> Author.
* Create a new **Data Flow** and name it `GlobalVariable_DataFlow`.
* Add a **Source** transformation to read or generate data that will represent your variable value (e.g., from a file, database, or a computed value).
* Add a **Derived Column** transformation (optional) if you need to manipulate or transform the data.

### Step 2: Set Variable in Data Flow
* After your transformations, add a **Sink** transformation if needed to persist data (e.g., in a database or storage).
* In the **Settings** tab of the Data Flow, set a **Variable** with the value you want to pass back to the parent pipeline using the **"Set Variable"** activity with a **Pipeline Return Value**.

### Step 3: Execute Data Flow from Parent Pipeline
* In your parent pipeline, use a **Data Flow activity** to execute the `GlobalVariable_DataFlow`.
* Capture the output of the Data Flow using the **"Pipeline Return Value"** variable type and pass it back to a variable or other activities in the parent pipeline.

-----

## ...via SQL Database  
   
### Step 1: Set Up a SQL Database  
   
* Navigate to SQL Database >> Query Editor
* Execute the following T-SQL query: `CREATE TABLE GlobalVariables ( Name NVARCHAR(64) PRIMARY KEY, Value NVARCHAR(MAX) );` 
   
### Step 2: Create `GlobalVariable_SQL` Pipeline  
   
* Navigate to Data Factory Studio >> Author 
* Create a pipeline and name it `GlobalVariable_SQL`
* First we want to confirm that the variable exists in the SQL table
   * Add a 'Lookup' activity named `Count` to the pipeline canvas with settings:
     * Source Dataset: `...globalvariables`
     * First Row Only: checked
     * Use Query: 'Query' with logic: `SELECT COUNT(*) AS RecordCount FROM [GlobalVariables] WITH (NOLOCK) WHERE [Name] = 'X'`
   * Add an 'If Condition' activity named `Confirm Data Availability` to the pipeline canvas with settings:
     * Expression: `@equals(activity('Count').output.firstRow.RecordCount, 0)`
     * Click the pencil icon on the 'True' case
       * Add a 'Script' activity named 'Insert Variable' to the 'True activities' pipeline canvas with settings:
         * Linked Service: `...sqldatabase`
         * Script: `NonQuery` with logic: `INSERT INTO GlobalVariables ([Name], [Value]) VALUES ('X', NULL)`
   * Return to the main pipeline canvas
   * Create success dependency from the 'Count' lookup activity to the 'If Condition' activity
* Next, we'll capture the value of variable
   * Add a 'Lookup' activity named `Value` to the pipeline canvas with settings:
     * Source Dataset: `...globalvariables`
     * First Row Only: checked
     * Use Query: 'Query' with logic: `SELECT [Value] FROM [GlobalVariables] WHERE [Name] = 'X'`
   * Create success dependency from the 'If Condition' activity to the 'Value' lookup activity
   * Add a new variable named `X` of type String with no default value
   * Add a 'Set Variable' activity to the pipeline canvas with settings:
     * Name: `X`
     * Value: `@activity('Lookup').output.firstRow.Value`
   * Create success dependency from the 'Value' lookup activity to the 'Set Variable' activity
* Finally, we'll write a value back to SQL
   * Add a 'Script' activity named `Update Variable` to the pipeline canvas with settings:
      * Linked Service: `...sqldatabase`
      * Script: `NonQuery` with logic: `UPDATE GlobalVariables SET Value = 'NewValue' WHERE Name = 'X'`  
   * Create success dependency from the 'Set Variable' activity to the 'Update Variable' activity
