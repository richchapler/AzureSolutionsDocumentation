# AI: Deploy

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/20ef5226-59b5-4876-b8b2-789373480cb4" width="1000" />

## Use Case
* "We are implementing OpenAI + AI Search and rapidly iterating through configuration enhancements"
  * "Creating and updating the AI Search configuration with Azure Portal, "Import data", and "Import and vectorize data" is difficult"
  * "We want to capture our AI Search configuration in our GitHub or DevOps repository"
* "Our AI Search index must include multiple data sources"
* "Both semantic and vector should be activated"
* "To optimize response quality, we should include synonyms for country codes"
* "All keys must be stored in Key Vault"

## Proposed Solution
* Deployment Application: Prepare for automated deployment of a multi-source index (and skillset, indexer, synonym map)
* Source Control: Create a pull request in a DevOps repo
* Custom Skillset API (Bonus Exercise): Use a Function App to instantiate a simple API for use with AI Search, custom skillset

## Solution Requirements
* [AI Search](https://azure.microsoft.com/en-us/products/search)

* [AI Services](https://learn.microsoft.com/en-us/azure/cognitive-services/)

* [DevOps](https://azure.microsoft.com/en-us/products/devops/) with organization and project
 
* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  - Subscription-Id
  - ResourceGroup-Name
  - AISearch-Name
  - AISearch-Key
  - AIServices-Name
  - AIServices-Key
  - OpenAI-Name
  - OpenAI-Key
  - OpenAI-Deployment-Embedding  
 
* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview) with "text-embedding-3-large" [deployment model](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource) 
 
* [Postman](https://www.postman.com/product/workspaces/) (with Desktop Agent for localhost testing)
 
* [Storage Account](Infrastructure_StorageAccount.md) with a container and uploaded sample data {e.g., [IRS Tax Forms](https://www.irs.gov/forms-instructions)}
 
* [SQL Server](https://learn.microsoft.com/en-us/azure/azure-sql) and [Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview?view=azuresql) with [AdventureWorks sample data](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)
 
* [Visual Studio](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload and connected to your DevOps project

<br>
If you intend to prepare a custom skillset, also instantiate:

* [Function App](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured for .NET 7, with dependencies:
  * [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
  * [Application Service](https://learn.microsoft.com/en-us/azure/app-service/)
  * [Storage Account](Infrastructure_StorageAccount.md)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----
-----

## Exercise 1: Deployment Application
In this exercise, we will prepare for automated deployment of a multi-source index (and skillset, indexer, synonym map).

### Step 1: Data Sources

_Philosophical Note: When I write (either articles or code), I strive to limit myself to only what is absolutely necessary and never duplicative with existing functionality. Azure AI Search, Data Source functionality is perfectly adequate and re-creating it would add no value. So, in this step, we'll simply review the creation of a data source (with slight specifics for this use case)._

Navigate to Azure portal >> AI Search >> Data Sources and then click "+ Add data source". Complete the form (examples below):

#### Blob Storage
<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e94da228-1906-4d78-bd19-a0b6e5cca203" width="800" title="Snipped: June 26, 2024" />

Blob storage has standard fields {e.g., `metadata_title`} which can be mapped to stardardized multi-source index columns (like `name`).

#### SQL Database

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ba9fa875-607a-4e3b-9047-1f29588d75bd" width="800" title="Snipped: June 26, 2024" />

SQL database tables will not have standard fields, so we add a SQL Query that provides necessary mapping for inclusion in a multi-source index; example: `SELECT [AddressID] [id], [AddressLine1] [name] FROM [SalesLT].[Address] WITH (NOLOCK)`

### Step 2: Create Visual Studio Project

Open Visual Studio and click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/317959b5-dfd7-4c97-af0c-0578f9e89429" width="600" title="Snipped: October 10, 2023" />

On the "**Create a new project**" form, search for and select "**Console App**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/db8c2898-b607-4441-8b1e-4f4f3dbd56b4" width="600" title="Snipped: October 10, 2023" />

Complete the "**Configure your new project**" form, then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2408d491-ba3b-4ba7-9d84-02caf1dab54d" width="600" title="Snipped: October 10, 2023" />

Complete the "**Additional information**" form, then click "**Create**".

-----

### Step 3: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/464851d5-30c0-4b72-87d5-cb95658d919d" width="800" title="Snipped: October 11, 2023" />

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/00248893-859f-4db5-96b4-dcbbf5cbc752" width="800" title="Snipped: May 29, 2024" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**AzureSolutions.Helpers**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".
<br>When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.
<br>When complete, close the "**NuGet - Solution**" tab.

_Note: I am steadily moving more and more functionality from one-off solutions to AzureSolutions.Helpers and I'm happy to share those details to interested parties. Ping me!_

-----

### Step 4: Synonym Map

Right-click on the project and add new file `synonyms.json`.

```json
[
  [ "US", "USA", "U.S.A.", "United States", "United States of America" ],
  [ "MX", "Mexico" ]
]  
```

-----

### Step 5: `Program.cs`

Replace the default code in `Program.cs` with the following C#:

```csharp
#pragma warning disable IDE1006 // Disable Warning: Naming Styles

using AzureSolutions.Helpers;

namespace AI_Deploy
{
    public class Program
    {
        public static AzureSolutions.Helpers.KeyVault.Configuration kvc { get; set; } = new AzureSolutions.Helpers.KeyVault.Configuration();
        public static AzureSolutions.Helpers.AISearch.Configuration aisc { get; set; } = new AzureSolutions.Helpers.AISearch.Configuration();
        public static AzureSolutions.Helpers.OpenAI.Configuration oaic { get; set; } = new AzureSolutions.Helpers.OpenAI.Configuration();

        public static async Task Main()
        {
            try
            {
                /* ************************* User Input: KeyVault and Configurations */

                var KeyVault_Name = AzureSolutions.Helpers.KeyVault.Name.Get("KeyVault_Name");
                if (KeyVault_Name == null) throw new ArgumentNullException(nameof(KeyVault_Name));

                kvc = AzureSolutions.Helpers.KeyVault.Configuration.Prepare(KeyVault_Name);
                if (kvc?.Client == null) throw new Exception("KeyVault Client is not initialized.");
                Console.WriteLine($".KeyVault '{kvc.Name}' prepared...");

                aisc = AzureSolutions.Helpers.AISearch.Configuration.Prepare(kvc.Client);
                if (aisc.Name == null || aisc.Key == null || aisc.Index == null) { throw new ArgumentNullException("Config_AISearch.Name or Config_AISearch.Key is null"); }
                Console.WriteLine($".AI Search '{aisc.Name}' prepared...");

                oaic = AzureSolutions.Helpers.OpenAI.Configuration.Prepare(kvc.Client);
                if (oaic.Name == null || oaic.Key == null) { throw new ArgumentNullException("Config_OpenAI.Name or Config_OpenAI.Key is null"); }
                Console.WriteLine($".OpenAI '{oaic.Name}' prepared...");

                /* ************************* Required Secrets */

                var Secrets = AzureSolutions.Helpers.KeyVault.Secret.GetList(kvc, [
                    "Subscription-Id",
                    "ResourceGroup-Name",
                    "AISearch-Name",
                    "AISearch-Key",
                    "AIServices-Name",
                    "AIServices-Key",
                    "OpenAI-Name",
                    "OpenAI-Key",
                    "OpenAI-Deployment-Embedding"
                ]);

                /* ************************* User Input: Data Sources */

                Console.WriteLine($"\nExisting Data Sources on Azure AI Search instance '{aisc.Name}'");

                List<AzureSolutions.Helpers.AISearch.DataSource> DataSources = [];

                var existingDataSources = await AzureSolutions.Helpers.AISearch.DataSource.List_Existing(aisc);

                existingDataSources.Select((dataSource, index) => $"{index + 1}. {dataSource.Name}").ToList().ForEach(Console.WriteLine);

                Console.WriteLine("\nEnter Data Sources using a comma-separated list {e.g., 1,2,4}:");
                var selectedIndices = Console.ReadLine().Split(',').Select(int.Parse).ToList();

                foreach (var index in selectedIndices)
                {
                    if (index < 1 || index > existingDataSources.Count) { throw new ArgumentException("Invalid selection."); }
                    var selectedDataSource = existingDataSources[index - 1];
                    DataSources.Add(selectedDataSource);
                }


                Console.WriteLine("\n************************* Delete Resources");

                foreach (var DataSource in DataSources)
                {
                    await AzureSolutions.Helpers.AISearch.Indexer.Delete(aisc, $"indexer-{DataSource.Name}");

                    await AzureSolutions.Helpers.AISearch.Skillset.Delete(aisc, $"skillset-{DataSource.Name}");

                    await AzureSolutions.Helpers.AISearch.SynonymMap.Delete(aisc);
                }

                await AzureSolutions.Helpers.AISearch.Index.Delete(aisc); /* Fully-articulated because "Index" is a reserved word */

                Console.WriteLine("\n************************* Create Resources");

                await AzureSolutions.Helpers.AISearch.Index.Create(aisc, oaic);

                await AzureSolutions.Helpers.AISearch.SynonymMap.Create(aisc, IndexName: aisc.Index, SynonymsJson_Path: $"..\\..\\..\\synonyms.json");

                for (int i = 0; i < DataSources.Count; i++)
                {
                    var DataSource = DataSources[i];

                    await AzureSolutions.Helpers.AISearch.Skillset.Create(kvc, aisc, oaic, Name: $"skillset-{DataSource.Name}", DataSource.Type);

                    await AzureSolutions.Helpers.AISearch.Indexer.Create(aisc,
                        Name: $"indexer-{DataSource.Name}",
                        Type: DataSource.Type,
                        DataSource_Name: DataSource.Name,
                        Index_Name: aisc.Index,
                        Skillset_Name: $"skillset-{DataSource.Name}"
                        );
                }
            }
            catch (Exception ex) { Console.WriteLine($"Exception: {ex}\n"); }
        }
    }
}
```

-----

### Step 6: Confirm Success

#### Visual Studio Debug

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/94ba39d5-0a55-4439-b2d1-188917087aa4" width="800" title="Snipped: October 11, 2023" />

Save your changes and then click "**Debug**" >> "**Start Debugging**" in the menubar.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d00e11c1-5611-49ed-b31e-228eee5428b2" width="600" title="Snipped: October 11, 2023" />

A "Microsoft Visual Studio Debug" window will open (as snipped above).

#### AI Search Index

Navigate to AI Search, then "**Indexes**" in the "**Search management**" grouping of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/60745fe0-95f6-40f7-a91b-94b10332c237" width="800" title="Snipped: October 12, 2023" />

You should see the index that you programmatically created {e.g., "rchaplerss-index"}. Click to open.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/302f8a28-e828-44de-b04c-cb9511bf19a2" width="800" title="Snipped: October 12, 2023" />

Click the "**Search**" button and review results.

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 2: Source Control
In this exercise, we will create a pull request in a DevOps repo.

### Step 1: Create and Push

Open Visual Studio, navigate to "**View**" >> "**Git Changes**", and then click "**Create Git Repository**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/54ffccf9-64db-4e1b-9177-2bde140fa473" width="600" title="Snipped: October 12, 2023" />

Complete the resulting "Create a Git repository" pop-up form, then click "**Create and Push**".

-----

**Congratulations... you have successfully completed this exercise**

-----

## Bonus Exercise: Custom Skillset API
In this exercise, we will use a Function App to instantiate a simple API for use with AI Search, custom skillset.
<br>_Note: Complete this exercise to include a custom skillset in the AI Search deployment application_

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/user-attachments/assets/0a3b222f-09ad-4b5b-8add-6f29c8492f1b" width="800" title="Snipped: July 30, 2024" />

On the "Create a new project" page, search for and select "Azure Functions", then click "Next".

<img src="https://github.com/user-attachments/assets/9da2a11c-f5e8-4008-adf5-96ca152e5b99" width="800" title="Snipped: July 30, 2024" />

Complete the "Configure your new project" form and then click "Next".

<img src="https://github.com/user-attachments/assets/2e730487-319b-424d-9369-f7998fe472a5" width="800" title="Snipped: July 30, 2024" />

Complete the "Additional information" form:

Prompt | Entry
:----- | :-----
Functions worker | .NET 8.0 Isolated
Function | Http trigger
Use Azurite... | Checked
Authorization level | Function

Click "Create".

-----

### Step 2: Code Function

Rename "Function1.cs" to "CustomSkillset.cs". When prompted "You are renaming a file...", click "**Yes**" to perform rename on all references.

<img src="https://github.com/user-attachments/assets/83be381a-ab8d-4e09-9edd-4e8c733cc710" width="800" title="Snipped: July 30, 2024" />

Replace the default logic in "CustomSkillset" with:

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using System.Net;
using System.Text.Json;

namespace FunctionApp_CustomSkillset
{
    public class CustomSkillset
    {
        [Function("CustomSkillset")]
        public HttpResponseData Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData hrd)
        {
            string request = string.Empty;

            using (var reader = new StreamReader(hrd.Body)) { request = reader.ReadToEnd(); }

            var d = new Dictionary<string, object> { { "values", new List<Dictionary<string, object>>() } };

            foreach (var value in JsonDocument.Parse(request).RootElement.GetProperty("values").EnumerateArray())
            {
                var r = new Dictionary<string, object>
                {
                    { "recordId", value.GetProperty("recordId").GetString() ?? string.Empty },
                    { "data", new Dictionary<string, object> { { "myColumn", value.GetProperty("data") } } },
                    { "errors", string.Empty },
                    { "warnings", string.Empty }
                };

                ((List<Dictionary<string, object>>)d["values"]).Add(r);
            }

            var response = hrd.CreateResponse(HttpStatusCode.OK);
            response.WriteAsJsonAsync(d);
            return response;
        }
    }
}
```

-----

### Step 3: Publish Function

<img src="https://github.com/user-attachments/assets/406e884b-5364-4e71-8470-4567169029c1" width="600" title="Snipped: July 30, 2024" />

Right-click on your project in the Solution Explorer pane and select "Publish" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5a090114-5879-44d0-a07c-c78a006d92ff" width="600" title="Snipped: October 30, 2023" />

On the "Publish" >> "Target" page, select "Azure", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/de97bbd3-ee15-4b1f-af0e-4299f14d6085" width="600" title="Snipped: October 30, 2023" />

On the "Publish" >> "Specific target" page, select "Azure App Service (Windows)", then click "Next".

<img src="https://github.com/user-attachments/assets/1a02b31a-f7b2-4672-a39f-33e7c3071212" width="600" title="Snipped: July 30, 2024" />

On the "Publish" >> "Functions instance" page, select your Function App and then click "Finish".

<img src="https://github.com/user-attachments/assets/355bd31a-1b70-4a72-a843-6e1b30596025" width="600" title="Snipped: July 30, 2024" />

On the "Publish profile creation progress" >> "Finish" page, click "Close".

<img src="https://github.com/user-attachments/assets/fe69130a-e418-4ada-9a5d-526cbe75db88" width="800" title="Snipped: July 30, 2024" />

Back on the "...Publish" page, click "Publish", allow time for processing, and confirm successful publication.

-----

### Step 4: Confirm Success

Navigate to your Function App >> "CustomSkillset" function >> "Code + Test".

<img src="https://github.com/user-attachments/assets/be9424fb-21dd-401e-8ffb-e1e592c1f22e" width="800" title="Snipped: July 30, 2024" />

Click "Test/Run" and on the resulting pop-out, "Input" tab, paste the following "Body" value:

```json
{"values":[{"recordId":"0","data":{"text":"Lorem Ipsum"}}]}
```

Click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7f0f47af-5e58-4fd9-822b-215e910c6602" width="800" title="Snipped: October 31, 2023" />

The pop-out will switch to the "**Output**" tab and you can expect the following "**HTTP response content**" value:

```json
{
  "values": [
    {
      "recordId": "0",
      "data": {
        "myColumn": {
          "text": "Lorem Ipsum"
        }
      },
      "errors": "",
      "warnings": ""
    }
  ]
}
```

-----

**Congratulations... you have successfully completed this exercise**
