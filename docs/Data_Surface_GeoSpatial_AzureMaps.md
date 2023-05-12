# Data Surface: Streaming GeoSpatial Visualization (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0596e57b-78be-413e-96c5-828db0288d6a" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to analyze real-time, streaming GPS coordinates from our vehicle fleet"
* "We want to employ powerful geospatial capabilities and the freshest mapping data"
* "We tried Power BI and it doesn't keep up with the size of our dataset"

## Prerequisites
This solution requires the following resources:

* [**Application Service**](https://learn.microsoft.com/en-us/azure/app-service/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/)
  * [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
  * [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
  * Database permissions for IoT Central System-Assigned Managed Identity
* [IoT Central](https://learn.microsoft.com/en-us/azure/iot-central/)
  * System-Assigned **Managed Identity**: `On` 
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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2018501f-9d34-4a0a-b0be-ea2dc86319cf" width="800" title="Snipped: May 11, 2023" />

On the `WebApplication_Azure Maps...: Publish` page, click "**Publish**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/99e9a946-7dd8-4f02-9aed-65a0442c23ac" width="800" title="Snipped: May 11, 2023" />

When complete, Visual Studio will open the new web application in a browser window.

-----

**Congratulations... you have successfully completed this exercise**

-----
WIP

```
Telemetry
| project latitude = toreal(telemetry.geolocation.lat), longitude = toreal(telemetry.geolocation.lon)
| where not(isnull(latitude)) and not(isnull(longitude))
| extend polygon = replace_string(replace_string(tostring(geo_h3cell_to_polygon(geo_point_to_h3cell(longitude, latitude, 2)).coordinates),'[[[','[['),']]]',']]')
| summarize quantity = count() by latitude, longitude, polygon
| order by quantity desc
```

```
using Kusto.Data;
using Kusto.Data.Net.Client;
using Microsoft.AspNetCore.Mvc.RazorPages;
using System.Data;
using System.Diagnostics;

namespace WebApplication1.Pages
{
    public class IndexModel : PageModel
    {
        public DataTable? coordinates { get; set; }
        public string? polygons { get; set; }

        public void OnGet()
        {
            var kcsb = new KustoConnectionStringBuilder("https://rchaplerdec.westus3.kusto.windows.net", "rchaplerded")
                .WithAadApplicationKeyAuthentication(
                    applicationClientId: "731995c1-a129-4a24-ab99-064dc4cfe2ac",
                    applicationKey: "klm8Q~_0Lu2WHYu2-~rCf03WQXKFd2uwYrlQ1bXr",
                    authority: "16b3c013-d300-468d-ac64-7eda0820b6d3"
                    );

            /* ***** Coordinates ***** */
            coordinates = new DataTable();
            coordinates.Columns.Add("latitude", typeof(double));
            coordinates.Columns.Add("longitude", typeof(double));
            coordinates.Columns.Add("count", typeof(int));

            using (var kcf = KustoClientFactory.CreateCslQueryProvider(kcsb))
            {
                var q = "Telemetry\r\n| project latitude = toreal(telemetry.geolocation.lat), longitude = toreal(telemetry.geolocation.lon)\r\n| where not(isnull(latitude)) and not(isnull(longitude))\r\n| summarize quantity = count() by latitude, longitude\r\n| order by quantity desc";

                var reader = kcf.ExecuteQuery(q);

                while (reader.Read())
                {
                    coordinates.Rows.Add(reader.GetDouble(0), reader.GetDouble(1), reader.GetInt64(2));
                }
            }

            /* ***** Polygons ***** */
            using (var kcf = KustoClientFactory.CreateCslQueryProvider(kcsb))
            {
                var q = "Telemetry\r\n| project longitude = toreal(telemetry.geolocation.lon), latitude = toreal(telemetry.geolocation.lat)\r\n| where not(isnull(latitude)) and not(isnull(longitude))\r\n| extend polygon = geo_h3cell_to_polygon(geo_point_to_h3cell(longitude, latitude, 10)).coordinates\r\n| summarize polygons = replace_string(tostring(make_list(polygon)),'\"','')";

                var reader = kcf.ExecuteQuery(q);

                while (reader.Read()) { polygons = reader.GetString(0); }
            }
        }
    }
}
```

```
@page
@model WebApplication1.Pages.IndexModel
@{
    ViewData["Title"] = "Home page";
}
@using System.Data

<html>
<head>
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

    <style>
        html, body {
            margin: 0
        }

        .form {
            position: absolute;
            top: 10px;
            left: 10px;
            width: 350px;
            height: calc(100vh - 50px);
        }

        #map {
            position: absolute;
            top: 10px;
            left: 410px;
            width: calc(100vw - 420px);
            height: calc(100vh - 30px);
        }

    </style>

    <script type="text/javascript">
        function Initialize() {
            var map = new atlas.Map(
                'map', /* Identifier */
                {
                    authOptions: { authType: 'subscriptionKey', subscriptionKey: 'hX_ICym11lSzClUnZP2lS8MUq4EGdkIFhf_jR21-RDQ' },
                    style: 'grayscale_dark',
                    center: [-122.121, 47.673],
                    zoom: 8
                }
            )

            map.events.add('ready', function ()
            {
                map.controls.add([new atlas.control.StyleControl({ mapStyles: 'all' })], { position: 'top-right' });

                dataSource = new atlas.source.DataSource();
                map.sources.add(dataSource);

                map.layers.add(new atlas.layer.PolygonLayer(dataSource, null, { fillColor: 'yellow' }), 'labels');

                dataSource.add(new atlas.data.Feature(new atlas.data.Polygon(@Model.polygons)));

                updateStatus();
            });
        }
    </script>
</head>

<body onload="Initialize()">
    <fieldset class="form">
        @if (Model.coordinates != null)
        {
            <div style="height:50%;overflow-y:scroll">
                <table>
                    <tr>
                        <th>Latitude</th>
                        <th>Longitude</th>
                        <th>Count</th>
                    </tr>
                    @foreach (DataRow row in Model.coordinates.Rows)
                    {
                        <tr>
                            <td>@row["latitude"]</td>
                            <td>@row["longitude"]</td>
                            <td>@row["count"]</td>
                        </tr>
                    }
                </table>
            </div>
        }
    </fieldset>
    <div id="map"></div>

</body>
</html>

```

-----

## Reference

* Azure Data Explorer
  * [geo_point_to_h3cell()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function)
* Azure Maps
  * [Azure Maps Samples](https://samples.azuremaps.com/)
  * [Azure.ResourceManager.Maps Namespace](https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.maps?view=azure-dotnet)
https://learn.microsoft.com/en-us/dotnet/api/azure.resourcemanager.maps?view=azure-dotnet
* Azure IoT Central
  * [Export IoT data to Azure Data Explorer](https://learn.microsoft.com/en-us/azure/iot-central/core/howto-export-to-azure-data-explorer)
  * [Transform data inside your IoT Central application for export](https://learn.microsoft.com/en-us/azure/iot-central/core/howto-transform-data-internally)
