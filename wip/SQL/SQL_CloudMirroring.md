# SQL: Cloud "Mirroring"

## Introduction

Integrating on‑prem SQL Server with Azure SQL enables continuous data replication, ensuring up‑to‑date information for reporting, disaster recovery, and workload offloading. This document outlines integration methods, with a particular focus on Managed Instance Link—a new feature in SQL Server 2022 that simplifies near real‑time replication to an Azure SQL Managed Instance.

## Objectives

- **Provide an Overview:**
  Explain the available options for replicating on‑prem SQL Server data to Azure SQL.
- **Focus on Managed Instance Link:** 
  Detail how Managed Instance Link works, its benefits, limitations, and typical use cases.
- **Support Decision Making:**
  Help readers understand why Managed Instance Link is the recommended solution for hybrid deployments at this stage.

## Options

### Option #1: Managed Instance Link (SQL Server 2022)

Managed Instance Link is a new feature introduced with SQL Server 2022 that leverages Always On technology in a more streamlined and integrated way. It is designed to replicate data in near real‑time from your on‑prem SQL Server 2022 instance directly to an Azure SQL Managed Instance.

**How It Works:**

- The on‑prem SQL Server 2022 instance is configured to send transaction changes directly to the Azure SQL Managed Instance.
- The data flow is asynchronous, ensuring that changes are continuously pushed to the Azure target.
- The Azure replica is maintained as a read‑only copy, making it ideal for reporting or offloading read workloads.

**Benefits:**

- **Native Integration:** Simplified configuration and setup for hybrid environments.
- **Low Latency:** Near real‑time replication with minimal overhead.
- **Streamlined Management:** Integrated into SQL Server 2022 without needing additional replication tools.

**Limitations:**

- Requires SQL Server 2022 on‑prem and an Azure SQL Managed Instance.
- The Azure replica is read‑only; write operations must continue on‑prem.

**Typical Use Cases:**

- Hybrid read‑scale workloads
- Phased migrations to the cloud
- Disaster recovery with near real‑time data availability

-------------------------

### Option #2: Always On Availability Groups

Always On Availability Groups (AG) provide an enterprise‑grade high availability and disaster recovery solution. With a Distributed AG, you can extend an on‑prem AG to include an Azure SQL target (either an Azure SQL Managed Instance or a SQL Server running on an Azure virtual machine).

**How It Works:**

- The on‑prem SQL Server is part of an Availability Group, and a separate AG is set up in Azure.
- A Distributed AG links the two groups, allowing data to be replicated asynchronously from the on‑prem primary to the Azure secondary.
- The secondary in Azure is typically read‑only, though it can be used to offload reporting workloads.

**Benefits:**

- **Enterprise‑Grade:** Robust HA/DR capabilities with automatic failover (when properly configured).
- **Flexible Configuration:** Supports both synchronous and asynchronous replication modes.
- **Read Scale:** Can offload read-only workloads to the Azure secondary.

**Limitations:**

- Requires SQL Server Enterprise Edition for full Availability Group functionality.
- Configuration is more complex compared to Managed Instance Link.
- Generally associated with higher infrastructure costs.

**Typical Use Cases:**

- Mission‑critical applications requiring high availability and disaster recovery.
- Scenarios where on‑prem AGs already exist and extending to Azure is desired.
- Minimal downtime cloud migrations with a robust failover mechanism.

-------------------------

### Option #3: Transactional Replication

Transactional Replication is a long‑standing SQL Server feature that enables continuous, near real‑time data movement. It replicates transactions from the on‑prem SQL Server (the publisher) to an Azure SQL target (the subscriber) and allows for granular control over which tables and columns are replicated.

**How It Works:**

- Changes made on the on‑prem database are captured and published.
- A subscription is set up on the Azure SQL target (either Azure SQL Database or Managed Instance) to receive these transactions.
- The process is asynchronous, ensuring minimal latency between the on‑prem source and Azure.

**Benefits:**

- **Granular Control:** Selectively replicate specific tables or columns as needed.
- **Proven Technology:** A mature and well-documented method with a long history of successful deployments.
- **Near Real‑Time:** Provides timely data updates for reporting or analytical workloads.

**Limitations:**

- Can become complex when dealing with very large schemas.
- Some subscriber schema limitations may require careful planning.
- Requires ongoing monitoring and maintenance to ensure replication health.

**Typical Use Cases:**

- Replicating selected parts of a database for reporting or analytics.
- Scenarios where only a subset of data is needed in Azure.
- Situations that demand near real‑time data movement without the overhead of full database synchronization.

-------------------------

### Option #4: SQL Data Sync

SQL Data Sync is an Azure SQL Database feature that uses a hub‑and‑spoke model to synchronize data between on‑prem SQL Server instances and Azure SQL Database. It is particularly useful when bi‑directional synchronization is required, although it can be configured for one‑way replication.

**How It Works:**

- An Azure SQL Database is designated as the hub.
- On‑prem SQL Server instances (or additional Azure databases) act as spokes.
- Data changes are synchronized between the on‑prem database and the hub, and then optionally propagated to other nodes.

