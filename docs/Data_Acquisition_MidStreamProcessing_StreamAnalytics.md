# Data Acquisition: Mid-Stream Processing via Stream Analytics (WiP)

![image](https://user-images.githubusercontent.com/44923999/221875850-3e7ad7e9-4416-4dde-9c76-ea822f750db0.png)

## Use Case
This solution considers the following requirements:

* "We stream millions of messages per hour from an IoT Hub owned and controlled by another organization"
* "We want visibility into the live stream and the ability to troubleshoot in near real-time"
* "We want to route messages, mid-stream, to the appropriate table in Data Explorer without using a second layer of Event Hubs"
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
* Exercise 2: Create Stream Analytics Job

-----

## Exercise 1: Create Sample Stream
In this exercise, we will create a stream of sample data and surface it to IoT Hub.

We will follow the instructions at https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal<br>(loosely summarized below)

### Step 1: Create Device
Navigate to the IoT Hub and then "**Devices**" in the "**Device management**" grouping of the left-hand navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221903858-42e8284b-3178-4f1f-a7c7-49f07e12568d.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add Device**".

<img src="https://user-images.githubusercontent.com/44923999/221904124-ae190bef-f18b-432e-b8a0-81f73a80fbe8.png" width="800" title="Snipped: February 28, 2023" />

On the "**Create a device**" page, enter a "Device ID" value and then click **Save**.

<img src="https://user-images.githubusercontent.com/44923999/221904412-6676e526-9dad-4fbb-bb18-82f94211de66.png" width="800" title="Snipped: February 28, 2023" />

Click on the new Device link.

<img src="https://user-images.githubusercontent.com/44923999/221905410-1bbfc497-446c-4ffb-ab72-c22497927012.png" width="800" title="Snipped: February 28, 2023" />

Copy the "**Primary connection string**" value; example: `HostName=rchaplerih.azure-devices.net;DeviceId=Device001;SharedAccessKey=...`
Navigate to the Raspberry Pi Azure IoT Online Simulator (https://azure-samples.github.io/raspberry-pi-web-simulator/).

<img src="https://user-images.githubusercontent.com/44923999/221912699-cd996cda-a4f5-42ba-91ac-4f1cad2c3f58.png" width="800" title="Snipped: February 28, 2023" />

Update Line 15, `const connectionString = ...` with the copied "**Primary connection string**" value and then click **Run**.

## Exercise 2: Create Stream Analytics Job

Lorem Ipsum

-----

## Reference

* [Azure Data Explorer output from Azure Stream Analytics](https://learn.microsoft.com/en-us/azure/stream-analytics/azure-database-explorer-output)
* [Built-in Functions (Azure Stream Analytics) - Stream Analytics Query](https://learn.microsoft.com/en-us/stream-analytics-query/built-in-functions-azure-stream-analytics)
* [Troubleshoot Azure Stream Analytics using resource logs](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-job-diagnostic-logs)
