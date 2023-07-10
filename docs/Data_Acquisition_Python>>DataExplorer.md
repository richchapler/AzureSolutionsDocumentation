# Data Acquisition: Python >> Data Explorer, Secure Connection

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/88915f08-7bfa-4e93-86e6-1f462e3b66a4" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to connect our Python Web App to Data Explorer"
* "We do not want to use device or user login"
* "We do not want to bake keys or secrets into Python code"

## Proposed Solution
* Demonstrate secure Python >> Data Explorer connectivity in the following steps:
  * Install Libraries
  * Connect Python >> Data Explorer via Service Principal

## Solution Requirements
The proposed solution requires:
* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
* [**Databricks**](https://learn.microsoft.com/en-us/azure/databricks/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) with:
  * [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with [Secret](https://learn.microsoft.com/en-us/azure/key-vault/secrets)

-----

## Exercise 1: Create Demonstration
In this exercise, we will create a Databricks Notebook with Python code blocks.

### Step 1: Install Libraries

Open Databricks and click "**Launch Workspace**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c3a0ed84-f7e9-4fb5-b0a4-bc0aed92fc97" width="800" title="Snipped: July 10, 2023" />

Click "**Create a notebook**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fdfeaaae-39ce-4cd2-9952-9674516f14a4" width="800" title="Snipped: July 10, 2023" />

#### Azure-Identity

In the  `Cmd 1` block, paste the following Python:

```
%pip install azure-identity
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f68b296d-2a9a-48f2-af10-c706cf3b5a43" width="800" title="Snipped: July 10, 2023" />

Click "Run Cell" (from the right-pointing, triangle dropdown button in the upper-right of the cell).
<br>You may be prompted to attach to a compute resource... if so, create and or leverage an existing cluster, and click "**Start, attach and run**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3f12bf02-84b3-460f-9eb0-012332dc53dc" width="800" title="Snipped: July 10, 2023" />

You can expect a response like...

```
Python interpreter will be restarted.
Collecting azure-identity
  Downloading azure_identity-1.13.0-py3-none-any.whl (151 kB)
Requirement already satisfied: six>=1.12.0 in /databricks/python3/lib/python3.9/site-packages (from azure-identity) (1.16.0)
Collecting msal-extensions<2.0.0,>=0.3.0
  Downloading msal_extensions-1.0.0-py2.py3-none-any.whl (19 kB)
Collecting msal<2.0.0,>=1.20.0
  Downloading msal-1.22.0-py2.py3-none-any.whl (90 kB)
Collecting azure-core<2.0.0,>=1.11.0
  Downloading azure_core-1.27.1-py3-none-any.whl (174 kB)
Requirement already satisfied: cryptography>=2.5 in /databricks/python3/lib/python3.9/site-packages (from azure-identity) (3.4.8)
Collecting typing-extensions>=4.3.0
  Downloading typing_extensions-4.7.1-py3-none-any.whl (33 kB)
Requirement already satisfied: requests>=2.18.4 in /databricks/python3/lib/python3.9/site-packages (from azure-core<2.0.0,>=1.11.0->azure-identity) (2.27.1)
Requirement already satisfied: cffi>=1.12 in /databricks/python3/lib/python3.9/site-packages (from cryptography>=2.5->azure-identity) (1.15.0)
Requirement already satisfied: pycparser in /databricks/python3/lib/python3.9/site-packages (from cffi>=1.12->cryptography>=2.5->azure-identity) (2.21)
Collecting PyJWT[crypto]<3,>=1.0.0
  Downloading PyJWT-2.7.0-py3-none-any.whl (22 kB)
Collecting portalocker<3,>=1.0
  Downloading portalocker-2.7.0-py2.py3-none-any.whl (15 kB)
Requirement already satisfied: idna<4,>=2.5 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.11.0->azure-identity) (3.3)
Requirement already satisfied: charset-normalizer~=2.0.0 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.11.0->azure-identity) (2.0.4)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.11.0->azure-identity) (1.26.9)
Requirement already satisfied: certifi>=2017.4.17 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.11.0->azure-identity) (2021.10.8)
Installing collected packages: PyJWT, typing-extensions, portalocker, msal, msal-extensions, azure-core, azure-identity
  Attempting uninstall: typing-extensions
    Found existing installation: typing-extensions 4.1.1
    Not uninstalling typing-extensions at /databricks/python3/lib/python3.9/site-packages, outside environment /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6
    Can't uninstall 'typing-extensions'. No files were found to uninstall.
