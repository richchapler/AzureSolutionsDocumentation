## Deployment >> Parameterized Linked Services

Requirement statements might include:

* "We regularly deploy changes from our development environment (dev) to our production environment (prod)"
* "Linked Service definitions typically point at a specific environment... and that means, when changes are deployed, manual re-configuration is required"
* "We want to make our Linked Services dynamic {i.e., when in environment X, use configuration settings appropriate to X}"

### Preface

Different organizations provide for deployment with various architectural configurations: 1) separate subscriptions, 2) separate resource groups, and 3) separate resources. Parameterization will work for any of these configurations. This documentation will employ separate resource groups.

### Prepare Infrastructure
This solution requires two resource groups {i.e., dev and prod} with each of the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md) :: [Sample Data](https://docs.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=one-click-ingest)
* [**Synapse**](Infrastructure_Synapse.md)
* User-Assigned Managed Identity

* GitHub

#### Grant Access (Data Explorer > Event Hub)
Data Explorer requires special permissions to interact with Event Hub.

Complete the following steps:

* Navigate to your Event Hub Namespace, then "**Access Control (IAM)**" in the navigation

  <img src="https://user-images.githubusercontent.com/44923999/184709205-5f6e8ad5-92fe-4577-b759-8e3b2b14dca4.png" width="800" title="Snipped: August 15, 2022" />

### Reference
https://docs.microsoft.com/en-us/azure/data-factory/parameterize-linked-services?tabs=data-factory
