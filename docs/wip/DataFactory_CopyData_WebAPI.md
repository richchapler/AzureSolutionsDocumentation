# Data Factory: Create a Pipeline to Copy Data from Web API to SQL Table  
  
## Create Azure SQL Database  
Create an Azure SQL Database to store the data. Note down the server name, database name, username, and password.  
  
## Create a Data Factory  
Create a new Data Factory in the Azure portal. Start the Data Factory UI by clicking on the "Author & Monitor" tile in your new data factory's dashboard.  
  
## Set up Linked Services  
Create two linked services: one for the OpenWeatherMap API and one for the Azure SQL Database.  
  
### OpenWeatherMap API Linked Service  
* Navigate to Azure Data Factory >> Author & Monitor >> Connections >> Linked services >> New  
* Choose 'HTTP' as the data store type  
* Provide a name for the linked service  
* In the base URL, enter `http://api.openweathermap.org/data/2.5/`  
* Test the connection and create the linked service  
  
### Azure SQL Database Linked Service  
* Click on 'New' again under 'Linked services'  
* Choose 'Azure SQL Database' as the data store type  
* Provide the necessary details (server name, database name, username, password)  
* Create the linked service  
  
## Create Datasets  
Create two datasets: one for the source (OpenWeatherMap API) and one for the sink (Azure SQL Database).  
  
### OpenWeatherMap API Dataset  
* Navigate to Azure Data Factory >> Author & Monitor >> Author >> + Create pipeline >> Add data flow >> Add source  
* Choose the HTTP linked service you created earlier  
* In the relative URL, enter `weather?q={city name}&appid={your api key}`  
* Set the request method as 'GET'  
  
### Azure SQL Database Dataset  
* Click on 'Add destination'  
* Choose the Azure SQL Database linked service  
* Choose the table where you want to store the data  
  
## Create a Pipeline  
Create a new pipeline and add a 'Copy Data' activity to the pipeline.  
  
* In the source tab of the copy activity, select the dataset for the OpenWeatherMap API  
* In the sink tab, select the dataset for the Azure SQL Database  
  
## Mapping  
In the mapping tab of the copy activity, map the source fields to the destination fields.  
  
## Debugging and Publishing  
* Click on 'Debug' to test the pipeline  
* If it runs successfully, publish the pipeline  
  
> Note: Replace `{city name}` and `{your api key}` with an actual city name and your OpenWeatherMap API key respectively.  
