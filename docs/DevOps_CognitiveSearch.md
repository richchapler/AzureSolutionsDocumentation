# DevOps: Cognitive Search

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/20ef5226-59b5-4876-b8b2-789373480cb4" width="1000" />

## Use Case
* "We have implemented OpenAI with Cognitive Search and are rapidly iterating through enhancements to the index"
* "Creating and updating the Cognitive Search index can be difficult... we want a simpler, faster, more consistent experience"
* "We want to capture our Cogniive Search index creation process in our DevOps repo"

## Proposed Solution
* Develop App: Use the Cognitive Search Development Kit (SDK) to create a data source, index, skillset, and indexer
* Source Control: Create a pull request in a DevOps repo

## Solution Requirements
* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/) with organization and project
* [**Storage Account**](Infrastructure_StorageAccount.md) with a container and uploaded sample data {e.g., [IRS Tax Forms](https://www.irs.gov/forms-instructions)}
* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/) connected to your DevOps project

-----

## Exercise 1: Develop App
In this exercise, we will use the Cognitive Search Development Kit (SDK) to create a data source, index, skillset, and indexer.

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

Replace the default code on the "**Program.cs**" tab with the following C#:

```
using Azure;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;

public class Program
{
    public static void Main(string[] args)
    {

    }
}
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/464851d5-30c0-4b72-87d5-cb95658d919d" width="800" title="Snipped: October 11, 2023" />

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0b3bc3a-e6af-47ff-8ed2-8c0d0340e44e" width="800" title="Snipped: October 11, 2023" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Azure.Search.Documents**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b56a92c-594a-4a18-afaa-24a4872ac73b" width="300" title="Snipped: October 11, 2023" />

When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d9906b3b-848d-4807-bc0c-441daf502865" width="800" title="Snipped: October 11, 2023" />

Close the "**Nuget - Solution**" tab.

-----

### Step 3: Complete Code

#### Names and Keys
The variables set in this section will be used to identify and create various Cognitive Search resources.

Return to the "**Program.cs**" tab and add the following code to `Main`.

```
string serviceName = "rchaplerss";
string adminApiKey = "{COGNITIVESEARCH_PRIMARYADMINKEY}";
string dataSourceName = "rchaplerss-datasource";
string indexName = "rchaplerss-index";
string indexerName = "rchaplerss-indexer";
string skillsetName = "rchaplerss-skillset";
```

_Notes:_
* _Replace name values {e.g., `rchaplerss`} with values appropriate to your implementation_
* _Replace `COGNITIVESEARCH_PRIMARYADMINKEY` with your Cognitive Search API Key (and considering using a Key Vault)_

-----

#### Service Connection Objects
The variables set in this section will be used to manage the Cognitive Search resources.

Append the following code to the bottom of `Main`:

```
var serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
var credential = new AzureKeyCredential(adminApiKey);
var indexClient = new SearchIndexClient(serviceEndpoint, credential);
var indexerClient = new SearchIndexerClient(serviceEndpoint, credential);
```

Logic Explained:
* `var serviceEndpoint...` creates a new `Uri` object that represents the Cognitive Search service endpoint
* `var credential...` creates a new `AzureKeyCredential` object used to authenticate your requests to the Cognitive Search service
* `var indexClient...` creates a new `SearchIndexClient` object used to manage (create, delete, update) indexes in your search service
* `var indexerClient...` creates a new `SearchIndexerClient` object used to manage (run, reset, delete) indexers in your search service

-----

#### Data Source
The logic in this section will create a Cognitive Search Data Source.

Append the following code to the bottom of `Main`:

```
var sidsc = new SearchIndexerDataSourceConnection(
    dataSourceName,
    SearchIndexerDataSourceType.AzureBlob,
    "{STORAGEACCOUNT_CONNECTIONSTRING}",
    new SearchIndexerDataContainer("forms")
    );

indexerClient.CreateDataSourceConnection(sidsc);
```

Logic Explained:
* `var sidsc...` creates a new `SearchIndexerDataSourceConnection` object that represents a connection to a Blob Storage account
    * Replace `STORAGEACCOUNT_CONNECTIONSTRING` with your Storage Account Connection String (and considering using a Key Vault)
* `indexerClient.CreateDataSourceConnection...` creates a new data source connection using the `SearchIndexerDataSourceConnection` object
* `forms` refers to the Storage Account Container mentioned in Solution Requirements

-----

#### Index
The logic in this section will create a Cognitive Search Index.

Append the following code to the bottom of `Main`:

```
var index = new SearchIndex(indexName)
{
    Fields =
    {
        new SimpleField("id", SearchFieldDataType.String) { IsKey = true },
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
    }
};

indexClient.CreateIndex(index);
```

Logic Explained:
* `var index...` creates a new `SearchIndex` object that represents a search index in Cognitive Search
  * `new SimpleField(...`, `new SearchField(...`,`new SearchableField(...` lines add fields the index
    * Each field represents a piece of data that can be searched, filtered, sorted, or faceted in the search index
* `indexClient.CreateIndex...` creates a new index using the `SearchIndex` object

-----

#### Skillset
The logic in this section will create a Cognitive Search Skillset.

Append the following code to the bottom of `Main`:

```
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
    }
};

var skillset = new SearchIndexerSkillset(skillsetName, skills)
{
    CognitiveServicesAccount = new CognitiveServicesAccountKey(key: "33235fe372ff414a9abf54a29af08825")
};

indexerClient.CreateSkillset(skillset);
```

Logic Explained:
* `var skills...` creates a new list of `SearchIndexerSkill` objects
  * Each `SearchIndexerSkill` represents a skill that can be used in an indexing pipeline
  * Inside the `{...}` block, an `OcrSkill` is added to the list
    * The `OcrSkill` is used to extract text from image files
    * The input to the skill is the image field, which comes from the `/document/normalized_images/*` path in your data source
    * The output of the skill is the `text` field
* `var skillset...` creates a new `SearchIndexerSkillset` object that represents a skillset in Cognitive Search
  * Inside the `{...}` block, a `CognitiveServicesAccountKey` is set for the skillset
    * This key is used to authenticate your requests to the Cognitive Services
* `indexerClient.CreateSkillset...` creates a new skillset using the `SearchIndexerSkillset` object

-----

#### Indexer
The logic in this section will create a Cognitive Search Indexer.

Append the following code to the bottom of `Main`:

```
var indexer = new SearchIndexer(indexerName, dataSourceName, indexName)
{
    Parameters = new IndexingParameters()
    {
        IndexingParametersConfiguration = new IndexingParametersConfiguration()
        {
            ImageAction = BlobIndexerImageAction.GenerateNormalizedImagePerPage,
        }
    },
    SkillsetName = skillsetName
};

indexer.OutputFieldMappings.Add(
    new FieldMapping(sourceFieldName: "/document/normalized_images/*/text") { TargetFieldName = "text" }
    );

indexerClient.CreateIndexer(indexer);
```

Logic Explained:
* `var indexer...` creates a new SearchIndexer object that represents an indexer in Azure Cognitive Search
  * The indexer is associated with the data source named `dataSourceName` and the index named `indexName`
  * Inside the `{...}` block, parameters are set for the indexer; these parameters control how the indexer behaves during indexing
    * In this case, the `ImageAction` parameter is set to `GenerateNormalizedImagePerPage`, which means the indexer will generate a normalized image for each page of a document
  * Also inside the `{...}` block, the `SkillsetName` property is set to `skillsetName`
    * This means the indexer will use the skillset named `skillsetName` to transform and enrich your data during indexing
* `indexer.OutputFieldMappings...` adds a field mapping to the indexer’s output field mappings
  * A field mapping defines how a field in your data source maps to a field in your index
  * In this case, the `/document/normalized_images/*/text` field in your data source is mapped to the `text` field in your index
* `indexerClient.CreateIndexer...` creates a new indexer using the `SearchIndexer` object

-----

### Step 4: Confirm Success

#### Visual Studio Debug

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/94ba39d5-0a55-4439-b2d1-188917087aa4" width="800" title="Snipped: October 11, 2023" />

Save your changes and then click "**Debug**" >> "**Start Debugging**" in the menubar.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d00e11c1-5611-49ed-b31e-228eee5428b2" width="600" title="Snipped: October 11, 2023" />

A "Microsoft Visual Studio Debug" window will open (as snipped above).

#### Cognitive Search Index

Navigate to Cognitive Search, then "**Indexes**" in the "**Search management**" grouping of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/60745fe0-95f6-40f7-a91b-94b10332c237" width="800" title="Snipped: October 12, 2023" />

You should see the index that you programmatically created {e.g., "rchaplerss-index"}. Click to open.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/302f8a28-e828-44de-b04c-cb9511bf19a2" width="800" title="Snipped: October 12, 2023" />

Click the "**Search**" button and review results.

-----

## Exercise 2: Source Control
In this exercise, we will create a pull request in a DevOps repo.

### Step 1: Create Git Repository

Open Visual Studio, navigate to "**View**" >> "**Git Changes**", and then click "**Create Git Repository**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/54ffccf9-64db-4e1b-9177-2bde140fa473" width="800" title="Snipped: October 12, 2023" />

Complete the resulting "Create a Git repository" pop-up form.

-----

**Congratulations... you have successfully completed all exercises**
