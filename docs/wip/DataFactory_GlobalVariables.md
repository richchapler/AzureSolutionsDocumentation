# Azure Data Factory: "Global Variables"

While Azure Data Factory (ADF) doesn't support global variables in the traditional sense, you can achieve similar functionality using parameters and pipeline activities.  
   
## ...via Pipeline

### Step 1: Create `GlobalVariable_Parent` Pipeline
  
* Navigate to Data Factory Studio >> Author
* Create a pipeline and name it `GlobalVariable_Parent`
* Add a new variable named `globalVariable` of type Array with default value `["Initial Value"]`

### Step 2: Create `GlobalVariable_Child` Pipeline
  
* Create another pipeline and name it `GlobalVariable_Child`
* Add a new parameter named `passedValue` of type Array (with no default value)
* Add a new variable named `localVariable` of type Array (with no default value)
* Add a 'Set Variable' activity to the pipeline canvas with settings:
  * Name: `localVariable`
  * Value: `@pipeline().parameters.passedValue`
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
  * Parameters >> `passedValue` >> Value: `@variables('globalVariable')`

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
  
### Step 5: Pass the Changed Variable Value Back to the Parent Pipeline  
  
* In the `GlobalVariable_Child` pipeline, add an 'Output' activity to the pipeline canvas with settings:  
  * Value: `@variables('localVariable')`  
* In the `GlobalVariable_Parent` pipeline, add a new variable named `receivedValue` of type string (with no default value)  
* In the 'Execute Pipeline' activity, under 'Settings', add an output parameter:  
  * Name: `outputValue`  
  * Value: `@activity('Name of your Execute Pipeline activity').output.firstRow.localVariable`  
  
### Step 6: Confirm Success  
  
* Click on 'Debug' to run the pipeline  
* Once the pipeline run is complete, navigate to the 'Monitor' tab  
* Click on the pipeline run to view the details  
* Under 'Activity Runs', click on the 'Execute Pipeline' activity  
* In the 'Output' tab, you should see the changed value of the variable  
  
Please note that the 'Output' activity is not a built-in activity in Azure Data Factory. You can use a 'Set Variable' activity to set the value of a variable in the parent pipeline, and then use a 'Web' activity to send the value to an HTTP endpoint, or a 'Lookup' activity to retrieve the value from a database or other data source.  