Successfully installed PyJWT-2.7.0 azure-core-1.27.1 azure-identity-1.13.0 msal-1.22.0 msal-extensions-1.0.0 portalocker-2.7.0 typing-extensions-4.7.1
Python interpreter will be restarted.
```

Click the downward-pointing carat in the upper-right of the cell and select "Add Cell Below" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4dfb28cd-f353-4ecf-bf6d-6ababc8ccd23" width="800" title="Snipped: July 10, 2023" />

#### Azure-Kusto-Data

In the  `Cmd 2` block, paste and run the following Python:

```
%pip install azure-kusto-data
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4b31f376-712d-4378-9943-71813f1e8963" width="800" title="Snipped: July 10, 2023" />

You can expect a response like...

```
Python interpreter will be restarted.
Collecting azure-kusto-data
  Downloading azure_kusto_data-4.2.0-py2.py3-none-any.whl (54 kB)
Requirement already satisfied: azure-identity<2,>=1.5.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-kusto-data) (1.13.0)
Requirement already satisfied: requests>=2.13.0 in /databricks/python3/lib/python3.9/site-packages (from azure-kusto-data) (2.27.1)
Requirement already satisfied: msal<2,>=1.9.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-kusto-data) (1.22.0)
Collecting ijson~=3.1
  Downloading ijson-3.2.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (111 kB)
Requirement already satisfied: azure-core<2,>=1.11.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-kusto-data) (1.27.1)
Requirement already satisfied: python-dateutil>=2.8.0 in /databricks/python3/lib/python3.9/site-packages (from azure-kusto-data) (2.8.2)
Requirement already satisfied: typing-extensions>=4.3.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-core<2,>=1.11.0->azure-kusto-data) (4.7.1)
Requirement already satisfied: six>=1.11.0 in /databricks/python3/lib/python3.9/site-packages (from azure-core<2,>=1.11.0->azure-kusto-data) (1.16.0)
Requirement already satisfied: msal-extensions<2.0.0,>=0.3.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-identity<2,>=1.5.0->azure-kusto-data) (1.0.0)
Requirement already satisfied: cryptography>=2.5 in /databricks/python3/lib/python3.9/site-packages (from azure-identity<2,>=1.5.0->azure-kusto-data) (3.4.8)
Requirement already satisfied: cffi>=1.12 in /databricks/python3/lib/python3.9/site-packages (from cryptography>=2.5->azure-identity<2,>=1.5.0->azure-kusto-data) (1.15.0)
Requirement already satisfied: pycparser in /databricks/python3/lib/python3.9/site-packages (from cffi>=1.12->cryptography>=2.5->azure-identity<2,>=1.5.0->azure-kusto-data) (2.21)
Requirement already satisfied: PyJWT[crypto]<3,>=1.0.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from msal<2,>=1.9.0->azure-kusto-data) (2.7.0)
Requirement already satisfied: portalocker<3,>=1.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from msal-extensions<2.0.0,>=0.3.0->azure-identity<2,>=1.5.0->azure-kusto-data) (2.7.0)
Requirement already satisfied: idna<4,>=2.5 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.13.0->azure-kusto-data) (3.3)
Requirement already satisfied: charset-normalizer~=2.0.0 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.13.0->azure-kusto-data) (2.0.4)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.13.0->azure-kusto-data) (1.26.9)
Requirement already satisfied: certifi>=2017.4.17 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.13.0->azure-kusto-data) (2021.10.8)
Installing collected packages: ijson, azure-kusto-data
Successfully installed azure-kusto-data-4.2.0 ijson-3.2.2
Python interpreter will be restarted.
```

Add a cell, then paste and run the following Python:

