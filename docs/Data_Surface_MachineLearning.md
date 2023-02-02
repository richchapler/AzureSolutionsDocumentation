# Surface Data to Azure Machine Learning

![image](https://user-images.githubusercontent.com/44923999/215880260-d581ec97-3b29-4490-94ae-0f473fcc3612.png)

## Use Case
This solution considers the following requirements:

* "We want to surface data from Data Explorer in Machine Learning"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster](Infrastructure_DataExplorer_Cluster.md), [Database](Infrastructure_DataExplorer_Database.md), and [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)
* [**Machine Learning**](https://learn.microsoft.com/en-us/azure/machine-learning/) with a [Compute Instance](https://learn.microsoft.com/en-us/azure/machine-learning/concept-compute-instance)

-----

Steps
* Use Azure ML Command Line to install "azure-kusto-data"

![image](https://user-images.githubusercontent.com/44923999/216428803-786b082a-43a6-4fdc-b423-d3f7ae3fc835.png)

### Command Line
```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
```

```
sudo apt  install pypy  # version 7.3.1+dfsg-2
```

```
pypy -m pip install azure-kusto-data azure-kusto-ingest
```
NOT CLEAR IF THE ONES ABOVE IS NECESSARY!!

### Command #1
```
pip install azure-kusto-data
```

### Command #2
```
pip install azure-kusto-ingest
```

### Command #3
```
from azure.kusto.data import KustoClient, KustoConnectionStringBuilder, ClientRequestProperties
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
```

### Command #4
```
kcsb = KustoConnectionStringBuilder.with_aad_application_key_authentication("https://rchaplerdec.westus3.kusto.windows.net","75afc8e9-f297-4ba4-8b5b-5ce3495258a1",".8q8Q~5MYV2sBVLwhaBAOWBJranmpErUJ9gHYaDP","16b3c013-d300-468d-ac64-7eda0820b6d3")
kcsb.authority_id = "16b3c013-d300-468d-ac64-7eda0820b6d3"
```

### Command #4
```
client = KustoClient(kcsb)
response = client.execute("rchaplerded", "StormEvents1 | count")
df = dataframe_from_result_table(response.primary_results[0])
df.head(3)
```



https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py


Here's a code snippet in Python that demonstrates how to connect to Kusto using the kqlmagic library in a Jupyter Notebook:

python
Copy code
!pip install kqlmagic

%load_ext Kqlmagic

# Connect to Kusto cluster and database
%kql kusto://code.kusto.windows.net/MyDatabase

# Run a KQL query
%kql query_string = "StormEvents | count"
In the above code:

The pip install kqlmagic command installs the kqlmagic library.
The %load_ext Kqlmagic command loads the kqlmagic extension into the Jupyter Notebook.
The %kql kusto://code.kusto.windows.net/MyDatabase command connects to the specified Kusto cluster and database.
The %kql query_string = "StormEvents | count" command runs a sample KQL query to count the number of rows in the StormEvents table.



Regenerate response




* Connect ADX


To connect to an Azure Data Explorer (Kusto) cluster from an Azure ML Notebook, you will need to perform the following steps:
1.	Install the azure-kusto-data package in the Azure ML Notebook environment using the command !pip install azure-kusto-data.
2.	Import the necessary modules using the command from azure.kusto.data import KustoClient, KustoConnectionStringBuilder.
3.	Build the connection string using the KustoConnectionStringBuilder class. You will need the following information to build the connection string:
•       The cluster name
•       The database name
•       The client ID and client secret or a managed identity for the Azure ML Notebook.
4.	Create a Kusto client instance using the KustoClient class and the connection string.
5.	Use the Kusto client instance to query the database by calling the execute method.
 
For the permissions, you will need the Data Reader role at the database level, or the Data Contributor role if you want to write data as well. You can assign these roles using the Azure portal, the Azure CLI, or Azure Resource Manager templates.
 



![image](https://user-images.githubusercontent.com/44923999/216399872-eb7c206d-c656-491d-a9d6-1f7a6ba3d70b.png)

```
# Step 1: Install the package
!pip install azure-kusto-data
 
# Step 2: Import necessary modules
from azure.kusto.data import KustoClient, KustoConnectionStringBuilder
 
# Step 3: Build the connection string
connection_string = KustoConnectionStringBuilder.with_aad_device_authentication(https://rchaplerdec.kusto.windows.net).with_database("rchaplerded").build()
 
# Step 4: Create the Kusto client
client = KustoClient(connection_string)
 
# Step 5: Query data
query = "StormEvents | count"
results = client.execute("rchaplerded", query)
 
# Step 6: Print the results
print(results.primary_results[0])
```

## Exercise 1: Lorem Ipsum
In this exercise, we will lorem ipsum

### Step 1: Lorem Ipsum

<img src="https://user-images.githubusercontent.com/44923999/blah.png" width="800" title="Snipped: February 1, 2023" />
