# Data Factory: Schema Drift

This guide demonstrates how to handle schema drift using Azure Data Factory (ADF) by ingesting multiple versions of a source dataset, detecting schema changes, and adjusting the pipeline accordingly.

## Step 1: Source Data

### Version1.json
```json
{  
  "id": "1",  
  "name": "John Doe",  
  "email": "johndoe@email.com",  
  "phone": "123-456-7890",  
  "age": 30,  
  "address": {  
    "street": "123 Main St",  
    "city": "Anytown",  
    "state": "Anystate",  
    "zip": "12345"  
  }  
}
```

### Version2.json
```json
{
  "id": "1",
  "gender": "Male"
}
```

### Version3.json
```json
{
  "id": "2",
  "name": "Jane Doe",
  "email": "janedoe@email.com",
  "phone": "987-654-3210",
  "age": 28,
  "address": {
    "street": "456 Main St",
    "city": "Anytown",
    "state": "Anystate",
    "zip": "54321"
  }
}
```

### Version4.json
```json
{
  "id": "1",
  "phone": null,
  "dateOfBirth": "1991-01-01"
}
```

### Version5.json
```json
{
  "id": "3",
  "name": "Mark Smith",
  "email": "marksmith@email.com",
  "dateOfBirth": "1990-12-12",
  "gender": "Male",
  "address": {
    "street": "789 Main St",
    "city": "Anytown",
    "state": "Anystate",
    "zip": "67890"
  }
}
```

## Step 2: Data Factory

* Navigate to Azure Data Factory >> Manage >> Linked Services
  * Create a new Azure Blob Storage linked service for the Storage Account (using System-Assigned Managed Identity) 
* Navigate to Azure Data Factory >> Author
  * Create a new Azure Blob Storage >> JSON dataset for the Storage Account / container
* Create a Data Flow
  * Lorem



----- 
Pipeline

* Create a pipeline and name it `SchemaDrift`
  * Add a 'Copy Data' activity to the pipeline canvas with settings:
    * Source >> Source Dataset: JSON dataset created in Step 1

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
