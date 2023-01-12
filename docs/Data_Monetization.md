# Data Monetization
_aka "Analytics as a Service", "AaaS"_

![image](https://user-images.githubusercontent.com/44923999/211827398-33d50c55-7fad-42df-999a-78a7a6b18026.png)

## Use Case
This solution considers the following requirements:

* "We have customer-specific data that we want to share with customers securely"
* "We want to monetize the analysis and data shared with customers"

## Proposed Solution
This solution will address requirements in three steps:

*	Exercise 1: Surface data via API (Function App GET customers, products, etc.) 
*	Exercise 2: Surface API via APIM (calls from customers will come through API for security, organization, throttling, “subscription model”, etc.) … http://api.blah.com/customer?id=7
*	Exercise 3: APIM Monetization? … with Stripe?
*	Exercise 4: Subscribe to API in APIM (persona: customer)
*	Exercise 5: Power BI template that uses API

## Required Infrastructure
This solution requires the following resources:

* [Application Registration](Infrastructure_ApplicationRegistration.md) ... two instances for "Customer1" and "Customer2"
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and two [**databases**](Infrastructure_DataExplorer_Database.md) ("Customer1" and "Customer2", both with StormEvents sample data)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured to use [**Application Insights**](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) for monitoring
* [Visual Studio](https://visualstudio.microsoft.com/) with **Azure development** workload


Create Function with "Http trigger with OpenAPI" to make an API

## Reference

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-the-sdk

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/netfx/about-kusto-data

> https://learn.microsoft.com/en-us/azure/data-explorer/kusto/api/connection-strings/kusto

> https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard

> https://stackoverflow.com/questions/53397728/kusto-query-from-c-sharp

## Code

```
using Kusto.Cloud.Platform.Data;
using Kusto.Data.Common;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;

namespace DataMonetization
{
    public static class StormEvents
    {
        [FunctionName("StormEvents")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
            ILogger log)
        {

            //string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            //string name = req.Query["name"]; // replace with query parameter?

            //dynamic data = JsonConvert.DeserializeObject(requestBody);

            var kcsb = new Kusto.Data.KustoConnectionStringBuilder(
                connectionString: "https://rchaplerdec.westus3.kusto.windows.net").WithAadApplicationKeyAuthentication(
                    applicationClientId: "21e11b4a-d067-40e0-9ed2-47f4a60c0218", // rchaplerar-c1
                    applicationKey: "pY88Q~JLYM_zxfgdec9NrbgsojBCUr7UXIn1ccba",
                    authority: "16b3c013-d300-468d-ac64-7eda0820b6d3"
                );

            try
            {
                string x = "";

                using (var cqp = Kusto.Data.Net.Client.KustoClientFactory.CreateCslQueryProvider(kcsb))
                {
                    var q = "StormEvents | limit 10 | project StartTime, EventType, State | as SampleRecords";
                    var crp = new ClientRequestProperties() { ClientRequestId = Guid.NewGuid().ToString() };

                    using (var reader = cqp.ExecuteQuery(databaseName: "Customer1", query: q, properties: crp))
                    {
                        while (reader.Read())
                        {
                            DateTime time = reader.GetDateTime(0);
                            string type = reader.GetString(1);
                            string state = reader.GetString(2);

                            x += time + "|" + type + "|" + state + "|| ";
                        }
                    }
                }

                return new OkObjectResult(x);
            }
            catch (Exception e)
            {
                return new OkObjectResult(e.ToString());
            }
        }
    }
}
```
