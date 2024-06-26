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

-----










-----

#### Synonym Map

##### `synonyms.json`

```json
[
  [ "US", "USA", "U.S.A.", "United States", "United States of America" ],
  [ "MX", "Mexico" ]
]  
```

-----

### Step 5: `Program.cs`

Replace the default code on the "**Program.cs**" tab with the following C#:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using AzureSolutions.Helpers.AISearch;
using System.Text.RegularExpressions;

namespace AI_Deploy
{
    public class Program
    {
        private static SecretClient? keyvault;

        private static string GetUserChoice(string variableName)
        {
            var options = new Dictionary<string, Dictionary<string, (string Choice, string Final)>>
            {
                {
                    "Data Source Type?",
                    new Dictionary<string, (string Choice, string Final)>
                    {
                        { "1", ("Azure Blob Storage", "blob") },
                        { "2", ("Azure SQL", "sql") }
                    }
                },
                {
                    "Create Method?",
                    new Dictionary<string, (string Choice, string Final)>
                    {
                        { "1", ("API", "API") },
                        { "2", ("SDK", "SDK") }
                    }
                }
            };

            var message = $"\n{variableName}: " + string.Join(" or ", options[variableName].Select(o => $"{o.Key}. {o.Value.Choice}"));

            Console.WriteLine(message);
            string? userChoice = Console.ReadLine();
            return userChoice != null && options[variableName].TryGetValue(userChoice, out var value)
                ? value.Final
                : string.Empty;
        }

