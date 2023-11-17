# AI Search + OpenAI: Improve Response Quality

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fe54b808-31eb-4932-a561-892f53854750" width="1000" />

## Use Case
* "We believe that our source data includes ambiguities that, if resolved, would result in better OpenAI response to prompts"

## Proposed Solution
* Stage Resources: Create AI Search Index and Open AI Deployment
* Test Prompts: Experiment with prompts to learn how OpenAI uses source data
* Implement Synonyms: Programmatically update the AI Search Index with Synonym Maps

## Solution Requirements
* [AI Search](https://azure.microsoft.com/en-us/products/search)
* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)
* [SQL Server](https://learn.microsoft.com/en-us/azure/azure-sql) and [Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview?view=azuresql) with [AdventureWorks sample data](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)
* [Visual Studio](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload installed

-----
-----

## Exercise 1: Stage Resources
In this exercise, we will import AdventureWorks sample data into AI Search and then use the index in the OpenAI Chat Playground.
<br>_Note: the instructions below are for creating a minimum viable index {i.e., no bells-and-whistles}_

### Step 1: Create AI Search Index
Navigate to AI Search > "Overview", click "Import data" and on the resulting page, select "Azure SQL Database" from the "Data Source" dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b67ee133-abc4-441d-bb38-3eb2ba2ffca9" width="800" title="Snipped: November 16, 2023" />

Click the "Choose an existing connection" link and select the SQL Database that will be the source.
<br>Complete the "Import data" form and then click "Test connection".
<br>Once the connection is validated, select the "[SalesLT].[Product]" table from the added "Table/View" dropdown.
<br>Click "Next: Add cognitive skills (Optional)".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d467d9c9-4a10-4479-b383-89a561be3b22" width="800" title="Snipped: November 16, 2023" />

Click "Skip to: Customize target index".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0377231d-3f77-451b-b7c5-a6bb1586d046" width="800" title="Snipped: November 16, 2023" />

Check the top "Retrievable" and "Searchable" boxes (which will check all boxes for all fields).
<br>Click "Next: Create an indexer".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4e908d25-3801-42c6-b0bb-07b3290eef04" width="800" title="Snipped: November 16, 2023" />

Click "Submit".

#### Confirm Success

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/248f073f-b490-47b0-b561-a7aa25e2173d" width="800" title="Snipped: November 16, 2023" />

Navigate to the new index and confirm success.

### Step 2: Create OpenAI Deployment
Navigate to OpenAI Studio > "Chat playground".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc174b2b-9db3-4757-b426-69b9bc65ff37" width="800" title="Snipped: November 16, 2023" />

Navigate to "Add your data..." and then click "Add a data source" on the "Assistant setup" pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/65fe3909-2010-49a1-ba6f-2772bd25b5bd" width="800" title="Snipped: November 16, 2023" />

Complete the "Add data" forms, then click "Save and close".

#### Confirm Success
Enter a prompt in the "Chat session" pane.
<br>_Note: In Exercise 2, we will focus on ambiguous Product Size data, so I started with: `Describe a large product`_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eeb2fcff-b5f5-4728-bd93-ebbc752b77b6" width="800" title="Snipped: November 16, 2023" />

Confirm response; for example:

```
The retrieved document provides information about a large product, specifically a "Touring-Panniers" with the model number "PA-T100". It is grey in color. The product has a price of 125.00 units, though it's not clear what currency this is in. There's also a number 51.5625 associated with the product, but without further context, it's unclear what this number represents.
```

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 2: Experiment with Prompts
In this exercise, we will experiment with prompts to learn how OpenAI uses source data.

### Step 1: Source Data, SQL
First, we get an idea what to expect from SQL.
<br>Navigate to the SQL Database >> Query Editor and login.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6ba15e7d-84fd-4ec9-b1c3-b9631518d60e" width="800" title="Snipped: November 16, 2023" />

Run the following T-SQL query:
```
SELECT TOP 10 ProductID, Name, Color, Size
FROM [SalesLT].[Product]
```

You will note that the results do not include anything specifically called out as "Large", which aligns with the limited response we saw from OpenAI earlier.
<br>There is, however, a column named Size and rows that contain the value "L", which we can assume stands for "Large"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/afa55b23-efe1-4656-8fec-806c62cae098" width="800" title="Snipped: November 16, 2023" />

Run the following T-SQL query:
```
SELECT TOP 10 ProductID, Name, Color, Size
FROM [SalesLT].[Product]
WHERE [Name] like '%Large%'
```

If we specifically query for rows with "Large" in the Name column, we see the single value that surfaced earlier in OpenAI.

### Step 2: Source Data, AI Search
Next, we get an idea what to expect from AI Search.
<br>Navigate to the AI Search index.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/200d99cd-1afd-4ed9-bdf1-6a949fb352d2" width="800" title="Snipped: November 16, 2023" />

On the "Search explorer" tab, enter Query Phrase `Large` and press "Search".

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes('azuresql-index')/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 2.70413,
      "ProductID": "842",
      "Name": "Touring-Panniers, Large",
      "ProductNumber": "PA-T100",
      "Color": "Grey",
      "StandardCost": "51.5625",
      "ListPrice": "125.0000",
      "Size": null,
      "Weight": null,
      "ProductCategoryID": 39,
      "ProductModelID": 120,
      "SellStartDate": "2006-07-01T00:00:00Z",
      "SellEndDate": "2007-06-30T00:00:00Z",
      "DiscontinuedDate": null,
      "ThumbnailPhotoFileName": "no_image_available_small.gif",
      "rowguid": "56334fff-91d4-495e-bf98-933bc1010f23",
      "ModifiedDate": "2008-03-11T10:01:36.827Z",
      "keyphrases": [
        "842"
      ]
    }
  ]
}
```

### Step 3: OpenAI Prompt Experimentation
Finally, we compare our expectations to reality to determine whether it is a schema or data issue.

<br>Navigate to OpenAI Studio >> Chat Playground, and re-add data source (if disconnected).

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/65fbe2ea-3968-48b1-8528-5fe178b13357" width="800" title="Snipped: November 16, 2023" />

#### Base Question
Prompt: `what large products are there?`
<br>Response:
```
The retrieved document mentions a product called "Touring-Panniers, Large" with the product code "PA-T100". It is available in grey color.
```

_Note: This answer demonstrates that OpenAI is using the "Color" column (name and data) in spite of not being prompted for that information._
<br>_This answer confirms that OpenAI is: 1) not using the "Size" column, and / or 2) confused by use of "L" as an abbreviation for "Large"_

<br>

#### Follow-up: Can I ambiguously reference columns that should be available?

Prompt: `how much does it cost?`
<br>Response:
```
The price of the "Touring-Panniers, Large" product is $125.00.
```

_Note: This answer implies use and some understanding of column names... it choose a column that corresponds to the request, but is not clear why it chose List Price instead of Standard Cost (specifically when the question was about cost)"_

<br>

#### Follow-up: Does specific column reference help?
Prompt: `is that standard cost or list price?`
<br>Response:
```
The $125.00 price for the "Touring-Panniers, Large" product is the list price.
```

_Note: Specificity does help and capitalization / spacing of column names does not appear to matter_

### Conclusions
1. OpenAI uses column information without including metadata in row-level data
2. OpenAI appear to be fairly comfortable dealing with schema ambiguity
3. OpenAI does not deal with data ambiguity quite so well

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 3: Implement Synonyms
In this exercise, we will programmatically update the AI Search Index with Synonym Maps and confirm OpenAI response enhancement.
<br>_Note: This section builds on skills learned from: [AI Search, DevOps](AISearch_DevOps.md)_

### Step 1: Create Visual Studio Project

Open Visual Studio and click "**Create a new project**".
<br>On the "**Create a new project**" form, search for and select "**Console App**", then click "**Next**".
<br>Complete the "**Configure your new project**" form, then click "**Next**".
<br>Complete the "**Additional information**" form, then click "**Create**".

-----

### Step 2: Install NuGet

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**".
<br>On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Azure.Search.Documents**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".
<br>When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.

#### Additional Packages

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
        /* ************************* Names */

        string nameAISearch_Index = "azuresql-index";
        string nameAISearch_SynonymMap = "azuresql-synonymmap";

        /* ************************* URIs */

        var uriAISearch = new Uri($"https://rchaplerss.search.windows.net/");
        var uriKeyVault = new Uri($"https://rchaplerk.vault.azure.net/");

        /* ************************* Keys */

        var sc = new SecretClient(uriKeyVault, new DefaultAzureCredential());
        var keyAISearch = sc.GetSecret("Key-AISearch").Value.Value.ToString() ?? string.Empty;

        /* ************************* Clients */

        var credential = new AzureKeyCredential(keyAISearch);
        var indexClient = new SearchIndexClient(uriAISearch, credential);

        /* ************************* SynonymMap */

        var d = new Dictionary<string, List<string>>
            {
                { "Large", new List<string> { "L" } },
                { "Medium", new List<string> { "M" } },
                { "Small", new List<string> { "S" } }
            };

        string synonyms = "";
        foreach (var synonym in d) { synonyms += $"{synonym.Key}, {string.Join(", ", synonym.Value)}\n"; }

        var sm = new SynonymMap(nameAISearch_SynonymMap, synonyms);
        indexClient.CreateOrUpdateSynonymMap(sm);

        var index = indexClient.GetIndex(nameAISearch_Index);

        int sizeIndex = -1;
        for (int i = 0; i < index.Value.Fields.Count; i++)
        {
            if (index.Value.Fields[i].Name == "Size")
            {
                sizeIndex = i;
                break;
            }
        }

        if (sizeIndex != -1)
        {
            index.Value.Fields[sizeIndex].SynonymMapNames.Add(sm.Name);
        }
        else
        {
            Console.WriteLine("Field 'Size' not found.");
        }


        //index.Value.Fields[0].SynonymMapNames.Add(sm.Name);

        indexClient.CreateOrUpdateIndex(index);
    }
}
```

