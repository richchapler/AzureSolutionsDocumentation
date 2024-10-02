# Azure Data Factory: "Global Variables"

While Azure Data Factory (ADF) doesn't support global variables in the traditional sense, you can achieve similar functionality using parameters and pipeline activities.

Here's a simple demonstration:  
   
## Step 1: Create a Master Pipeline:  
  
   This pipeline will act as the main orchestrator. It will set the initial values of the variables and call the other pipelines.  
  
   - Go to the ADF authoring UI and click on '+', then 'Pipeline'.  
   - Name it 'MasterPipeline'.  
   - Add a 'Set variable' activity to initialize your variable. Name the variable 'MyVariable' and set its value to whatever you want.  
   - Add an 'Execute pipeline' activity to call the other pipeline. Set the 'Pipeline name' to the name of your other pipeline and pass the variable as a parameter.  
   
2. **Create a Child Pipeline:**  
  
   This pipeline will receive the variable value, use it, and update it.  
  
   - Create a new pipeline and name it 'ChildPipeline'.  
   - In the pipeline's parameters, create a new parameter. Name it 'MyParameter'.  
   - Add a 'Set variable' activity to update the variable value. Use the expression '@add(pipeline().parameters.MyParameter, 1)' to increment the value by 1.  
   - Add a 'Web activity' to return the updated value. Set the 'URL' to 'http://httpbin.org/post', the 'Method' to 'POST', and the 'Body' to '@variables('MyVariable')'.  
   
3. **Create a Data Flow:**  
  
   This data flow will also receive the variable value and use it.  
  
   - Create a new data flow and name it 'MyDataFlow'.  
   - In the data flow's parameters, create a new parameter. Name it 'MyParameter'.  
   - Use the parameter in your transformations. For example, you can use the expression 'iif({MyParameter} > 10, 'High', 'Low')' to categorize your data based on the variable value.  
   
4. **Update the Master Pipeline:**  
  
   - Add another 'Execute pipeline' activity to call the data flow. Set the 'Pipeline name' to 'MyDataFlow' and pass the variable as a parameter.  
   - Add a 'Get variable' activity to retrieve the updated variable value. Set the 'Name' to 'MyVariable'.  
   
Now, when you run the MasterPipeline, it will initialize the variable, pass it to the ChildPipeline and the data flow, and retrieve the updated value. This is a simple demonstration, but you can extend it to fit your needs.
