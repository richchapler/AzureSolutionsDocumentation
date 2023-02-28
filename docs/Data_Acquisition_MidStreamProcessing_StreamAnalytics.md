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

* Exercise 1: Create Stream
* Exercise 2: Create Job

-----

## Exercise 1: Create Stream
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

## Exercise 2: Configure Job
### Stream Analytics Job, Add Input
Navigate to theStream Analytics Job and then "**Inputs**" in the "**Job topology**" grouping of the left-hand navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221920094-c96b4c8a-f5ca-4af3-88a1-bb2ba9b850a7.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add stream input**", select "**IoT Hub**" from the resulting dropdown menu, complete the resulting "**IoT Hub**" pop-out, and then click **Save**.

### Data Explorer, Create Destination Table
Navigate to Data Explorer and then **Query** in the **Data** grouping of the left-hand navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221956094-af2d8851-25d6-4926-a42a-300b1403120d.png" width="800" title="Snipped: February 28, 2023" />

Execute the following KQL:

```
.create table t (
    messageId:int,
    deviceId:string,
    generationId:string,
    temperature:double,
    humidity:double
    )
```

### Stream Analytics Job, Add Output
Navigate to "**Outputs**" in the "**Job topology**" grouping of the left-hand navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221955364-58c61210-4ac2-45b8-8598-1829f5fb9194.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add**", select "**Azure Data Explorer**" from the resulting dropdown menu, complete the resulting "**Azure Data Explorer**" pop-out, and then click **Save**.

```
SELECT messageId, deviceId, IoTHub.ConnectionDeviceGenerationId generationId, temperature, humidity
INTO [rchaplerdec-rchaplerded-temp-gt27]
FROM rchaplerih
WHERE Temperature > 27; -- route #1

SELECT messageId, deviceId, IoTHub.ConnectionDeviceGenerationId generationId, temperature, humidity
INTO [rchaplerdec-rchaplerded-temp-ltoet27]
FROM rchaplerih
WHERE Temperature <= 27 -- route #2
```

![image](https://user-images.githubusercontent.com/44923999/221941546-8255ac64-422d-436d-a006-4839a0f36b6e.png)

-----

## Reference

* [Azure Data Explorer output from Azure Stream Analytics](https://learn.microsoft.com/en-us/azure/stream-analytics/azure-database-explorer-output)
* [Built-in Functions (Azure Stream Analytics) - Stream Analytics Query](https://learn.microsoft.com/en-us/stream-analytics-query/built-in-functions-azure-stream-analytics)
* [Troubleshoot Azure Stream Analytics using resource logs](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-job-diagnostic-logs)