**Benefits:**

- **Ease of Use:** Configuration is handled through the Azure Portal, making it accessible even for smaller deployments.
- **Bi‑Directional Sync:** Supports two‑way data synchronization if needed.
- **Cost‑Effective:** Suitable for smaller or distributed databases.

**Limitations:**

- Limited to Azure SQL Database as the hub; it is not applicable for Azure SQL Managed Instance.
- Not a full high‑availability or disaster recovery solution.
- Conflict resolution can be challenging in bi‑directional scenarios.

**Typical Use Cases:**

- Distributed applications with smaller data sets.
- Scenarios where occasional connectivity exists and multi‑directional data flow is needed.
- Branch office environments or distributed setups where data synchronization is the primary requirement.

-------------------------

### Option #5: ETL Tools (SSIS / Azure Data Factory)

**Overview:**
While not purely a native SQL replication feature, ETL tools such as SQL Server Integration Services (SSIS) or Azure Data Factory (ADF) can be used to capture changes (often via Change Data Capture, CDC) and continuously load data from on‑prem SQL Server to Azure SQL. This approach is particularly useful for data warehousing or complex transformation scenarios.

**How It Works:**

- On‑prem SQL Server CDC is enabled to track data changes.
- ETL tools are configured to extract these changes at scheduled intervals or near real‑time.
- The data is then transformed as needed and loaded into the Azure SQL target (either Database or Managed Instance).

**Benefits:**

- **Flexibility:** Supports complex data transformations and business logic during the load process.
- **Broad Applicability:** Works with any SQL Server version that supports CDC.
- **Decoupled Architecture:** Separates the data movement from the operational database, allowing for greater control over data processing.

**Limitations:**

- Not a built‑in HA/DR solution; it is primarily used for data movement and transformation.
- Requires additional infrastructure and management of the ETL process.
- May introduce latency depending on the scheduling and processing of data batches.

**Typical Use Cases:**

- Building data warehouses or analytical environments.
- Scenarios where data transformation is needed along with replication.
- Incremental data loads where near real‑time performance is acceptable with slight latency.

-------------------------

### Recommendations & Considerations

- **For a straightforward, near real‑time read‑only replica:**
   If you are running SQL Server 2022 on‑prem and have an Azure SQL Managed Instance available, **Managed Instance Link** is the simplest and most integrated solution.
   
- **For enterprise-grade high availability and disaster recovery:**
   Consider **Always On Availability Groups (Distributed AG)** if you need robust HA/DR capabilities and already use Availability Groups on‑prem.
   
- **For granular, selective data replication:**
   **Transactional Replication** is ideal if you need to replicate only a subset of your data to Azure, offering fine‑grained control over what gets replicated.
   
- **For distributed scenarios with smaller data sets:**
   **SQL Data Sync** offers an easy-to-use solution, especially if you require bi‑directional data flow across multiple endpoints in Azure SQL Database.
   
- **For scenarios involving data transformation or analytics:**
   ETL tools such as **SSIS** or **Azure Data Factory** can be leveraged to capture on‑prem changes and load them into Azure, though these are more focused on data processing than native HA/DR replication.
------------------------- ------------------------- ------------------------- -------------------------

## Deep-Dive: Managed Instance Link

### Provision Resources

#### Virtual Machine

