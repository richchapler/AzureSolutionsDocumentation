# Data Enrichment: Cognitive Search, Custom "Get Data" Skillset

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d7ad1df6-3f08-4397-88b6-a4b7b67ea90d" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to enrich our Cognitive Search index with metadata from SQL"

## Proposed Solution
* Create a "Get Data" API using a Function App
* Create a Cognitive Search index that includes a custom skillset

_Note: Our pretend scenario will be that users of a "hotels" search index want to also know the count of customers in a given city {i.e. using sample "hotels" data from Cognitive Search and sample "CustomerAddress" data from SQL Server, AdventureWorks}_

## Solution Requirements
The proposed solution requires:
* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview), including dependencies:
  * [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
  * [Application Service](https://learn.microsoft.com/en-us/azure/app-service/)
  * [Storage Account](Infrastructure_StorageAccount.md)
* [**Postman**](https://www.postman.com/product/workspaces/)
* [**SQL**](https://learn.microsoft.com/en-us/azure/azure-sql) with [AdventureWorks sample data](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)
* [**Visual Studio**](https://visualstudio.microsoft.com/) with **Azure development** workload

-----

## Exercise 1: Create API
In this exercise, we will create and publish a Function App-based API that will receive a parameter, query SQL, and package a response.

### Step 1: Create Visual Studio Project

Open Visual Studio.

<img src="https://user-images.githubusercontent.com/44923999/212137484-599c9cd8-5e0e-46b1-818d-a3a008fecd5b.png" width="600" title="Snipped: January 12, 2023" />

Click "**Create a new project**".

<img src="https://user-images.githubusercontent.com/44923999/212137783-9ee34157-17fc-4364-a2d1-d572afbb4d8b.png" width="600" title="Snipped: January 12, 2023" />

On the "**Create a new project**" page, search for and select "**Azure Functions**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0d7c77a-af1b-4f72-ade8-a800a34729dc" width="600" title="Snipped: July 5, 2023" />

Complete the "**Configure your new project**" form and then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/cf45d8ed-27a4-40e4-ac5a-40efbeb30f99" width="600" title="Snipped: July 5, 2023" />

Complete the "**Additional information**" form:

Prompt | Entry
:----- | :-----
**Functions worker** | **.NET 6.0 (Long Term Support)**
**Function** | **Http trigger**
**Use Azurite...** | Checked
**Authorization level** | Function

Click "**Create**".

-----

### Step 2: Install NuGet

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1695a4b5-ce8d-4c40-b082-c0f716bf40f3" width="800" title="Snipped: July 5, 2023" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Microsoft.Data.SqlClient**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e21b7833-b2ef-459e-929d-6398c3f24ff8" width="800" title="Snipped: July 5, 2023" />

On the resulting pop-out, check the Project in the list, and then click "**Install**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc4eae77-2f17-4869-8e2b-3d033719b8ed" width="800" title="Snipped: July 5, 2023" />

Navigate to the "**Updates**" tab, check "**Select all packages**" and then click **Update** (as applicable).

-----

### Step 3: Update Logic

Rename "Function1.cs" to "GetData.cs" and open.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ce428390-fb00-410b-aa8e-8d6441a0589d" width="800" title="Snipped: July 6, 2023" />

Replace the default code with:

```
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;

namespace CognitiveSearch_CustomSkillsetAPI
{
    public static class getData
    {
        [FunctionName("getData")]
        public static async Task<IActionResult> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req, ILogger log)
        {
            SqlConnectionStringBuilder scsb = new SqlConnectionStringBuilder();
            scsb.DataSource = "{SQL_SERVER_NAME}.database.windows.net";
            scsb.UserID = "{SQL_ADMIN_LOGIN}";
            scsb.Password = "{SQL_ADMIN_PASSWORD}";
            scsb.InitialCatalog = "{SQL_DATABASE_NAME}";

            string request = await new StreamReader(req.Body).ReadToEndAsync();

            var d = new Dictionary<string, object> { { "values", new List<Dictionary<string, object>>() } };

            foreach (var value in JObject.Parse(request)["values"])
            {
                string c = "0";

                string sql = "SELECT COUNT(*)\r\nFROM [SalesLT].[CustomerAddress] CA\r\nINNER JOIN [SalesLT].[Address] A ON CA.AddressId=A.AddressId\r\nWHERE A.City = '" + (string)value["data"]["text"] + "'\r\n";

                using (SqlConnection sc = new SqlConnection(scsb.ConnectionString))
                {
                    using (SqlCommand command = new SqlCommand(cmdText: sql, connection: sc))
                    {
                        sc.Open();

                        using (SqlDataReader sdr = command.ExecuteReader()) { if (sdr.Read()) { c = sdr.GetInt32(0).ToString(); } }

                        sc.Close();
                    }
                }

                var r = new Dictionary<string, object>
                {
                    { "recordId", (string)value["recordId"] },
                    { "data", new Dictionary<string, string> { { "customercount", c } } },
                    { "errors", null },
                    { "warnings", null }
                };

                ((List<Dictionary<string, object>>)d["values"]).Add(r);
            }

            req.HttpContext.Response.Headers.Add("Content-Type", "application/json");

            log.LogInformation(JsonConvert.SerializeObject(d));

            return new OkObjectResult(JsonConvert.SerializeObject(d));
        }
    }
}
```

Logic Explained:

* `using Microsoft.Data.SqlClient`... necessary to execute Data Explorer queries
* [SqlConnectionStringBuilder](https://learn.microsoft.com/en-us/dotnet/api/microsoft.data.sqlclient.sqlconnectionstringbuilder?view=sqlclient-dotnet-standard-5.1)
  * `{SQL_SERVER_NAME}`, etc... replace with values appropriate for your instantiation
* `string request =...` reads the contents of the HTTP Request Body
* `var d = new Dictionary...` prepares for response with the creation of a Dictionary; this must occur before the `foreach` loop to allow for iterative additions
* `foreach (var value in JObject...` begins iteration through the HTTP Request Body
* `using (SqlConnection...`, etc... runs T-SQL query logic to pull the count of customers for a given City
* `var r = new Dictionary...` prepares for addition of a new record to previously-created response Dictionary
* `((List<Dictionary...` adds the new record to the previously-created response Dictionary
* `req.HttpContext.Response...` sets the `Content-Type` header of the response to indicate that the response body contains JSON data
* `return new OkObjectResult(...` returns an `HTTP 200 OK` response and serialized JSON version of the previously-created response Dictionary

#### Sample Request Body (abridged)

`{"values":[{"recordId":"0","data":{"text":"Seattle"}},{"recordId":"1","data":{"text":"Redmond"}},...,{"recordId":"49","data":{"text":"Santa Clara"}}]}`

#### Sample Response Body (abridged)

`{"values":[{"recordId":"0","data":{"customercount":"4"},"errors":null,"warnings":null},{"recordId":"1","data":{"customercount":"2"},"errors":null,"warnings":null},...,{"recordId":"49","data":{"customercount":"0"},"errors":null,"warnings":null}]}`

-----

### Step 4: Publish to Azure

In Visual Studio, right-click on the project and select **Publish** from the resulting drop-down menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a48fdfe9-1afb-4628-938d-b327234ebfab" width="800" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Where are you publishing...**" page, select "**Azure**" and then click **Next**.

<img src="https://user-images.githubusercontent.com/44923999/212152235-d37d84e9-9889-48cb-8d02-6d1b040e34e8.png" width="600" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Which Azure service...**" page, select "**Azure Function App (Windows)**" and then click **Next**.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b0f6a61e-b741-4b48-a3d6-eb2ac5f711a7" width="600" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Select existing or...**" page, select your Function App and then click **Finish**.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2333abe2-d9e0-4674-84ca-aba15be28be9" width="600" title="Snipped: July 6, 2023" />

Back on the "...Publish" page, click **Publish** and confirm successful publication in the "**Output**" pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e31d913e-62ee-4d6f-a117-aac39e57d4c9" width="800" title="Snipped: July 6, 2023" />

-----

### Step 5: Confirm Success

Navigate to the **GetData** function, and then "**Code + Test**" in the "**Developer**" grouping of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/dc516c29-900f-47ab-8afc-ebc7b5cc0dbb" width="800" title="Snipped: July 6, 2023" />

Click "**Test/Run**" and on the resulting pop-out, "**Input**" tab, paste the following "**Body**" value:

```
{"values":[{"recordId":"0","data":{"text":"Seattle"}},{"recordId":"1","data":{"text":"Redmond"}},{"recordId":"2","data":{"text":"Bothell"}},{"recordId":"3","data":{"text":"St. Louis"}},{"recordId":"4","data":{"text":"Portland"}},{"recordId":"5","data":{"text":"Memphis"}},{"recordId":"6","data":{"text":"Kirkland"}},{"recordId":"7","data":{"text":"West Hartford"}},{"recordId":"8","data":{"text":"San Diego"}},{"recordId":"9","data":{"text":"Mountain View"}},{"recordId":"10","data":{"text":"Detroit"}},{"recordId":"11","data":{"text":"Durham"}},{"recordId":"12","data":{"text":"Wilsonville"}},{"recordId":"13","data":{"text":"Atlanta"}},{"recordId":"14","data":{"text":"Austin"}},{"recordId":"15","data":{"text":"New York"}},{"recordId":"16","data":{"text":"Seattle"}},{"recordId":"17","data":{"text":"Dallas"}},{"recordId":"18","data":{"text":"San Antonio"}},{"recordId":"19","data":{"text":"Portland"}},{"recordId":"20","data":{"text":"Redmond"}},{"recordId":"21","data":{"text":"Aventura"}},{"recordId":"22","data":{"text":"Sarasota"}},{"recordId":"23","data":{"text":"Atlanta"}},{"recordId":"24","data":{"text":"Bellevue"}},{"recordId":"25","data":{"text":"Metairie"}},{"recordId":"26","data":{"text":"San Jose"}},{"recordId":"27","data":{"text":"Seattle"}},{"recordId":"28","data":{"text":"Chicago"}},{"recordId":"29","data":{"text":"Miami"}},{"recordId":"30","data":{"text":"Bellevue"}},{"recordId":"31","data":{"text":"Tulsa"}},{"recordId":"32","data":{"text":"New York"}},{"recordId":"33","data":{"text":"Fargo"}},{"recordId":"34","data":{"text":"Honolulu"}},{"recordId":"35","data":{"text":"Boise"}},{"recordId":"36","data":{"text":"Albuquerque"}},{"recordId":"37","data":{"text":"San Francisco"}},{"recordId":"38","data":{"text":"Cambridge"}},{"recordId":"39","data":{"text":"Scottsdale"}},{"recordId":"40","data":{"text":"Washington D.C."}},{"recordId":"41","data":{"text":"Lexington"}},{"recordId":"42","data":{"text":"Nashville"}},{"recordId":"43","data":{"text":"Denver"}},{"recordId":"44","data":{"text":"Boston"}},{"recordId":"45","data":{"text":"Arlington"}},{"recordId":"46","data":{"text":"San Francisco"}},{"recordId":"47","data":{"text":"New York"}},{"recordId":"48","data":{"text":"Tampa"}},{"recordId":"49","data":{"text":"Santa Clara"}}]}
```

Click "**Run**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3b96537a-872f-4f67-b126-b295aaad42e5" width="800" title="Snipped: July 6, 2023" />

The pop-out will switch to the "**Output**" tab and you can expect the following "**HTTP response content**" value:

```
{"values":[{"recordId":"0","data":{"customercount":"4"},"errors":null,"warnings":null},{"recordId":"1","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"2","data":{"customercount":"4"},"errors":null,"warnings":null},{"recordId":"3","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"4","data":{"customercount":"3"},"errors":null,"warnings":null},{"recordId":"5","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"6","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"7","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"8","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"9","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"10","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"11","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"12","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"13","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"14","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"15","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"16","data":{"customercount":"4"},"errors":null,"warnings":null},{"recordId":"17","data":{"customercount":"6"},"errors":null,"warnings":null},{"recordId":"18","data":{"customercount":"5"},"errors":null,"warnings":null},{"recordId":"19","data":{"customercount":"3"},"errors":null,"warnings":null},{"recordId":"20","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"21","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"22","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"23","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"24","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"25","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"26","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"27","data":{"customercount":"4"},"errors":null,"warnings":null},{"recordId":"28","data":{"customercount":"5"},"errors":null,"warnings":null},{"recordId":"29","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"30","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"31","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"32","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"33","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"34","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"35","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"36","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"37","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"38","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"39","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"40","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"41","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"42","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"43","data":{"customercount":"2"},"errors":null,"warnings":null},{"recordId":"44","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"45","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"46","data":{"customercount":"1"},"errors":null,"warnings":null},{"recordId":"47","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"48","data":{"customercount":"0"},"errors":null,"warnings":null},{"recordId":"49","data":{"customercount":"0"},"errors":null,"warnings":null}]}
```

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Create Index
In this exercise, we will create a Cognitive Search index and then customize the index, indexer, and skillset.

### Step 1: Import Data

Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".










<img src="https://user-images.githubusercontent.com/44923999/226375829-57106809-9582-46b5-ba64-638d3348e36b.png" width="800" title="Snipped: March 20, 2023" />

Complete the "**Import Data**" >> "**Connect to your data**" form, including:

Prompt | Entry
:----- | :-----
**Data Source** | Select "**Azure SQL Database**"
**Data source name** | Enter a meaningful name aligned with your standard {e.g., SERVER-DATABASE}
**Connection string** | Click "**Choose an existing connection**", then select your SQL Database from the resulting pop-out menu
**Managed identity authentication** | Select "**System-assigned**"
**Table/View** | Enter "**SalesLT.Product**"

Click "**Next: Add cognitive skills**...".<br>
On the resulting "**Add cognitive skills**..." page, expand "**Attach Cognitive Services**".

<img src="https://user-images.githubusercontent.com/44923999/226380779-1feebb45-d656-4288-ae6b-f6e67c48a5e8.png" width="800" title="Snipped: March 20, 2023" />

Select your instance of Cognitive Services.<br>
Collapse "**Attach Cognitive Services**" and expand "**Add enrichments**".

<img src="https://user-images.githubusercontent.com/44923999/226403680-d650824d-0b63-4334-b3a9-1179890dfc44.png" width="800" title="Snipped: March 20, 2023" />

On the "**Add cognitive skills**..." tab,  select:
* "Extract people names"
* "Extract organization names"
* "Extract location names"
* "Extract key phrases"

Click "**Next: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/226404041-aa514da5-1a5c-4edd-90c9-a034488f15be.png" width="800" title="Snipped: March 20, 2023" />

On the "**Customize target index**" tab, enter **Suggester** name "**azuresql-suggester**" and select all available options.<br>
Click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226385246-bf6f57bf-c315-4513-9920-fd4254f7c4ec.png" width="800" title="Snipped: March 20, 2023" />

On the "**Create an indexer**" tab, confirm default values and then click "**Submit**".

### Step 2: Confirm Success
Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>

<img src="https://user-images.githubusercontent.com/44923999/226418120-1ab6b78c-8b5d-4abf-a75f-e50a19ab6061.png" width="800" title="Snipped: March 20, 2023" />

Click on the newly-created Indexer.

<img src="https://user-images.githubusercontent.com/44923999/226404925-b1cf0637-b9db-4093-b5e7-62bdaa7d2140.png" width="800" title="Snipped: March 20, 2023" />

Confirm successful execution.<br>
Navigate to the "**Indexer Definition (JSON)**" tab.

<img src="https://user-images.githubusercontent.com/44923999/226405366-d5a3beaf-4c32-40ad-bd8c-43e3c1059a84.png" width="800" title="Snipped: March 20, 2023" />

Review the produced JSON content... we will use the example below in Exercise Three.

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/$metadata#indexers/$entity",
  "@odata.etag": "\"0x8DB295E944A0232\"",
  "name": "azuresql-indexer",
  "description": "",
  "dataSourceName": "rchaplersds-rchaplersd",
  "skillsetName": "azuresql-skillset",
  "targetIndexName": "azuresql-index",
  "disabled": null,
  "schedule": null,
  "parameters": {
    "batchSize": null,
    "maxFailedItems": 0,
    "maxFailedItemsPerBatch": 0,
    "base64EncodeKeys": false,
    "configuration": {}
  },
  "fieldMappings": [],
  "outputFieldMappings": [
    {
      "sourceFieldName": "/document/ProductID/people",
      "targetFieldName": "people"
    },
    {
      "sourceFieldName": "/document/ProductID/organizations",
      "targetFieldName": "organizations"
    },
    {
      "sourceFieldName": "/document/ProductID/locations",
      "targetFieldName": "locations"
    },
    {
      "sourceFieldName": "/document/ProductID/keyphrases",
      "targetFieldName": "keyphrases"
    }
  ],
  "cache": null,
  "encryptionKey": null
}
```

Navigate to Cognitive Search, "**Overview**" and then the "**Indexes**" tab.<br>
Click on the newly-created index.

<img src="https://user-images.githubusercontent.com/44923999/226405949-7cc8aafa-4b24-44b0-90a6-2773ba0275ed.png" width="800" title="Snipped: March 20, 2023" />

Paste the following "**Query string**" value: `$top=1` and then click **Search**.<br>
Review "**Results**" content; example below:

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes('azuresql-index')/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "ProductID": "710",
      "Name": "Mountain Bike Socks, L",
      "ProductNumber": "SO-B909-L",
      "Color": "White",
      "StandardCost": "3.3963",
      "ListPrice": "9.5000",
      "Size": "L",
      "Weight": null,
      "ProductCategoryID": 27,
      "ProductModelID": 18,
      "SellStartDate": "2005-07-01T00:00:00Z",
      "SellEndDate": "2006-06-30T00:00:00Z",
      "DiscontinuedDate": null,
      "ThumbnailPhotoFileName": "no_image_available_small.gif",
      "rowguid": "161c035e-21b3-4e14-8e44-af508f35d80a",
      "ModifiedDate": "2008-03-11T10:01:36.827Z",
      "people": [],
      "organizations": [],
      "locations": [],
      "keyphrases": []
    }
  ]
}
```

LOREM IPSUM

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
* [Tips for AI enrichment in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-troubleshooting)
* [Debug an Azure Cognitive Search skillset in Azure portal](https://learn.microsoft.com/en-us/azure/search/cognitive-search-how-to-debug-skillset#debug-a-custom-skill-locally)
