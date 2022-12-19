# Data Acquisition: Mid-Stream Processing

![image](https://user-images.githubusercontent.com/44923999/208464152-33914e21-5ae5-49fc-8a9a-dc7dddcf0339.png)

## Consider Requirements
This solution considers the following requirements:

* "We stream millions of messages per hour from an Event Hub owned and controlled by another organization"
* "Each message requires special, mid-stream handling {e.g., format translation, decompression, re-packing of JSON, etc.}"

## Prepare Infrastructure
This solution requires the following resources:

* [**Event Hubs**](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group ... one to mimic the untouchable source and a second to mimic post-processing
* Function Apps ... one to mock data flowing through the untouchable source and a second to hand mid-stream processing
* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)

## Exercise 1: Mock Untouchable Source

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
