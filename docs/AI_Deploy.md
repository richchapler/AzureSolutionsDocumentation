# DevOps: AI Deployment

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/20ef5226-59b5-4876-b8b2-789373480cb4" width="1000" />

## Use Case
* "We have implemented OpenAI with AI Search and are rapidly iterating through enhancements to the index"
  * "We have found that creating and updating the AI Search index with the Azure Portal tools can be difficult... we want a simpler, faster, more consistent experience"
  * "We want to capture our AI Search index creation process in our GitHub or DevOps repository"
* "Both semantic and vector should be activated"
  * "Codified AI Search skills must include: OCR, Key Phrase Extraction, and a Custom Skillset... also, for vectorization, Split and OpenAI Embedding"
  * "Because a lot of the code for vectorization is not yet surfaced in the Search Development Kit (SDK), we must also be able to leverage Application Programming Interface (API)"
* "To optimize response quality, we should include synonyms for country codes"
* "All secrets must be stored in Key Vault"

## Proposed Solution
* Custom Skillset API: Use a Function App to instantiate a simple API for use with AI Search, custom skillset
* Deployment Application: Use a combination of API and SDK to create data source, index, skillset, indexer, and synonym map
* Source Control: Create a pull request in a DevOps repo

## Solution Requirements
* [**AI Search**](https://azure.microsoft.com/en-us/products/search)

* [**AI Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)

* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/) with organization and project
 
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  - Subscription-Id  
  - ResourceGroup-Name  
  - AISearch-Name  
  - AISearch-Key  
  - AISearch-DataSource-Name  
  - AISearch-DataSource-Blob-ConnectionString  
  - AISearch-DataSource-Blob-Container  
  - AISearch-DataSource-SQL-Server-Name  
  - AISearch-DataSource-SQL-Server-User  
  - AISearch-DataSource-SQL-Server-Password  
  - AISearch-DataSource-SQL-Database-Name  
  - AISearch-DataSource-SQL-Table  
  - AISearch-DataSource-SQL-Query  
  - AISearch-Index-Name  
  - AISearch-Skillset-Name  
  - AISearch-Indexer-Name  
  - AISearch-Suggester-Name  
  - AISearch-SynonymMap-Name  
  - AISearch-SemanticConfiguration-Name  
  - AISearch-VectorProfile-Name  
  - AISearch-VectorAlgorithm-Name  
  - AISearch-Vectorizer-Name  
  - AIServices-Name  
  - AIServices-Key  
  - OpenAI-Name  
  - OpenAI-Key  
  - OpenAI-Deployment-Embedding  
 
* [**OpenAI**](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview) with "text-embedding-ada-002" [deployment model](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource) 
 
