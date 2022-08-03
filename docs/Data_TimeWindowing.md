## Data Analysis... Time Windowing

Requirement statements might include:

* "We want to use our time-series data to characterize Device 'Runs' {i.e., the period between Active and Inactive}"
* "We need to know key metrics {e.g., temperature} at the start and end of a run"

### Step 1: Infrastructure Resources

This solution requires the following resources:

* Data Explorer [Cluster](Infrastructure_DataExplorer_Cluster.md)

### Step 2: Prepare Sample Data

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then on the **Home** page, click **Explore sample data with KQL**


* On the **Explore data samples** pop-up, click to select **IoT data** and then click **Explore**

  <img src="https://user-images.githubusercontent.com/44923999/182495753-2caf6e57-109f-43f9-b825-ed77438cd22f.png" width="800" title="Snipped: August 2, 2022" />

* First, we will transform the **RawSensorsData** sample dataset; paste the following KQL and then click **Run**

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
      , State = iff(value >= 0.1, "Active", "Inactive")
  | take 10
  ```

  _Note: The State column is added to characterize values Active and Inactive... the "iff( value > 0.1.." conditional is arbitrary_
