## Deployment of Synapse using Parameterized Linked Services

![image](https://user-images.githubusercontent.com/44923999/185416361-d8b12426-677b-48ec-9e46-d316cb7902d7.png)

This use case considers requirement statements like:

* "We regularly deploy changes from our development environment (dev) to our production environment (prod)"
* "Our current Linked Service definitions are static... when changes are deployed, re-configuration is required"
* "We want to make our Linked Services dynamic {i.e., when in environment X, use configuration settings appropriate to X}"

### Preface

Different organizations provide for deployment with various architectural configurations: 1) separate subscriptions, 2) separate resource groups, and 3) separate resources.<br><br>This documentation employs separate resource groups.

### Prepare Infrastructure

This solution requires:

* **GitHub Enterprise** with a [repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository), a "**development**" [branch](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository), and a "**production**" [branch](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository)

<br>...as well as two resource groups {i.e., dev and prod} with each of the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
* [**Synapse**](Infrastructure_Synapse.md), configured to use [GitHub Enterprise](Infrastructure_Synapse_GitConfiguration.md)

  <img src="https://user-images.githubusercontent.com/44923999/185621162-ea01a934-f17f-4dce-b2bd-6caaea601932.png" width="800" title="Snipped: August 18, 2022" />

#### Grant Access (Synapse > Data Explorer)
Permissions must be added to Data Explorer in both environments {i.e., dev and prod}.

Complete the following steps:

* Navigate to your development instance of Data Explorer, then "**Permissions**" in the navigation

  <img src="https://user-images.githubusercontent.com/44923999/185622002-bb053429-f0ab-4453-b41c-ff7bf1e97ac5.png" width="800" title="Snipped: August 15, 2022" />

* Click "**+ Add**" and then select **AllDatabasesAdmin** from the resulting dropdown menu
* On the resulting screen, click **+ Add** and select "**Add role assignment**" from the resulting dropdown menu

  <img src="https://user-images.githubusercontent.com/44923999/185622271-6d43b98f-6df4-420d-a14b-87cef140e396.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting pop-out, search for and then **Select** your development instance of Synapse

Repeat this process for your production instance of Data Explorer

### Step 1: Create a Static Linked Service
In this step, we will create a static linked service to help us understand that use case and challenges.
<br>_Note: If you interested in simply solving the challenge, feel free to skip ahead to Step 4_

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
In this step, we will create pull request to move changes into the "production" branch.

Complete the following steps:

* Continue in the development instance of Synapse and the "**development**" branch

  <img src="https://user-images.githubusercontent.com/44923999/185477727-848084df-6bd7-485f-b841-9817fca1e7af.png" width="800" title="Snipped: August 18, 2022" />

* Click the "**development branch**" dropdown menu and select "**Create pull request**..."

  <img src="https://user-images.githubusercontent.com/44923999/185481805-6c171547-0592-47c8-b1ba-2bb5e76ae5c0.png" width="800" title="Snipped: August 18, 2022" />

* Click "**Create pull request**"

  <img src="https://user-images.githubusercontent.com/44923999/185495949-297dc374-970c-4456-aa80-f66e12e40ede.png" width="800" title="Snipped: August 18, 2022" />

* Review settings on the resulting "**Open a pull request**" screen {e.g., from "development" to "production"} and then click "**Create pull request**"

  <img src="https://user-images.githubusercontent.com/44923999/185496188-3cd53754-cbcf-420b-ab06-1cfa6c10cc59.png" width="800" title="Snipped: August 18, 2022" />

* Review settings on the resulting screen and then click "**Merge pull request**" and on the resulting screen, click "**Confirm merge**"

  _Note: This exercise errs on the side of brevity, we are clearly not giving standard deployment process or pull requests their full due_

Successful merge of the pull request means that the "production" branch is ready to review on the production instance of Synapse. 

### Step 3: Review Result
In this step, we will review the result.

Complete the following steps:

* Navigate to **Synapse Studio** in your production instance of Synapse
* Confirm that you are working in the "**production**" branch
* Click the **Manage** navigation icon
* Select "**Linked services**" from the "**External connections**" grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/185497675-a0c5783e-7c3c-49bd-9f55-49dc09ff53f9.png" width="800" title="Snipped: August 18, 2022" />

You will see that the "static" Linked Service has been deployed to the production branch and arguably, could be published.<br>There are several problems, however:

* "**Connection failed**" - The system-assigned permissions that allow the development instance of Synapse and Data Explorer to interact do not work for the production instance of Synapse and the development instance of Data Explorer (and rightly so)
* **Wrong Data Explorer** - Even if there were not a permissions issue, we do not want to access data from the development environment
* **Dependent Datasets** - If we had dependent datasets, each of those would need to be updated to switch between Linked Services and making that change negatively impacts other settings {e.g., credentials}

The point of Steps 1 through 3 was to demonstrate that a static Linked Service is not a great choice for regular deployment of changes from a development instance of Synapse to a production instance of Synapse. We will stop here and flip to a more reasonable solution.

### Step 4: Create a Dynamic Linked Service
In this step, we will create a dynamic linked service to demonstrate a solution to the issues observed in the previous step.

Complete the following steps:

* Navigate to **Synapse Studio** in your development instance of Synapse
* Confirm that you are working in the "**development**" branch
* Click the **Manage** navigation icon
* Select "**Linked services**" from the "**External connections**" grouping in the resulting navigation
* Click "**+ New**"
* On the resulting "**New linked service**" pop-out, search for and select "**Azure Data Explorer (Kusto)**", then click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/185502998-9a7f6bd2-dd25-417d-928f-47c4cbbd4a41.png" width="800" title="Snipped: August 18, 2022" />

* Not surprisingly, we will complete the resulting "**New linked service**" >> "**Azure Data Explorer (Kusto)**" pop-out form differently than we did for "static"
  * First, click "**+ New**" under Parameters and enter the following values:
  
    Name | Type | Default value
    ------ | ------ | ------
    WorkspaceName | String | devsaw
    
    _Note: We are providing a "**Default value**" only temporarily... the value for the parameter should be passed in from the Datasets that use it_

  * Second, select the "**Enter manually**" radio button under "**Account selection method**"
  * Third, click on the **Endpoint** text box and then click the "**Add dynamic content...**" link that appears below the text box

    <img src="https://user-images.githubusercontent.com/44923999/185500863-0d0aa02d-6117-4dc3-8a78-887880f09355.png" width="800" title="Snipped: August 18, 2022" />

    * Modify the following expression and paste into the resulting "**Add dynamic content**" pop-out:
      ```
      @if(equals(linkedService().WorkspaceName,'devsaw')
       ,'https://devdec.westus3.kusto.windows.net'
       ,'https://proddec.westus3.kusto.windows.net'
      )
      ```
  
    * Click **OK**

  * Next, click on the **Database** text box and then click the "**Add dynamic content...**" link that appears below the text box

    <img src="https://user-images.githubusercontent.com/44923999/185500863-0d0aa02d-6117-4dc3-8a78-887880f09355.png" width="800" title="Snipped: August 18, 2022" />

    * Modify the following expression and paste into the resulting "**Add dynamic content**" pop-out:
      ```
      @if(equals(linkedService().WorkspaceName,'devsaw')
       ,'devded'
       ,'prodded'
      )
      ```
  
    * Click **OK**

  * Finally, click "**Test connection**", confirm successful connection, and then click **Save**

You might also use something like:

```
@concat('https://', linkedService().Environment, 'rchaplerdec.westus3.kusto.windows.net')
```

### Step 5: Deploy the Dynamic Linked Service
In this step, we will create another pull request to move new changes into the "production" branch.

Repeat the process described in "**Step 2: Deploy the Static Linked Service**"

### Step 6: Confirm Success
In this step, we will confirm success.

Complete the following steps:

* Navigate to **Synapse Studio** in your production instance of Synapse
* Confirm that you are working in the "**production**" branch
* Click the **Manage** navigation icon
* Select "**Linked services**" from the "**External connections**" grouping in the resulting navigation

  <img src="https://user-images.githubusercontent.com/44923999/185674922-3155b60c-2a62-4a3c-ac3f-0264646b14c7.png" width="800" title="Snipped: August 19, 2022" />

* Open the "**dynamic**" Linked Service and update the "**WorkspaceName**" to value "**prodsaw**"
<br>_Note: This parameter should be updated from the Dataset, rather than manually_

* Click "**Test connection**" and confirm successful connection

### What's Next?

You will reference your dynamic Linked Service from a Dataset with a parameter {e.g., "Environment"} and you will populate the "**Linked Service Properties**", parameter value with expression: `@dataset().Environment`

You will reference this Dataset with a parameter {e.g., "Environment"} and you will populate the "**Linked Service Properties**", parameter value with an expression like: `@substring(pipeline().DataFactory,0,1)`

<img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />

### Reference
https://docs.microsoft.com/en-us/azure/data-factory/parameterize-linked-services?tabs=data-factory