#### Logic Explanation

1. **Names**: The names of the Azure Cognitive Search index and synonym map are defined.
2. **URIs**: The URIs for Azure Cognitive Search and Azure Key Vault are defined.
3. **Keys**: The `SecretClient` is used to connect to Azure Key Vault using the `DefaultAzureCredential`. The secret named "Key-AISearch" is retrieved from the Key Vault.
4. **Clients**: An `AzureKeyCredential` is created using the secret retrieved from Key Vault. This credential is then used to create a `SearchIndexClient` which will be used to interact with Azure Cognitive Search.
5. **SynonymMap**: A dictionary is created to map synonyms. For example, "Large" is a synonym for "L", "Medium" for "M", and "Small" for "S". These synonyms are then added to a `SynonymMap` object which is created or updated in Azure Cognitive Search.
6. **Index**: The index named "azuresql-index" is retrieved from Azure Cognitive Search.
7. **Size Field**: The code then iterates over the fields in the index to find the field named "Size". If the "Size" field is found, the name of the synonym map is added to the field's `SynonymMapNames` property.
8. **Update Index**: Finally, the updated index is sent back to Azure Cognitive Search using the `CreateOrUpdateIndex` method.

_Note: Replace name values {e.g., `rchaplerss`} with values appropriate to your implementation_

