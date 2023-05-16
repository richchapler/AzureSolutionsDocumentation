# Data Surface: Geo-Spatial Visualization with Azure Maps (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0596e57b-78be-413e-96c5-828db0288d6a" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to analyze real-time, streaming GPS coordinates from our vehicle fleet"
* "We want to employ powerful geospatial capabilities (like in-map charting) and the freshest mapping data"
* "We tried Power BI and it doesn't keep up with the size of our dataset"

## Prerequisites
This solution requires the following resources:

* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
* [**Application Service**](https://learn.microsoft.com/en-us/azure/app-service/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
  * `Database User` permissions for the Application Registration
  * `Database Admin` permissions for the IoT Central System-Assigned Managed Identity
* [IoT Central](https://learn.microsoft.com/en-us/azure/iot-central/)
  * System-Assigned **Managed Identity**: `On` 
* [**Maps**](https://learn.microsoft.com/en-us/azure/azure-maps)
  * **CORS** >> Allowed Origins: `*`
  * System-Assigned **Managed Identity**: `On`
* [**Storage Account**](Infrastructure_StorageAccount.md)
* [**Visual Studio**](https://visualstudio.microsoft.com)

## Proposed Solution
Surface Azure Maps in a Web Application.

This documentation will address solution requirements in three exercises:

* Exercise 1: Create Web Application
* Exercise 2: Generate Sample Data
* Exercise 3: Visualize Geospatial Data

-----

## Exercise 1: Create Web Application
In this exercise, we will use Visual Studio to create a basic web application and publish to Azure.

### Step 1: Create Project

Open Visual Studio.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c19a84db-d898-4ab2-9456-b168c1cd9220" width="600" title="Snipped: May 10, 2023" />

Under "**Get started**", click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e67290b5-e6e6-40fa-931e-565225bc0c63" width="600" title="Snipped: May 10, 2023" />

On the "**Create a new project**" page, click "**ASP.NET Core Web App**" and then "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/67d7bed8-c452-4cb1-a29d-84fa5c6c4f5c" width="600" title="Snipped: May 10, 2023" />

Complete the "**Configure your new project**" form and then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/56dcd05f-9582-41ef-b16d-d9a933d3ba0c" width="600" title="Snipped: May 15, 2023" />

Complete the "**Aditional information**" form and then click "**Next**".

Prompt | Entry
:----- | :-----
**Framework** | `.NET 7.0 (Standard Term Support)`
**Authentication type** | `None`
**Configure for HTTPS** | unchecked

Click "**Create**".

### Step 2: Edit Homepage

Open "**Pages**" >> `index.cshtml`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8f2eec60-f614-408b-9f9e-aa1408b05473" width="800" title="Snipped: May 15, 2023" />

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
            left: 0px;
            width: 90vw;
            height: 85vh;
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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2757f785-c60c-4ffc-9a96-b3911d3db065" width="800" title="Snipped: May 15, 2023" />

When complete, Visual Studio will open the new web application in a browser window.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Generate Sample Data
In this exercise, we will use IoT Central to enable capture of streaming device data (using a mobile phone as source).

### Step 1: Configure Device

Open the IoT Central Application.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b5204083-cb41-4883-9f81-e9dabe821c1f" width="800" title="Snipped: May 15, 2023" />

Click "**Use phone as a device**" on the "**Devices**" page.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bce05e3d-bdd9-4140-ad87-d4af7b58acf6" width="800" title="Snipped: May 15, 2023" />

Follow directions on the resulting pop-up to setup your phone.

### Step 2: Prepare Destination

Open then Data Explorer Database, then "**Query**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/03821c39-1944-4922-b6d6-1a3908910c47" width="800" title="Snipped: May 15, 2023" />

Run the following KQL:

```
.create table Telemetry (applicationId: guid, deviceId: string, enqueuedTime: datetime, telemetry: dynamic)
```

Confirm success.

### Step 3: Data Export

Return to the IoT Central Application and then navigate to "**Data Export**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2a6d2729-9640-48b4-8e99-4ec6cc0a343e" width="800" title="Snipped: May 15, 2023" />

On the "**Destinations**" tab, click "**+ New destination**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ef2604b2-355a-45d9-bd6a-199f4f70d3f1" width="800" title="Snipped: May 15, 2023" />

Click "**Save**".

### Step 4: Data Export

Navigate to Iot Central Application >> "**Data Export**" page >> "**Exports**" tab.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/27039c35-b912-4c2d-be33-5f7a659d75b1" width="800" title="Snipped: May 15, 2023" />

Click "**+ New export**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9b5e612b-9c33-4147-8e8d-a792c6b87cb5" width="800" title="Snipped: May 15, 2023" />

Complete the "Exports" form:

Prompt | Entry
:----- | :-----
**Enter an export name** | Self-explanatory
**Type of data to export** | **Telemetry**

Click "**+ Destination**" and select the destination created in Step 3.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3627ff1c-eb9b-4587-acf2-f1ab97c8a0a2" width="800" title="Snipped: May 15, 2023" />

Click "**+ Transformation**" and on the resulting pop-out, select "**IoT Plug and Play mobile**", then click "**Add**".
<br>Back on the "**Exports**" >> "**Telemetry**" page, click "**Save**".

### Step 5: Confirm Success

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/750ceba2-53cc-442c-b155-40f0ed4ecdcc" width="800" title="Snipped: May 15, 2023" />

On the "**Data export**" page, you will see the new item, along with a status {e.g., "**Starting**" or "**Healthy**"}.
<br>Make sure that the "IoT PnP" app is running on your device and try moving around to capture interesting data.

_Note: The "IoT PnP" app only seems to log new "Geolocation" data when the app is opened (and closed / re-opened if already open)"_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9dd15a60-8ff7-4028-b693-fd4040ddb622" width="800" title="Snipped: May 15, 2023" />

Open Data Explorer Database >> "Query" and run the following KQL: `Telemetry`
<br>You can expect to see data flowing in (within a few minutes of initial configuration).

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 3: Visualize Geospatial Data
In this exercise, we will enhance the app from Exercise 1, and connect to the sample data from Exercise 2.

### Step 1: Nuget >> `Microsoft.Azure.Kusto.Data`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/94170541-f538-40a2-bab8-cc405dbf7d33" width="800" title="Snipped: May 16, 2023" />

Right-click on project `WebApplication_AzureMaps` and select "**Manage NuGet Packages...**" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0f743466-0fd7-4ecc-a758-4281dfde338c" width="800" title="Snipped: May 16, 2023" />

On the "**NuGet Package Manager**..." page, "**Browse**" tab, search for and select `Microsoft.Azure.Kusto.Data`, then click "**Install**" on the right-hand panel.

### Step 2: Prepare KQL
_Note: The goal of this step is to understand the query logic and the required dataset for steps that follow_

Open then Data Explorer Database, then "**Query**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/676d6066-26e9-414c-8345-34b0b8f01c69" width="800" title="Snipped: May 16, 2023" />

Run the following KQL:

```
Telemetry
| where not(isnull(telemetry.geolocation.lat)) and not(isnull(telemetry.geolocation.lon))
| summarize height = count() by
    polygon = tostring(geo_h3cell_to_polygon(geo_point_to_h3cell(toreal(telemetry.geolocation.lon), toreal(telemetry.geolocation.lat), 10)).coordinates)
    , color = toint(totimespan(now()-enqueuedTime) / 1h)
``` 

Logic explained:
* `Telemetry` is the table created and populated in Exercise 2
* `| where not(isnull(telemetry.geolocation.lat)) and...` filters out non-geospatial telemetry {e.g., barometer}
* `| summarize height = count()...` value will determine polygon extrusion height rendered on the map
* `polygon = ...` complex array of GPS coordinates {e.g., `[[[-123.2803,47.4275],[-123.2796,47.4280],...]]`}
* `geo_point_to_h3cell(...` calculates the H3 Cell token string value of a geographic location
* `geo_h3cell_to_polygon(...` calculates the polygon that represents the H3 Cell rectangular area
* `color = ...` generates a value to be used with C# `rgba` function to produce a gradient (lighter for newer data >> darker for older data)

### Step 3: `index.cshtml.cs` >> `public void OnGet()`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0e27ef99-a324-485b-9300-5558ebeaaca8" width="800" title="Snipped: May 16, 2023" />

Open `index.cshtml.com`, then modify / paste the following C# into the `OnGet` function:

```
var kcsb = new KustoConnectionStringBuilder("{DATAEXPLORER_URI}", "{DATAEXPLORER_DATABASENAME}")
    .WithAadApplicationKeyAuthentication(
        applicationClientId: "{APPLICATIONREGISTRATION_CLIENTID}",
        applicationKey: "{APPLICATIONREGISTRATION_CLIENTSECRET}",
        authority: "{TENANTID}"
    );

using (var kcf = KustoClientFactory.CreateCslQueryProvider(kcsb))
{
    var q = "Telemetry | where not(isnull(telemetry.geolocation.lat)) and not(isnull(telemetry.geolocation.lon)) | summarize height = count() by\r\n polygon = tostring(geo_h3cell_to_polygon(geo_point_to_h3cell(toreal(telemetry.geolocation.lon), toreal(telemetry.geolocation.lat), 10)).coordinates), color = toint(totimespan(now()-enqueuedTime) / 1h)";

    var reader = kcf.ExecuteQuery(q);

    while (reader.Read())
    {
        datasourceAdd_Polygons = string.Concat(datasourceAdd_Polygons, "dataSource.add(new atlas.data.Feature(new atlas.data.Polygon(", reader.GetString(0), "),{heightValue:", reader.GetInt64(2), ",colorValue:'rgba(0, 0, 255, 255-", reader.GetInt32(1), ")'}));\r\n");
    }
}
```

Logic explained:
* `var kcsb = new KustoConnectionStringBuilder...` creates a new instance of the class used to build connection strings for Data Explorer sources
* `.WithAadApplicationKeyAuthentication...` specifies authencation using an App Registration







LOREM ISPUM

-----
## WIP



### index.cshtml.cs
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
        public string? datasourceAdd_Polygons { get; set; }

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

            /* ***** datasourceAdd_Polygons ***** */
            using (var kcf = KustoClientFactory.CreateCslQueryProvider(kcsb))
            {
                var q = "Telemetry\r\n| where not(isnull(telemetry.geolocation.lat)) and not(isnull(telemetry.geolocation.lon))\r\n| summarize height = count() by\r\n    polygon = tostring(geo_h3cell_to_polygon(geo_point_to_h3cell(toreal(telemetry.geolocation.lon), toreal(telemetry.geolocation.lat), 10)).coordinates)\r\n    , color = toint(totimespan(now()-enqueuedTime) / 1h)";

                var reader = kcf.ExecuteQuery(q);

                while (reader.Read())
                {
                    datasourceAdd_Polygons = string.Concat(datasourceAdd_Polygons, "dataSource.add(new atlas.data.Feature(new atlas.data.Polygon(", reader.GetString(0), "),{heightValue:", reader.GetInt64(2), ",colorValue:'rgba(0, 0, 255, 255-", reader.GetInt32(1), ")'}));\r\n");
                }
            }
        }
    }
}
```

### index.cshtml
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
                    pitch: 60,
                    zoom: 8
                }
            )

            map.events.add('ready', function () {
                map.controls.add([new atlas.control.StyleControl({ mapStyles: 'all' })], { position: 'top-right' });

                dataSource = new atlas.source.DataSource();

                map.sources.add(dataSource);

                @Html.Raw(Model.datasourceAdd_Polygons);

                map.layers.add(
                    new atlas.layer.PolygonExtrusionLayer(dataSource, null, {
                        height: ['get', 'heightValue'],
                        fillColor: ['get', 'colorValue']
                    }), 'labels');

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
  * [Add a polygon extrusion layer to the map](https://learn.microsoft.com/en-us/azure/azure-maps/map-extruded-polygon)
  * [Add a bar chart layer](https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-bar-chart-layer)
* Azure IoT Central
  * [Export IoT data to Azure Data Explorer](https://learn.microsoft.com/en-us/azure/iot-central/core/howto-export-to-azure-data-explorer)
  * [Transform data inside your IoT Central application for export](https://learn.microsoft.com/en-us/azure/iot-central/core/howto-transform-data-internally)
