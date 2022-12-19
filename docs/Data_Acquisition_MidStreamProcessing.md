# Data Acquisition: Mid-Stream Processing

![image](https://user-images.githubusercontent.com/44923999/208464152-33914e21-5ae5-49fc-8a9a-dc7dddcf0339.png)

## Use Case
This solution considers the following requirements:

* "We stream millions of messages per hour from an Event Hub owned and controlled by another organization"
* "Each message requires special, mid-stream handling {e.g., format translation, decompression, re-packing of JSON, etc.}"

## Prepare Infrastructure
This solution requires the following resources:

* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Event Hubs**](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group ... one to mimic the untouchable source and a second to mimic post-processing
* Function Apps ... one to mock data flowing through the untouchable source and a second to hand mid-stream processing
* [Visual Studio](https://visualstudio.microsoft.com/) with **Azure development** workload

## Exercise 1: Mock Untouchable Source
In this exercise, we will use a Function App to mock the flow of messages coming from the source Event Hub.

### Step 1: Create Visual Studio Project

* Open Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/201695714-2c2be122-0b23-42ef-8a47-f71a4bc3ac76.png" width="800" title="Snipped: November 14, 2022" />

* Click "**Create a new project**"

  <img src="https://user-images.githubusercontent.com/44923999/201696928-02adc19a-6cfe-45b9-8271-eb71da588f0d.png" width="800" title="Snipped: November 14, 2022" />

* On the "**Create a new project**" page, search for and select "Azure Functions", then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/208470843-1698307e-c955-4c60-9173-798460baa275.png" width="800" title="Snipped: December 19, 2022" />

* Complete the "**Configure your new project**" form and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/201709567-ea5c59be-014e-4f38-9781-582afd44edea.png" width="800" title="Snipped: November 14, 2022" />

* Complete the "**Additional information**" form, including:

  | Prompt | Entry |
  | ------ | ----- |
  | **Functions worker** | Select "**.NET 6.0 (Long Term Support)**" |
  | **Function** | Select "**Timer trigger**" |
  | **Use Azurite...** | Checked |
  | **Schedule** | Enter "*/1 * * * *" (the CRON expression for every one minute) |

* Click **Create**

### Step 2: "Function1.cs" Logic, Update Method

* Open "Function1.cs"

  <img src="https://user-images.githubusercontent.com/44923999/208492132-99ce75ef-18f5-416a-8852-c34d37d35a09.png" width="800" title="Snipped: December 19, 2022" />

* Replace the default code `public void Run([TimerTrigger("*/1 * * * *")]TimerInfo myTimer, TraceWriter log)` with:

  ```
  public async Task Run(
    [TimerTrigger("*/1 * * * *")] TimerInfo theTimer,
    [EventHub("dest", Connection = "EventHubConnectionAppSetting")] IAsyncCollector<string> theEventHub,
    ILogger theLogger)
  ```

  Logic Explained:

  * `async Task` ... provides for use of asynchronous calls
  * `[EventHub...` ... provides for output to Event Hub (for later Data Explorer ingestion)

### Step 3: Install NuGet

* In the "**Error List**", you will notice errors like "The type or namespace 'EventHub' could not be found..."; these must be resolved by adding a NuGet Package

  <img src="https://user-images.githubusercontent.com/44923999/208492582-5cc800c9-12f4-4ec6-9c6b-0431d56cfb4e.png" width="800" title="Snipped: December 19, 2022" />

* Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**"

  <img src="https://user-images.githubusercontent.com/44923999/208492799-a3c55385-e5f7-48be-843f-ee4a5efc65e9.png" width="800" title="Snipped: December 19, 2022" />

* On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Microsoft.Azure.WebJobs.Extensions.EventHubs**"
* On the resulting pop-out, check project **MidStreamProcessing** and then click **Install**
* When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up

### Step 4: Update "Function1.cs"

* Return to "Function1.cs"

  <img src="https://user-images.githubusercontent.com/44923999/208499718-fb7658ab-5dff-4d8d-9384-752f62a120f7.png" width="800" title="Snipped: December 19, 2022" />

* Replace the default code `log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");` with:

  ```
  string output = "{\"rows\":[{\"id\":"+Guid.NewGuid()+ "},{\"id\":"+Guid.NewGuid()+"},{\"id\":"+Guid.NewGuid()+"}]}";

  theLogger.LogInformation(output);

  await theEventHub.AddAsync(output);
  ```

  Logic Explained:

  * `string output = ...` generates a nested JSON string like `{"rows":[{"id":5740c202-3ee8-43c0-8561-01a33506cb6f},{"id":78ec5ee8-95f1-4189-85db-d14a53916566},{"id":8fc61f2f-db87-4424-8860-d553983f908f}]}`
  * `await theEventHub.AddAsync(output);` sends the JSON string to the Event Hub

### Step 5: Update "local.settings.json"

* Open "local.settings.json"

  <img src="https://user-images.githubusercontent.com/44923999/208500134-9dcf4c03-4226-4ccb-b5af-32af5588b886.png" width="800" title="Snipped: December 19, 2022" />

* Modify the following JSON {e.g., replace placeholders like STORAGE_ACCOUNT_NAME with real values} and then replace default "local.settings.json" content

  ```
  {
    "IsEncrypted": false,
    "Values": {
      "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=STORAGE_ACCOUNT_NAME;AccountKey=STORAGE_ACCOUNT_KEY;EndpointSuffix=core.windows.net",
      "EventHubConnectionAppSetting": "Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME",
      "FUNCTIONS_WORKER_RUNTIME": "dotnet"
    }
  }
  ```

### Step 6: Confirm Success

* In the Visual Studio menu-bar, click **Debug** >> "**Start Debugging**"

  <img src="https://user-images.githubusercontent.com/44923999/208502174-2084dcfc-d5d2-48b6-88f2-cabf2c3a39f0.png" width="800" title="Snipped: December 19, 2022" />

* In the resulting Command window, monitor processing and confirm success

* Navigate to the Event Hub





# Delete Me

  _Note: When you publish to an Azure Function App, remember that "local.settings.json" will not be published directly... you will need to update the function app with environment settings directly._


```
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using System;
using System.Threading.Tasks;

namespace Event_Hub_Message_Generator
{
    public class Function1
    {
        [FunctionName("Function1")]
        public async Task Run(
            [TimerTrigger("*/1 * * * *")] TimerInfo theTimer,
            [EventHub("dest", Connection = "EventHubConnectionAppSetting")] IAsyncCollector<string> theEventHub,
            ILogger theLogger)
        {
            string output = "{\"rows\":[{\"id\":"+Guid.NewGuid()+ "},{\"id\":"+Guid.NewGuid()+"},{\"id\":"+Guid.NewGuid()+"}]}";

            theLogger.LogInformation(output);

            await theEventHub.AddAsync(output);
        }
    }
}
```
