## Data Acquisition... from IoT Simulation

![image](https://user-images.githubusercontent.com/44923999/187276879-6d8fffff-fb47-405a-ad53-bbc0dee92665.png)

This use case considers requirement statements like:

* "We are working hard to bring IoT online, but it is going to take time... we want to begin working on analytics before we have real data"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**IoT Central**](Infrastructure_IoTCentral.md)

### Step 2: Create Device Template
In this step, we will create a Device Template.

Complete the following steps:

  <img src="https://user-images.githubusercontent.com/44923999/187228017-7d569c6d-475b-4aad-af30-92b933e48c94.png" width="800" title="Snipped: August 29, 2022" />

* From the Azure Portal, **IoT Central Application**, **Overview** page, click on the "**IoT Central Application URL**"
* Select "**Diagnostic Settings**" in the **Monitoring** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/187228762-41bcae98-f945-49a5-afc2-c1affa8d783e.png" width="800" title="Snipped: August 29, 2022" />

* Select "**Device templates**" in the **Connect** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/187229096-7faf99db-baae-4fd2-bf2e-72e84f34623e.png" width="800" title="Snipped: August 29, 2022" />

* On the resulting "**Device templates**" page, click "**Create a device template**"

  <img src="https://user-images.githubusercontent.com/44923999/187230571-3bda18b2-2035-41c3-9e90-c183eaf586b6.png" width="800" title="Snipped: August 29, 2022" />

* On the resulting "**Device templates**" > "**Create a device template**" > "**Select type**" page, select "**IoT device**" and then click "**Next: Customize**"

  <img src="https://user-images.githubusercontent.com/44923999/187232369-19cdb883-35d6-4f92-b75d-8a0664de5994.png" width="800" title="Snipped: August 29, 2022" />

* On the resulting "**Device templates**" > "**Create a device template**" > "**Customize**" page, enter "Pump" and then click "**Next: Review**"

  <img src="https://user-images.githubusercontent.com/44923999/187232630-af106d45-1614-4a1e-91be-9cc1a2797f6b.png" width="800" title="Snipped: August 29, 2022" />

* On the resulting "**Device templates**" > "**Create a device template**" > "**Review**" page, review configuration and then click **Create**

### Step 3: Create Device Template
In this step, we will model our device by adding capabilities.

Complete the following steps:

* Navigate to your **IoT Central Application**, and then "**Device Templates**" in the left-hand navigation
* Select your Device Template and on the resulting page, click "**+ Add capability**"

  <img src="https://user-images.githubusercontent.com/44923999/187242299-7f1edd77-704b-456c-bd39-3933342a8ebc.png" width="800" title="Snipped: August 29, 2022" />
  
* Complete the resulting form, including:

  Prompt | Entry
  ------ | ------
  **Display name** and **Name** | Enter "Pressure"
  **Capability type** | Select "Telemetry"
  **Capability type** | Select "Telemetry"
  **Semantic type** | Select "Pressure"
  **Schema** | Select "Double"
  **Min value** | Enter 0
  **Max value** | Enter 30
  **Decimal places** | Enter 1
  **Unit** | Select "Pound per square inch"
  **Display unit** | Enter "PSI"

_Note: These values are derived from what I found from an internet search re: "industrial pump"... you are likely to have more meaningful values {i.e., values applicable to your use case and device}_

* Click **Save**

  <img src="https://user-images.githubusercontent.com/44923999/187244100-4a7673b7-2458-4aa3-891d-8870b6efe71a.png" width="800" title="Snipped: August 29, 2022" />

* Repeat for additional capabilities {e.g., Temperature and Flow, as depicted above}

  <img src="https://user-images.githubusercontent.com/44923999/187244354-46f573e1-cbb2-4c58-926c-2e2ab22ca633.png" width="800" title="Snipped: August 29, 2022" />

* When you are finished adding capabilities, click **Publish**, review configuration and then click **Publish** (again)

### Step 4: Add Device
In this step, we will instantiate device simulations using our Device Template.

Complete the following steps:

* Navigate to your **IoT Central Application**, and then **Devices** in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/187244787-3626aa10-926d-45ef-b221-523138a9327e.png" width="800" title="Snipped: August 29, 2022" />

* Click "**Add a device**"

  <img src="https://user-images.githubusercontent.com/44923999/187249302-9d0d29ac-4f93-4937-9c2a-3d1aa95d97ce.png" width="800" title="Snipped: August 29, 2022" />

* Complete the resulting "**Create a new device**" form, including:

  Prompt | Entry
  ------ | ------
  **Device template** | Select the previously-created device template
  **Simulate this device?** | Select **Yes**
  
* Click **Create**
* Repeat if you wish to add additional devices {e.g., Pump002, Pump003 .. PumpNNN}

### Step 5: Export Data
In this step, we will set up continuous export of data to Azure Data Explorer.

##### Event Hub, Connection String

Complete the following steps:

* Navigate to your Event Hub Namespace, and then "**Shared Access Policies**" in the **Settings** grouping of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/187298261-44addf59-1879-4a7f-b45d-a3e3e43b0378.png" width="800" title="Snipped: August 29, 2022" />

* Click to select **RootManageSharedAccessKey**
* On the resulting "**SAS Policy: RootManageSharedAccessKey**" pop-out menu, copy the "**Connection string-primary key**" value for later use

##### IoT Central, Data Export

Complete the following steps:

* Navigate to your **IoT Central Application**, and then "**Data export**" in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/187250824-11a1cefe-f729-4730-8d49-0aeb376fe467.png" width="800" title="Snipped: August 29, 2022" />
  
* Click "**Add an export**"
