## Data Analysis: JSON Discovery

Requirement statements might include:

* "Our data includes a dynamic column filled with JSON subject to frequent schema changes"
* "We want a way to discover all of the column headers that might be derived from the JSON over many rows"

### Step 1: Stage Sample Data

First, we will stage a `datatable` with some simple data.

  <img src="https://user-images.githubusercontent.com/44923999/184379741-939e57b0-7ffd-4c32-9f31-833fe06661f3.png" width="800" title="Snipped: August 12, 2022" />

Complete the following steps:

* Navigate to **Query** on https://dataexplorer.azure.com/
* Paste the following KQL and then click Run

  ```
  let dt = datatable(c:string)
  ['{"c1":"lorem","c2":"ipsum"}','{"c2":"dolor","c4":"sit"}','{"c3":"amet"}'];
  dt
  ```

* Confirm expected resultset {i.e., JSON-formatted columns with different names and data}

### Step 2: Enumerate Root Keys

Next, we will extract column header metadata.

  <img src="https://user-images.githubusercontent.com/44923999/184382729-3d241895-74db-484a-8736-7313cc2e6218.png" width="800" title="Snipped: August 12, 2022" />

Complete the following steps:

* Append the following KQL and then click Run

  ```
  | project headers = bag_keys(todynamic(c))
  ```

* Confirm expected resultset {i.e., JSON-formatted, column headers only}

### Step 3: Finalize Result

Finally, we will prepare the deliverable result.

  <img src="https://user-images.githubusercontent.com/44923999/184383036-6b0d6307-0634-40b3-a8f3-182aee304902.png" width="800" title="Snipped: August 12, 2022" />

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
