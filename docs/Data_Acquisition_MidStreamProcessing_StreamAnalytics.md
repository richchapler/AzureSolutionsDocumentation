# Data Acquisition: Mid-Stream Processing via Stream Analytics (WiP)

![image](https://user-images.githubusercontent.com/44923999/221875850-3e7ad7e9-4416-4dde-9c76-ea822f750db0.png)

## Use Case
This solution considers the following requirements:

* "We stream millions of messages per hour from an IoT Hub owned and controlled by another organization"
* "We want to make sure that the solution will "pick up where it left off" if there is a problem with the connection to Event Hub"
* "We want visibility into the live stream and the ability to troubleshoot in near real-time"
* "We want to bake in minor transformations {e.g., JSON parsing, rounding, etc.}"
* "We want to route messages, mid-stream, to the appropriate table in Data Explorer without using a second layer of Event Hubs"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
* [**IoT Hub**](https://learn.microsoft.com/en-us/azure/iot-hub/)
* [**Log Analytics**](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
* [**Raspberry Pi Simulator**](https://azure-samples.github.io/raspberry-pi-web-simulator/)
* [**Stream Analytics**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Create Stream
* Exercise 2: Create Job
* Exercise 3: Route Data

-----

## Exercise 1: Create Stream
In this exercise, we will create a stream of sample data and surface it to IoT Hub.

We will follow the instructions at https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal<br>(loosely summarized below)

### Step 1: Create Device
Navigate to "**Devices**" in the "**Device management**" grouping of the IoT Hub navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221903858-42e8284b-3178-4f1f-a7c7-49f07e12568d.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add Device**".

<img src="https://user-images.githubusercontent.com/44923999/221904124-ae190bef-f18b-432e-b8a0-81f73a80fbe8.png" width="800" title="Snipped: February 28, 2023" />

On the "**Create a device**" page, enter a "Device ID" value and then click **Save**.

<img src="https://user-images.githubusercontent.com/44923999/221904412-6676e526-9dad-4fbb-bb18-82f94211de66.png" width="800" title="Snipped: February 28, 2023" />

Click on the new Device link.

<img src="https://user-images.githubusercontent.com/44923999/221905410-1bbfc497-446c-4ffb-ab72-c22497927012.png" width="800" title="Snipped: February 28, 2023" />

Copy the "**Primary connection string**" value; example: `HostName=rchaplerih.azure-devices.net;DeviceId=Device001;SharedAccessKey=...`

### Step 2: Activate Simulator

Navigate to the Raspberry Pi Azure IoT Online Simulator (https://azure-samples.github.io/raspberry-pi-web-simulator/).

<img src="https://user-images.githubusercontent.com/44923999/221912699-cd996cda-a4f5-42ba-91ac-4f1cad2c3f58.png" width="800" title="Snipped: February 28, 2023" />

Update Line 15, `const connectionString = ...` with the copied "**Primary connection string**" value and then click **Run**.

## Exercise 2: Configure Job

### Step 1: Add Job Input

Navigate to "**Inputs**" in the "**Job topology**" grouping of the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221920094-c96b4c8a-f5ca-4af3-88a1-bb2ba9b850a7.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add stream input**", select "**IoT Hub**" from the resulting dropdown menu, complete the resulting "**IoT Hub**" pop-out, and then click **Save**.

### Step 2: Create Destination Table

Navigate to **Query** in the **Data** grouping of the Data Explorer navigation pane.

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

### Step 3: Add Job Output

Navigate to "**Outputs**" in the "**Job topology**" grouping of the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221990599-36b1bbb3-1c80-4529-b6d9-c52b4760ae38.png" width="800" title="Snipped: February 28, 2023" />

Click "**+ Add**", select "**Azure Data Explorer**" from the resulting dropdown menu, complete the resulting "**Azure Data Explorer**" pop-out, and then click **Save**.

### Step 4: Start Job

Navigate to "**Overview**" on the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221994420-6a2b2577-bf84-487b-853c-70ff99ccef1d.png" width="800" title="Snipped: February 28, 2023" />

Click **Start**.

### Step 5: Test Query

Navigate to "**Query**" in the "**Job topology**" grouping of the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221994564-0b354243-e02d-4bdc-887c-53a3dac42179.png" width="800" title="Snipped: February 28, 2023" />

Paste the following query logic:
```
SELECT messageId, deviceId, IoTHub.ConnectionDeviceGenerationId generationId, temperature, humidity
INTO [rchaplerdec-rchaplerded]
FROM rchaplerih
```

Click "**Test query**" and confirm result.

### Step 6: Confirm Success

#### Job Diagram

Navigate to "**Job diagram...**" in the "**Developer tools**" grouping of the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221995347-df89e605-b331-42de-b6cf-395513efe4a1.png" width="800" title="Snipped: February 28, 2023" />

Select metrics "**Input Events**" and "**Output Events**", click **Apply**, then confirm the flow of events.

#### Data Explorer

Navigate to "**Query**" in the "**Data**" grouping of the Data Explorer navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/221996613-aa02268b-7b20-4da3-817f-8250728fe5f5.png" width="800" title="Snipped: February 28, 2023" />

Confirm output data by executing the following KQL: `t`

#### Troubleshooting

Navigate to "**Diagnostic settings**" in the "**Monitoring**" grouping of the Stream Analytics Job navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/222199724-dc5060c0-00fc-43d7-9d8f-feae09bb78a2.png" width="800" title="Snipped: March 1, 2023" />

Click "**+ Add diagnostic setting**"

<img src="https://user-images.githubusercontent.com/44923999/222199954-fe81df4d-5357-4317-84b0-cd43e2437e03.png" width="800" title="Snipped: March 1, 2023" />

Complete the form and click **Save**.<br>

<img src="https://user-images.githubusercontent.com/44923999/222200799-6915e965-4a26-42a8-9fba-d1b2ffcd39d4.png" width="800" title="Snipped: March 1, 2023" />

Navigate to "**Logs**" in the "**Monitoring**" grouping of the Stream Analytics Job navigation pane and browse canned queries.

## Exercise 3: Route Data
To demonstrate routing, we'll automate data movement based on the value of the Temperature field.

### Step 1: Create Tables and Job

Navigate to **Query** in the **Data** grouping of the Data Explorer navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/222198121-70ef4d8a-db34-415b-bac5-c4c472b53a36.png" width="800" title="Snipped: March 1, 2023" />

Execute the following KQL:

```
.create table temperature_gt27 (
    deviceId:string,
    temperature:double
    );
    
.create table temperature_ltoet27 (
    deviceId:string,
    temperature:double
    );
```

Lorem Ipsum

-----

## Reference

* [Azure Data Explorer output from Azure Stream Analytics](https://learn.microsoft.com/en-us/azure/stream-analytics/azure-database-explorer-output)
* [Built-in Functions (Azure Stream Analytics) - Stream Analytics Query](https://learn.microsoft.com/en-us/stream-analytics-query/built-in-functions-azure-stream-analytics)
* [Troubleshoot Azure Stream Analytics using resource logs](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-job-diagnostic-logs)
