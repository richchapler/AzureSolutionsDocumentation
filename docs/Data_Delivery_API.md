# Surface Data with an API
_Note: This documentation is a prelude to a larger, future conversation about Data Monetization_

![image](https://user-images.githubusercontent.com/44923999/214113810-5d6ffd3c-0b84-4f3b-9ac9-266ef1fa3302.png)

## Use Case
This solution considers the following requirements:

* "We have customer-specific data that we want to share via API"

## Proposed Solution
This solution will address requirements in one exercise:

*	Exercise 1: Create an API using Function App, Data Explorer and StormEvents sample data

## Required Infrastructure
This solution requires the following resources:

* [**API Management**](https://learn.microsoft.com/en-us/azure/api-management/)
* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md) (with StormEvents sample data)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured to use [**Application Insights**](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) for monitoring
* [**Visual Studio**](https://visualstudio.microsoft.com/) with **Azure development** workload

## Exercise 1: Create API
In this exercise, we will create an API using Function App, Data Explorer and StormEvents sample data.

### Step 1: Create Visual Studio Project

* Open Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/212137484-599c9cd8-5e0e-46b1-818d-a3a008fecd5b.png" width="600" title="Snipped: January 12, 2023" />

* Click "**Create a new project**"

  <img src="https://user-images.githubusercontent.com/44923999/212137783-9ee34157-17fc-4364-a2d1-d572afbb4d8b.png" width="600" title="Snipped: January 12, 2023" />

* On the "**Create a new project**" page, search for and select "**Azure Functions**", then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/212138137-5eb64402-c7dd-4b35-8285-d0ffd527f2f9.png" width="600" title="Snipped: January 12, 2023" />

* Complete the "**Configure your new project**" form and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/212139114-5874d64a-42dc-449f-8635-d90d2cd319c8.png" width="600" title="Snipped: January 12, 2023" />

* Complete the "**Additional information**" form:

  | Prompt | Entry |
  | ----- | ----- |
  | **Functions worker** | Select "**.NET 6.0 (Long Term Support)**" |
  | **Function** | Select "**Http trigger with OpenAPI**" |
  | **Use Azurite...** | Checked |
  | **Authorization level** | Confirm default "Function" |

* Click **Create**

### Step 2: Install NuGet

* In this step, we will install Nuget to pre-empt errors when we update the logic

  <img src="https://user-images.githubusercontent.com/44923999/212140679-25ac45f7-34e2-4c12-8c60-a7cea234fb2d.png" width="800" title="Snipped: January 12, 2023" />

* Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**"

  <img src="https://user-images.githubusercontent.com/44923999/212140965-c3691ba9-69fe-4d0b-9035-37acac31605b.png" width="800" title="Snipped: January 12, 2023" />

* On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Microsoft.Azure.Kusto.Data**"
* On the resulting pop-out, check project **DataMonetization** and then click "**Install**"
* When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up

  <img src="https://user-images.githubusercontent.com/44923999/212141406-3d1bbf08-1259-4b4c-9a0d-241e0fa72f1b.png" width="800" title="Snipped: January 12, 2023" />

* Navigate to the **Updates** tab, check "**Select all packages**" and then click **Update** (as applicable)
* When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up

### Step 3: Update Logic

* Rename "Function1.cs" to "StormEvents.cs" and open

  <img src="https://user-images.githubusercontent.com/44923999/208928042-53491842-c66e-4501-a208-46ca5eadadb2.png" width="800" title="Snipped: January 12, 2023" />

* Replace the default code with:

  ```
  using Kusto.Cloud.Platform.Data;
  using Kusto.Data.Common;
  using Microsoft.AspNetCore.Http;
  using Microsoft.AspNetCore.Mvc;
  using Microsoft.Azure.WebJobs;
  using Microsoft.Azure.WebJobs.Extensions.Http;
  using Microsoft.Azure.WebJobs.Extensions.OpenApi.Core.Attributes;
  using Microsoft.Azure.WebJobs.Extensions.OpenApi.Core.Enums;
  using Microsoft.Extensions.Logging;
  using Microsoft.OpenApi.Models;
  using System;
  using System.Net;
  using System.Threading.Tasks;

  namespace StormEvents
  {
      public class StormEvents
      {
          private readonly ILogger<StormEvents> _logger;

          public StormEvents(ILogger<StormEvents> log) { _logger = log; }

          [FunctionName("StormEvents")]
          [OpenApiOperation(operationId: "Run", tags: new[] { "name" })]
          [OpenApiSecurity("function_key", SecuritySchemeType.ApiKey, Name = "code", In = OpenApiSecurityLocationType.Query)]
          [OpenApiParameter(name: "name", In = ParameterLocation.Query, Required = true, Type = typeof(string), Description = "The **Name** parameter")]
          [OpenApiResponseWithBody(statusCode: HttpStatusCode.OK, contentType: "text/plain", bodyType: typeof(string), Description = "This is the StormEvent data!")]

          public async Task<IActionResult> Run(
              [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req)
          {
              var kcsb = new Kusto.Data.KustoConnectionStringBuilder(
                  connectionString: "https://rchaplerdec.westus3.kusto.windows.net").WithAadApplicationKeyAuthentication(
                      applicationClientId: "21e11b4a-d067-40e0-9ed2-47f4a60c0218", // rchaplerar-c1
                      applicationKey: "pY88Q~JLYM_zxfgdec9NrbgsojBCUr7UXIn1ccba",
                      authority: "16b3c013-d300-468d-ac64-7eda0820b6d3"
                  );

              try
              {
                  var cqp = Kusto.Data.Net.Client.KustoClientFactory.CreateCslQueryProvider(kcsb);

                  var q = "StormEvents | limit 3 | project StartTime, EventType, State";
                  var crp = new ClientRequestProperties() { ClientRequestId = Guid.NewGuid().ToString() };

                  var result = cqp.ExecuteQuery(databaseName: "Customer1", query: q, properties: crp).ToJsonString();

                  return new OkObjectResult(result);
              }
              catch (Exception e)
              {
                  return new OkObjectResult(e.ToString());
              }
          }
      }
  }
  ```

  Logic Explained:

  * `using Kusto.Cloud.Platform.Data` and `using Kusto.Data.Common`... necessary to execute Data Explorer queries
  * [KustoConnectionStringBuilder](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/connection-strings/kusto)
  * `var q = "StormEvents | limit 3...` limits the result to just three rows to keep the exercise simple 

### Step 4: Publish to Azure

* Continue in Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/212151843-6221cfc1-86cf-429e-a2a1-dc835102e989.png" width="800" title="Snipped: January 12, 2023" />

* Right-click on the project and select **Publish** from the resulting drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/212152235-d37d84e9-9889-48cb-8d02-6d1b040e34e8.png" width="600" title="Snipped: January 12, 2023" />

* On the **Publish** >> "**Where are you publishing...**" page, select "**Azure**" and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/212152644-c26e3467-4499-4ce4-946b-9b7ed376b3b3.png" width="600" title="Snipped: January 12, 2023" />

* On the **Publish** >> "**Which Azure service...**" page, select "**Azure Function App (Windows)**" and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/212153003-da72ba24-8d5b-4929-b187-913e0ba620cc.png" width="600" title="Snipped: January 12, 2023" />

* On the **Publish** >> "**Select existing or...**" page, select your Function App and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/212163888-82580072-ae94-4825-95f4-43f18fcc34c7.png" width="600" title="Snipped: January 12, 2023" />

* On the **Publish** >> "**Enable API consumption...**" page, expand your instance of **API Management**, select "**echo-api**" and then click **Finish**

  <img src="https://user-images.githubusercontent.com/44923999/212164072-b11eafdb-f4b5-4ac3-8f5c-bab9f33340ae.png" width="600" title="Snipped: January 12, 2023" />

* On the **Publish profile creation progress**" page, confirm success and then click **Close**

  <img src="https://user-images.githubusercontent.com/44923999/212164829-7fd5ee7a-2f7a-44d6-acef-cb3cd4568fda.png" width="800" title="Snipped: January 12, 2023" />

* Back on the "...Publish" page, click **Publish** and confirm successful publication

### Step 5: Confirm Success

* Open the Azure Portal and navigate to the **StormEvents** function

  <img src="https://user-images.githubusercontent.com/44923999/212186832-c9f2d533-de7f-4f8b-85ab-5043c100619d.png" width="800" title="Snipped: January 12, 2023" />

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

_Note: This JSON is not likely your finally desired result; additional massaging will be required_

--------------------------

## Reference

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-the-sdk

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-kusto-data

> https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard

> https://stackoverflow.com/questions/53397728/kusto-query-from-c-sharp