Sign in to the [Azure Portal](https://portal.azure.com) using your Azure credentials and create a virtual machine:

- Image: `SQL Server 2022 Standard on Windows Server 2022...` (or equivalent)

- Size: `Standard_D2s_v3`

- SQL Authentication: `Enabled`

#### Managed Instance

Create a "Azure SQL Managed Instance":

- Authentication Method: `Use both SQL and Microsoft Entra authentication`

------------------------- -------------------------

### Prepare Source

Connect to the Virtual Machine and configure the on-prem SQL Server instance.

#### SQL Server Configuration Manager

- Open SQL Server Configuration Manager on the Azure virtual machine
- In the left pane, select SQL Server Services
- Right-click `SQL Server (MSSQLSERVER)` and choose "Properties" from the resulting menu

##### Enable Always On Availability Groups

- Select the "Always On Availability Groups" tab
- Check “Enable Always On Availability Groups”

##### Enable Recommended Trace Flags

Managed Instance Link leverages trace flags to optimize performance and address specific functionality requirements.

- Select the "Startup Parameters" tab
-  Add the following parameters:  `-T1800` and `-T9567`

##### Finally...

Click "OK" and restart `SQL Server (MSSQLSERVER)`

#### SQL Server Management Studio

- Open SQL Server Management Studio on the Azure virtual machine
- Connect to the on‑prem SQL Server instance

##### Confirm Sysadmin Role Membership

- Expand Security >> Logins, double-click your login
- In the "Login Properties..." pop-up, click "Server Roles” and confirm that "sysadmin" is checked

##### Create Database Master Key

```sql
USE master
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourStrongPassword'
```

##### Create Certificate for Endpoint Authentication

```sql
USE master
CREATE CERTIFICATE MyAGCert WITH SUBJECT = 'Certificate for Managed Instance Link'
```

##### Create Mirroring Endpoint

```sql
USE master
CREATE ENDPOINT Hadr_endpoint
    STATE = STARTED
    AS TCP (LISTENER_PORT = 5022)
    FOR DATA_MIRRORING (
        ROLE = ALL,
        AUTHENTICATION = CERTIFICATE MyAGCert,
        ENCRYPTION = REQUIRED ALGORITHM AES
    )
```

##### Create Source Database

```sql
CREATE DATABASE SourceDB;
```

##### Create Source Tables

```sql
USE SourceDB;

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Email NVARCHAR(100)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT FOREIGN KEY REFERENCES Customers(CustomerID),
    OrderDate DATETIME,
    Amount DECIMAL(10,2)
);
```

##### Populate Source Tables

```sql
INSERT INTO Customers (CustomerID, FirstName, LastName, Email)
VALUES (1, 'John', 'Doe', 'john.doe@example.com');

INSERT INTO Orders (OrderID, CustomerID, OrderDate, Amount)
VALUES (101, 1, GETDATE(), 150.00);
```

##### Backup Database

```sql
BACKUP DATABASE SourceDB TO DISK = 'C:\Temp\SourceDB.bak' WITH INIT, COMPRESSION, STATS = 10;
```

------------------------- -------------------------

### Configure Managed Instance Link

Right‑click the `SourceDB` database and select "Azure SQL Managed Instance link" > "New"

Complete steps in the "New SQL Managed Instance link" popup:

- Specify Link Options

  - Link Name: `SourceDB-DestinationDB`

  - Failover Intent: `unchecked`

    If your primary goal is one-way replication (on-prem to Azure) with a read-only secondary, you don’t need bi-directional failover. Leave this unchecked unless you plan to fail over to Azure and potentially reverse replication later.

  - Connectivity Troubleshooting: `checked`

    This option helps diagnose network and authentication issues by gathering additional diagnostic information, making it easier to pinpoint and resolve connectivity problems.

- Requirements

  - Confirm server is ready and then click "Next"

- Select Databases

  - Check the `SourceDB` box and then click "Next"

- Specify Secondary Replica

  - Replicas
    - Click "Add secondary replica" and sign in to Azure

  - Endpoints

  - Backup

  - Link Endpoint






RESUME HERE!!!

- 

- Enter the endpoint of the Azure SQL Managed Instance (as shown in the Azure Portal)

- Provide the SQL Server authentication credentials for the Managed Instance

- Specify which objects (schemas, tables) you want to replicate—in this demonstration, select the tables and objects from "SourceDB" that you just created

Confirm and apply the configuration

The system will initiate an initial snapshot of the selected data. Monitor the Managed Instance Link status in SSMS to ensure the process is active and the data is being replicated

These revised steps ensure that the necessary databases and schema are created before you configure Managed Instance Link, providing a clear and dependency‑aware process for setting up data replication.

------------------------- -------------------------

### Configure Managed Instance Link on the On‑Prem SQL Server

Launch SQL Server Management Studio on the Azure virtual machine and connect to the on‑prem SQL Server instance.

Identify the database that will be replicated (or create it if needed). For this demonstration, a custom sample database will be created later

#### Launch the Managed Instance Link Configuration  
- Right‑click the target database and choose the option to configure Managed Instance Link  
- The configuration wizard will appear  
- Enter the endpoint of the Azure SQL Managed Instance (as shown in the Azure Portal)  
- Provide the SQL authentication credentials for the Managed Instance  
- Specify which objects (schemas, tables) you want to replicate—for this demonstration, choose the custom sample database objects that will be created

#### Confirm the Configuration  
After reviewing the settings, confirm and apply the configuration. The system will initiate an initial snapshot of the selected data, and the Managed Instance Link status should update accordingly in SQL Server Management Studio

-------------------------

### Test the Data Replication

#### Perform Data Changes on the On‑Prem SQL Server  
In SQL Server Management Studio on the Azure virtual machine, execute operations (inserts, updates, deletes) on the SourceDB tables. For example:

```sql
UPDATE Customers SET Email = 'john.updated@example.com' WHERE CustomerID = 1;
INSERT INTO Orders (OrderID, CustomerID, OrderDate, Amount) VALUES (102, 1, GETDATE(), 200.00);
DELETE FROM Orders WHERE OrderID = 101;
```

#### Monitor Replication Status  
- Check the Managed Instance Link status within SQL Server Management Studio to ensure that the changes are being captured and sent  
- Look for any errors or latency issues in the replication log or the status dashboard

#### Verify Data on the Azure SQL Managed Instance  
Connect to the Managed Instance using SQL Server Management Studio and query the SourceDB. For example:

```sql
SELECT * FROM Customers;
SELECT * FROM Orders;
```

Confirm that the data reflects the changes made on the on‑prem server
