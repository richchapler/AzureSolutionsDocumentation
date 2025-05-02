# Data Explorer: Migrate to Fabric

## Activate [EventHouse](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)

> These steps will not be necessary for everyone

Navigate to https://app.fabric.microsoft.com, then click your profile picture in the upper-right of the screen.

In the resulting menu, **Switch tenant** (if necessary), and then click "Start Fabric trial".

On the resulting **Activate your 60-day free Fabric trial capacity** popup, confirm region and then click **Activate**.







Enable EventHouse on the Fabric Admin Portal:

Navigate to https://admin.microsoft.com/ >> **All admin centers** >> 

Go to Microsoft 365 Admin Center

Navigate to Fabric Admin Portal > Tenant Settings

Look for EventHouse or Real-Time Analytics preview features

Enable for your tenant or specific security groups

## [Create EventHouse](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/create-eventhouse)

Navigate to https://app.fabric.microsoft.com, then click **+ Create**.

Scroll down on the **New** page to **Real-Time Intelligence**, and click **Eventhouse**



1. Select "EventHouse" under "Real-Time Intelligence" (if not visible, your Fabric license level or region may not support it yet)
2. Click "New EventHouse"
3. Fill required fields:
   - Name
   - Workspace
   - Capacity (select Fabric capacity that supports EventHouse, typically F SKU or P SKU)
4. Create EventHouse

Reality check:  
- EventHouse is tightly linked to Fabric Real-Time hub concepts.  
- You will not get the same direct Azure portal feel like you did with Data Explorer (Kusto).
- Think of EventHouse as "Fabric-native Data Explorer lite" with native table schemas and event streams.

## Prepare Sample Data in Azure Data Explorer (Kusto)

1. Use an existing ADX cluster or spin up a new Dev SKU cluster
2. Create a database (if needed)

```kusto
.create database ['SampleDB']
```

3. Create a simple table

```kusto
.create table SensorData (DeviceId:string, Timestamp:datetime, Temperature:real)
```

4. Insert sample data

```kusto
.ingest inline into table SensorData <|
    "device01", datetime(2024-01-01T00:00:00Z), 22.5,
    "device02", datetime(2024-01-01T01:00:00Z), 23.0,
    "device03", datetime(2024-01-01T02:00:00Z), 21.7
```

Reality check:  
- Kusto types map closely to EventHouse types, but not 1:1.
- You must manually define EventHouse tables.

## Migrate Data from Azure Data Explorer to EventHouse

No direct "native" migration tools exist (yet). You have two realistic options:

**Option 1: Export from ADX and Import to EventHouse**

1. Export data from ADX to CSV/Parquet

```kusto
.export async compressed to csv (
    h@"https://{storageaccount}.blob.core.windows.net/{container}?{SAS}"
) 
<|
SensorData
```

2. Go to Fabric workspace
3. Use Dataflow Gen2 or Fabric Pipelines
4. Load CSV into EventHouse:
   - Define EventHouse table manually to match schema
   - Use built-in ingestion tools to pull from Storage Account

**Option 2: Query Stream into EventHouse**

(Only if real-time matters and you don't mind intermediate staging)

- Create EventStream in Fabric
- Connect ADX data via EventHub export (if you expose ADX data to EventHub — not common for just migration)
- Use EventStream to write into EventHouse

Reality check:  
- Option 2 is overkill unless you're migrating *live* streaming data
- For a batch move of history, Option 1 is what everyone is doing today

### Mapping Tips

| Azure Data Explorer | EventHouse | Notes |
|---------------------|------------|-------|
| string               | string     | ok    |
| real                 | float      | ok    |
| datetime             | datetime   | ok    |
| int                  | int        | ok    |
| dynamic              | not native | flatten first |

Dynamic columns (e.g., JSON blobs) need to be flattened first before loading to EventHouse.

---

Would you also like a PowerShell or REST-based version of these steps, since manual Fabric UI clicking is slow if you are scripting or at scale?  
Or an ARM template version (for setting up Fabric resources, although Fabric automation is still kind of messy)?