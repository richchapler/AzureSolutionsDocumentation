# SQL Server: Database Administration  

**Reference:**  
• [SQL Server Educational Resources](https://learn.microsoft.com/en-us/sql/sql-server/educational-sql-resources)

**Overview:**  
This lab offers hands-on experience with core administrative tasks—covering server setup, maintenance, troubleshooting, database creation, and user/permission management. It is structured to support both on-premises SQL Server and Azure SQL environments, with exam-style questions to reinforce key concepts. Enhancements include updated guidance on modern T-SQL commands, detailed troubleshooting steps, and best practices for both environments.

---

## Lab Objectives

- **Fundamentals of Database Administration**  
  - Master server installation, configuration, and routine maintenance.  
  - Learn to monitor and adjust performance settings, including memory, CPU, and error logs.

- **Database Creation, Users & Permissions**  
  - Create databases using T-SQL and the SSMS GUI.  
  - Understand the difference between logins and database users, and practice mapping logins to users with appropriate roles.  
  - Compare legacy commands (e.g., `sp_addrolemember`) with modern alternatives (e.g., `ALTER ROLE ... ADD MEMBER`)—noting compatibility and exam expectations.

- **Exam Guidance & Best Practices**  
  - Develop familiarity with exam-style scenarios such as user authentication, role membership, and troubleshooting access issues.  
  - Reinforce learning with exam-style questions that mirror real-world and exam scenarios.

- **Dual Platform Coverage**  
  - **On-Premises SQL Server:** Detailed procedures, updated best practices, and troubleshooting guidelines for a traditional on-prem environment.  
  - **Azure SQL:** Essential tasks including setup via the Azure portal, firewall configurations, and diagnostic tools alongside SSMS.

---

## Part 1: On-Premises SQL Server Lab

### 1. Environment Setup (On-Premises)

**Pre-requisites:**  

- Install the latest version of SQL Server and SQL Server Management Studio (SSMS).  
- Ensure that you have the necessary permissions and network access to your SQL Server instance.  
- **Tip:** Use Windows Authentication where appropriate and keep your SSMS updated to leverage new features and security improvements.

**Initial Connection:**  

- Launch SSMS and connect to your SQL Server instance.  

- Run the following query to verify connectivity and confirm the server version:  

  ```sql
  SELECT @@VERSION;
  ```

- **Enhanced Note:** Consider also verifying SQL Server services via SQL Server Configuration Manager.

### 2. Exploring On-Premises SQL Server Configuration

**Reviewing Server Properties:**  

- In SSMS, right-click the server node and select **Properties**.  
- Review key settings such as memory allocation, CPU usage, and check the error logs.  
- **Enhanced Checklist:**  
  - Verify that max memory settings are optimized for your workload.  
  - Confirm the configuration of tempdb files.  
  - Check SQL Server Agent and maintenance plan settings.

**Exam Guidance:**  

- Questions often test your ability to navigate server properties and understand performance settings.

### 3. Database Creation (On-Premises)

**Using T-SQL:**  

- Create a new database named `TrainingDB`:

  ```sql
  CREATE DATABASE TrainingDB;
  GO
  ```

- Refresh Object Explorer to confirm the new database.

**Using SSMS GUI (Optional):**  

- Right-click **Databases**, select **New Database…**, and follow the prompts.

**Enhanced Best Practice:**  

- Consider reviewing the file layout (data and log files) for performance tuning and recovery objectives.

**Exam Guidance:**  

- Know both methods; exam questions may ask which T-SQL command creates a new database.

### 4. Creating and Managing Users (On-Premises)

**Creating a SQL Server Login:**  

- Execute the following to create a login:

  ```sql
  CREATE LOGIN TestUser WITH PASSWORD = 'YourPassword123';
  GO
  ```

**Mapping Login to a Database User:**  

- Switch to the `TrainingDB` database and create a user:

  ```sql
  USE TrainingDB;
  GO
  CREATE USER TestUser FOR LOGIN TestUser;
  GO
  ```

- **Assigning a Role:**  

  - **Legacy Method (still common in exam scenarios):**

    ```sql
    EXEC sp_addrolemember 'db_datareader', 'TestUser';
    GO
    ```

  - **Modern Alternative:**  

    ```sql
    ALTER ROLE db_datareader ADD MEMBER TestUser;
    GO
    ```

  - **Enhanced Note:** Although many exams reference `sp_addrolemember`, note that Microsoft recommends the `ALTER ROLE ... ADD MEMBER` syntax in newer versions of SQL Server.

**Verifying Permissions:**  

- Optionally, log in as TestUser and run a query (e.g., `SELECT TOP 10 * FROM SomeTable;`) to validate access.

**Exam Guidance:**  

- Understand the distinction between logins and users, and be prepared to map them correctly in exam questions.

### 5. Basic Troubleshooting (On-Premises)

**Simulated Scenario:**  

- If TestUser is unable to query data from TrainingDB, review the error message and follow these steps:

**Troubleshooting Checklist:**  

- Confirm that TestUser exists in TrainingDB.  
- Verify that TestUser is assigned to the appropriate role (e.g., `db_datareader`).  
- Check SQL Server error logs and Windows Event Logs for any related diagnostic information.  
- **Enhanced Tip:** Use the SQL Server Activity Monitor to check for blocking or performance issues.

**Exam Guidance:**  

- Exam questions may involve troubleshooting permission issues, so be comfortable with these steps.

---

## Part 2: Azure SQL Lab

### 1. Environment Setup (Azure SQL)

**Pre-requisites:**  

- Have an active Azure subscription and access to the [Azure Portal](https://portal.azure.com/).  
- Ensure you have the latest version of SSMS or Azure Data Studio installed.

**Initial Connection:**  

- Connect via SSMS using your Azure SQL server details.  

- Execute:

  ```sql
  SELECT @@VERSION;
  ```

  - **Note:** The output will indicate that this is an Azure SQL Database instance.

**Enhanced Considerations:**  

- Verify that your client IP is allowed through the Azure SQL firewall settings.

**Exam Guidance:**  

- Recognize the connectivity differences and how version information is presented in Azure SQL.

### 2. Exploring Azure SQL Configuration

**Reviewing Database Settings:**  

- In the Azure portal, navigate to your Azure SQL Database and review:
  - Performance tier and DTU/vCore settings.
  - Firewall rules and virtual network configurations.
  - Auditing and threat detection settings.

**Using SSMS:**  

- You can also execute T-SQL commands to review configurations and monitor performance metrics.

**Enhanced Note:**  

- Leverage the built-in diagnostic tools and performance insights available in the Azure portal for deeper analysis.

**Exam Guidance:**  

- Understanding both the Azure portal and SSMS methods is key for troubleshooting and exam scenarios.

### 3. Database Creation (Azure SQL)

**Using T-SQL in SSMS:**  

- Create a new database named `AzureTrainingDB`:

  ```sql
  CREATE DATABASE AzureTrainingDB;
  GO
  ```

**Using the Azure Portal (Optional):**  

- Select **Create a resource**, choose **SQL Database**, and follow the guided steps.

**Enhanced Best Practice:**  

- Configure backup and geo-replication settings based on your data protection requirements.

**Exam Guidance:**  

- Be familiar with both creation methods; exam questions may include Azure-specific features.

### 4. Creating and Managing Users (Azure SQL)

**Creating Contained Database Users:**  

- **SQL-Authenticated User:**

  ```sql
  CREATE USER AzureUser WITH PASSWORD = 'YourAzurePassword123';
  GO
  ```

- **Azure AD User:**  
  (For environments integrated with Azure Active Directory)

  ```sql
  CREATE USER [user@domain.com] FROM EXTERNAL PROVIDER;
  GO
  ```

**Assigning Roles:**  

- Add the user to a role using:

  ```sql
  EXEC sp_addrolemember 'db_datareader', 'AzureUser';
  GO
  ```

- **Modern Alternative (if supported):**

  ```sql
  ALTER ROLE db_datareader ADD MEMBER AzureUser;
  GO
  ```

**Enhanced Note:**  

- Azure SQL often uses contained database users, so ensure you understand the differences from on-premises user creation.

**Exam Guidance:**  

- Examine the differences in authentication methods and role assignment for Azure SQL.

### 5. Basic Troubleshooting (Azure SQL)

**Simulated Scenario:**  

- If an Azure SQL user cannot access data, perform these checks:

**Troubleshooting Checklist:**  

- Verify the user’s existence and role membership in the database.  
- Confirm that the client IP address is allowed by the Azure SQL firewall rules.  
- Use the Azure portal’s diagnostic tools to monitor and identify issues (e.g., Query Performance Insight).  
- Review auditing logs for additional context on failed connections.

**Exam Guidance:**  

- Azure-specific issues (like firewall settings) are common exam topics, so be sure to cover these aspects.

---

## Test Section

### On-Premises SQL Server Exam-Style Questions

**Question 1:**  
You have just created a new SQL Server login named TestUser. Which of the following commands correctly maps this login to a database user in TrainingDB and assigns the user to the db_datareader role?

**A.**  

```sql
USE TrainingDB;  
GO  
CREATE USER TestUser FOR LOGIN TestUser;  
GO  
EXEC sp_addrolemember 'db_datareader', 'TestUser';  
GO
```

**B.**  

```sql
CREATE USER TestUser FOR LOGIN TestUser;  
ALTER ROLE db_datareader ADD MEMBER TestUser;  
GO
```

**C.**  

```sql
USE TrainingDB;  
GO  
CREATE USER TestUser FOR LOGIN TestUser;  
GO  
GRANT SELECT TO TestUser;  
GO
```

**D.** Both A and C are correct.

**Answer:** A  
**Enhanced Explanation:** Option A explicitly creates the database user and assigns it to the `db_datareader` role using the legacy (but exam-recognized) stored procedure. Although Option B uses the modern `ALTER ROLE ... ADD MEMBER` syntax—which is valid on newer versions—exams often expect Option A.

---

**Question 2:**  
Which T-SQL command creates a new database named TrainingDB on an on-premises SQL Server?

**A.** `CREATE DATABASE TrainingDB;`  
**B.** `NEW DATABASE TrainingDB;`  
**C.** `MAKE DATABASE TrainingDB;`  
**D.** `ADD DATABASE TrainingDB;`

**Answer:** A  
**Explanation:** The correct T-SQL command to create a new database is `CREATE DATABASE TrainingDB;`.

---

**Question 3:**  
A user is unable to query data from TrainingDB on your on-premises server. What is the first step you should take to troubleshoot this issue?

**A.** Verify that the SQL Server instance is running.  
**B.** Check the SQL Server error logs.  
**C.** Verify that the user is correctly mapped to TrainingDB and has the appropriate role.  
**D.** Restart the SQL Server service.

**Answer:** C  
**Explanation:** The most direct troubleshooting step is to ensure the user is properly mapped to the database and has the necessary permissions.

---

### Azure SQL Exam-Style Questions

**Question 4:**  
You have created a contained database user in an Azure SQL Database named AzureTrainingDB. Which of the following commands creates a contained user with SQL authentication?

**A.**  

```sql
CREATE USER AzureUser WITH PASSWORD = 'YourAzurePassword123';
GO
```

**B.**  

```sql
CREATE LOGIN AzureUser WITH PASSWORD = 'YourAzurePassword123';
GO
```

**C.**  

```sql
CREATE USER [user@domain.com] FROM EXTERNAL PROVIDER;
GO
```

**D.** Both A and C are correct.

**Answer:** A  
**Explanation:** In Azure SQL, a contained database user is created directly in the database using `CREATE USER` with a password. Option C is used for Azure AD authentication.

---

**Question 5:**  
Which method is commonly used to connect to an Azure SQL Database for administrative tasks?

**A.** SQL Server Management Studio (SSMS)  
**B.** Azure Data Studio  
**C.** Azure Portal Query Editor  
**D.** All of the above

**Answer:** D  
**Explanation:** All these methods can be used to connect to and manage an Azure SQL Database.

---

**Question 6:**  
When troubleshooting access issues in an Azure SQL Database, what is one additional configuration to verify that is not typically an on-premises concern?

**A.** Database user mapping  
**B.** Role membership  
**C.** Firewall rules in the Azure portal  
**D.** SQL Server error logs

**Answer:** C  
**Explanation:** In Azure SQL, firewall rules set in the Azure portal can block access, making them an important troubleshooting aspect that is unique to cloud environments.

---

## Next Steps

1. **Walk Through the Lab:**  
   - Set up your on-premises SQL Server and/or Azure SQL environment.  
   - Execute each section’s instructions, noting any differences between the environments.

2. **Practice Exam-Style Questions:**  
   - Work through the test section to reinforce your understanding of key concepts.  
   - Consider practicing both legacy and modern T-SQL command syntax where appropriate.

3. **Explore Enhancements:**  
   - Review the troubleshooting checklists and enhanced best practices.  
   - Use diagnostic tools (e.g., SQL Server Activity Monitor, Azure Performance Insights) to deepen your understanding of performance and security management.

This enhanced lab document aims to not only guide you through basic administration tasks but also prepare you for exam scenarios and real-world troubleshooting. As you work through the lab, keep these enhancements in mind and adjust based on your specific environment and requirements.

Happy learning and best of luck with your lab walkthrough!
