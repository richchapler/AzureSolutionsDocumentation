# Data Factory: HTTP Connectors for GET and POST

## GET: Fetch Data from Web API

- Create an Azure SQL Database to store the data.
  - Note down the following details:
    - Server name
    - Database name
    - Username
    - Password
- Create a Data Factory in the Azure portal.
  - In your Data Factory dashboard, click on the "Author & Monitor" tile to launch the Data Factory UI.

### Set up Linked Services  
You'll need to create two linked services: one for the JSONPlaceholder API (source) and one for the Azure SQL Database (sink).

#### JSONPlaceholder API Linked Service  
- Navigate to **Azure Data Factory** > **Author & Monitor** > **Connections** > **Linked Services** > **New**.  
- Choose **HTTP** as the data store type.  
- Provide a name for the linked service (e.g., `HttpLinkedService`).  
- In the **URL** field, enter:  
  `https://jsonplaceholder.typicode.com`  
- Click **Test connection** and verify the connection is successful.  
- Once confirmed, click **Create** to save the linked service.

#### Azure SQL Database Linked Service  
- Click **New** again under **Linked Services**.  
- Choose **Azure SQL Database** as the data store type.  
- Provide the necessary details:
  - Server name
  - Database name
  - Username
  - Password  
- Click **Create** to save the linked service.

### Create Datasets  
Create two datasets: one for the source (JSONPlaceholder API) and one for the sink (Azure SQL Database).

#### JSONPlaceholder API Dataset  
- Navigate to **Azure Data Factory** > **Author & Monitor** > **Author** > **+ Create pipeline**.  
- Click **Add data flow** > **Add source**.  
- Choose the HTTP linked service you created earlier (e.g., `HttpLinkedService`).  
- In the **Relative URL** field, enter `posts` (to retrieve the post data).  
- Set the **Format** to **JSON**.  

#### Azure SQL Database Dataset  
- Click **Add destination**.  
- Choose the Azure SQL Database linked service.  
- Choose the table where you want to store the data.

### Create a Pipeline  
- Create a new pipeline in the **Author** section of Azure Data Factory.  
- Add a **Copy Data** activity to the pipeline.  
- In the **Source** tab of the **Copy Data** activity, select the dataset for the JSONPlaceholder API.  
- In the **Sink** tab, select the dataset for the Azure SQL Database.

### Mapping  
- In the **Mapping** tab of the **Copy Data** activity, map the source fields (from the JSONPlaceholder API) to the destination fields (in the Azure SQL table).

---

## POST: Interact with Web API Using POST Method

### Using POST Method in Azure Data Factory

We will use the **Reqres API**, which is a free API for testing and doesn't require an API key. Specifically, you can use the "Create User" endpoint.

- Navigate to **Azure Data Factory** > **Author & Monitor** > **Author** > **Pipelines** > **New**.  
- Add a **Web** activity to the pipeline.  
- In the **URL** field, enter the following Reqres API endpoint:  
  `https://reqres.in/api/users`  
- In the **Method** field, select **POST**.  
- In the **Body** field, enter the JSON payload for the POST request. For example:  
  ```json
  {
    "name": "John Doe",
    "job": "Software Developer"
  }
