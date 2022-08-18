## Deployment of Synapse using Parameterized Linked Services

![image](https://user-images.githubusercontent.com/44923999/185416361-d8b12426-677b-48ec-9e46-d316cb7902d7.png)

This use case included requirement statements like:

* "We regularly deploy changes from our development environment (dev) to our production environment (prod)"
* "Our current Linked Service definitions are static... when changes are deployed, re-configuration is required"
* "We want to make our Linked Services dynamic {i.e., when in environment X, use configuration settings appropriate to X}"

### Preface

Different organizations provide for deployment with various architectural configurations: 1) separate subscriptions, 2) separate resource groups, and 3) separate resources.<br><br>This documentation employs separate resource groups.

### Prepare Infrastructure

This solution requires:

* **GitHub Enterprise** with a [repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository), a "**development**" [branch](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository), and a "**production**" [branch](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository)

<br>...as well as two resource groups {i.e., dev and prod} with each of the following resources:

* **Data Explorer** [cluster](Infrastructure_DataExplorer_Cluster.md) and [database](Infrastructure_DataExplorer_Database.md)
* [**Synapse**](Infrastructure_Synapse.md), configured to use [GitHub Enterprise](Infrastructure_Synapse_GitConfiguration.md)

  <img src="https://user-images.githubusercontent.com/44923999/185439267-ac9df2cc-8257-4ebf-8d6f-f0378ade3598.png" width="800" title="Snipped: August 18, 2022" />

### Step 1: Create a Static Linked Service
First, we will create a static linked service to help us understand that use case and challenges.

Complete the following steps:

* Navigate to **Synapse Studio** in your development instance of Synapse
* Confirm that you are working in the "**development**" branch
* Click the **Manage** navigation icon
* Select "**Linked services**" from the "**External connections**" grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/185463845-8d7c3c6f-4d7d-4cee-a616-c9bab4637547.png" width="800" title="Snipped: August 18, 2022" />

* Click "**+ New**"

  <img src="https://user-images.githubusercontent.com/44923999/185464087-cfcac130-63be-4182-b197-a7f3a90ba28e.png" width="800" title="Snipped: August 18, 2022" />

* On the resulting "**New linked service**" pop-out, search for and select "**Azure Data Explorer (Kusto)**", then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/185464259-94e986a1-3db6-425d-8b8c-ac9ac5298481.png" width="800" title="Snipped: August 18, 2022" />

* Complete the resulting "**New linked service**" >> "**Azure Data Explorer (Kusto)**" pop-out form, including:

  Prompt | Entry
  ------ | ------
  **Connect via…** | Confirm default selection, **AutoResolveIntegrationRuntime**
  **Authentication method** | Select "**System Assigned Managed Identity**"
  **Account selection method** | Select "**From Azure subscription**"
  **Cluster** | Select your **development** instance of "**Data Explorer**"
  **Database** | Select the database in your **development** instance of "**Data Explorer**"

* Click "**Test connection**" and confirm successful connection
* Click **Commit**

### Step 2: Deploy the Static Linked Service
Next, we will create pull request to move changes into the "production" branch.

Complete the following steps:

* Continue in the development instance of Synapse and the "**development**" branch

  <img src="https://user-images.githubusercontent.com/44923999/185477727-848084df-6bd7-485f-b841-9817fca1e7af.png" width="800" title="Snipped: August 18, 2022" />

* Click the "**development branch**" dropdown menu and select "**Create pull request**..."

  <img src="https://user-images.githubusercontent.com/44923999/185481805-6c171547-0592-47c8-b1ba-2bb5e76ae5c0.png" width="800" title="Snipped: August 18, 2022" />

* Click "**Create pull request**"

**Lorem Ipsum...**

### Step 3: Create a Dynamic Linked Service

**Lorem Ipsum...**

### Step 4: Deploy the Dynamic Linked Service

**Lorem Ipsum...**

### Reference
https://docs.microsoft.com/en-us/azure/data-factory/parameterize-linked-services?tabs=data-factory