* [**Postman**](https://www.postman.com/product/workspaces/) (with Desktop Agent for localhost testing)
 
* [**Storage Account**](Infrastructure_StorageAccount.md) with a container and uploaded sample data {e.g., [IRS Tax Forms](https://www.irs.gov/forms-instructions)}
 
* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload and connected to your DevOps project

<br>
If you intend to prepare a custom skillset, also instantiate:

* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured for .NET 7, with dependencies:
  * [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
  * [Application Service](https://learn.microsoft.com/en-us/azure/app-service/)
  * [Storage Account](Infrastructure_StorageAccount.md)

-----

## Exercise 1: Custom Skillset API
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

```
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

```
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

-----
-----

## Exercise 2: Deployment Application
In this exercise, we will prepare to use a combination of API and SDK to create data source, index, skillset, indexer, and synonym map.

### Step 1: Create Visual Studio Project

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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/00248893-859f-4db5-96b4-dcbbf5cbc752" width="800" title="Snipped: May 29, 2024" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**AzureSolutions.Helpers**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".
<br>When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.
<br>When complete, close the "**NuGet - Solution**" tab.

-----

### Step 3: Helper Classes

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

#### KeyVault.cs

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1a2987e0-2e70-4652-8041-1625dc27cc37" width="600" title="Snipped: January 18, 2024" />

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "KeyVault.cs" on the resulting popup and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0c414788-dc12-43ae-b15c-79c4a8751a99" width="800" title="Snipped: January 18, 2024" />

Replace the default code with:

```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace DevOps_AIDeployment.Helpers
{
    internal class KeyVault
    {
        private SecretClient client;

        public KeyVault()
        {
            client = new SecretClient(
                vaultUri: new Uri($"https://{KeyVaultName}.vault.azure.net/"),
                credential: new DefaultAzureCredential()
                );
        }

        public string getSecret(string secretName)
        {
            Response<KeyVaultSecret> responseKeyVaultSecret = client.GetSecret(secretName);
            return responseKeyVaultSecret.Value.Value;
        }
    }
}
```

_Note:_
* _Replace `{KeyVaultName}` with the name of your Key Vault_

#### AISearch.cs

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "AISearch.cs" on the resulting popup and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2e76e089-db09-4c7d-ad77-4b6e2f461f87" width="800" title="Snipped: January 18, 2024" />

Replace the default code with:

```
using Azure;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;
using System.Text;
using System.Text.Json;
using System.Text.RegularExpressions;

namespace DevOps_AIDeployment.Helpers
{
    internal class AISearch
    {
        KeyVault keyvault = new();

        public string prefix { get; set; }
        public SearchIndexClient clientIndex { get; private set; }
        public SearchIndexerClient clientIndexer { get; private set; }

        public AISearch()
        {
            prefix = keyvault.getSecret("ResourceGroup-Name"); /* common start for each resource name */

            clientIndex = new SearchIndexClient(
                endpoint: new Uri($"https://{prefix + "ss"}.search.windows.net"),
                credential: new AzureKeyCredential(keyvault.getSecret("AISearch-Key"))
                );

            clientIndexer = new SearchIndexerClient(
                endpoint: new Uri($"https://{prefix + "ss"}.search.windows.net"),
                credential: new AzureKeyCredential(keyvault.getSecret("AISearch-Key"))
                );
        }

        public async Task createResource_API(string type)
        {
            Console.WriteLine($"Creating {prefix}ss-{type} using REST API");

            string jsonDefinition = System.IO.File.ReadAllText(path: Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, $"..\\..\\..\\Definitions\\{type}.json")));

            var matches = Regex.Matches(jsonDefinition, @"\{(.+?)\}");

            foreach (Match match in matches)
            {
                string secretName = match.Groups[1].Value;

                jsonDefinition = jsonDefinition.Replace($"{{{secretName}}}", keyvault.getSecret(secretName));
            }

            HttpClient client = new();
            client.DefaultRequestHeaders.Add("api-key", keyvault.getSecret("AISearch-Key"));

            string typePlural = type == "index" ? "indexes" : $"{type}s";

            var response = await client.PutAsync(
                requestUri: $"https://{prefix + "ss"}.search.windows.net/{typePlural}/{prefix + "ss" + $"-{type}"}?api-version=2023-10-01-Preview",
                content: new StringContent(jsonDefinition, Encoding.UTF8, "application/json")
                );

            if (!response.IsSuccessStatusCode)
            {
                var errorDetails = await response.Content.ReadAsStringAsync();
                Console.WriteLine($"Failed to create {type}./nStatus code: {response.StatusCode}./nReason: {response.ReasonPhrase}/nError details: {errorDetails}/n");
            }
        }

        public async Task createDataSource_SDK()
        {
            Console.WriteLine("Creating " + prefix + "ss" + "-datasource using .NET SDK");

            var sidsc = new SearchIndexerDataSourceConnection(
                prefix + "ss-datasource",
                type: SearchIndexerDataSourceType.AzureBlob,
                connectionString: keyvault.getSecret("Storage-ConnectionString"),
                container: new SearchIndexerDataContainer(name: keyvault.getSecret("Storage-ContainerName"))
            );
            /* 20240118: AI Search does not support blob deletion detection directly */

            await clientIndexer.CreateDataSourceConnectionAsync(sidsc);
        }

        public async Task createIndex_SDK()
        {
            Console.WriteLine("Creating " + prefix + "ss" + "-index using .NET SDK");

            SearchIndex index = new(prefix + "ss" + "-index")
            {
                /* ************************* Fields */
                Fields = {
                    new SimpleField("id", SearchFieldDataType.String) { IsKey = true }
                        /* "id" field captures required, unique identifier generated by the indexer */

                     /* ************************* Fields: Standard Blob Storage */

                    , new SearchField("metadata_author", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_content_type", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_creation_date", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_language", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_content_type", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_file_extension", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_last_modified", SearchFieldDataType.DateTimeOffset) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_name", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_path", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_storage_size", SearchFieldDataType.Int64) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchField("metadata_title", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    , new SearchableField("content") { AnalyzerName = LexicalAnalyzerName.StandardLucene } 

                     /* ************************* Fields: Skillset Additions */

                    , new SearchableField("keyphrases_content", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene }
                        /* "keyphrases_content" captures key phrases from the "content" field as generated by the KeyPhraseExtractionSkill */

                    , new SearchableField("text", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene }
                        /* "text" captures text extracted from image by the OcrSkill */

                    , new SearchableField("keyphrases_text", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene }
                        /* "keyphrases_text" captures key phrases from the "text" field as generated by the KeyPhraseExtractionSkill */

                    /* ************************* Fields: Vector Additions (not possible with .NET SDK as of Jan 6 2024) */

                    ////, new SimpleField("chunk_id", SearchFieldDataType.String) { /* IsKey = true,*/ IsFilterable = true, IsSortable = true }
                    ////    /* "chunk_id" captures unique identifier for each chunk of data and MIGHT replace key field in the index */

                    ////, new SimpleField("parent_id", SearchFieldDataType.String) { IsFilterable = true, IsSortable = true, IsFacetable = true }
                    ////    /* "parent_id" contains the identifier of the original document from which the chunk was derived */

                    ////, new SearchableField("chunk") { IsFilterable = true, IsSortable = true, IsFacetable = true }
                    ////    /* "chunk" contains the actual chunk of data that has been vectorized */

                    ////, new SearchField("vector", SearchFieldDataType.Collection(SearchFieldDataType.Single))
                    ////{
                    ////    IsSearchable = true
                    ////    , IsFilterable = false
                    ////    , IsSortable = false
                    ////    , IsFacetable = false
                    ////    , VectorSearchDimensions = 1536
                    ////    , VectorSearchProfileName = prefix + "ss" + "-vectorprofile"
                    ////}
                    ////    /* "vector" contains the vector representation of the chunks */

                    /* ************************* Custom Skillset Additions */

                    //, new SearchField("customColumn", SearchFieldDataType.String) { IsFacetable = true, IsFilterable = true, IsSortable = true }
                    /* "customColumn" field captures data generated by a custom skillset {i.e., WebApiSkill} */
                },

                /* ************************* Suggesters */
                Suggesters = { new SearchSuggester(name: prefix + "ss" + "-suggester", sourceFields: new[] { "metadata_storage_name" }) },
                /* Suggesters enable "type-ahead" experience, either auto-complete or suggestions */

                /* ************************* Semantic Search */
                SemanticSearch = new SemanticSearch
                {
                    Configurations =
                    {
                        new SemanticConfiguration(
                            name: prefix + "ss" + "-semanticconfiguration",
                            prioritizedFields: new SemanticPrioritizedFields()
                            {
                                TitleField = new SemanticField(fieldName: "metadata_title" ),
                                ContentFields =  {    new SemanticField(fieldName: "content"),  new SemanticField(fieldName: "text") },
                                KeywordsFields = { new SemanticField(fieldName: "metadata_title") }
                            }
                        )
                    }
                },

                /* ************************* Vector Search */

                //VectorSearch = new()
                //{
                //    Profiles =
                //    {
                //        new VectorSearchProfile( name:prefix + "ss" + "-vectorprofile",  algorithmConfigurationName: prefix + "ss" + "-vectoralgorithm" )
                //        {
                //            //Vectorizer = prefix + "ss" + "-vectorizer"
                //        }
                //    },
                //    Algorithms =
                //    {
                //        new HnswAlgorithmConfiguration(name: prefix + "ss" + "-vectoralgorithm")
                //        /* Instantiates with default values: Bi-direction link count 4, efConstruction 400, efSearch 500, Similarity metric cosine */
                //    },
                //    Vectorizers =
                //    {
                //        new AzureOpenAiVectorizer(name + "-vectorizer", "https://{ResourceGroup-Name}ai.openai.azure.com", "{ResourceGroup-Name}ai-ada2", "34b586dacbfb4115b15d5c167438a11c"),
                //    }
                //}
            };

            await clientIndex.CreateIndexAsync(index);
        }

        public async Task addSynonyms_SDK()
        {
            Console.WriteLine("Adding Synonyms to " + prefix + "ss" + "-index using .NET SDK");

            string j = System.IO.File.ReadAllText(Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, $"..\\..\\..\\Definitions\\synonyms.json")));

            var options = new JsonSerializerOptions { PropertyNameCaseInsensitive = true, };

            var d = JsonSerializer.Deserialize<Dictionary<string, List<string>>>(j, options);

            string synonyms = "";
            if (d != null) { foreach (var synonym in d) { synonyms += $"{synonym.Key}, {string.Join(", ", synonym.Value)}\n"; } }

            var sm = new SynonymMap(keyvault.getSecret("AISearch-Name") + "-synonymmap", synonyms);

            await clientIndex.CreateOrUpdateSynonymMapAsync(sm);

            var index = clientIndex.GetIndex(keyvault.getSecret("AISearch-IndexName"));

            var field = clientIndex.GetIndex(keyvault.getSecret("AISearch-IndexName")).Value.Fields.FirstOrDefault(f => f.Name == "merged_content");
            if (field != null) { field.SynonymMapNames.Add(sm.Name); }

            await clientIndex.CreateOrUpdateIndexAsync(index);
        }

        public async Task createSkillset_SDK()
        {
            Console.WriteLine("Creating " + prefix + "ss" + "-skillset using .NET SDK");

            var skills = new List<SearchIndexerSkill>
            {
                    /* ************************* Optical Character Recognition (OCR) */

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

                    /* ************************* Key Phrases */

                    new KeyPhraseExtractionSkill(
                        inputs: new List<InputFieldMappingEntry>
                        {
                            new InputFieldMappingEntry("text") { Source = "/document/content" } /* source: blob indexer */
                        },
                        outputs: new List<OutputFieldMappingEntry>
                        {
                            new OutputFieldMappingEntry("keyPhrases") { TargetName = "keyphrases_content" }
                        }
                    )
                    {
                        Context = "/document/content"
                    },

                    new KeyPhraseExtractionSkill(
                        inputs: new List<InputFieldMappingEntry>
                        {
                            new InputFieldMappingEntry("text") { Source = "/document/normalized_images/*/text" } /* source: OcrSkill */
                        },
                        outputs: new List<OutputFieldMappingEntry>
                        {
                            new OutputFieldMappingEntry("keyPhrases") { TargetName = "keyphrases_text" }
                        }
                    )
                    {
                        Context = "/document/normalized_images/*"
                    },

                    /* ************************* Custom (WebAPI) */

                    //new WebApiSkill(
                    //    inputs: new List<InputFieldMappingEntry>
                    //    {
                    //        new InputFieldMappingEntry("text") { Source = "/document/metadata_storage_file_extension" }
                    //    },
                    //    outputs: new List<OutputFieldMappingEntry>
                    //    {
                    //        new OutputFieldMappingEntry("customColumn") { TargetName = "customColumn" }
                    //    },
                    //    uri: "https://{ResourceGroup-Name}fa.azurewebsites.net/api/CustomSkillset?code=Oi4QJyFaVnDKBlhK9PKaIQ90MIi1Q_kJ10EWwTrZk5oiAzFulZd4jg=="
                    //)
                    //{
                    //    Context = "/document/content"
                    //}
            };

            SearchIndexerSkillset skillset = new(name: prefix + "ss" + "-skillset", skills)
            {
                CognitiveServicesAccount = new CognitiveServicesAccountKey(key: keyvault.getSecret("AIServices-Key"))
            };

            await clientIndexer.CreateSkillsetAsync(skillset);
        }

        public async Task createIndexer_SDK()
        {
            Console.WriteLine("Creating " + prefix + "ss" + "-indexer using .NET SDK");

            var si = new SearchIndexer(name: prefix + "ss" + "-indexer", dataSourceName: prefix + "ss" + "-datasource", targetIndexName: prefix + "ss" + "-index")
            {
                Parameters = new IndexingParameters()
                {
                    IndexingParametersConfiguration = new IndexingParametersConfiguration()
                    {
                        ImageAction = BlobIndexerImageAction.GenerateNormalizedImagePerPage, /* re: OCR of multi-page PDFs */
                    }
                },
                SkillsetName = prefix + "ss" + "-skillset",
                //IsDisabled = true /* Prevents indexer from auto-processing after creation */
            };

            /* ************************* Optical Character Recognition (OcrSkill) */
            si.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/normalized_images/*/text") { TargetFieldName = "text" });

            /* ************************* Key Phrase Extraction (KeyPhraseExtractionSkill) */
            si.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/content/keyphrases") { TargetFieldName = "keyphrases" });

            /* ************************* Custom (WebApiSkill) */
            //indexer.OutputFieldMappings.Add(new FieldMapping(sourceFieldName: "/document/content/customColumn") { TargetFieldName = "customColumn" });

            await clientIndexer.CreateIndexerAsync(si);
        }

        public async Task deleteExisting()
        {
            Console.WriteLine("Attempting deletion of " + prefix + "ss" + "-indexer using .NET SDK");
            try
            {
                var existing = await clientIndexer.GetIndexerAsync(prefix + "ss" + "-indexer");
                await clientIndexer.DeleteIndexerAsync(prefix + "ss" + "-indexer");
            }
            catch (Exception) { }

            Console.WriteLine("Attempting deletion of " + prefix + "ss" + "-skillset using .NET SDK");
            try
            {
                var existing = await clientIndexer.GetSkillsetAsync(prefix + "ss" + "-skillset");
                await clientIndexer.DeleteSkillsetAsync(prefix + "ss" + "-skillset");
            }
            catch (Exception) { }

            Console.WriteLine("Attempting deletion of " + prefix + "ss" + "-index using .NET SDK");
            try
            {
                var existing = await clientIndex.GetIndexAsync(prefix + "ss" + "-index");
                await clientIndex.DeleteIndexAsync(prefix + "ss" + "-index");
            }
            catch (Exception) { }

            Console.WriteLine("Attempting deletion of " + prefix + "ss" + "-datasource using .NET SDK");
            try
            {
                var existing = await clientIndexer.GetDataSourceConnectionAsync(prefix + "ss" + "-datasource");
                await clientIndexer.DeleteDataSourceConnectionAsync(prefix + "ss" + "-datasource");
            }
            catch (Exception) { }
        }
    }
}
```

_Notes:_
* _Uncomment `new WebApiSkill(...` lines if you are including a custom skillset_
* _Uncomment `IsDisabled = true` to stop the indexer from running immediately after creation_

-----

### Step 4: Definitions

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Definitions".

#### index.json

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fe8abcf5-dd66-4357-81bd-b58f5ceabd33" width="600" title="Snipped: January 18, 2024" />

Right-click on the "Definitions" folder, select "Add" >> "New Item" from the resulting dropdowns, search for and select "JSON", enter name "index.json" on the resulting popup and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bb13c1e4-9834-4759-9410-c6c33ea9b910" width="800" title="Snipped: January 18, 2024" />

Replace the default code with:

```
{
  "name": "{ResourceGroup-Name}ss-index",
  "defaultScoringProfile": "",
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": true,
      "analyzer": "keyword"
    },
    {
      "name": "metadata_title",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_storage_name",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_storage_path",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_storage_content_type",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_storage_file_extension",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_storage_size",
      "type": "Edm.Int64",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
    },
    {
      "name": "metadata_author",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_language",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "metadata_creation_date",
      "type": "Edm.DateTimeOffset",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
    },
    {
      "name": "metadata_storage_last_modified",
      "type": "Edm.DateTimeOffset",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
    },
    {
      "name": "content",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "text",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "layoutText",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "merged_content",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "keyphrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene"
    },
    {
      "name": "parent_id",
      "type": "Edm.String",
      "searchable": true,
      "filterable": true,
      "retrievable": true,
      "sortable": true,
      "facetable": true,
      "key": false
    },
    {
      "name": "chunk",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
    },
    {
      "name": "vector",
      "type": "Collection(Edm.Single)",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "dimensions": 1536,
      "vectorSearchProfile": "{ResourceGroup-Name}ss-vectorprofile"
    }
  ],
  "similarity": {
    "@odata.type": "#Microsoft.Azure.Search.BM25Similarity"
  },
  "semantic": {
    "defaultConfiguration": "{AISearch-SemanticConfigurationName}",
    "configurations": [
      {
        "name": "{AISearch-SemanticConfigurationName}",
        "prioritizedFields": {
          "titleField": {
            "fieldName": "metadata_title"
          },
          "prioritizedContentFields": [
            {
              "fieldName": "merged_content"
            }
          ],
          "prioritizedKeywordsFields": [
            {
              "fieldName": "keyphrases"
            }
          ]
        }
      }
    ]
  },
  "vectorSearch": {
    "algorithms": [
      {
        "name": "{ResourceGroup-Name}ss-vectoralgorithm",
        "kind": "hnsw",
        "hnswParameters": {
          "metric": "cosine",
          "m": 4,
          "efConstruction": 400,
          "efSearch": 500
        }
      }
    ],
    "profiles": [
      {
        "name": "{ResourceGroup-Name}ss-vectorprofile",
        "algorithm": "{ResourceGroup-Name}ss-vectoralgorithm",
        "vectorizer": "{ResourceGroup-Name}ss-vectorizer"
      }
    ],
    "vectorizers": [
      {
        "name": "{ResourceGroup-Name}ss-vectorizer",
        "kind": "azureOpenAI",
        "azureOpenAIParameters": {
          "resourceUri": "https://{OpenAI-Name}.openai.azure.com",
          "deploymentId": "{OpenAI-Name}-ada2",
          "apiKey": "{OpenAI-Key}",
          "authIdentity": null
        }
      }
    ]
  }
}
```

_Notes:_
* _The included fields are standard for: 1) blob file indexing, 2) OCR and related merge, 3) Key Phrase Extraction, and 4) vectorization_
* _Parameters will be wrapped in curly brackets {e.g., `{ResourceGroup-Name}`} and replaced with values in the related code_
* _`{ResourceGroup-Name}` is used as a prefix throughout the code... each created resource will start with `{ResourceGroup-Name}`, and follow with an acronym describing the resource type (as well as any other related identifying information)_

#### synonyms.json

Right-click on the "Definitions" folder, select "Add" >> "New Item" from the resulting dropdowns, search for and select "JSON", enter name "synonyms.json" on the resulting popup and then click "Add".

Replace the default code with:

```
{
  "United States": [ "US", "USA" ],
  "Mexico": [ "MX" ],
  "Canada": [ "CA" ]
}
```

#### skillset.json

Right-click on the "Definitions" folder, select "Add" >> "New Item" from the resulting dropdowns, search for and select "JSON", enter name "skillset.json" on the resulting popup and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7cc8e7e4-f577-4fb7-af39-a13f9331ae7d" width="800" title="Snipped: January 18, 2024" />

Replace the default code with:

```
{
  "name": "{ResourceGroup-Name}ss-skillset",
  "skills": [
    {
      "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
      "name": "#1",
      "description": "Step #1: Extract text from image",
      "context": "/document/normalized_images/*",
      "lineEnding": "Space",
      "defaultLanguageCode": "en",
      "detectOrientation": true,
      "inputs": [
        {
          "name": "image",
          "source": "/document/normalized_images/*"
        }
      ],
      "outputs": [
        {
          "name": "text",
          "targetName": "text"
        },
        {
          "name": "layoutText",
          "targetName": "layoutText"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "name": "#2",
      "description": "Step #2: Merge content, text, and contentOffset",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " ",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        },
        {
          "name": "itemsToInsert",
          "source": "/document/normalized_images/*/text"
        },
        {
          "name": "offsets",
          "source": "/document/normalized_images/*/contentOffset"
        }
      ],
      "outputs": [
        {
          "name": "mergedText",
          "targetName": "merged_content"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
      "name": "#3",
      "description": "Step #3: Extract keyphrases from merged_content",
      "context": "/document/merged_content",
      "inputs": [
        {
          "name": "text",
          "source": "/document/merged_content"
        }
      ],
      "outputs": [
        {
          "name": "keyPhrases",
          "targetName": "keyphrases"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "description": "Step #4: Split Chunks, merged_content > pages",
      "context": "/document",
      "defaultLanguageCode": "en",
      "textSplitMode": "pages",
      "maximumPageLength": 2000,
      "pageOverlapLength": 500,
      "maximumPagesToTake": 0,
      "inputs": [
        {
          "name": "text",
          "source": "/document/merged_content"
        }
      ],
      "outputs": [
        {
          "name": "textItems",
          "targetName": "pages"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.AzureOpenAIEmbeddingSkill",
      "description": "Step #5: Generate Embeddings, pages > vector",
      "context": "/document/pages/*",
      "resourceUri": "https://{OpenAI-Name}.openai.azure.com",
      "apiKey": "{OpenAI-Key}",
      "deploymentId": "{OpenAI-Name}-ada2",
      "inputs": [
        {
          "name": "text",
          "source": "/document/pages/*"
        }
      ],
      "outputs": [
        {
          "name": "embedding",
          "targetName": "vector"
        }
      ]
    }
  ],
  "cognitiveServices": {
    "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
    "description": "/subscriptions/{Subscription-Id}/resourceGroups/{ResourceGroup-Name}/providers/Microsoft.CognitiveServices/accounts/{ResourceGroup-Name}ais",
    "key": "{AIServices-Key}"
  },
  "indexProjections": {
    "selectors": [
      {
        "targetIndexName": "{ResourceGroup-Name}ss-index",
        "parentKeyFieldName": "parent_id",
        "sourceContext": "/document/pages/*",
        "mappings": [
          {
            "name": "metadata_author",
            "source": "/document/metadata_author"
          },
          {
            "name": "metadata_creation_date",
            "source": "/document/metadata_creation_date"
          },
          {
            "name": "metadata_language",
            "source": "/document/metadata_language"
          },
          {
            "name": "metadata_storage_content_type",
            "source": "/document/metadata_storage_content_type"
          },
          {
            "name": "metadata_storage_file_extension",
            "source": "/document/metadata_storage_file_extension"
          },
          {
            "name": "metadata_storage_last_modified",
            "source": "/document/metadata_storage_last_modified"
          },
          {
            "name": "metadata_storage_name",
            "source": "/document/metadata_storage_name"
          },
          {
            "name": "metadata_storage_path",
            "source": "/document/metadata_storage_path"
          },
          {
            "name": "metadata_storage_size",
            "source": "/document/metadata_storage_size"
          },
          {
            "name": "metadata_title",
            "source": "/document/metadata_title"
          },
          {
            "name": "content",
            "source": "/document/content"
          },
          {
            "name": "text",
            "source": "/document/normalized_images/*/text"
          },
          {
            "name": "layoutText",
            "source": "/document/normalized_images/*/layoutText"
          },
          {
            "name": "merged_content",
            "source": "/document/merged_content"
          },
          {
            "name": "keyphrases",
            "source": "/document/merged_content/keyphrases"
          },
          {
            "name": "chunk",
            "source": "/document/pages/*"
          },
          {
            "name": "vector",
            "source": "/document/pages/*/vector"
          }
        ]
      }
    ],
    "parameters": {
      "projectionMode": "skipIndexingParentDocuments"
    }
  }
}
```

_Notes:_
* _Healthy debate about the ordering or keyword generation re: vectorization chunking is both necessary and reasonable... diagram below starts to describe the necessary conversation_

  <img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1bd79083-5713-4ef2-ab9f-c1c7db4e62f6" width="800" title="Snipped: January 18, 2024" />

#### indexer.json

Right-click on the "Definitions" folder, select "Add" >> "New Item" from the resulting dropdowns, search for and select "JSON", enter name "indexer.json" on the resulting popup and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/824d69e8-8c1c-43c5-af39-45656d96dcd8" width="800" title="Snipped: January 18, 2024" />

Replace the default code with:

```
{
  "name": "{ResourceGroup-Name}ss-indexer",
  "description": "",
  "dataSourceName": "{ResourceGroup-Name}ss-datasource",
  "skillsetName": "{ResourceGroup-Name}ss-skillset",
  "targetIndexName": "{ResourceGroup-Name}ss-index",
  "disabled": false,
  "schedule": null,
  "parameters": {
    "batchSize": null,
    "maxFailedItems": 0,
    "maxFailedItemsPerBatch": 0,
    "base64EncodeKeys": null,
    "configuration": {
      "imageAction": "generateNormalizedImagePerPage",
      "dataToExtract": "contentAndMetadata",
      "parsingMode": "default"
    }
  },
  "outputFieldMappings": [
    {
      "sourceFieldName": "/document/normalized_images/*/text",
      "targetFieldName": "text"
    },
    {
      "sourceFieldName": "/document/normalized_images/*/layoutText",
      "targetFieldName": "layoutText"
    },
    {
      "sourceFieldName": "/document/merged_content",
      "targetFieldName": "merged_content"
    },
    {
      "sourceFieldName": "/document/merged_content/keyphrases",
      "targetFieldName": "keyphrases"
    }
  ],
  "cache": null,
  "encryptionKey": null
}
```

-----

### Step 5: `Program.cs`

Replace the default code on the "**Program.cs**" tab with the following C#:

```
using DevOps_AIDeployment.Helpers;

public class Program
{
    public static async Task Main(string[] args)
    {
        AISearch aisearch = new();

        await aisearch.deleteExisting();

        await aisearch.createDataSource_SDK();

        //await aisearch.createIndex_SDK();
        await aisearch.createResource_API("index");

        await aisearch.addSynonyms_SDK();

        //await aisearch.createSkillset_SDK();
        await aisearch.createResource_API("skillset");

        //await aisearch.createIndexer_SDK();
        await aisearch.createResource_API("indexer");
    }
}
```

_Notes:_
* _API handling was added to address the fact that, as of Jan 2024, Vectorization had not been completely surfaced in the `Azure.Search.Documents` library_
* _Calls included for both `...SDK` and `...API` versions... comment, as needed, to achieve goals_

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

#### SynonymMaps

Use Postman to test a GET call to https://rchaplerss.search.windows.net/synonymmaps?api-version=2023-11-01 with Headers "api-key" and "Content-Type" "application/json"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a7e51dac-9516-47f9-9f5c-9903f565fe45" width="800" title="Snipped: January 28, 2024" />

You can expect response JSON like:

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/$metadata#synonymmaps",
  "value": [
    {
      "@odata.etag": "\"0x8DC1DEB5153ECD1\"",
      "name": "rchaplerss-index-synonymmap",
      "format": "solr",
      "synonyms": "United States, US, USA\nMexico, MX\nCanada, CA\n",
      "encryptionKey": null
    }
  ]
}
```

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

**Congratulations... you have successfully completed all exercises**
