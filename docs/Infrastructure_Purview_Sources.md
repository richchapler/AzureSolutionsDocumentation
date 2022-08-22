## Purview >> Sources

![image](https://user-images.githubusercontent.com/44923999/185973242-b30abab5-6b25-4d03-a403-58c5c42f172c.png)

### Step 1: Source Permissions

#### Azure Data Explorer

Complete the following steps:

* Navigate to your Data Explorer Cluster, then **Permissions** from the **Security + networking** grouping of the navigation

  <img src="https://user-images.githubusercontent.com/44923999/184018326-6c224eeb-3c10-468a-b04b-bd3f4ea9fa57.png" width="800" title="Snipped: August 10, 2022" />

* On the resulting screen, click **+ Add** and select **AllDatabasesViewer** from the resulting dropdown menu
* On the resulting pop-out, search for and select the managed identity of your Purview instance, then click **Select**

#### Azure Data Lake, Gen 2

Complete the following steps:

* Navigate to your Data Lake, then **Access Control (IAM)** in the navigation

  <img src="https://user-images.githubusercontent.com/44923999/184259786-22c36812-ae2a-410d-8c62-cf3e48022e5e.png" width="800" title="Snipped: August 11, 2022" />

* On the resulting screen, click **+ Add** and select **Add role assignment** from the resulting dropdown menu

  <img src="https://user-images.githubusercontent.com/44923999/184259881-65add184-5505-4beb-b072-e340bcb6bfaa.png" width="800" title="Snipped: August 11, 2022" />

* On the resulting **Add role assignment** screen, search for and select **Storage Blob Data Reader**, then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/184260378-4e203be5-1956-4ffa-964f-e32de9c3f76d.png" width="800" title="Snipped: August 11, 2022" />

* On the resulting **Add role assignment** screen, select the **Managed identity** radio button, and then click **+ Select members**
* On the resulting pop-out, search for and select the managed identity of your Purview instance, then click **Select**
* Back on the **Add role assignment** screen, click **Review + assign**

  <img src="https://user-images.githubusercontent.com/44923999/184260718-84a5cb5c-d9b5-45c5-ae40-7e1e1f453fd6.png" width="800" title="Snipped: August 11, 2022" />

* On the resulting **Add role assignment** screen, confirm configuration and then click **Review + assign**

### Step 2: Register Source

#### Azure Data Explorer

Complete the following steps:

* Navigate to **Data map** > **Sources** in the **Microsoft Purview Governance Portal**
* Click **Register**

  <img src="https://user-images.githubusercontent.com/44923999/184013736-581011e6-f230-49ff-b888-5ba44137badb.png" width="800" title="Snipped: August 10, 2022" />

* On the resulting **Register sources** pop-out form, search for and select **Azure Data Explorer (Kusto)** and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184014298-5035b63e-a98b-489a-971e-f1c7d8703771.png" width="800" title="Snipped: August 10, 2022" />

* Complete the resulting **Register sources (Azure Data Explorer (Kusto))** pop-out form and then click **Register**

### Step 3: Scan Source

Complete the following steps:

* Navigate to **Data map** > **Sources** in the **Microsoft Purview Governance Portal**

  <img src="https://user-images.githubusercontent.com/44923999/184015328-b6933bad-80df-4daf-a596-25320f0d941b.png" width="800" title="Snipped: August 10, 2022" />

* Click the **New Scan** icon on your registered source

  <img src="https://user-images.githubusercontent.com/44923999/184017490-3fcdd40e-9e11-497a-81a9-3e945b254e19.png" width="800" title="Snipped: August 10, 2022" />

* Complete the resulting pop-out form, click **Test Connection** to confirm successful connection, and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184018634-6b7b579f-c05c-4c37-88ea-5ea9473a197c.png" width="800" title="Snipped: August 10, 2022" />

* Confirm selections on the resulting **Scope your scan** pop-out form, and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184018996-cca7f92c-6475-4daf-abaa-f6929eb3e72b.png" width="800" title="Snipped: August 10, 2022" />

* Select **AzureDataExplorer** on the resulting **Select a scan rule set** pop-out form, and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184019391-81329502-7ed4-4e17-9a24-a8aaaaaa2feb.png" width="800" title="Snipped: August 10, 2022" />

* Complete the resulting **Set a scan trigger** pop-out form, and then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/184019677-3642f802-5774-48e1-abf0-5e0c6f942797.png" width="800" title="Snipped: August 10, 2022" />

* Confirm values on the resulting **Review your scan** pop-out form, and then click **Save and run**

### Step 4: Confirm Success

Complete the following steps:

* Navigate to **Data map** > **Monitoring** in the **Microsoft Purview Governance Portal**

  <img src="https://user-images.githubusercontent.com/44923999/184021732-42a98402-f764-468c-a450-98571d5a876c.png" width="800" title="Snipped: August 10, 2022" />

* Confirm successful scan
* Navigate to **Data catalog** > **Browse** in the **Microsoft Purview Governance Portal**

  <img src="https://user-images.githubusercontent.com/44923999/184021555-2cbda974-1708-41c9-8299-68aa6d1f7e84.png" width="800" title="Snipped: August 10, 2022" />

* Browse and confirm the scanned source

### Reference
https://docs.microsoft.com/en-us/azure/purview/register-scan-azure-data-explorer