        public static async Task Main()
        {
            try
            {
                /* ************************* User Choices */

                Console.WriteLine("\nConfigure...");
                
                string DataSourceType = GetUserChoice("Data Source Type?");
                string CreateMethod = GetUserChoice("Create Method?");

                /* ************************* KeyVault */

                Console.WriteLine("\nKeyVault Name?");

                string? KeyVault_Name = Console.ReadLine();

                keyvault = new(vaultUri: new Uri($"https://{KeyVault_Name}.vault.azure.net"), credential: new DefaultAzureCredential());

                Dictionary<string, string> secret = [];

                string[] secretNames = [
                    "Subscription-Id",
                    "ResourceGroup-Name",
                    "AISearch-Name",
                    "AISearch-Key",
                    "AISearch-DataSource-Name",
                    "AISearch-DataSource-Blob-ConnectionString",
                    "AISearch-DataSource-Blob-Container",
                    "AISearch-DataSource-SQL-Server-Name",
                    "AISearch-DataSource-SQL-Server-User",
                    "AISearch-DataSource-SQL-Server-Password",
                    "AISearch-DataSource-SQL-Database-Name",
                    "AISearch-DataSource-SQL-Table",
                    "AISearch-DataSource-SQL-Query",
                    "AISearch-Index-Name",
                    "AISearch-Skillset-Name",
                    "AISearch-Indexer-Name",
                    "AISearch-Suggester-Name",
                    "AISearch-SynonymMap-Name",
                    "AISearch-SemanticConfiguration-Name",
                    "AISearch-VectorProfile-Name",
                    "AISearch-VectorAlgorithm-Name",
                    "AISearch-Vectorizer-Name",
                    "AIServices-Name",
                    "AIServices-Key",
                    "OpenAI-Name",
                    "OpenAI-Key",
                    "OpenAI-Deployment-Embedding"
                ];

                foreach (var secretName in secretNames)
                {
                    string secretValue = AzureSolutions.Helpers.KeyVault.GetSecret(keyvault, secretName, () =>
                    {
                        Console.WriteLine($"\nEnter value for missing KeyVault Secret '{secretName}':");
                        return Console.ReadLine() ?? string.Empty;
                    });

                    secret[secretName] = secretValue;
                }

                switch (DataSourceType)
                {
                    case "Azure Blob Storage":
                        string Storage_ConnectionString = GetSecret(keyvault, "AISearch-DataSource-Blob-ConnectionString");
                        string Storage_ContainerName = GetSecret(keyvault, "AISearch-DataSource-Blob-Container");
                        break;
                    case "Azure SQL Database":
                        break;
                }

                /* ************************* Delete Existing */

                Console.WriteLine("\nDelete Existing...");

                await Indexer.Delete(secret["AISearch-Name"], secret["AISearch-Key"], secret["AISearch-Indexer-Name"]);

                await Skillset.Delete(secret["AISearch-Name"], secret["AISearch-Key"], secret["AISearch-Skillset-Name"]);

                await AzureSolutions.Helpers.AISearch.Index.Delete(secret["AISearch-Name"], secret["AISearch-Key"], secret["AISearch-Index-Name"]);
                /* Fully-articulated because "Index" is a reserved word */

                await SynonymMap.Delete(secret["AISearch-Name"], secret["AISearch-Key"], secret["AISearch-SynonymMap-Name"]);

                await DataSource.Delete(secret["AISearch-Name"], secret["AISearch-Key"], secret["AISearch-DataSource-Name"]);

                /* ************************* Create New */

                Console.WriteLine("\nCreate New...");

                await DataSource.Create(
                    CreateMethod,
                    AISearch_Name: secret["AISearch-Name"],
                    AISearch_Key: secret["AISearch-Key"],
                    AISearch_DataSource_Name: secret["AISearch-DataSource-Name"],
                    jsonDefinition: GetDefinition($"datasource_{DataSourceType}.json")
                    );

                await AzureSolutions.Helpers.AISearch.Index.Create(
                    CreateMethod,
                    AISearch_Name: secret["AISearch-Name"],
                    AISearch_Key: secret["AISearch-Key"],
                    AISearch_Index_Name: secret["AISearch-Index-Name"],
                    AISearch_Suggester_Name: secret["AISearch-Suggester-Name"],
                    AISearch_SemanticConfiguration_Name: secret["AISearch-SemanticConfiguration-Name"],
                    jsonDefinition: GetDefinition($"index_{DataSourceType}.json")
                    );

                await Skillset.Create(
                    CreateMethod,
                    AISearch_Name: secret["AISearch-Name"],
                    AISearch_Key: secret["AISearch-Key"],
                    AISearch_Skillset_Name: secret["AISearch-Skillset-Name"],
                    AIServices_Key: secret["AIServices-Key"],
                    jsonDefinition: GetDefinition($"skillset_{DataSourceType}.json")
                    );

                await Indexer.Create(
                    CreateMethod,
                    AISearch_Name: secret["AISearch-Name"],
                    AISearch_Key: secret["AISearch-Key"],
                    AISearch_Indexer_Name: secret["AISearch-Indexer-Name"],
                    AISearch_DataSource_Name: secret["AISearch-DataSource-Name"],
                    AISearch_Index_Name: secret["AISearch-Index-Name"],
                    AISearch_Skillset_Name: secret["AISearch-Skillset-Name"],
                    jsonDefinition: GetDefinition($"indexer_{DataSourceType}.json")
                    );

                await SynonymMap.Create(
                    AISearch_Name: secret["AISearch-Name"],
                    AISearch_Key: secret["AISearch-Key"],
                    AISearch_Index_Name: secret["AISearch-Index-Name"],
                    AISearch_SynonymMap_Name: secret["AISearch-SynonymMap-Name"]
                    );
            }
            catch (Exception ex) { Console.WriteLine($"Exception: {ex}\n"); }
        }

        private static string GetSecret(SecretClient keyvault, string secretName)
        {
            return keyvault.GetSecret(secretName).Value.Value ?? string.Empty;
        }

        private static string GetDefinition(string fileName)
        {
            string directory = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "..\\..\\..\\Definitions\\");

            string path = Path.GetFullPath(Path.Combine(directory, fileName));

            string jsonDefinition = File.ReadAllText(path);

