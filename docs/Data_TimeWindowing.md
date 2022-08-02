## Data Analysis... Time Windowing

Requirement statements might include:

* "We want to use our time-series data to characterize Device 'Runs' {i.e., the period between State A and State B}"
* "We need to know key metrics {e.g., temperature} at the start and end of a run"

### Step 1: Infrastructure Resources

This solution requires the following resources:

* Data Explorer [Cluster](Infrastructure_DataExplorer_Cluster.md)

### Step 2: Sample Data

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then on the **Home** page, click **Explore sample data with KQL**


* On the **Explore data samples** pop-up, click to select **IoT data** and then click **Explore**

  <img src="https://user-images.githubusercontent.com/44923999/182453777-e1010579-29eb-4d54-9d4f-aff9d4d33b9c.png" width="800" title="Snipped: August 2, 2022" />

* To preview our transformation of the sample dataset, **Run** the following KQL:

  ```
  RawSensorsData
  | mvexpand sensors = rawdata['data'] to typeof(string)
  | extend jsondata = parse_json(sensors)
  | extend name = tostring(jsondata.name)
  | extend timestamp = jsondata.timestamp
  | mv-expand value = jsondata.values to typeof(double)
      , t1 = jsondata["timeDelta"] to typeof(long)
  | project Sensor = name
      , Timestamp = unixtime_microseconds_todatetime(t1 + timestamp)
      , Value = value
      , State = iff(value >= 0.1, "Active", "Dormant")
  | take 10
  ```

lorem
