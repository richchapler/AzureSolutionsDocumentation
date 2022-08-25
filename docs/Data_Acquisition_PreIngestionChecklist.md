## Data Acquisition... Pre-Ingestion Checklist

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:
* "We are planning to kick-off ingestion (both continuous and historical)... is there anything we should consider before we begin?"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** [Cluster](Infrastructure_DataExplorer_Cluster.md) and [Database](Infrastructure_DataExplorer_Database.md)

### Step 2: Create Linked Service
In this step, we will create the Linked Service we will use to get our Bearer Token.

Complete the following steps:

* Open **Synapse Studio** and click the **Manage** navigation icon
* Click "**Linked services**" from the "**External connections**" grouping in the resulting navigation
* Click "**+ New**"

  <img src="https://user-images.githubusercontent.com/44923999/186210475-7a993950-f5ba-4775-bb55-0dcb165a57a2.png" width="800" title="Snipped: August 23, 2022" />
