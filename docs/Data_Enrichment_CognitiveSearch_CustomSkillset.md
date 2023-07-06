# Data Enrichment: Cognitive Search, Custom "Get Data" Skillset

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d7ad1df6-3f08-4397-88b6-a4b7b67ea90d" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to enrich our Cognitive Search index with metadata from SQL"

## Proposed Solution
* Create a "Get Data" API using a Function App
* Add a custom skillset to the Cognitive Search index

## Solution Requirements
The proposed solution requires:
* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview), including dependencies:
  * [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
  * [Application Service](https://learn.microsoft.com/en-us/azure/app-service/)
  * [Storage Account](Infrastructure_StorageAccount.md)
* [**SQL**](https://learn.microsoft.com/en-us/azure/azure-sql) with [AdventureWorks sample data](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)
* [**Visual Studio**](https://visualstudio.microsoft.com/) with **Azure development** workload

-----

## Exercise 1: Create Project
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
            scsb.Password = "{SQL_ADMIN_PASSWORD";
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

### Step 4: Publish to Azure

In Visual Studio, right-click on the project and select **Publish** from the resulting drop-down menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ad817645-8678-4d8c-9b92-c63824a61162" width="800" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Where are you publishing...**" page, select "**Azure**" and then click **Next**.

<img src="https://user-images.githubusercontent.com/44923999/212152235-d37d84e9-9889-48cb-8d02-6d1b040e34e8.png" width="600" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Which Azure service...**" page, select "**Azure Function App (Windows)**" and then click **Next**.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b0f6a61e-b741-4b48-a3d6-eb2ac5f711a7" width="600" title="Snipped: July 6, 2023" />

On the **Publish** >> "**Select existing or...**" page, select your Function App and then click **Finish**.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2333abe2-d9e0-4674-84ca-aba15be28be9" width="600" title="Snipped: July 6, 2023" />

Back on the "...Publish" page, click **Publish** and confirm successful publication in the "**Output**" pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e31d913e-62ee-4d6f-a117-aac39e57d4c9" width="800" title="Snipped: July 6, 2023" />

### Step 5: Confirm Success

Open the Azure Portal and navigate to the **StormEvents** function

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d9e12021-7b36-4586-96f2-dca2b0e3ec98" width="800" title="Snipped: July 6, 2023" />







* Click "**Get Function URL**" and copy the value from the resulting pop-up

  <img src="https://user-images.githubusercontent.com/44923999/214117591-4d1bbf24-bcf0-48cf-a809-237779d209ea.png" width="800" title="Snipped: January 23, 2023" />

* Open a new tab on your browser and paste the copied URL... you can expect to see a JSON response with StormEvents data {abbreviated and prettified below}:

  ```
  {
    "Tables": [
      {
        "TableName": "Table_0",
        "Columns": [
          {
            "ColumnName": "StartTime",
            "DataType": "DateTime",
            "ColumnType": "datetime"
          },
          {
            "ColumnName": "EventType",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "State",
            "DataType": "String",
            "ColumnType": "string"
          }
        ],
        "Rows": [
          [
            "2007-01-01T00:00:00Z",
            "Thunderstorm Wind",
            "NORTH CAROLINA"
          ],
          [
            "2007-01-01T00:00:00Z",
            "Winter Storm",
            "WISCONSIN"
          ],
          [
            "2007-01-01T00:00:00Z",
            "Winter Storm",
            "WISCONSIN"
          ]
        ]
      },
      {
        "TableName": "Table_1",
        "Columns": [
          {
            "ColumnName": "Value",
            "DataType": "String",
            "ColumnType": "string"
          }
        ],
        "Rows": [
          [
            "{\"Visualization\":null,\"Title\":null,\"XColumn\":null,\"Series\":null,\"YColumns\":null,\"AnomalyColumns\":null,\"XTitle\":null,\"YTitle\":null,\"XAxis\":null,\"YAxis\":null,\"Legend\":null,\"YSplit\":null,\"Accumulate\":false,\"IsQuerySorted\":false,\"Kind\":null,\"Ymin\":\"NaN\",\"Ymax\":\"NaN\",\"Xmin\":null,\"Xmax\":null}"
          ]
        ]
      },
      {
        "TableName": "Table_2",
        "Columns": [
          {
            "ColumnName": "Timestamp",
            "DataType": "DateTime",
            "ColumnType": "datetime"
          },
          {
            "ColumnName": "Severity",
            "DataType": "Int32",
            "ColumnType": "int"
          },
          {
            "ColumnName": "SeverityName",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "StatusCode",
            "DataType": "Int32",
            "ColumnType": "int"
          },
          {
            "ColumnName": "StatusDescription",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "Count",
            "DataType": "Int32",
            "ColumnType": "int"
          },
          {
            "ColumnName": "RequestId",
            "DataType": "Guid",
            "ColumnType": "guid"
          },
          {
            "ColumnName": "ActivityId",
            "DataType": "Guid",
            "ColumnType": "guid"
          },
          {
            "ColumnName": "SubActivityId",
            "DataType": "Guid",
            "ColumnType": "guid"
          },
          {
            "ColumnName": "ClientActivityId",
            "DataType": "String",
            "ColumnType": "string"
          }
        ],
        "Rows": [
          [
            "2023-01-23T18:13:12.421014Z",
            4,
            "Info",
            0,
            "Query completed successfully",
            2,
            "be124417-f656-4ba2-a708-6092de5841f8",
            "be124417-f656-4ba2-a708-6092de5841f8",
            "38de4bc5-73f9-4448-8ec7-b028aa0968ac",
            "768f1424-d75f-448e-a78d-4788bbfd4dac"
          ],
          [
            "2023-01-23T18:13:12.421014Z",
            6,
            "Stats",
            0,
            "{\"ExecutionTime\":0.0,\"resource_usage\":{\"cache\":{\"memory\":{\"hits\":0,\"misses\":0,\"total\":0},\"disk\":{\"hits\":0,\"misses\":0,\"total\":0},\"shards\":{\"hot\":{\"hitbytes\":52739,\"missbytes\":0,\"retrievebytes\":0},\"cold\":{\"hitbytes\":0,\"missbytes\":0,\"retrievebytes\":0},\"bypassbytes\":0}},\"cpu\":{\"user\":\"00:00:00\",\"kernel\":\"00:00:00\",\"total cpu\":\"00:00:00\"},\"memory\":{\"peak_per_node\":1573200},\"network\":{\"inter_cluster_total_bytes\":2268,\"cross_cluster_total_bytes\":0}},\"input_dataset_statistics\":{\"extents\":{\"total\":1,\"scanned\":1,\"scanned_min_datetime\":\"2023-01-12T13:43:43.0092037Z\",\"scanned_max_datetime\":\"2023-01-12T13:43:43.0092037Z\"},\"rows\":{\"total\":59066,\"scanned\":59066},\"rowstores\":{\"scanned_rows\":0,\"scanned_values_size\":0},\"shards\":{\"queries_generic\":1,\"queries_specialized\":0}},\"dataset_statistics\":[{\"table_row_count\":3,\"table_size\":100}],\"cross_cluster_resource_usage\":{}}",
            1,
            "be124417-f656-4ba2-a708-6092de5841f8",
            "be124417-f656-4ba2-a708-6092de5841f8",
            "38de4bc5-73f9-4448-8ec7-b028aa0968ac",
            "768f1424-d75f-448e-a78d-4788bbfd4dac"
          ]
        ]
      },
      {
        "TableName": "Table_3",
        "Columns": [
          {
            "ColumnName": "Ordinal",
            "DataType": "Int64",
            "ColumnType": "long"
          },
          {
            "ColumnName": "Kind",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "Name",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "Id",
            "DataType": "String",
            "ColumnType": "string"
          },
          {
            "ColumnName": "PrettyName",
            "DataType": "String",
            "ColumnType": "string"
          }
        ],
        "Rows": [
          [
            0,
            "QueryResult",
            "PrimaryResult",
            "0e601b81-d776-473a-a8bf-20ad84770a8c",
            ""
          ],
          [
            1,
            "QueryProperties",
            "@ExtendedProperties",
            "4f4d08e9-8887-4a79-80b2-dd47310ce96f",
            ""
          ],
          [
            2,
            "QueryStatus",
            "QueryStatus",
            "00000000-0000-0000-0000-000000000000",
            ""
          ]
        ]
      }
    ]
  }
  ```

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
