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

### Command #5
```
client = KustoClient(kcsb)
response = client.execute("rchaplerded", "StormEvents1 | count")
```

### Command #6
```
for row in response.primary_results[0]: print("EventType:{}".format(row["EventType"]))
```

https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py

![image](https://user-images.githubusercontent.com/44923999/216399872-eb7c206d-c656-491d-a9d6-1f7a6ba3d70b.png)


## Exercise 1: Lorem Ipsum
In this exercise, we will lorem ipsum

### Step 1: Lorem Ipsum

<img src="https://user-images.githubusercontent.com/44923999/blah.png" width="800" title="Snipped: February 1, 2023" />
