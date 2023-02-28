# Data Acquisition: Mid-Stream Processing via Stream Analytics (WiP)

![image](https://user-images.githubusercontent.com/44923999/221875850-3e7ad7e9-4416-4dde-9c76-ea822f750db0.png)

## Use Case
This solution considers the following requirements:

* "We stream millions of messages per hour from an IoT Hub owned and controlled by another organization"
* "We want visibility into the live stream and the ability to troubleshoot in near real-time"
* "We want to route messages, mid-stream, to the appropriate destination table in Azure Data Explorer without using a second layer of Event Hubs"
* "We want to bake in minor transformations {e.g., JSON parsing, rounding, etc.}"
* "We want to make sure that the solution will "pick up where it left off" if there is a problem with the connection to Event Hub"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
* [**IoT Hub**](https://learn.microsoft.com/en-us/azure/iot-hub/)
* [Raspberry Pi Simulator](https://azure-samples.github.io/raspberry-pi-web-simulator/)
* [**Stream Analytics**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction)

## Proposed Solution
This solution will address requirements in two exercises:

* Exercise 1: Create Sample Stream
* Exercise 2: Configure Routing via Stream Analytics

-----

## Exercise 1: Create Sample Stream
In this exercise, we will create a stream of sample data and surface it to IoT Hub.

We will follow the instructions at https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal (loosely summarized below).

### Step 1: Lorem Ipsum
In this step, we will: 1) load sample data and 2) run a KQL query to perform an export to Blob Storage.

Load sample data as specified in [Quickstart: Ingest sample data into Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard).

<img src="https://user-images.githubusercontent.com/44923999/214618239-312d447b-230b-4ed2-acbb-2dbf2b520ce7.png" width="800" title="Snipped: January 25, 2023" />

```
.export to csv ( "https://rchaplers.blob.core.windows.net/stormevents;STORAGEACCOUNT_ACCESSKEY" )
with ( includeHeaders = "all" ) <| StormEvents
```

-----

## Reference

* [Quickstart - Create a Stream Analytics job by using the Azure portal](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal)
* [Raspberry Pi Azure IoT Web Simulator](https://azure-samples.github.io/raspberry-pi-web-simulator/)
* [Azure Data Explorer output from Azure Stream Analytics](https://learn.microsoft.com/en-us/azure/stream-analytics/azure-database-explorer-output)
* [Built-in Functions (Azure Stream Analytics) - Stream Analytics Query](https://learn.microsoft.com/en-us/stream-analytics-query/built-in-functions-azure-stream-analytics)
* [Troubleshoot Azure Stream Analytics using resource logs](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-job-diagnostic-logs)
