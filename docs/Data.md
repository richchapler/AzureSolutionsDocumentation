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
  [...from Bing Maps API](Data_Acquisition_BingMapsAPI.md) | Synapse Pipeline | Bing Maps API | Data Explorer | Low Code
  [...from Cost Management API](Data_Acquisition_CostManagement.md) | Synapse Pipeline, Logic App, and Function App | Azure API | Data Explorer | various
  [...from IoT Simulation](Data_Acquisition_IoTSimulation.md) | Continuous Ingestion (Event Hub) | IoT Central | Data Explorer | No Code
  [...from Log Analytics API](Data_Acquisition_LogAnalyticsAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Purview API](Data_Acquisition_PurviewAPI.md) | Synapse Pipeline | Azure API | Data Lake | Low Code
  [...from Resource Diagnostics](Data_Acquisition_ResourceDiagnostics.md) | Continuous Ingestion (Event Hub) | Resource Diagnostics | Data Explorer | No Code
  [Python >> ADX](Data_Acquisition_Python>>DataExplorer.md) | Databricks | Data Explorer | Data Frame | Python

  

### Enrich

  Use Case | Method | Source | Target | Type
  :----- | :----- | :----- | :----- | :-----
  [Cognitive Search, Custom "Get Data" Skillset](Data_Enrichment_CognitiveSearch_CustomSkillset.md) | Custom Skills >> API | SQL | Cognitive Search Index | C# (Function App)
  [Mid-Stream Processing via Function App](Data_Enrichment_MidStreamProcessing_viaFunctionApp.md) | Continuous Ingestion | Event Hub | Data Explorer | C# (Function App)
  [Mid-Stream Processing via Stream Analytics](Data_Enrichment_MidStreamProcessing_viaStreamAnalytics.md) | Stream Analytics | IoT Hub | Data Explorer | SQL
  [OpenAI](Data_Enrichment_OpenAI.md) | Synapse Pipeline | OpenAI | Data Explorer | API

### Analyze

  Use Case | Method | Language
  :----- | :----- | :-----
  [JSON Discovery](Data_Analysis_JSONDiscovery.md) | Data Explorer | KQL
  [Time Windowing](Data_Analysis_TimeWindowing.md) | Data Explorer | KQL

### Surface

  Use Case | Source | Surface | Method
  :----- | :----- | :----- | :-----
  [Application Programming Interface (API)](Data_Surface_API.md) | | Function App + APIM | C# (Function App)
  [Bulk Export](Data_Surface_BulkExport.md) | | Synapse Pipeline | Low Code
  [Cognitive Search, from Data Explorer](Data_Surface_CognitiveSearch_fromDataExplorer.md) | Data Explorer (via export) | Cognitive Search | JSON and HTML
  [Cognitive Search, Multi-Source Index](Data_Surface_CognitiveSearch_MultiSourceIndex.md) | SQL and Blob Storage | Cognitive Search | JSON
  [Cognitive Search, PowerApp + Security](Data_Surface_CognitiveSearch_PowerApp+Security.md) | Sample Data | Power Apps | JSON
  [Geo-Spatial Visualization with Power BI](Data_Surface_GeoSpatial_PowerBI.md) | Data Explorer | Power BI | No Code
  [Geo-Spatial Visualization with Azure Maps](Data_Surface_GeoSpatial_AzureMaps.md) | IoT Central + Data Explorer | Web App + Azure Map | CSHTML + Javascript

### Govern

  Use Case | Source | Surface | Method
  :----- | :----- | :----- | :-----
  [Purview >> Alation Bridge (WiP)](Data_Governance_PurviewAlationBridge.md) | Purview | Alation | Logic Apps + API
  
