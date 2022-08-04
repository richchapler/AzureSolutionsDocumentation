## Data Analysis: Time Windowing

Requirement statements might include:

* "We want to use our time-series data to characterize Sensor 'Activity Runs' {what we call the period between Active and Inactive}"

### Step 1: Infrastructure Resources

This solution requires the following resource:

* [Data Explorer](Infrastructure_DataExplorer.md) >> Cluster

### Step 2: Prepare Sample Data

First, we will transform the **RawSensorsData** sample dataset.

  <img src="https://user-images.githubusercontent.com/44923999/182669711-cfb91e83-c71f-490d-887c-d5b54156a212.png" width="800" title="Snipped: August 2, 2022" />

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then on the **Home** page, click **Explore sample data with KQL**

* On the **Explore data samples** pop-up, click to select **IoT data** and then click **Explore**

* Paste the following KQL and then click **Run**

  ```
  let t = 
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
      | where Sensor == "sensor-99";
  t
  ```

  _Notes:_<br>
  _* `let...` is used to encapsulate logic because we lack `.create function...` permissions to the sample database_<br>
  _* `project ... State` is used to characterize rows as Active or Inactive... the `iff( value > 0.1..` conditional is arbitrary_<br>
  _* `where ... sensor == "sensor-99"` is used to limit the displayed data but won't be used in future steps_

### Step 3: Serialize Transformed Data

Next, we will serialize "start of run" from the previously transformed data.

  <img src="https://user-images.githubusercontent.com/44923999/182633642-fb10c967-e219-4224-bb08-1e25fc266583.png" width="800" title="Snipped: August 2, 2022" />

Complete the following steps:

* Add the following KQL and then click **Run**

  ```
  | where State == "Active"
  | project Sensor, Start = Timestamp
  | sort by Sensor, Start asc 
  | serialize 
  | extend nextStart = next(Start, 1)
  ```

### Step 4: Finalize Analysis

Finally, we will join "end of run", include an Activity Run Identifier and calculate Delta.

  <img src="https://user-images.githubusercontent.com/44923999/182653189-c5519d5f-c695-4617-b713-f780897b7f6f.png" width="800" title="Snipped: August 2, 2022" />

Complete the following steps:

* Add the following KQL and then click **Run**

```
  | join kind=inner ( t | where State == "Inactive" | project Sensor, End = Timestamp ) on Sensor
  | where End between (Start .. nextStart)
  | sort by Start asc
  | project Sensor, RunId = tostring(row_number()), Start, nextStart, End, Delta = datetime_diff('millisecond', End, Start)
```
