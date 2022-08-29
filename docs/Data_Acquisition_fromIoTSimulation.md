## Data Acquisition... from IoT Simulation

![image](https://user-images.githubusercontent.com/44923999/185979268-83aad799-8e0a-4e82-9f82-f81616cf8cff.png)

This use case considers requirement statements like:

* "We are working hard to bring IoT online, but it is going to take time... we want to begin working on analytics before we have real data"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* [**IoT Central**](Infrastructure_IoTCentral.md)

### Step 2: Create Device Template
First, we will create a Device Template.

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
