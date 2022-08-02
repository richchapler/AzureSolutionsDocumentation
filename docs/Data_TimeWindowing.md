## Data Analysis... Time Windowing

Requirement statements might include:

* "We want to use our time-series data to characterize Device 'Runs' {i.e., the period between State A and State B}"
* "We need to know key metrics {e.g., temperature} at the start and end of a run"

### Step 1: Infrastructure Resources

This solution requires the following resources:

* Data Explorer [Cluster](Infrastructure_DataExplorer_Cluster.md)

### Step 2: Sample Data

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/, default landing page **Home**
* Click **Explore sample data with KQL**

  <img src="https://user-images.githubusercontent.com/44923999/182449010-e0a5130d-6186-4988-a4e0-5682912484b0.png" width="800" title="Snipped: August 2, 2022" />

* On the **Explore data samples** pop-up, click to select **IoT data** and then click **Explore**

Click Ingest Data on the resulting Data Management page
* Open **Synapse Studio** and click the **Manage** navigation icon
* Click **Linked services** from the **External connections** grouping in the resulting navigation

