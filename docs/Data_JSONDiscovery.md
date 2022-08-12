## Data Analysis: JSON Discovery

Requirement statements might include:

* "Our data includes a dynamic column filled with JSON subject to frequent schema changes"
* "We want a way to discover all of the column headers that might be derived from the JSON over many rows"

### Step 1: Stage Sample Data

Lorem Ipsum

### Step 2: Enumerate Root Keys

First, we will transform the **RawSensorsData** sample dataset.

  <img src="https://user-images.githubusercontent.com/44923999/182669711-cfb91e83-c71f-490d-887c-d5b54156a212.png" width="800" title="Snipped: August 2, 2022" />

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then on the **Home** page, click **Explore sample data with KQL**

  ```
  let dt = datatable(c:string)
  [
  '{"c1":"lorem","c2":"ipsum"}'
  ,'{"c2":"dolor","c4":"sit"}'
  ,'{"c3":"amet"}'
  ];
  dt
  | project headers = bag_keys(todynamic(c))
  | mv-expand headers
  | distinct tostring(headers)
  | sort by headers asc
  ```

### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/bagkeysfunction
