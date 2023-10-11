# DevOps: Cognitive Search

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/20ef5226-59b5-4876-b8b2-789373480cb4" width="1000" />

## Use Case
* "We have implemented OpenAI with Cognitive Search and are rapidly iterating through enhancements to the index"
* "Creating and updating the Cognitive Search index can be difficult... we want a simpler, faster, more consistent experience"
* "We want to capture our Cogniive Search index creation process in our DevOps repo"

## Proposed Solution
* Sample Data: Download sample PDF files and upload to a Blob Container
* Develop Logic: Use Visual Studio (C#) and the Cognitive Search Development Kit (SDK) to codify creation of data source, index, skillset, and indexer
* Check-In Logic: Create a pull request in a DevOps repo

## Solution Requirements
* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/)
* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/)

-----

## Exercise 1: Sample Data
In this exercise, we will download sample PDF files and upload to a Blob Container.

LOREM IPSUM

-----

## Exercise 2: Develop Logic
In this exercise, we will add create a Cognitive Search index, skillset, and indexer using a Console App.

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

#### Service Connection Objects
The variables set in this section will be used to manage the Cognitive Search resources.

Append the following code at the bottom of `Main`:

```
var serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
var credential = new AzureKeyCredential(adminApiKey);
var indexClient = new SearchIndexClient(serviceEndpoint, credential);
var indexerClient = new SearchIndexerClient(serviceEndpoint, credential);
```

Logic Explained:
* `var serviceEndpoint...` creates a new Uri object that represents the Cognitive Search service endpoint
* `var credential...` creates a new AzureKeyCredential object used to authenticate your requests to the Cognitive Search service
* `var indexClient...` creates a new SearchIndexClient object used to manage (create, delete, update) indexes in your search service
* `var indexerClient...` creates a new SearchIndexerClient object used to manage (run, reset, delete) indexers in your search service

#### Data Source

The logic in this section will create a Cognitive Search Data Source.

Append the following code at the bottom of `Main`:

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
* `var sidsc...` creates a new SearchIndexerDataSourceConnection object that represents a connection to a Blob Storage account
    * Replace `STORAGEACCOUNT_CONNECTIONSTRING` with your Storage Account Connection String (and considering using a Key Vault)
* `indexerClient.CreateDataSourceConnection...` creates a new data source connection using the SearchIndexerDataSourceConnection object

#### ************************* Index

The logic in this section will create a Cognitive Search Index.

Append the following code at the bottom of `Main`:

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
* `var index...` creates a new SearchIndex object that represents a search index in Cognitive Search
  * `new SimpleField(...`, `new SearchField(...`,`new SearchableField(...` lines add fields the index
    * Each field represents a piece of data that can be searched, filtered, sorted, or faceted in the search index
* `indexClient.CreateIndex...` creates a new index using the SearchIndex object






-----

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
            }
        };

        var skillset = new SearchIndexerSkillset(skillsetName, skills)
        {
            CognitiveServicesAccount = new CognitiveServicesAccountKey(key: "33235fe372ff414a9abf54a29af08825")
        };

        indexerClient.DeleteSkillset(skillset);

        indexerClient.CreateSkillset(skillset);

        /* ************************* Indexer */

        SearchIndexer indexer = new SearchIndexer(indexerName, dataSourceName, indexName)
        {
            Parameters = new IndexingParameters()
            {
                IndexingParametersConfiguration = new IndexingParametersConfiguration()
                {
                    ImageAction = BlobIndexerImageAction.GenerateNormalizedImagePerPage, /* re: OCR */
                }
            },
            SkillsetName = skillsetName
        };

        indexer.OutputFieldMappings.Add(
            new FieldMapping(sourceFieldName: "/document/normalized_images/*/text") { TargetFieldName = "text" }
            );

        indexerClient.DeleteIndexer(indexer);

        indexerClient.CreateIndexer(indexer);
    }
}
```

## Content, Merged_Content, and Text
In Azure Cognitive Search, the fields `content`, `merged_content`, and `text` have different roles when it comes to OCR scanned PDF documents:

1. **Content**: This field is generated when Azure Cognitive Search "cracks" each document to read content from different file formats. The found text originating in the source file is placed into this generated `content` field, one for each document⁴.

2. **Merged_content**: This field is typically used when you want to merge the textual representation of images (text from an OCR skill, or the caption of an image) into the content field of a document². For example, when you enable OCR and select the checkbox "merge all text into merged_content field", all of the enriched field options become visible³.

3. **Text**: This is the plain text extracted from the image by the OCR skill². The OCR skill recognizes printed and handwritten text in image files and extracts this text².

In summary, `content` is the original text content of the document, `merged_content` is a combination of original and additional enriched content (like OCR results), and `text` is specifically the output from the OCR process.

## OCRSkill
The OcrSkill is a prebuilt AI skill for text extraction from image files. It takes an image as input and outputs the text found in the image.

Here’s what each part does:

new InputFieldMappingEntry("image") { Source = "/document/normalized_images/*" }: This line is defining the input to the OCR skill. It’s saying that the input images will come from the normalized_images field of the documents in your search index.

new OutputFieldMappingEntry("text") { TargetName = "text" }: This line is defining where the output of the OCR skill (the extracted text) will be stored. In this case, it’s stored in a field named text.

So, to sum up, this code is setting up an OCR skill that takes images from the normalized_images field of your documents, extracts text from them, and stores that text in a field named text. This can be useful for making the text within images searchable.
