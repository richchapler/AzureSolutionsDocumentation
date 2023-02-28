# Data Acquisition: Mid-Stream Processing via Function App

![image](https://user-images.githubusercontent.com/44923999/208464152-33914e21-5ae5-49fc-8a9a-dc7dddcf0339.png)

## Use Case

This solution considers the following requirements:

* "We stream millions of messages per hour from an Event Hub owned and controlled by another organization"
* "Each message requires special, mid-stream handling {e.g., format translation, decompression, re-packing of JSON, etc.}"

## Proposed Solution

This solution will address requirements in three steps:

* Mimic Source - For this exercise, we must loosely mirror the source Event Hub... in the real-world, we would just connect to the real source
* Process Mid-Stream - create a Function App with incoming Event Hub (source), processing logic {e.g. unpack JSON} and outgoing Event Hub (destination)
* Ingest Data - ingest data into Data Explorer

## Required Infrastructure

This solution requires the following resources:

* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](https://learn.microsoft.com/en-us/azure/event-hubs/)... one namespace with two event hubs (...incoming and ...outgoing, with corresponding Shared Acces Policies and Consumer Groups)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) configured to use [**Application Insights**](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) for monitoring
* [Visual Studio](https://visualstudio.microsoft.com/) with **Azure development** workload

## Exercise 1: Mimic Source

In this exercise, we will use a Function App to mock the flow of messages coming from the source Event Hub.

### Step 1: Create Visual Studio Project

* Open Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/201695714-2c2be122-0b23-42ef-8a47-f71a4bc3ac76.png" width="800" title="Snipped: November 14, 2022" />

* Click "**Create a new project**"

  <img src="https://user-images.githubusercontent.com/44923999/201696928-02adc19a-6cfe-45b9-8271-eb71da588f0d.png" width="800" title="Snipped: November 14, 2022" />

* On the "**Create a new project**" page, search for and select "**Azure Functions**", then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/208800336-d2a27a2b-b910-43b1-ac8f-bee628e8c8d9.png" width="800" title="Snipped: December 20, 2022" />

* Complete the "**Configure your new project**" form and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/208800514-a7a5a588-e44b-44a8-943d-846d6ac56f98.png" width="800" title="Snipped: December 20, 2022" />

* Complete the "**Additional information**" form:

  | Prompt | Entry |
  | ----- | ----- |
  | **Functions worker** | Select "**.NET 6.0 (Long Term Support)**" |
  | **Function** | Select "**Timer trigger**" |
  | **Use Azurite...** | Checked |
  | **Schedule** | Enter "0 */1 * * * *" (the CRON expression for every one minute) |

* Click **Create**

### Step 2: Install NuGet

* In this step, we will install Nuget to pre-empt errors when we update the logic

  <img src="https://user-images.githubusercontent.com/44923999/208803586-942b5d0f-b09d-4df3-a094-878f17ed8015.png" width="800" title="Snipped: December 20, 2022" />

* Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**"

  <img src="https://user-images.githubusercontent.com/44923999/208803810-34581b76-62b6-4d28-b5cf-5fcde3e45516.png" width="800" title="Snipped: December 20, 2022" />

* On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Microsoft.Azure.WebJobs.Extensions.EventHubs**"
* On the resulting pop-out, check project **DataAcquisition_MidStreamProcessing** and then click "**Install**"
* When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up

### Step 3: Update Logic

* Rename "Function1.cs" to "Exercise1.cs" and open

  <img src="https://user-images.githubusercontent.com/44923999/208928042-53491842-c66e-4501-a208-46ca5eadadb2.png" width="800" title="Snipped: December 21, 2022" />

* Replace the default code `[FunctionName("Function1")]` with `[FunctionName("Exercise1")]`
* Replace the default code `public void Run([TimerTrigger("*/1 * * * *")]TimerInfo myTimer, TraceWriter log)` with:

  ```
  public async Task Run(
    [TimerTrigger("*/1 * * * *")] TimerInfo timer
    , [EventHub(eventHubName:"rchaplereh-incoming", Connection="incoming")] IAsyncCollector<string> incoming
    , ILogger log)
  ```

  Logic Explained:

  * `async Task` ... provides for use of asynchronous calls
  * `[EventHub...` ... provides for **output** to Event Hub

* Replace the default code `log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");` with:

  ```
  string json = "{\"rows\":[{\"id\":\"" + Guid.NewGuid() + "\"},{\"id\":\"" + Guid.NewGuid() + "\"},{\"id\":\"" + Guid.NewGuid() + "\"}]}";
  log.LogInformation(json);
  await incoming.AddAsync(json);
  ```

  Logic Explained:

  * `string output = ...` generates a nested JSON string like `{"rows":[{"id":"5740c202-3ee8-43c0-8561-01a33506cb6f"},...]}`
  * `await theEventHub.AddAsync(output);` sends the JSON string to the Event Hub

### Step 4: Update "local.settings.json"

* Open "local.settings.json"

  <img src="https://user-images.githubusercontent.com/44923999/208928679-39b0a6f2-cc7d-4a87-b9a0-edac092dfc79.png" width="800" title="Snipped: December 21, 2022" />

* Modify the following JSON {e.g., replace placeholders like STORAGE_ACCOUNT_NAME with real values}

  ```
  {
    "IsEncrypted": false,
    "Values": {
      "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=STORAGE_ACCOUNT_NAME;AccountKey=STORAGE_ACCOUNT_KEY;EndpointSuffix=core.windows.net",

      "incoming": "Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME",

      "FUNCTIONS_WORKER_RUNTIME": "dotnet"
    }
  }
  ```

* Use modified JSON to replace default "local.settings.json" content

### Step 5: Confirm Success (local)

* In the Visual Studio menu-bar, click **Debug** >> "**Start Debugging**" and in the resulting Command window, monitor processing and confirm success

  <img src="https://user-images.githubusercontent.com/44923999/208929770-4bac3ce0-56ad-4f5c-a30b-09a5506be055.png" width="800" title="Snipped: December 21, 2022" />

* Navigate to the Event Hub and confirm **Incoming Messages**

  <img src="https://user-images.githubusercontent.com/44923999/208930352-4c6764b9-9221-4a03-ba8b-2478abd658ad.png" width="800" title="Snipped: December 21, 2022" />

### Step 6: Publish to Azure

* Return to Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/208931362-0c5fa72f-2451-4ad0-ac9d-6762d0f818b8.png" width="800" title="Snipped: December 19, 2022" />

* Right-click on the project and select Publish from the resulting drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/208524701-66855dbf-aa15-4166-823b-9a3ab7335312.png" width="600" title="Snipped: December 19, 2022" />

* On the **Publish** pop-up, **Target** tab, select "**Azure**" and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/208525720-9123df26-a749-469e-89bc-47a22ad8e997.png" width="600" title="Snipped: December 19, 2022" />

* On the **Publish** pop-up, "**Specific target**" tab, select "**Azure Function App (Windows)**" and then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/208526223-fab7fdcc-64a6-4e23-80d1-c3c20ae5c846.png" width="600" title="Snipped: December 19, 2022" />

* On the **Publish** pop-up, "**Functions instance**" tab, select your subscription, expand to and select your Function App, then click **Finish**

  <img src="https://user-images.githubusercontent.com/44923999/208931759-d1d257bc-e16e-4d00-a3d9-b8daee52d2fd.png" width="600" title="Snipped: December 21, 2022" />

* On the **Publish** pop-up, "**Finish**" tab, monitor publish profile creation progress and then click **Close**

  <img src="https://user-images.githubusercontent.com/44923999/208932581-fb373a70-0dcb-4da0-a8af-d9942ac010c1.png" width="800" title="Snipped: December 21, 2022" />

* On the "...Publish" page, click **Publish**

### Step 7: Configure Application Settings

* Navigate to the Function App, then **Configuration** in the **Settings** group of the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/208932948-df391957-b2b9-4865-8b07-9a5b6a7d47d1.png" width="800" title="Snipped: December 21, 2022" />

* Click "**+ New application setting**"

  <img src="https://user-images.githubusercontent.com/44923999/208933467-f54ae533-de52-403d-a897-858669261cb9.png" width="800" title="Snipped: December 21, 2022" />

* Complete the resulting "**Add/Edit application setting**" pop-out, including:

  | Prompt | Entry |
  | --------- | ------------------------------------------------------------ |
  | **Name** | Enter "**incoming**" |
  | **Value** | Modify and enter the following string:<br> `Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME` |

* Click **OK** to close the pop-out and then **Save** on the **Configuration** page, "**Application setting**" tab

### Step 8: Confirm Success (Azure)

* Navigate to function Exercise1, then **Monitor** in the **Developer** group of the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/208934008-f664ffb1-e21e-4620-a695-978f452cb061.png" width="800" title="Snipped: December 21, 2022" />

* Confirm **Success** messages
* Navigate to the Event Hub and confirm Incoming Messages

## Exercise 2: Process Mid-Stream

<img src="https://user-images.githubusercontent.com/44923999/208710138-a6f25c3e-88a9-4195-aebf-54c8016d5b5c.png" width="800" title="Snipped: December 20, 2022" />

Were we to ingest the source data directly {i.e., without mid-stream processing}, our best strategy would be to pull it in Data Format "TXT", and then convert to dynamic / parse with KQL in Data Explorer. This, of course, implies that data is accessible at the level described in our simple first exercise. Mid-stream processing addresses far more than JSON parsing; for example: encoding, compression, custom formats, etc.

In Exercise 2, we will create a Function App with incoming Event Hub (source), processing logic {e.g. unpack JSON} and outgoing Event Hub (destination).

### Step 1: Add New Azure Function

* Open Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/210417129-33cdc7bf-7d56-4ab3-b261-bd959bdae5da.png" width="800" title="Snipped: January 3, 2023" />

* In Solution Explorer, right-click on the project, then **Add** >> "**New Azure Function**" on the resulting drop-down menus

  <img src="https://user-images.githubusercontent.com/44923999/210417520-b45a481a-fdaf-4fb5-bdb1-c1ba31763288.png" width="600" title="Snipped: January 3, 2023" />

* On the "**Add New Item...**" pop-up form, enter a **Name** and then click **Add**

  <img src="https://user-images.githubusercontent.com/44923999/210417898-465add07-8fe2-4760-b504-03caf5a061ff.png" width="600" title="Snipped: January 3, 2023" />

* Complete the resulting "**New Azure Function...**" pop-up form, select "Event Hub trigger" and enter the following values:

  | Prompt | Entry |
  | ----- | ----- |
  | **Connection string setting name** | Enter "**incoming**" |
  | **Event Hub name** | Modify and enter "**EVENTHUB_NAME**" |

* Click **Add**

### Step 2: Update Logic

* Open "Exercise.cs"
  
  <img src="https://user-images.githubusercontent.com/44923999/210444860-9b64264d-ceee-4231-979d-5180c5d2a4f2.png" width="800" title="Snipped: January 3, 2023" />

* Replace default code with:

  ```
  using Azure.Messaging.EventHubs;
  using Microsoft.Azure.WebJobs;
  using Microsoft.Extensions.Logging;
  using System.Text.Json;
  using System.Threading.Tasks;

  namespace DataAcquisition_MidStreamProcessing
  {
    public static class Exercise2
    {
      [FunctionName("Exercise2")]
      public static async Task Run(
        [EventHubTrigger(eventHubName: "rchaplereh-incoming", Connection = "incoming")] EventData[] incoming,
        [EventHub(eventHubName: "rchaplereh-outgoing", Connection = "outgoing")] IAsyncCollector<string> outgoing,
        ILogger log
        )
      {
        foreach (EventData ed in incoming)
        {
          data d = JsonSerializer.Deserialize<data>(ed.EventBody);

          foreach (row r in d.rows)
          {
            log.LogInformation(JsonSerializer.Serialize(r));

            await outgoing.AddAsync(JsonSerializer.Serialize(r));
          }
        }
      }
    }

    public class data { public row[] rows { get; set; } }
    public class row { public string id { get; set; } }
  }
  ```

### Step 3: Update "local.settings.json"

* Open "local.settings.json"

  <img src="https://user-images.githubusercontent.com/44923999/210446109-49a0c8ba-a07f-4f36-aaec-a9cd92d58cc5.png" width="800" title="Snipped: January 3, 2023" />

* Modify the following JSON {e.g., replace placeholders like STORAGE_ACCOUNT_NAME with real values}

  ```
  {
    "IsEncrypted": false,
    "Values": {
      "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=STORAGE_ACCOUNT_NAME;AccountKey=STORAGE_ACCOUNT_KEY;EndpointSuffix=core.windows.net",

      "incoming": "Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME",

      "outgoing": "Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME",

      "FUNCTIONS_WORKER_RUNTIME": "dotnet"
    }
  }
  ```

* Use modified JSON to replace default "local.settings.json" content

### Step 4: Confirm Success (local)

* In the Visual Studio menu-bar, click **Debug** >> "**Start Debugging**" and in the resulting Command window, monitor processing and confirm success

  <img src="https://user-images.githubusercontent.com/44923999/210446414-773ec9a3-f9f0-416f-8f51-6ff0a785ae83.png" width="800" title="Snipped: January 3, 2023" />

  _Notice that both Exercise1 and Exercise are running_

* Navigate to the Event Hub and confirm both incoming and outgoing messages

  <img src="https://user-images.githubusercontent.com/44923999/210447013-9338a4b7-d04f-4241-b386-828db0f3a38a.png" width="800" title="Snipped: January 3, 2023" />

### Step 5: Publish to Azure

* Return to Visual Studio

  <img src="https://user-images.githubusercontent.com/44923999/208931362-0c5fa72f-2451-4ad0-ac9d-6762d0f818b8.png" width="800" title="Snipped: December 19, 2022" />

* Right-click on the project and select Publish from the resulting drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/210577539-91370965-6fd3-44b4-812b-2bcba51bb6ef.png" width="800" title="Snipped: January 4, 2023" />

* On the "...Publish" page, click **Publish**

### Step 6: Configure Application Settings

* Navigate to the Function App, then **Configuration** in the **Settings** group of the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/208932948-df391957-b2b9-4865-8b07-9a5b6a7d47d1.png" width="800" title="Snipped: December 21, 2022" />

* Click "**+ New application setting**"

  <img src="https://user-images.githubusercontent.com/44923999/210579151-63d36d51-1a8d-41d7-824a-a321e9eeb387.png" width="800" title="Snipped: January 4, 2023" />

* Complete the resulting "**Add/Edit application setting**" pop-out, including:

  | Prompt | Entry |
  | ----- | ----- |
  | **Name** | Enter "**outgoing**" |
  | **Value** | Modify and enter the following string:<br> `Endpoint=sb://EVENTHUB_NAMESPACE_NAME.servicebus.windows.net/;SharedAccessKeyName=EVENTHUB_SHAREDACCESSPOLICY_NAME;SharedAccessKey=EVENTHUB_SHAREDACCESSPOLICY_KEY;EntityPath=EVENTHUB_NAME` |

* Click **OK** to close the pop-out and then **Save** on the **Configuration** page, "**Application setting**" tab

### Step 7: Confirm Success (Azure)

* Navigate to function Exercise2, then **Monitor** in the **Developer** group of the left-hand navigation pane
* Confirm **Success** messages
* Navigate to the Event Hub and confirm Incoming Messages

## Ingest Data

In this quick exercise, we will ingest data from the outgoing Event Hub into Data Explorer.

* Navigate to https://dataexplorer.azure.com/home, then click **Data** in the left-hand navigation pane

  <img src="https://user-images.githubusercontent.com/44923999/210590504-6bef6af9-cfc8-4c09-b0a8-21eaf3d11133.png" width="800" title="Snipped: January 4, 2023" />

* Click **Ingest**

  <img src="https://user-images.githubusercontent.com/44923999/210591461-d9128620-8715-4c7c-bad6-276321cf5593.png" width="800" title="Snipped: January 4, 2023" />

* Complete the resulting "**Ingest data**" >> "**Destination**" form, including an appropriate name for the destination table
* Click "**Next: Source**"

  <img src="https://user-images.githubusercontent.com/44923999/210592005-3623ffb8-78b0-4242-ac91-945ebf2035a6.png" width="800" title="Snipped: January 4, 2023" />

* Complete the resulting "**Ingest data**" >> "**Source**" form, including those values appropriate to the outgoing Event Hub
* Click "**Next: Schema**"

  <img src="https://user-images.githubusercontent.com/44923999/210593077-99e8633d-3fdf-4521-bfc0-2eb01d4a85e1.png" width="800" title="Snipped: January 4, 2023" />

* Complete the resulting "**Ingest data**" >> "**Schema**" form, including selection of data format **JSON**
* Click "**Next: Start ingestion**"

  <img src="https://user-images.githubusercontent.com/44923999/210593527-d6471456-e4e9-4bcb-8033-e2283e3f13a8.png" width="800" title="Snipped: January 4, 2023" />

* Confirm success and then click **Close**

  <img src="https://user-images.githubusercontent.com/44923999/210594399-6c3ab17b-003d-4eaa-8272-0a916b8a5519.png" width="800" title="Snipped: January 4, 2023" />

* Navigate to Query and run KQL {e.g., `t | take 5`} to confirm successful ingestion of data
