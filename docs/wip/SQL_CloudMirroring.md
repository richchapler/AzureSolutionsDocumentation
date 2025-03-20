# SQL: On-Prem to Cloud "Mirroring"

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

Always On Availability Groups (AG) provide an enterprise‑grade high availability and disaster recovery solution. With a Distributed AG, you can extend an on‑prem AG to include an Azure SQL target (either an Azure SQL Managed Instance or a SQL Server running on an Azure VM).

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

### Provision Virtual Machine

Sign in to the [Azure Portal](https://portal.azure.com) using your Azure credentials

#### Create a New Virtual Machine  

- Navigate to "Virtual Machines" and click "Create"  
- Select a resource group or create a new one for organizational purposes  
- Choose a Windows Server image (for example, Windows Server 2019 or 2022)  
- Select a VM size that meets the performance and licensing requirements for SQL Server  
- Configure networking by choosing an appropriate virtual network and subnet (a public IP is optional based on your connectivity needs)  
- Set administrative credentials (note that SQL Server authentication will be used later for Managed Instance Link)  
- Review your settings and click "Create" to provision the VM

#### Connect to the Azure VM via Remote Desktop  

Once the VM is running, use Remote Desktop Protocol (RDP) to connect and begin the SQL Server installation

#### Install SQL Server 2022  

- If SQL Server is not already installed, download the SQL Server 2022 installer from the Microsoft website  
- Run the installer on the VM and follow the installation wizard to set up a new SQL Server instance  
- During installation, select mixed‑mode authentication to enable SQL Server authentication  
- After installation, configure Windows Firewall to open port 1433 (the default SQL Server port)  
- Open SQL Server Management Studio (SSMS) and connect to your new SQL Server instance to verify that it is running

-------------------------

### Provision the Azure SQL Managed Instance

#### Create a New SQL Managed Instance  

- In the Azure Portal, search for "SQL Managed Instance" and select "Create"  
- Provide a name for the Managed Instance, choose an existing or create a new resource group, and select the appropriate region  
- Choose the compute and storage configurations that match your expected workload  
- Under networking, ensure that the Managed Instance is deployed in a virtual network configured to allow communication with your Azure VM  
- Set the administrative login credentials for the Managed Instance  
- Review your settings and click "Create" to start provisioning (provisioning may take some time)

#### Confirm Provisioning Completion  

Monitor the status in the Azure Portal and wait until the Managed Instance is fully provisioned before proceeding

-------------------------

### Ensure Connectivity Between the Azure VM and Managed Instance

#### Verify Virtual Network Settings  

Confirm that both the Azure VM and the Managed Instance reside in virtual networks (or subnets) that allow communication; this may involve ensuring they are in the same virtual network or configuring appropriate peering

#### Configure Firewall Rules  

If necessary, adjust network security group (NSG) rules or Managed Instance firewall settings to permit inbound connections on the SQL Server port (usually 1433)

#### Test Connectivity  

From the Azure VM, open SSMS and attempt to connect to the Managed Instance using its fully qualified domain name (FQDN) and the SQL Server authentication credentials configured earlier; a successful connection confirms that the network is correctly set up

-------------------------

### Configure Managed Instance Link on the On‑Prem SQL Server

#### Open SSMS on the Azure VM  

Launch SSMS on the Azure VM and connect to the on‑prem SQL Server instance

#### Select the Database to Replicate  

Identify the database that will be replicated (or create it if needed). For this demonstration, a custom sample database will be created later

#### Launch the Managed Instance Link Configuration  

- Right‑click the target database and choose the option to configure Managed Instance Link  
- The configuration wizard will appear  
- Enter the endpoint of the Azure SQL Managed Instance (as shown in the Azure Portal)  
- Provide the SQL authentication credentials for the Managed Instance  
- Specify which objects (schemas, tables) you want to replicate—for this demonstration, choose the custom sample database objects that will be created

#### Confirm the Configuration  

After reviewing the settings, confirm and apply the configuration. The system will initiate an initial snapshot of the selected data, and the Managed Instance Link status should update accordingly in SSMS

-------------------------

### Set Up a Custom Sample Database on the On‑Prem SQL Server

#### Create a New Database  

In SSMS, execute a T‑SQL command or use the GUI to create a new database (for example, "SampleDB")

#### Create Sample Tables and Schema  

Create a simple schema that might include tables such as Customers and Orders. For example:

```sql
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

#### Populate the Sample Database  

Insert a few rows into the tables to simulate transactional data:

```sql
INSERT INTO Customers (CustomerID, FirstName, LastName, Email)
VALUES (1, 'John', 'Doe', 'john.doe@example.com');

INSERT INTO Orders (OrderID, CustomerID, OrderDate, Amount)
VALUES (101, 1, GETDATE(), 150.00);
```

These sample records will serve as the initial data that the Managed Instance Link will replicate to the Azure SQL Managed Instance

-------------------------

### Test the Data Replication

#### Perform Data Changes on the On‑Prem SQL Server  

In SSMS on the Azure VM, execute operations (inserts, updates, deletes) on the SampleDB tables. For example:

```sql
UPDATE Customers SET Email = 'john.updated@example.com' WHERE CustomerID = 1;
INSERT INTO Orders (OrderID, CustomerID, OrderDate, Amount) VALUES (102, 1, GETDATE(), 200.00);
DELETE FROM Orders WHERE OrderID = 101;
```

#### Monitor Replication Status  

- Check the Managed Instance Link status within SSMS to ensure that the changes are being captured and sent  
- Look for any errors or latency issues in the replication log or the status dashboard

#### Verify Data on the Azure SQL Managed Instance  

Connect to the Managed Instance using SSMS and query the SampleDB. For example:

```sql
SELECT * FROM Customers;
SELECT * FROM Orders;
```

Confirm that the data reflects the changes made on the on‑prem server
