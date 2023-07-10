# Data Acquisition: Python >> Data Explorer, Secure Connection

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/88915f08-7bfa-4e93-86e6-1f462e3b66a4" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to connect our Python Web App to Data Explorer"
* "We do not want to use device or user login"
* "We do not want to bake keys or secrets into Python code"

## Proposed Solution
* Demonstrate secure Python >> Data Explorer connectivity

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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/abab94b0-e0f3-4141-9ecd-66bf4b8a0fde" width="800" title="Snipped: July 10, 2023" />









In the  `Cmd 1` block, paste the following Python:

```
%pip install azure-identity

%pip install azure-kusto-data
import azure.kusto.data
print(azure.kusto.data.__version__)

%pip install azure-keyvault-secrets
```

In the  `Cmd 1` block, paste the following Python:

```
%pip install azure-identity

%pip install azure-kusto-data
import azure.kusto.data
print(azure.kusto.data.__version__)

%pip install azure-keyvault-secrets
```

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
* [Tips for AI enrichment in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-troubleshooting)
* [Debug an Azure Cognitive Search skillset in Azure portal](https://learn.microsoft.com/en-us/azure/search/cognitive-search-how-to-debug-skillset#debug-a-custom-skill-locally)