            foreach (Match m in Regex.Matches(jsonDefinition, @"\{(.+?)\}").Cast<Match>())
            {
                string secretName = m.Groups[1].Value;
                if (keyvault != null) { jsonDefinition = jsonDefinition.Replace(oldValue: $"{{{secretName}}}", newValue: GetSecret(keyvault, secretName)); }
            }

            //Log.Write(message: $"JSON Definition...\n{jsonDefinition}");

            return jsonDefinition;
        }
    }
}
```

-----

### Step 4: Confirm Success

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

## Exercise 3: Source Control
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
<br>_Note: Only complete this exercise if you intend to include a custom skillset in the AI Search deployment app_

### Step 1: Create Project

Open Visual Studio and click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2b9ad75a-370d-4642-98a0-78600a43b772" width="600" title="Snipped: October 27, 2023" />

On the "**Create a new project**" page, search for and select "**Azure Functions**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/877158e8-9b6e-4c88-9346-307bc73095f6" width="600" title="Snipped: October 27, 2023" />

Complete the "**Configure your new project**" form and then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/39eeeab1-76b4-4325-8772-0c7aafa3c6d9" width="600" title="Snipped: October 27, 2023" />

Complete the "**Additional information**" form:

Prompt | Entry
:----- | :-----
**Functions worker** | **.NET 7.0 Isolated**
**Function** | **Http trigger**
**Use Azurite...** | Checked
**Authorization level** | Function

Click "**Create**".

-----

### Step 2: Code Function

Rename "Function1.cs" to "CustomSkillset.cs". When prompted "You are renaming a file...", click "**Yes**" to perform rename on all references.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/af1d52cb-5dbd-43b2-9198-ea390dede565" width="800" title="Snipped: October 27, 2023" />

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

Logic Explained:
* `using (var reader = new StreamReader(hrd.Body)...` parses request body JSON
* `foreach (var value in JsonDocument.Parse(request)` iterates through request values and packages a response
* `response.WriteAsJsonAsync(d)... return...` completes response

_Note: This is a template function ONLY... add additional logic based on your use case_

-----

### Step 3: Publish Function

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1bed8055-1d94-4c88-bde2-e1c6e557625e" width="800" title="Snipped: October 30, 2023" />

Right-click on your project in the Solution Explorer pane and select "**Publish**" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5a090114-5879-44d0-a07c-c78a006d92ff" width="600" title="Snipped: October 30, 2023" />

On the "**Publish**" popup, "**Target**" tab, select "**Azure**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/de97bbd3-ee15-4b1f-af0e-4299f14d6085" width="600" title="Snipped: October 30, 2023" />

On the "**Publish**" popup, "**Specific target**" tab, select "**Azure App Service (Windows)**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/af141980-d358-4879-b2c9-0277d90882c6" width="600" title="Snipped: October 30, 2023" />

On the **Publish** >> "**Select existing or...**" page, select your Function App and then click "**Finish**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8ed84d8e-1b06-4abf-8cf3-2a158c339d53" width="600" title="Snipped: October 30, 2023" />

On the **Publish profile creation progress** >> "**Finish**" page, click "**Close**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/85acb60f-bb87-43a2-9c4a-e4b5f4c12f11" width="800" title="Snipped: October 30, 2023" />

Back on the "...Publish" page, click **Publish**, allow time for processing, and confirm successful publication.

-----

### Step 4: Confirm Success

Navigate to your Azure Function App, then the "**CustomSkillset**" function, and then "**Code + Test**" in the "**Developer**" grouping of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a48bae96-a071-440e-912f-dff11e09da87" width="800" title="Snipped: October 31, 2023" />

Click "**Test/Run**" and on the resulting pop-out, "**Input**" tab, paste the following "**Body**" value:

```
{"values":[{"recordId":"0","data":{"text":"Lorem Ipsum"}}]}
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4d9fd06a-62d8-4ebe-9304-bdcc2834e860" width="800" title="Snipped: October 31, 2023" />

Click "**Run**".

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
