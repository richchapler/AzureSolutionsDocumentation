# Data

![image](https://user-images.githubusercontent.com/44923999/185972867-64465cc3-0769-4045-bc5d-672f573854c7.png)

## Migrate

Use Case | Method | Source | Target | Type
:----- | :----- | :----- | :----- | :-----
[...from ADX to ADX](Data_Migration_ADXtoADX.md) | Synapse Pipeline | Data Explorer | Data Explorer | Low Code
  
## Acquire

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
[Python >> ADX, Secure Connection](Data_Acquisition_Python>>DataExplorer.md) | Databricks | Data Explorer | Data Frame | Python

## Enrich

Use Case | Method | Source | Target | Type
:----- | :----- | :----- | :----- | :-----
[Mid-Stream Processing via Function App](Data_Enrichment_MidStreamProcessing_viaFunctionApp.md) | Continuous Ingestion | Event Hub | Data Explorer | C# (Function App)
[Mid-Stream Processing via Stream Analytics](Data_Enrichment_MidStreamProcessing_viaStreamAnalytics.md) | Stream Analytics Job | IoT Hub | Data Explorer | SQL
[Stream Analytics, Geo-Fencing](Data_Enrichment_StreamAnalytics_GeoFencing.md) | Stream Analytics Job | Event Hub and Storage Account | Not Applicable | Stream Analytics Query and Function

## Analyze

Use Case | Method | Language
:----- | :----- | :-----
[JSON Discovery](Data_Analysis_JSONDiscovery.md) | Data Explorer | KQL
[Time Windowing](Data_Analysis_TimeWindowing.md) | Data Explorer | KQL

## Surface

Use Case | Source | Surface | Method
:----- | :----- | :----- | :-----
[Application Programming Interface (API)](Data_Surface_API.md) | | Function App + APIM | C# (Function App)
[Bulk Export](Data_Surface_BulkExport.md) | | Synapse Pipeline | Low Code
[Geo-Spatial Visualization with Power BI](Data_Surface_GeoSpatial_PowerBI.md) | Data Explorer | Power BI | No Code
[Geo-Spatial Visualization with Azure Maps](Data_Surface_GeoSpatial_AzureMaps.md) | IoT Central + Data Explorer | Web App + Azure Map | CSHTML + Javascript

## Govern

Use Case | Source | Surface | Method
:----- | :----- | :----- | :-----
[Purview Asset Inventory](Data_Governance_PurviewAssetInventory.md) | Purview | Blob Storage | Logic Apps + API
[Purview >> Alation Bridge](Data_Governance_PurviewAlationBridge.md) | Purview | Alation | Logic Apps + API

## Manage

Use Case | Source | Surface | Method
:----- | :----- | :----- | :-----
[Data Factory: Debugging](DataFactory_Debugging.md) | non-specific | non-specific | Data Factory, Log Analytics
