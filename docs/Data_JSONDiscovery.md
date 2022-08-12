## Data Analysis: JSON Discovery

Requirement statements might include:

* "Our data includes a dynamic column filled with JSON subject to frequent schema changes"
* "We want a way to discover all of the column headers that might be derived from the JSON over many rows"

### Step 1: Stage Sample Data

First, we will stage a `datatable` with some simple data.

Complete the following steps:

* Navigate to **Query** on https://dataexplorer.azure.com/

  <img src="https://user-images.githubusercontent.com/44923999/184379741-939e57b0-7ffd-4c32-9f31-833fe06661f3.png" width="800" title="Snipped: August 12, 2022" />

* Paste the following KQL and then click Run
* 
  ```
  let dt = datatable(c:string)
  ['{"c1":"lorem","c2":"ipsum"}','{"c2":"dolor","c4":"sit"}','{"c3":"amet"}'];
  dt
  ```

* Confirm expected resultset {i.e., JSON-formatted columns with different names and data}

### Step 2: Enumerate Root Keys

Complete the following steps:

* Append the following KQL and then click Run

  ```
  | project headers = bag_keys(todynamic(c))
  ```

* Confirm expected resultset {i.e., JSON-formatted, column headers only}

### Step 3: Finalize Result

Complete the following steps:

* Append the following KQL and then click Run

  ```
  | mv-expand headers
  | distinct tostring(headers)
  | sort by headers asc
  ```

* Confirm expected resultset {i.e., alphabetized list of column headers}

### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/bagkeysfunction