```
import azure.kusto.data
print(azure.kusto.data.__version__)
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ff7c675b-d674-4bb7-b05f-288fc9c907fa" width="800" title="Snipped: July 10, 2023" />

You can expect a response like...

```
4.2.0
```

_Note: Checking Azure-Kusto-Data Library version is not necessary, but might be useful for troubleshooting_

#### Azure-KeyVault-Secrets

Add a cell, then paste and run the following Python:

```
%pip install azure-keyvault-secrets
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/591de671-395b-4c6a-a33a-5730adcd806c" width="800" title="Snipped: July 10, 2023" />

You can expect a response like...

```
Python interpreter will be restarted.
Collecting azure-keyvault-secrets
  Downloading azure_keyvault_secrets-4.7.0-py3-none-any.whl (348 kB)
Collecting azure-common~=1.1
  Downloading azure_common-1.1.28-py2.py3-none-any.whl (14 kB)
Collecting isodate>=0.6.1
  Downloading isodate-0.6.1-py2.py3-none-any.whl (41 kB)
Requirement already satisfied: azure-core<2.0.0,>=1.24.0 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-keyvault-secrets) (1.27.1)
Requirement already satisfied: typing-extensions>=4.0.1 in /local_disk0/.ephemeral_nfs/envs/pythonEnv-d8929e04-eb72-4534-a423-fe0bc382b0d6/lib/python3.9/site-packages (from azure-keyvault-secrets) (4.7.1)
Requirement already satisfied: six>=1.11.0 in /databricks/python3/lib/python3.9/site-packages (from azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (1.16.0)
Requirement already satisfied: requests>=2.18.4 in /databricks/python3/lib/python3.9/site-packages (from azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (2.27.1)
Requirement already satisfied: idna<4,>=2.5 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (3.3)
Requirement already satisfied: charset-normalizer~=2.0.0 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (2.0.4)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (1.26.9)
Requirement already satisfied: certifi>=2017.4.17 in /databricks/python3/lib/python3.9/site-packages (from requests>=2.18.4->azure-core<2.0.0,>=1.24.0->azure-keyvault-secrets) (2021.10.8)
Installing collected packages: isodate, azure-common, azure-keyvault-secrets
Successfully installed azure-common-1.1.28 azure-keyvault-secrets-4.7.0 isodate-0.6.1
Python interpreter will be restarted.
```

### Step 2: Connect Python >> Data Explorer via Service Principal

Continuing in the notebook... add a cell, then paste and run the following Python:

