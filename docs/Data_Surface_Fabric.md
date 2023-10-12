* SQL Server Download... https://www.microsoft.com/en-us/sql-server/sql-server-downloads
* SQL Server Management Studio (SSMS)... https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms
* AdventureWorks2022... https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure

SQL On-Prem >> Fabric Data Gateway >> Incremental Load >> OneLake >> Power BI

* Fabric, Data Engineering... https://msit.powerbi.com/home?experience=data-engineering

Microsoft Fabric provides two data stores: Data Warehouse and Lakehouse. The choice between the two depends on your specific needs and circumstances 12.

If you’re dealing with large volumes of unstructured or semi-structured data and have developers skilled in Spark, a Lakehouse may be the best choice. On the other hand, if you’re primarily dealing with structured data and your developers are more comfortable with SQL, a Warehouse might be more suitable 2.

Create Warehouse... rchaplerfwh

* Add On-Prem Data Gateway... https://learn.microsoft.com/en-us/data-integration/gateway/service-gateway-install
  * Download and install gateway
  * The name Gateway Cluster is misleading... you might add more On-Prem Data Gateways to the "cluster", but the "cluster" is created when you create the first on-prem data gateway (and can be used at that point)

Create Data Gateway... Settings >> Manage Connections and Gateways >> + New >> rchaplerdg
