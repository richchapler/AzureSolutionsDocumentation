# Data Factory: Schema Drift

This guide demonstrates how to handle schema drift using Azure Data Factory (ADF) by ingesting multiple versions of a source dataset, detecting schema changes, and adjusting the pipeline accordingly.

## Version 1: Initial Load

### 1. Create a Data Factory Instance
1. Go to the [Azure portal](https://portal.azure.com).
2. Search for **Azure Data Factory** and click **Create**.
3. Select your subscription, resource group, and give your Data Factory a unique name.
4. Choose the appropriate region and click **Review + Create**.
5. Click **Create** after the validation completes.

### 2. Create a Linked Service
Create a linked service to connect to the storage location where your JSON files are stored.

1. In Data Factory, go to **Manage > Linked Services > New**.
2. Select the linked service type based on where your files are stored (e.g., Azure Blob Storage or Azure Data Lake).
3. Configure the connection by providing the necessary credentials (storage account name, access key, etc.).
4. Test the connection and click **Create**.

### 3. Create a Dataset for JSON Files
1. In the ADF pane, go to **Author > Datasets > New Dataset**.
2. Select **JSON** as the dataset type.
3. Choose your previously created linked service and provide the path to the folder where your versioned JSON files are stored.
4. In the **Schema** section, allow the schema to drift by enabling the "Allow Schema Drift" option.
5. Click **OK** to create the dataset.

### 4. Create a Data Pipeline
1. Go to **Author > Pipelines > New Pipeline**.
2. Name the pipeline **SchemaDriftPipeline**.
3. In the **Activities** pane, drag in a **Copy Data** activity to the canvas.
4. Configure the **Source** tab:
   - Choose the JSON dataset created in step 3.
   - Enable **Schema Drift** handling by checking the appropriate option.
5. Configure the **Sink** tab:
   - Choose your target dataset or storage location for the ingested data.
   - Optionally, enable a staging area if necessary.
   
### 5. Add Schema Drift Demonstration Logic
We’ll now demonstrate schema drift handling by ingesting each file version and inspecting the results.

#### a. Ingest **Version 1** Data
1. On the **Source** dataset, set the file path to the file `Version1.json`.
2. Click **Debug** to run the pipeline.
3. After the run completes, inspect the output in your target storage or dataset to observe how ADF handled the schema.

## Version 2: Lorem Load
#### b. Ingest **Version 2** Data
1. Modify the file path on the source dataset to point to `Version2.json`.
2. Click **Debug** to run the pipeline.
3. Observe the handling of the new "gender" field introduced in **Version 2**.

#### c. Ingest **Version 3** Data
1. Change the file path to `Version3.json`.
2. Run the pipeline and inspect the results.
3. Notice the introduction of new data for **id 2** and how the schema evolves.

#### d. Ingest **Version 5** Data
1. Set the file path to `Version5.json`.
2. Run the pipeline and observe how the pipeline handles the "dateOfBirth" and "gender" fields.

### 6. Analyze Results
After each file version is processed, compare the results to ensure that schema drift was handled correctly and no data was lost or mishandled.

### 7. Automate Schema Drift Handling
To automate schema drift handling, you can enable ADF’s **mapping data flows** and configure a **Derived Column** transformation to map new or altered fields dynamically.

1. Go to **Author > Data Flows > New Mapping Data Flow**.
2. Add a **Source** transformation pointing to your JSON dataset.
3. Enable **Schema Drift** in the source settings.
4. Use the **Select** transformation to choose specific fields or use **Auto Mapping**.
5. Add a **Sink** to define where the transformed data should be stored.

### Conclusion
This pipeline demonstrates how Azure Data Factory can be used to handle schema drift across different data versions. The results of each version’s ingestion should show how ADF adapts to schema changes without requiring manual intervention.
