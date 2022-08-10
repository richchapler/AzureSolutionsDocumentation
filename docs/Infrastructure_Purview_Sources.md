## Purview >> Sources

### Step 1: Permissions

#### Azure Data Explorer

Complete the following steps:

* Navigate to your Data Explorer Cluster, then **Permissions** from the **Security + networking** grouping of the navigation

  <img src="https://user-images.githubusercontent.com/44923999/184018326-6c224eeb-3c10-468a-b04b-bd3f4ea9fa57.png" width="800" title="Snipped: August 10, 2022" />

* On the resulting screen, click **+ Add** and select **AllDatabasesViewer** from the resulting dropdown menu
* On the resulting pop-out, search for and select the managed identity of your Purview instance, then click **Select**

### Step 1: Register Source

#### Azure Data Explorer

Complete the following steps:

* Navigate to **Data map** > **Sources** in the **Microsoft Purview Governance Portal**
* Click **Register**

  <img src="https://user-images.githubusercontent.com/44923999/184013736-581011e6-f230-49ff-b888-5ba44137badb.png" width="800" title="Snipped: August 10, 2022" />

* Search for and select **Azure Data Explorer (Kusto)** and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184014298-5035b63e-a98b-489a-971e-f1c7d8703771.png" width="800" title="Snipped: August 10, 2022" />

* Complete the resulting pop-out form and then click **Register**

### Step 2: Scan

Complete the following steps:

* Navigate to **Data map** > **Sources** in the **Microsoft Purview Governance Portal**

  <img src="https://user-images.githubusercontent.com/44923999/184015328-b6933bad-80df-4daf-a596-25320f0d941b.png" width="800" title="Snipped: August 10, 2022" />

* Click the **New Scan** icon on your registered source

  <img src="https://user-images.githubusercontent.com/44923999/184017490-3fcdd40e-9e11-497a-81a9-3e945b254e19.png" width="800" title="Snipped: August 10, 2022" />

* Complete the resulting pop-out form, click **Test Connection** to confirm successful connection, and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184018634-6b7b579f-c05c-4c37-88ea-5ea9473a197c.png" width="800" title="Snipped: August 10, 2022" />

* Confirm selections on the resulting **Scope your scan** pop-out form, and then click **Continue**

### Reference
https://docs.microsoft.com/en-us/azure/purview/register-scan-azure-data-explorer
