## [Data Explorer](Infrastructure_DataExplorer.md) >> **Ingestion Mapping**

![image](https://user-images.githubusercontent.com/44923999/185751628-db6489fe-5e2f-418d-a3df-1a1b19d51479.png)

### Create with Azure Portal

Complete the following steps:

* Navigate to Data Explorer and then select Query from the navigation

  <img src="https://user-images.githubusercontent.com/44923999/182936856-ef5c140b-2cc1-45a8-ba15-b1a89cca71a7.png" width="800" title="Snipped: August 4, 2022" />

* Modify and **Run** the following KQL:

  ```
  .create table t ingestion json mapping 't_mapping' '[{"column":"d","path":"$","datatype":"dynamic"}]'
  ```
