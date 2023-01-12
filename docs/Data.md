## Data

![image](https://user-images.githubusercontent.com/44923999/185972867-64465cc3-0769-4045-bc5d-672f573854c7.png)

### Migration

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [...from ADX to ADX](Data_Migration_ADXtoADX.md) | Synapse Pipeline | Data Explorer | Data Explorer | Low Code
  
### Acquisition

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [Pre-Ingestion Checklist](Data_Acquisition_PreIngestionChecklist.md) | Data Explorer | Sample Data | | KQL
  [One-Click Ingestion](Data_Acquisition_OneClickIngestion.md) | Data Explorer Wizard | Data Lake | Data Explorer | No Code
  [...from Cost Management API](Data_Acquisition_CostManagement.md) | Synapse Pipeline, Logic App, and Function App | Azure API | Data Explorer | various
  [...from IoT Simulation](Data_Acquisition_IoTSimulation.md) | Continuous Ingestion (Event Hub) | IoT Central | Data Explorer | No Code
  [...from Log Analytics API](Data_Acquisition_LogAnalyticsAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Purview API](Data_Acquisition_PurviewAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Resource Diagnostics](Data_Acquisition_ResourceDiagnostics.md) | Continuous Ingestion (Event Hub) | Resource Diagnostics | Data Explorer | No Code
  [Mid-Stream Processing](Data_Acquisition_MidStreamProcessing.md) | Continuous Ingestion (Event Hub) | Event Hub | Data Explorer | C# (Function App)

### Analysis

  Use Case | Method | Language
  :----- | :----- | :-----
  [Geo-Spatial Visualization](Data_Analysis_GeoSpatial.md) | Data Explorer >> Power BI | No Code
  [JSON Discovery](Data_Analysis_JSONDiscovery.md) | Data Explorer | KQL
  [Time Windowing](Data_Analysis_TimeWindowing.md) | Data Explorer | KQL

### Delivery

  Use Case | Method | Language
  :----- | :----- | :-----
  [Data Monetization](Data_Delivery_Monetization.md) | APIM | C#
