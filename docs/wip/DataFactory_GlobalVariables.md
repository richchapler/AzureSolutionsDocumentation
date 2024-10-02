# Azure Data Factory: "Global Variables"

While Azure Data Factory (ADF) doesn't support global variables in the traditional sense, you can achieve similar functionality using parameters and pipeline activities. Here's a simple demonstration:  
   
## Step 1: Create `GlobalVariable` Pipeline
This pipeline will act as the main orchestrator. It will set the initial values of the variables and call the other pipelines.  
  
* Navigate to Data Factory Studio >> Author
* Click on '+', then 'Pipeline' in the resulting dropdown
* In the 'Properties' pop-out, change default name to `GlobalVariable`
* Create new variable `theVariable` of type string (with no default value)
* Drag-and-drop a 'Set Variable' activity onto the pipeline canvas
  * Configure 'Settings' to set `theVariable` to `Initial Value`
   
## Step 2: Create `GlobalVariable_Child` Pipeline
This pipeline will receive the variable value, use it, and update it.  
  
* Create a pipeline and name it `GlobalVariable_Child`
* Drag-and-drop a 'Set variable' activity onto the pipeline canvas
* Configure 'Settings' to set `theVariable` to `Changed Value`

## Step 3: Enhance `GlobalVariable` Pipeline

* Return to the `GlobalVariable` Pipeline
* Drag-and-drop an 'Execute Pipeline' activity onto the pipeline canvas
  * Configure 'Settings' to set `Invoked pipeline` to `GlobalVariable_Child`
* Add an 'Execute pipeline' activity to call the other pipeline. Set the 'Pipeline name' to the name of your other pipeline and pass the variable as a parameter.

   
3. **Create a Data Flow:**  
  
   This data flow will also receive the variable value and use it.  
  
   - Create a new data flow and name it 'MyDataFlow'.  
   - In the data flow's parameters, create a new parameter. Name it 'MyParameter'.  
   - Use the parameter in your transformations. For example, you can use the expression 'iif({MyParameter} > 10, 'High', 'Low')' to categorize your data based on the variable value.  
   
4. **Update the Master Pipeline:**  
  
   - Add another 'Execute pipeline' activity to call the data flow. Set the 'Pipeline name' to 'MyDataFlow' and pass the variable as a parameter.  
   - Add a 'Get variable' activity to retrieve the updated variable value. Set the 'Name' to 'MyVariable'.  
   
Now, when you run the MasterPipeline, it will initialize the variable, pass it to the ChildPipeline and the data flow, and retrieve the updated value. This is a simple demonstration, but you can extend it to fit your needs.
