### Lab: Identifying Problematic Records in Azure Data Factory Pipeline

### **Objective**

1. Enable row-level error logging in an ADF pipeline
2. Capture detailed error logs in Azure Blob Storage
3. Parse error logs to extract meaningful details (row numbers, column names, error messages)
4. Optionally store parsed error details in Azure SQL Database

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
   - Problematic rows:
     - **Row 3**: Missing `Name`.
     - **Row 4**: Non-numeric value for `Age`.

2. **Upload the File to Blob Storage**:
   - Upload `sample_data.csv` to a container (e.g., `input/`) in your Azure Blob Storage account.

---

#### **Step 2: Set Up the ADF Pipeline**

1. **Create a Pipeline**:
   - Name the pipeline `DetectProblematicRecords`.

2. **Add a Copy Data Activity**:
   - Configure the Copy Data activity as follows:
     - **Source**:
       - Dataset: Use a delimited text dataset pointing to `sample_data.csv`.
       - Format: CSV with headers.
     - **Sink**:
       - Dataset: Use an Azure SQL Database table (`ValidRecords`) as the Sink.
       - Table definition:
         ```sql
         CREATE TABLE ValidRecords (
             ID INT,
             Name NVARCHAR(100),
             Age INT
         );
         ```
     - **Fault Tolerance**:
       - Enable **Skip incompatible rows**.
       - Set the error file path to a folder in your Blob Storage, such as `copyactivity-logs/`.
       - Enable logging with the following settings:
         ```json
         "logSettings": {
             "enableCopyActivityLog": true,
             "copyActivityLogSettings": {
                 "logLevel": "Warning",
                 "enableReliableLogging": true
             },
             "logLocationSettings": {
                 "linkedServiceName": {
                     "referenceName": "AzureBlobStorage2",
                     "type": "LinkedServiceReference"
                 },
                 "path": "copyactivity-logs/"
             }
         }
         ```

---

#### **Step 3: Parse the Error Logs**
1. **Create a Dataset for Error Logs**:
   - Type: **Delimited Text**
   - File path: Point to the `copyactivity-logs/` folder.
   - Use wildcard file selection to process log files dynamically.
   - Schema: Include fields such as `Timestamp`, `Level`, `OperationName`, `OperationItem`, `Message`.

2. **Add a Data Flow**:
   - Add a **Data Flow** activity to parse the error log file.

3. **Data Flow Configuration**:
   - **Source**:
     - Use the error log dataset.
   - **Derived Column Transformation**:
     - Use expressions to extract the following details from the `Message` column:
       - **RowNumber**:
         Extract the first value inside quotes (e.g., `"4"`).
         Expression: 
         ```expression
         substring(Message, indexOf(Message, 'TabularRowSkip,') + 15, indexOf(Message, ',') - indexOf(Message, 'TabularRowSkip,') - 15)
         ```
       - **ColumnName**:
         Extract the column name causing the error (e.g., `"Age"`).
         Expression:
         ```expression
         substring(Message, indexOf(Message, "column name '") + 13, indexOf(Message, "' from type") - indexOf(Message, "column name '") - 13)
         ```
       - **ErrorDescription**:
         Extract the full error description.
         Expression:
         ```expression
         substring(Message, indexOf(Message, 'Exception occurred'), len(Message))
         ```
   - **Sink**:
     - Write the parsed data to an Azure SQL Database table (`Errors`).

4. **Create an Errors Table**:
   - Use the following SQL script to create the table:
     ```sql
     CREATE TABLE Errors (
         RowNumber INT,
         ColumnName NVARCHAR(100),
         ErrorDescription NVARCHAR(255)
     );
     ```

---

#### **Step 4: Test the Pipeline**
1. **Upload Test Data**:
   - Use the `sample_data.csv` file containing known errors.
2. **Run the Pipeline**:
   - Trigger the pipeline and monitor its execution.
3. **Verify the Results**:
   - Check the `ValidRecords` table for successfully processed rows.
   - Inspect the `Errors` table for row-level error details.
   - Optionally, review the raw error file in the `copyactivity-logs/` folder.

---

### **Summary of Results**
- **Valid Records**:
  The `ValidRecords` table will contain rows with IDs `1`, `2`, and `5`.

- **Error Records**:
  The `Errors` table will contain entries like:
  ```plaintext
  RowNumber | ColumnName  | ErrorDescription
  --------------------------------------------
  3         | Name        | Missing value
  4         | Age         | Invalid value 'not_a_number'
  ```

- **Error Logs**:
  The raw error logs in Blob Storage (`copyactivity-logs/`) provide traceability for debugging.
