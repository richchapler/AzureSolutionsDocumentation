## Synapse Deployment using Parameterized Linked Services

![image](https://user-images.githubusercontent.com/44923999/185416361-d8b12426-677b-48ec-9e46-d316cb7902d7.png)

This use case included requirement statements like:

* "We regularly deploy changes from our development environment (dev) to our production environment (prod)"
* "Our current Linked Service definitions are static... when changes are deployed, re-configuration is required"
* "We want to make our Linked Services dynamic {i.e., when in environment X, use configuration settings appropriate to X}"

### Preface

Different organizations provide for deployment with various architectural configurations: 1) separate subscriptions, 2) separate resource groups, and 3) separate resources.<br><br>This documentation will employ separate resource groups.

### Prepare Infrastructure

  <img src="https://user-images.githubusercontent.com/44923999/185439267-ac9df2cc-8257-4ebf-8d6f-f0378ade3598.png" width="800" title="Snipped: August 18, 2022" />

This solution requires two resource groups {i.e., dev and prod} with each of the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md) :: [Sample Data](https://docs.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=one-click-ingest)
* [**Synapse**](Infrastructure_Synapse.md)

You will also require:

* **GitHub** >> [Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)

### Step 1: Static Linked Service
First, we will create a static linked service.

Complete the following steps:

* Navigate to **Manage** >> "**Linked services**" in your Synapse Studio

  <img src="https://user-images.githubusercontent.com/44923999/185442780-b9b8ae0b-e89e-4488-867a-2f2a2bcc43b9.png" width="800" title="Snipped: August 18, 2022" />

### Reference
https://docs.microsoft.com/en-us/azure/data-factory/parameterize-linked-services?tabs=data-factory
