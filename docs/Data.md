## Data

![image](https://user-images.githubusercontent.com/44923999/185972867-64465cc3-0769-4045-bc5d-672f573854c7.png)

### Migrate

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [...from ADX to ADX](Data_Migration_ADXtoADX.md) | Synapse Pipeline | Data Explorer | Data Explorer | Low Code
  
### Acquire

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [Pre-Ingestion Checklist](Data_Acquisition_PreIngestionChecklist.md) | Data Explorer | Sample Data | | KQL
  [One-Click Ingestion](Data_Acquisition_OneClickIngestion.md) | Data Explorer Wizard | Data Lake | Data Explorer | No Code
  [...from Cost Management API](Data_Acquisition_CostManagement.md) | Synapse Pipeline, Logic App, and Function App | Azure API | Data Explorer | various
  [...from IoT Simulation](Data_Acquisition_IoTSimulation.md) | Continuous Ingestion (Event Hub) | IoT Central | Data Explorer | No Code
  [...from Log Analytics API](Data_Acquisition_LogAnalyticsAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Purview API](Data_Acquisition_PurviewAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Resource Diagnostics](Data_Acquisition_ResourceDiagnostics.md) | Continuous Ingestion (Event Hub) | Resource Diagnostics | Data Explorer | No Code

### Enrich

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [Mid-Stream Processing via Function App](Data_Enrichment_MidStreamProcessing_viaFunctionApp.md) | Continuous Ingestion | Event Hub | Data Explorer | C# (Function App)
  [Mid-Stream Processing via Stream Analytics](Data_Enrichment_MidStreamProcessing_viaStreamAnalytics.md) | Stream Analytics | IoT Hub | Data Explorer | SQL
  [OpenAI](Data_Enrichment_OpenAI.md) | Synapse Pipeline | OpenAI | Data Explorer | API

### Analyze

  Use Case | Method | Language
  :----- | :----- | :-----
  [Geo-Spatial Visualization](Data_Analysis_GeoSpatial.md) | Data Explorer >> Power BI | No Code
  [JSON Discovery](Data_Analysis_JSONDiscovery.md) | Data Explorer | KQL
  [Time Windowing](Data_Analysis_TimeWindowing.md) | Data Explorer | KQL

### Surface

  Use Case | Method | Language
  :----- | :----- | :-----
  [...with an API](Data_Surface_API.md) | Function App + APIM | C# (Function App)
  [...with Bulk Export](Data_Surface_BulkExport.md) | Synapse Pipeline | Low Code
  [...with Cognitive Search](Data_Surface_CognitiveSearch.md) | Cognitive Search | Low Code
  [...with Cognitive Search (multi-source index)](Data_Surface_CognitiveSearch_MultiSourceIndex.md) | Cognitive Search | Low Code