```
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient
from azure.kusto.data import KustoClient, KustoConnectionStringBuilder
from azure.kusto.data.helpers import dataframe_from_result_table

# Data Explorer
adxCluster = "{DATAEXPLORER_URI}"
adxDatabase = "{DATAEXPLORER_DATABASE}"
clientId = "{APPLICATIONREGISTRATION_CLIENTID}"
clientSecret = "{APPLICATIONREGISTRATION_CLIENTSECRET}"
authorityId = "{TENANT_ID}"

kcsb = KustoConnectionStringBuilder.with_aad_application_key_authentication( adxCluster, clientId, clientSecret, authorityId )

kc = KustoClient(kcsb)
adxQuery = "StormEvents | take 10"
response = kc.execute(adxDatabase, adxQuery)

df = dataframe_from_result_table(response.primary_results[0])

print(df)
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc7bd2f2-40c2-49a9-867c-b23f45768e4c" width="800" title="Snipped: July 10, 2023" />

Logic Explained:

* `from azure...` imports Azure classes
* `{DATAEXPLORER_URI}`, etc... should be replaced with values specific to your configuration
* `kcsb =...` creates a KustoConnectionStringBuilder object that can be used to connect to Data Explorer using Azure Active Directory, Application Registration authentication
* `kc =...` creates a KustoClient object that can query Data Explorer databases
* `response = kc.execute(...` executes a query on Data Explorer
* `df = dataframe_from_result_table(...` converts a Data Explorer result table into a Pandas DataFrame object

You can expect a response like...

```
                  StartTime                   EndTime  EpisodeId  EventId  \
0 2007-01-01 00:00:00+00:00 2007-01-01 00:00:00+00:00       2592    13208   
1 2007-01-01 00:00:00+00:00 2007-01-01 05:00:00+00:00       4171    23358   
2 2007-01-01 00:00:00+00:00 2007-01-01 05:00:00+00:00       4171    23357   
3 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9494   
4 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9488   
5 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9487   
6 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9485   
7 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9486   
8 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9493   
9 2007-01-01 00:00:00+00:00 2007-01-01 06:00:00+00:00       1930     9489   

            State          EventType  InjuriesDirect  InjuriesIndirect  \
0  NORTH CAROLINA  Thunderstorm Wind               0                 0   
1       WISCONSIN       Winter Storm               0                 0   
2       WISCONSIN       Winter Storm               0                 0   
3        NEW YORK     Winter Weather               0                 0   
4        NEW YORK     Winter Weather               0                 0   
5        NEW YORK     Winter Weather               0                 0   
6        NEW YORK     Winter Weather               0                 0   
7        NEW YORK     Winter Weather               0                 0   
8        NEW YORK     Winter Weather               0                 0   
9        NEW YORK     Winter Weather               0                 0   

   DeathsDirect  DeathsIndirect  ...                  Source  BeginLocation  \
0             0               0  ...                  Public          CASAR   
1             0               0  ...           COOP Observer                  
2             0               0  ...           COOP Observer                  
3             0               0  ...  Department of Highways                  
4             0               0  ...  Department of Highways                  
5             0               0  ...  Department of Highways                  
6             0               0  ...  Department of Highways                  
7             0               0  ...  Department of Highways                  
8             0               0  ...  Department of Highways                  
9             0               0  ...  Department of Highways                  

  EndLocation BeginLat BeginLon  EndLat  EndLon  \
0       CASAR    35.52   -81.63   35.52  -81.63   
1                 <NA>     <NA>    <NA>    <NA>   
2                 <NA>     <NA>    <NA>    <NA>   
3                 <NA>     <NA>    <NA>    <NA>   
4                 <NA>     <NA>    <NA>    <NA>   
5                 <NA>     <NA>    <NA>    <NA>   
6                 <NA>     <NA>    <NA>    <NA>   
7                 <NA>     <NA>    <NA>    <NA>   
8                 <NA>     <NA>    <NA>    <NA>   
9                 <NA>     <NA>    <NA>    <NA>   

                                    EpisodeNarrative       EventNarrative  \
0  A small cluster of thunderstorms moved rapidly...  Several trees down.   
1  A powerful storm system moved from the souther...                        
2  A powerful storm system moved from the souther...                        
3  A weak area of low pressure moved across Ontar...                        
4  A weak area of low pressure moved across Ontar...                        
5  A weak area of low pressure moved across Ontar...                        
6  A weak area of low pressure moved across Ontar...                        
7  A weak area of low pressure moved across Ontar...                        
8  A weak area of low pressure moved across Ontar...                        
9  A weak area of low pressure moved across Ontar...                        

                                        StormSummary  
0  {'TotalDamages': 0, 'StartTime': '2007-01-01T0...  
1  {'TotalDamages': 0, 'StartTime': '2007-01-01T0...  
2  {'TotalDamages': 0, 'StartTime': '2007-01-01T0...  
3  {'TotalDamages': 2000, 'StartTime': '2007-01-0...  
4  {'TotalDamages': 2000, 'StartTime': '2007-01-0...  
5  {'TotalDamages': 2000, 'StartTime': '2007-01-0...  
6  {'TotalDamages': 3000, 'StartTime': '2007-01-0...  
7  {'TotalDamages': 3000, 'StartTime': '2007-01-0...  
8  {'TotalDamages': 2000, 'StartTime': '2007-01-0...  
9  {'TotalDamages': 2000, 'StartTime': '2007-01-0...  

[10 rows x 22 columns]
```




You can expect a response like...

```
Python interpreter will be restarted.
Collecting azure-keyvault-secrets

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
* [Tips for AI enrichment in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-troubleshooting)
* [Debug an Azure Cognitive Search skillset in Azure portal](https://learn.microsoft.com/en-us/azure/search/cognitive-search-how-to-debug-skillset#debug-a-custom-skill-locally)
