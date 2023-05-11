# Data Surface: GeoSpatial Visualization with Azure Maps (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0596e57b-78be-413e-96c5-828db0288d6a" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to analyze real-time, streaming GPS coordinates"
* "We want to employ powerful geospatial capabilities and the freshest mapping data"
* "We tried Power BI and it doesn't keep up with the size of our dataset"

## Prerequisites
This solution requires the following resources:

* [**Application Service**](https://learn.microsoft.com/en-us/azure/app-service/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [**Maps**](https://learn.microsoft.com/en-us/azure/azure-maps)
  * **CORS** >> Allowed Origins: `*`
  * System-Assigned **Managed Identity**: `On`
* [**Storage Account**](Infrastructure_StorageAccount.md)
* [**Visual Studio**](https://visualstudio.microsoft.com)

## Proposed Solution
Surface Azure Maps in a Web Application.

This documentation will address solution requirements in ##### exercises:

* Exercise 1: Create Basic Web Application
* Exercise 2: Prepare Sample Data Stream
* Exercise 3: Connect and Visualize Data

-----

## Exercise 1: Create Basic Web Application
In this exercise, we will use Visual Studio to create a basic web application and publish to Azure.

### Step 1: Create Project

Open Visual Studio.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c19a84db-d898-4ab2-9456-b168c1cd9220" width="600" title="Snipped: May 10, 2023" />

Under "**Get started**", click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e67290b5-e6e6-40fa-931e-565225bc0c63" width="600" title="Snipped: May 10, 2023" />

On the "**Create a new project**" page, click "**ASP.NET Core Web App**" and then "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/67d7bed8-c452-4cb1-a29d-84fa5c6c4f5c" width="600" title="Snipped: May 10, 2023" />

Complete the "**Configure your new project**" form and then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/22b966d0-4eae-4610-a8fc-a37490aec2a3" width="600" title="Snipped: May 10, 2023" />

Complete the "**Aditional information**" form and then click "**Next**".

Prompt | Entry
:----- | :-----
**Framework** | `.NET 7.0 (Standard Term Support)`
**Authentication type** | `None`
**Configure for HTTPS** | checked

Click "**Create**".

### Step 2: Edit Homepage

Open "**Pages**" >> `index.cshtml`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a54060a7-a6d8-476d-9227-a4770484cca0" width="800" title="Snipped: May 10, 2023" />

Replace the default `index.cshtml` code with:
```
@page
@{
    ViewData["Title"] = "Home page";
}

<html>
<head>
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <style>
        html, body {
            margin: 0
        }
        #myMap {
            position: relative;
            top: 0px;
            left: 300px;
            width: calc(90vw - 300px);
            height: 80vh;
        }
    </style>

    <script type="text/javascript">
        function InitMap() {
            var map = new atlas.Map('myMap', {
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '{AZUREMAPS_PRIMARYKEY}'
                }
            });
        }
    </script>
</head>

<body onload="InitMap()">
    <div id="myMap"></div>
</body>
</html>
```

Logic explained:
* `<link rel="stylesheet"...` establishes a link to the Azure Maps Web SDK stylesheet
* `<script src="https://atlas.microsoft.com...` establishes a link to the Azure Maps Web SDK
* `<style>... #myMap...` describes layout of the Map component using both percent of viewport and pixel count
* `<script...> function InitMap()...` defines a function that creates a map authenticated using Azure Maps, Primary Key
<br><br>  _Note: Replace `{AZUREMAPS_PRIMARYKEY}` with your Azure Maps, Primary Key (found in "Settings" >> "**Authentication**")_

### Step 3: Publish

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4625385c-2f9d-4c3b-969a-ba44c59a19da" width="800" title="Snipped: May 10, 2023" />

Right-click on project `WebApplication_AzureMaps` and select "**Publish**" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bb34770b-c57a-4d81-8f0d-e07069ed774f" width="600" title="Snipped: May 10, 2023" />

On the "**Publish**" popup, "**Target**" tab, select "**Azure**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e2d0a43f-5ddf-4e4d-b486-9135ca778f6c" width="600" title="Snipped: May 11, 2023" />

On the "**Publish**" popup, "**Specific target**" tab, select "**Azure App Service (Windows)**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5b195526-9410-4c0a-a672-d9fdba90142c" width="600" title="Snipped: May 11, 2023" />

On the "**Publish**" popup, "**App Service**" tab, select your Application Service, then click "**Finish**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/47f7b5b3-4143-4ca4-9420-4cc19a2ed7ca" width="600" title="Snipped: May 11, 2023" />

Confirm success on the "**Publish**" popup, "**Finish**" tab, select your Application Service, then click "**Close**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/32b3204a-7c0d-4c32-84ca-371aab262075" width="800" title="Snipped: May 11, 2023" />

On the `WebApplication_Azure Maps...: Publish` page, click "**Publish**".

-----

## Reference

* [Azure Maps Samples](https://samples.azuremaps.com/)
* [Azure.ResourceManager.Maps Namespace](https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.maps?view=azure-dotnet)
https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.maps?view=azure-dotnet
