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

### Step 3: Definitions
JSON Definitions are used by API logic to create AI Search resources.

<br>Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Definitions".

#### Data Source
Definition JSON for Blob and SQL versions are sufficiently different to merit separate handling... examples for both are included below.

##### `datasource_blob.json`
Right-click on the "Definitions" folder, select "Add" >> "New Item" from the resulting dropdowns, search for and select "JSON", enter name "datasource_blob.json" on the resulting popup and then click "Add".

Replace the default code with:

```json
{
  "type": "azureblob",
  "credentials": {
    "connectionString": "{AISearch-DataSource-Blob-ConnectionString}"
  },
  "container": {
    "name": "{AISearch-DataSource-Blob-Container}"
  },
  "identity": null,
  "name": "{AISearch-DataSource-Name}"
}
```

##### `datasource_sql.json`

```json
{
  "name": "{AISearch-DataSource-Name}",
  "description": null,
  "type": "azuresql",
  "subtype": null,
  "credentials": {
    "connectionString": "Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;Server=tcp:{SQL-Server-Name}.database.windows.net,1433;Database={SQL-Database-Name};User ID={SQL-Server-User};Password={SQL-Server-Password};"
  },
  "container": {
    "name": "{AISearch-DataSource-SQL-Table}",
    "query": "{AISearch-DataSource-SQL-Query}"
  },
  "dataChangeDetectionPolicy": {
    "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
    "highWaterMarkColumnName": "ModifiedDate"
  },
  "dataDeletionDetectionPolicy": null,
  "encryptionKey": null,
  "identity": null
}
```
_Note: Parameters are wrapped in curly brackets {e.g., `{ResourceGroup-Name}`} and replaced with values at run-time_

#### Index

##### `index_blob.json`
```json
{
  "name": "{AISearch-Index-Name}",
  "fields": [
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
      "name": "vectors",
      "type": "Collection(Edm.Single)",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "dimensions": 3072,
      "vectorSearchProfile": "{AISearch-VectorProfile-Name}"
    }
  ],
  "similarity": {
    "@odata.type": "#Microsoft.Azure.Search.BM25Similarity"
  },
  "semantic": {
    "defaultConfiguration": "{AISearch-SemanticConfiguration-Name}",
    "configurations": [
      {
        "name": "{AISearch-SemanticConfiguration-Name}",
        "prioritizedFields": {
          "titleField": {
            "fieldName": "metadata_title"
          },
          "prioritizedContentFields": [
            {
              "fieldName": "chunk"
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
        "name": "{AISearch-VectorAlgorithm-Name}",
        "kind": "hnsw",
        "hnswParameters": {
          "metric": "cosine",
          "m": 4,
          "efConstruction": 400,
          "efSearch": 500
        }
      }
    ],
    "vectorizers": [
      {
        "name": "{AISearch-Vectorizer-Name}",
        "kind": "azureOpenAI",
        "azureOpenAIParameters": {
          "resourceUri": "https://{OpenAI-Name}.openai.azure.com",
          "apiKey": "{OpenAI-Key}",
          "deploymentId": "{OpenAI-Deployment-Embedding}",
          "authIdentity": null
        }
      }
    ],
    "profiles": [
      {
        "name": "{AISearch-VectorProfile-Name}",
        "algorithm": "{AISearch-VectorAlgorithm-Name}",
        "vectorizer": "{AISearch-Vectorizer-Name}"
      }
    ]
  }
}
```

##### `index_sql.json`

```json
{
  "name": "{AISearch-Index-Name}",
  "fields": [
    {
      "name": "ProductDescriptionID",
      "type": "Edm.String",
      "searchable": true,
      "filterable": true,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": true
    },
    {
      "name": "Description",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
    },
    {
      "name": "ModifiedDate",
      "type": "Edm.DateTimeOffset",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false
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
    }
  ],
  "semantic": {
    "defaultConfiguration": "{AISearch-SemanticConfiguration-Name}",
    "configurations": [
      {
        "name": "{AISearch-SemanticConfiguration-Name}",
        "prioritizedFields": {
          "titleField": {
            "fieldName": "ProductDescriptionID"
          },
          "prioritizedContentFields": [
            {
              "fieldName": "Description"
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
  }
}  
```

#### Skillset

##### `skillset_blob.json`
```json
```

##### `skillset_sql.json`

```json
```

#### Indexer

##### `indexer_blob.json`
```json
```

##### `indexer_sql.json`

```json
```

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