-----

### Step 4: Confirm Success

#### Visual Studio Debug

Save your changes and then click "**Debug**" >> "**Start Debugging**" in the menubar.
<br>A "Microsoft Visual Studio Debug" window will open.

#### AI Search Index JSON

Navigate to the AI Search Index, then click "Edit JSON".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/48877403-e635-4c63-b2e1-f4b63175d2b0" width="800" title="Snipped: November 17, 2023" />

Scroll to the entry for `Size`:

```
{
  "name": "Size",
  "type": "Edm.String",
  "searchable": true,
  "filterable": false,
  "retrievable": true,
  "sortable": false,
  "facetable": false,
  "key": false,
  "indexAnalyzer": null,
  "searchAnalyzer": null,
  "analyzer": "standard.lucene",
  "normalizer": null,
  "dimensions": null,
  "vectorSearchProfile": null,
  "synonymMaps": [
    "**azuresql-synonymmap**"
  ]
}
```

You can expect to see the `synonymMaps` reference (as above).
<br>Sadly, this is as much as you can expect to see as proof in the UI... any additional evidence needs to be pulled via API or .NET.

#### AI Search Index Query

Navigate to the AI Search Index, "Search Explorer" tab.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/176c6023-54cf-4c74-834c-495ec5817b0b" width="800" title="Snipped: November 17, 2023" />

You should see a much longer resultset, including entries like:

```
{
  "@search.score": 2.9267395,
  "ProductID": "715",
  "Name": "Long-Sleeve Logo Jersey, L",
  "ProductNumber": "LJ-0192-L",
  "Color": "Multi",
  "StandardCost": "38.4923",
  "ListPrice": "49.9900",
  "Size": "L",
  "Weight": null,
  "ProductCategoryID": 25,
  "ProductModelID": 11,
  "SellStartDate": "2005-07-01T00:00:00Z",
  "SellEndDate": null,
  "DiscontinuedDate": null,
  "ThumbnailPhotoFileName": "awc_jersey_male_small.gif",
  "rowguid": "34cf5ef5-c077-4ea0-914a-084814d5cbd5",
  "ModifiedDate": "2008-03-11T10:01:36.827Z"
}
```

You will note that this does not have the word "Large", but the `Size` field does have value "L".

#### OpenAI, Chat Playground

Navigate to OpenAI Studio >> Chat Playground, and re-add data source (if disconnected).
<br>Re-enter the original prompt: `what large products are there?`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c4779d49-6089-47a3-8e1b-7d8b830c695c" width="800" title="Snipped: November 17, 2023" />

You will see that the response is much improved!

-----

**Congratulations... you have successfully completed all exercises**
