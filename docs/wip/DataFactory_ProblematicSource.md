### Lab: Identifying Problematic Records in Azure Data Factory Pipeline

This lab demonstrates how to create an Azure Data Factory (ADF) pipeline that identifies and logs problematic records during a data copy operation. The errors will be logged to Azure Blob Storage or Data Lake, with an optional step to load them into an Azure SQL Database.

---

### **Objective**

- Create a sample dataset with problematic records
- Build an ADF pipeline to process the dataset and handle errors
- Log errors in Azure Blob Storage or Data Lake
- Optionally, load error logs into an Azure SQL Database for further analysis

---

### **Steps**

#### **Step 1: Create Sample Data**
1. **Create a CSV File**:
   - Name the file `sample_data.csv` with the following content:
     ```csv
     ID,Name,Age
     1,John,30
     2,Jane,25
     3,,40
     4,Emily,not_a_number
     5,Michael,35
     ```
   - Problematic records:
     - Record 3: Missing value in `Name`
     - Record 4: Non-numeric value in `Age`

2. **Upload the File to Blob Storage or Data Lake**:
   - Upload the file to a container or folder, e.g., `input/` in your Azure Blob Storage or Data Lake.

---

#### **Step 2: Set Up the Azure SQL Database (Optional)**
1. **Create a Table for Valid Records**:
   ```sql
   CREATE TABLE ValidRecords (
       ID INT,
       Name NVARCHAR(100),
       Age INT
   );
   ```

2. **Create a Table for Error Logs**:
   ```sql
   CREATE TABLE Errors (
       RowNumber INT,
       ColumnName NVARCHAR(100),
       ErrorDescription NVARCHAR(255)
   );
   ```

3. **Grant Permissions to ADF’s Managed Identity**:
   - In SQL Server:
     ```sql
     CREATE LOGIN [<ManagedIdentityObjectID>] FROM EXTERNAL PROVIDER;
     USE <DatabaseName>;
     CREATE USER [<ManagedIdentityObjectID>] FOR LOGIN [<ManagedIdentityObjectID>];
     ALTER ROLE db_datareader ADD MEMBER [<ManagedIdentityObjectID>];
     ALTER ROLE db_datawriter ADD MEMBER [<ManagedIdentityObjectID>];
     ```
   - Replace `<ManagedIdentityObjectID>` with the object ID of ADF’s managed identity.

---

#### **Step 3: Set Up the ADF Pipeline**

1. **Create the Pipeline**:
   - In Azure Data Factory, create a pipeline named `DetectProblematicRecords`.

2. **Add a Copy Data Activity**:
   - Drag a **Copy Data** activity onto the pipeline canvas.
   - **Source**:
     - Dataset: Point to the `sample_data.csv` file in Blob Storage or Data Lake.
   - **Sink**:
     - Dataset: Point to the `ValidRecords` table in Azure SQL Database (if configured).
   - **Fault Tolerance**:
     - Enable **Skip incompatible rows**.
     - Set the **Error log file path** to a location in Blob Storage or Data Lake, e.g., `errors/`.

3. **Add a Dataset for Error Logs**:
   - Create a dataset for the error log file:
     - Format: **Delimited Text** or **JSON**.
     - Linked Service: Point to the Blob Storage or Data Lake location where the error logs are written.

---

#### **Step 4: Optional - Load Errors into SQL**
1. **Add a Second Copy Data Activity**:
   - **Source**: Use the dataset pointing to the error logs.
   - **Sink**: Use the dataset pointing to the `Errors` table in Azure SQL Database.
   - Map the error log fields (`RowNumber`, `ColumnName`, `ErrorDescription`) to the table columns.

---

#### **Step 5: Test and Debug**
1. **Run the Pipeline**:
   - Trigger the pipeline with the `sample_data.csv` file.
   - Verify that:
     - Valid records are written to the `ValidRecords` table.
     - Errors are logged to Blob Storage or Data Lake.

2. **Optional - Verify SQL Error Logs**:
   - Check the `Errors` table for records like:
     ```plaintext
     RowNumber | ColumnName  | ErrorDescription
     --------------------------------------------
     3         | Name        | Missing value
     4         | Age         | Invalid value 'not_a_number'
     ```

---

### **Key Notes**
- **Storage for Error Logs**: ADF writes errors by default to Blob Storage or Data Lake. Use this as the primary mechanism for error logging.
- **Optional SQL Integration**: If SQL-based analysis is required, error logs can be loaded into the `Errors` table.
- **Advanced Error Handling**: Use ADF's **Data Flow** activity for more complex error validation and processing.

This lab equips you with the ability to handle data errors in ADF pipelines efficiently, ensuring both valid data processing and clear error tracking.
