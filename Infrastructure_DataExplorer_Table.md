## [Data Explorer](Infrastructure_DataExplorer.md) >> **Table**

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

### Create with Azure Portal
In this step, we will use the Azure Portal to create a table in our Data Explorer database.

* Navigate to Data Explorer and then select Query from the navigation

  <img src="https://user-images.githubusercontent.com/44923999/182936394-a49e10dc-9a28-4e4e-834a-ff432e28cd3e.png" width="800" title="Snipped: August 4, 2022" />

* Modify and **Run** the following KQL:

  ```
  .create table t ( d:dynamic )
  ```
