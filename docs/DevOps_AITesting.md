# DevOps: AI Testing (WORK IN PROGRESS)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/20ef5226-59b5-4876-b8b2-789373480cb4" width="1000" />

## Use Case
* "We want to automate testing of hundreds of prompts with various configurations of AI Search and OpenAI"

## Proposed Solution
* Stage AI Search Index
* Customize DevOps Test Case entity
* Automate Population of DevOps Test Cases

## Solution Requirements
* [**AI Search**](https://azure.microsoft.com/en-us/products/search) with dependency:
  * [**AI Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)
* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/) with organization and project
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured for .NET 7, with dependencies:
  * [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
  * [Application Service](https://learn.microsoft.com/en-us/azure/app-service/)
  * [Storage Account](Infrastructure_StorageAccount.md)
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * ConnectionString_BlobStorage
  * Key_AISearch
  * Key_AIServices
* [**Postman**](https://www.postman.com/product/workspaces/) (with Desktop Agent for localhost testing)
* [**Storage Account**](Infrastructure_StorageAccount.md) with a container and uploaded sample data {e.g., [IRS Tax Forms](https://www.irs.gov/forms-instructions)}
* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload and connected to your DevOps project

-----

## Exercise 1: Stage AI Search Index
In this exercise we will use the AI Search Development Kit (SDK) to create a data source, index, skillset, and indexer

### Step 1: Create Visual Studio Project
In this exercise, we will use the AI Search Development Kit (SDK) to create a data source, index, skillset, and indexer.

Open Visual Studio and click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/317959b5-dfd7-4c97-af0c-0578f9e89429" width="600" title="Snipped: October 10, 2023" />

On the "**Create a new project**" form, search for and select "**Console App**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/db8c2898-b607-4441-8b1e-4f4f3dbd56b4" width="600" title="Snipped: October 10, 2023" />

Complete the "**Configure your new project**" form, then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2408d491-ba3b-4ba7-9d84-02caf1dab54d" width="600" title="Snipped: October 10, 2023" />

Complete the "**Additional information**" form, then click "**Create**".

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/464851d5-30c0-4b72-87d5-cb95658d919d" width="800" title="Snipped: October 11, 2023" />

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0b3bc3a-e6af-47ff-8ed2-8c0d0340e44e" width="800" title="Snipped: October 11, 2023" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Azure.Search.Documents**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b56a92c-594a-4a18-afaa-24a4872ac73b" width="300" title="Snipped: October 11, 2023" />

When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d9906b3b-848d-4807-bc0c-441daf502865" width="800" title="Snipped: October 11, 2023" />

**Additional Packages**

Repeat this process for the following NuGet packages:

* Azure.Identity
* Azure.Security.KeyVault.Secrets

Close the "**NuGet - Solution**" tab.

-----

### Step 3: Code Application

Replace the default code on the "**Program.cs**" tab with the following C#:

```
using Azure;
using Azure.Identity;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;
using Azure.Security.KeyVault.Secrets;

public class Program
{
    public static void Main(string[] args)
    {

    }
}
```

**Names, URIs, and Keys**
The variables set in this section will be used to identify and create various resources.

Return to the "**Program.cs**" tab and add the following code to `Main`.

```
/* ************************* Names */

string nameBlobStorage_Container = "rchaplersc";
string nameAISearch_DataSource = "rchaplerss-datasource";
string nameAISearch_Index = "rchaplerss-index";
string nameAISearch_Indexer = "rchaplerss-indexer";
string nameAISearch_SemanticConfiguration = "rchaplerss-semanticconfiguration";
string nameAISearch_Skillset = "rchaplerss-skillset";
string nameAISearch_Suggester = "rchaplerss-suggester";

/* ************************* URIs */

var uriAISearch = new Uri($"https://rchaplerss.search.windows.net/");
var uriKeyVault = new Uri($"https://rchaplerk.vault.azure.net/");

/* ************************* Keys */

var sc = new SecretClient(uriKeyVault, new DefaultAzureCredential());

var ConnectionString_BlobStorage = sc.GetSecret("ConnectionString-BlobStorage").Value.Value.ToString() ?? string.Empty;
var Key_AISearch = sc.GetSecret("Key-AISearch").Value.Value.ToString() ?? string.Empty;
var Key_AIServices = sc.GetSecret("Key-AIServices").Value.Value.ToString() ?? string.Empty;
/* use of double ".Value" is a necessary oddity */
```

_Notes:_
* _Replace name values {e.g., `rchaplerss`} with values appropriate to your implementation_
* _Replace `AISEARCH_PRIMARYADMINKEY` with your AI Search API Key (and considering using a Key Vault)_

-----

**Clients**
The variables set in this section will be used to manage the AI Search resources.

Append the following code to the bottom of `Main`:

```
/* ************************* Clients */

var credential = new AzureKeyCredential(Key_AISearch);
var indexClient = new SearchIndexClient(uriAISearch, credential);
var indexerClient = new SearchIndexerClient(uriAISearch, credential);
```

Logic Explained:
* `var credential...` creates a new `AzureKeyCredential` object used to authenticate your requests to the AI Search service
* `var indexClient...` creates a new `SearchIndexClient` object used to manage (create, delete, update) indexes in your search service
* `var indexerClient...` creates a new `SearchIndexerClient` object used to manage (run, reset, delete) indexers in your search service

-----

**Data Source**
The logic in this section will create a AI Search Data Source.

Append the following code to the bottom of `Main`:

```
/* ************************* Data Source */

var sidsc = new SearchIndexerDataSourceConnection(
    name: nameAISearch_DataSource,
    type: SearchIndexerDataSourceType.AzureBlob,
    connectionString: ConnectionString_BlobStorage,
    container: new SearchIndexerDataContainer(nameBlobStorage_Container)
    );

indexerClient.DeleteIndexer(nameAISearch_Indexer); /* indexer must be deleted before data source connection */
indexerClient.DeleteDataSourceConnection(nameAISearch_DataSource);

indexerClient.CreateDataSourceConnection(sidsc);
```

Logic Explained:
* `var sidsc...` creates a new `SearchIndexerDataSourceConnection` object that represents a connection to a Blob Storage account
    * Replace `STORAGEACCOUNT_CONNECTIONSTRING` with your Storage Account Connection String (and considering using a Key Vault)
* `indexerClient.CreateDataSourceConnection...` creates a new data source connection using the `SearchIndexerDataSourceConnection` object

-----

**Index**
The logic in this section will create a AI Search Index.

Append the following code to the bottom of `Main`:

```
/* ************************* Index */

var index = new SearchIndex(nameAISearch_Index)
{
    Fields = {
        new SimpleField("id", SearchFieldDataType.String) { IsKey = true }, /* SimpleField = non-searchable */
        new SearchField("metadata_author", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_content_type", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_creation_date", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_language", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_content_type", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_file_extension", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_last_modified", SearchFieldDataType.DateTimeOffset) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_name", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_path", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_storage_size", SearchFieldDataType.Int64) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchField("metadata_title", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true },
        new SearchableField("content") { AnalyzerName = LexicalAnalyzerName.StandardLucene },
        new SearchableField("text", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
        new SearchableField("keyphrases", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
        new SearchField("myColumn", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
    },
    Suggesters = {
        new SearchSuggester(name: nameAISearch_Suggester, sourceFields: new[] { "metadata_storage_name" })
    }
};

indexClient.DeleteIndex(index);
indexClient.CreateIndex(index);
```

Logic Explained:
* `var index...` creates a new `SearchIndex` object that represents a search index in AI Search
  * `new SimpleField(...`, `new SearchField(...`,`new SearchableField(...` lines add fields the index
    * Each field represents a piece of data that can be searched, filtered, sorted, or faceted in the search index
* `new SearchableField("text"...` will be used by the OCR Skill
* `new SearchableField("keyphrases"...` will be used by the Key Phrases Extraction Skill
* `new SearchableField("myColumn"...` will be used by the WebAPI Skill
* `indexClient.DeleteIndex...` deletes any existing index with the same name using the `SearchIndex` object
* `indexClient.CreateIndex...` creates a new index using the `SearchIndex` object

-----

**Skillset**
The logic in this section will create a AI Search Skillset.

Append the following code to the bottom of `Main`:

```
/* ************************* Skillset */

var skills = new List<SearchIndexerSkill>
{
    new OcrSkill(
        inputs: new List<InputFieldMappingEntry>
        {
            new InputFieldMappingEntry("image") { Source = "/document/normalized_images/*" }
        },
        outputs: new List<OutputFieldMappingEntry>
        {
            new OutputFieldMappingEntry("text") { TargetName = "text" }
        }
    )
    {
        Context = "/document/normalized_images/*"
    },
    new KeyPhraseExtractionSkill(
        inputs: new List<InputFieldMappingEntry>
        {
            new InputFieldMappingEntry("text") { Source = "/document/content" }
        },
        outputs: new List<OutputFieldMappingEntry>
        {
            new OutputFieldMappingEntry("keyPhrases") { TargetName = "keyphrases" } /* camel-case required for source */
        }
    )
    {
        Context = "/document/content"
    },
    new WebApiSkill(
        inputs: new List<InputFieldMappingEntry>
        {
            new InputFieldMappingEntry("text") { Source = "/document/metadata_storage_file_extension" }
        },
        outputs: new List<OutputFieldMappingEntry>
        {
            new OutputFieldMappingEntry("myColumn") { TargetName = "myColumn" }
        },
        uri: "{FUNCTIONAPP_URL}"
    )
    {
        Context = "/document/content"
    }
};

var skillset = new SearchIndexerSkillset(nameAISearch_Skillset, skills)
{
    AIServicesAccount = new AIServicesAccountKey(key: Key_AIServices)
};

indexerClient.DeleteSkillset(skillset);
indexerClient.CreateSkillset(skillset);
```

Logic Explained:
* `var skills...` creates a new list of `SearchIndexerSkill` objects
  * Each `SearchIndexerSkill` represents a skill that can be used in an indexing pipeline
  * Inside the `{...}` block, an `OcrSkill` is added to the list
    * The `OcrSkill` is used to extract text from image files
      * The input to the skill is the `image` field, which comes from the `/document/normalized_images/*` path in your data source
      * The output of the skill is the `text` field
    * The `KeyPhraseExtractionSkill` evaluates unstructured text and returns a list of key phrases for each record
      * The input to the skill is `text`, which comes from the `/document/content` path in your data source
      * The output of the skill is the `keyPhrases` field
    * The `WebApiSkill` sends data to a custom web API endpoint (specified by `{FUNCTIONAPP_URL}`) and receives transformed data in return
      * The input is text, sourced from `/document/metadata_storage_file_extension` (used for simplicity)
      * The output from the web API is stored in a field named `myColumn`
* `var skillset...` creates a new `SearchIndexerSkillset` object that represents a skillset in AI Search
  * Inside the `{...}` block, a `AIServicesAccountKey` is set for the skillset
    * This key is used to authenticate your requests to the AI Services
* `indexerClient.DeleteSkillset...` deletes any existing skillset with the same name using the `SearchIndexerSkillset` object
* `indexerClient.CreateSkillset...` creates a new skillset using the `SearchIndexerSkillset` object

-----

**Indexer**
The logic in this section will create a AI Search Indexer.

Append the following code to the bottom of `Main`:

```
/* ************************* Indexer */

var indexer = new SearchIndexer(nameAISearch_Indexer, nameAISearch_DataSource, nameAISearch_Index)
{
    Parameters = new IndexingParameters()
    {
        IndexingParametersConfiguration = new IndexingParametersConfiguration()
        {
            ImageAction = BlobIndexerImageAction.GenerateNormalizedImagePerPage, /* re: OCR */
        }
    },
    SkillsetName = nameAISearch_Skillset,
    IsDisabled = true
};

indexer.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/normalized_images/*/text") { TargetFieldName = "text" });
indexer.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/content/keyphrases") { TargetFieldName = "keyphrases" });
indexer.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/content/myColumn") { TargetFieldName = "myColumn" });

indexerClient.DeleteIndexer(indexer);
indexerClient.CreateIndexer(indexer);
```

Logic Explained:
1. **SearchIndexer Creation**: A `SearchIndexer` named `indexer` is created with a specified indexer name, data source name, and index name. The indexer is configured with specific parameters and a skillset name
2. **IndexingParametersConfiguration**: The `IndexingParametersConfiguration` is set to `BlobIndexerImageAction.GenerateNormalizedImagePerPage`, which means the indexer will perform Optical Character Recognition (OCR) on images in blobs and generate a normalized image per page
3. **IsDisabled**: The indexer is initially disabled (`IsDisabled = true`) to prevent it from auto-processing after creation... re-enable by modifying Indexer >> "Indexer Definition (JSON)"
4. **OutputFieldMappings**: These mappings define how the output of a skill is mapped to a field in an index schema:
   <br>`text`... from `OcrSkill` and `/document/normalized_images/*/text`
   <br>`keyphrases`... from `KeyPhraseExtractionSkill` and `/document/content/keyphrases`
   <br>`myColumn`... from `WebApiSkill` and `/document/content/myColumn` (custom skillset)
6. **DeleteIndexer** and **CreateIndexer**: The existing indexer with the same name is deleted if it exists, and then the new indexer is created.

-----

### Step 4: Confirm Success

**Visual Studio Debug**

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/94ba39d5-0a55-4439-b2d1-188917087aa4" width="800" title="Snipped: October 11, 2023" />

Save your changes and then click "**Debug**" >> "**Start Debugging**" in the menubar.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d00e11c1-5611-49ed-b31e-228eee5428b2" width="600" title="Snipped: October 11, 2023" />

A "Microsoft Visual Studio Debug" window will open (as snipped above).

**AI Search Index**

Navigate to AI Search, then "**Indexes**" in the "**Search management**" grouping of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/60745fe0-95f6-40f7-a91b-94b10332c237" width="800" title="Snipped: October 12, 2023" />

You should see the index that you programmatically created {e.g., "rchaplerss-index"}. Click to open.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/302f8a28-e828-44de-b04c-cb9511bf19a2" width="800" title="Snipped: October 12, 2023" />

Click the "**Search**" button and review results.

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 2: Customize DevOps




LOREM IPSUM

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 3: Automate Test Case Creation
In this exercise, we will test prompts programmatically interact with OpenAI + AI Search index:

### Helper Class: DevOps

```
using Microsoft.TeamFoundation.WorkItemTracking.WebApi;
using Microsoft.TeamFoundation.WorkItemTracking.WebApi.Models;
using Microsoft.VisualStudio.Services.Common;
using Microsoft.VisualStudio.Services.WebApi;
using Microsoft.VisualStudio.Services.WebApi.Patch.Json;

namespace ConsoleApp1.Helpers
{
    internal class DevOps
    {
        KeyVault keyvault = new();

        private WorkItemTrackingHttpClient client;

        public DevOps()
        {

            client = new VssConnection(
                baseUrl: new Uri(keyvault.getSecret("DevOps-Url")),
                credentials: new VssBasicCredential(
                    userName: string.Empty,
                    password: keyvault.getSecret("DevOps-PersonalAccessToken") /* DevOps Personal Access Token with "Read, write, & manage" permissions */
                    )
                ).GetClient<WorkItemTrackingHttpClient>();            
        }

        public async Task<List<WorkItem>> getTestCases()
        {
            Wiql q = new()
            {
                Query = @"SELECT [System.Id], [System.State], [Custom.SystemMessage], [Custom.Prompt]
                            FROM workitems
                            WHERE [System.WorkItemType] = 'Test Case' AND [System.State] = 'Ready for OpenAI'"
            };

            WorkItemQueryResult wiqr = await client.QueryByWiqlAsync(q);

            List<WorkItem> testCases = new();

            if (wiqr != null && wiqr.WorkItems.Any())
            {
                int[] ids = wiqr.WorkItems.Select(item => item.Id).ToArray();
                testCases = await client.GetWorkItemsAsync(ids);
            }

            return testCases;
        }

        public void updateWorkItem(int id, string title, string prompt, string responseSimple, string responseSemantic, string steps)
        {
            Microsoft.VisualStudio.Services.WebApi.Patch.Json.JsonPatchDocument devopsWorkItem_Definition = new();
            addField(devopsWorkItem_Definition, "/fields/System.Title", title);
            addField(devopsWorkItem_Definition, "/fields/System.State", "Ready for Human");
            addField(devopsWorkItem_Definition, "/fields/Prompt", prompt);
            addField(devopsWorkItem_Definition, "/fields/Response_Simple", responseSimple);
            addField(devopsWorkItem_Definition, "/fields/Response_Simple_Ranking", "3-Neutral");
            addField(devopsWorkItem_Definition, "/fields/Response_Semantic", responseSemantic);
            addField(devopsWorkItem_Definition, "/fields/Response_Semantic_Ranking", "3-Neutral");
            addField(devopsWorkItem_Definition, "/fields/Microsoft.VSTS.TCM.Steps", steps);

            var workItem = client.UpdateWorkItemAsync(devopsWorkItem_Definition, id).Result;
        }

        private void addField(Microsoft.VisualStudio.Services.WebApi.Patch.Json.JsonPatchDocument devopsWorkItem_Definition, string path, string value)
        {
            devopsWorkItem_Definition.Add(
                new JsonPatchOperation
                {
                    Operation = Microsoft.VisualStudio.Services.WebApi.Patch.Operation.Add,
                    Path = path,
                    Value = value
                }
            );
        }

        public string defaultSteps()
        {
            return @"
            <steps id='0' last='3'>
                <step id='2' type='ValidateStep'>
                    <parameterizedString isformatted='true'>Review User Message (aka Prompt) and System Message</parameterizedString>
                    <parameterizedString isformatted='true'>Confirm that messages are well-formed</parameterizedString>
                    <description/>
                </step>
                <step id='3' type='ValidateStep'>
                    <parameterizedString isformatted='true'>Evaluate Responses</parameterizedString>
                    <parameterizedString isformatted='true'>Rank Responses from each model</parameterizedString>
                    <description/>
                </step>
            </steps>";
        }
    }
}
```

### Helper Class: KeyVault

```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace ConsoleApp1.Helpers
{
    internal class KeyVault
    {
        private SecretClient client;

        public KeyVault()
        {
            client = new SecretClient(
                vaultUri: new Uri("https://rchaplerkv.vault.azure.net/"),
                credential: new DefaultAzureCredential()
                );
        }

        public KeyVaultSecret getSecret(string secretName)
        {
            Response<KeyVaultSecret> responseKeyVaultSecret = client.GetSecret(secretName);
            return responseKeyVaultSecret.Value;
        }
    }
}
```

### Helper Class: OpenAI

```
using Azure;
using Azure.AI.OpenAI; /* pre-release NuGet Package: Azure.AI.OpenAI v1.0.0-beta.9 */

namespace ConsoleApp1.Helpers
{
    internal class OpenAI
    {
        KeyVault keyvault = new();

        private OpenAIClient client;

        public OpenAI()
        {
            client = new OpenAIClient(
                endpoint: new Uri(keyvault.getSecret("OpenAI-Endpoint")),
                keyCredential: new AzureKeyCredential(keyvault.getSecret("OpenAI-Key"))
            );
        }

        public async Task<string> Prompt(string queryType, string systemMessage, string userMessage)
        {
            AzureCognitiveSearchQueryType type;

            switch (queryType.ToLower())
            {
                case "simple": type = AzureCognitiveSearchQueryType.Simple; break;
                case "semantic": type = AzureCognitiveSearchQueryType.Semantic; break;
                default: throw new ArgumentException($"Invalid query type: {queryType}");
            }

            AzureCognitiveSearchChatExtensionConfiguration config = new()
            {
                SearchEndpoint = new Uri(keyvault.getSecret("AISearch-Url")),
                IndexName = keyvault.getSecret("AISearch-IndexName"),
                QueryType = type,
                ShouldRestrictResultScope = true,
                DocumentCount = 5,
                SemanticConfiguration = keyvault.getSecret("AISearch-SemanticConfiguration") /* Ignored when queryType != Semantic */
            };
            config.SetSearchKey(searchKey: keyvault.getSecret("AISearch-Key"));

            ChatCompletionsOptions cco = new()
            {
                DeploymentName = keyvault.getSecret("OpenAI-DeploymentName"),
                AzureExtensionsOptions = new AzureChatExtensionsOptions() { Extensions = { config } }
            };

            cco.Messages.Add(new ChatMessage(ChatRole.System, systemMessage));
            cco.Messages.Add(new ChatMessage(ChatRole.User, userMessage));

            var response = await client.GetChatCompletionsAsync(cco);
            return response.Value.Choices[0].Message.Content;
        }
    }
}
```

### Program.cs

```
using ConsoleApp1.Helpers;

DevOps devops = new(); OpenAI openai = new();

var testCases = await devops.getTestCases();

foreach (var testCase in testCases)
{
    int id;
    if (testCase.Id.HasValue) { id = testCase.Id.Value; } else { continue; }

    string? systemMessage = testCase.Fields["Custom.SystemMessage"].ToString();
    string? userMessage = testCase.Fields["Custom.Prompt"].ToString();

    if (systemMessage != null && userMessage != null)
    {
        string title = $"Prompt: '{string.Join(" ", userMessage.Split(' ').Take(5))}...'";
        Console.WriteLine(title);

        var responseSimple = await openai.Prompt(queryType: "Simple", systemMessage, userMessage);
        var responseSemantic = await openai.Prompt(queryType: "Semantic", systemMessage, userMessage);

        devops.updateWorkItem(
            id,
            title,
            userMessage,
            responseSimple,
            responseSemantic,
            steps: devops.defaultSteps()
            );
    }
}
```

Reference:
* https://learn.microsoft.com/en-us/azure/ai-services/openai/
* https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.OpenAI_1.0.0-beta.9/sdk/openai/Azure.AI.OpenAI/README.md
* https://learn.microsoft.com/en-us/azure/devops/boards/queries/import-work-items-from-csv?view=azure-devops

LOREM IPSUM

-----

**Congratulations... you have successfully completed all exercises**
