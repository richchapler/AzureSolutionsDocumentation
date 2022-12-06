# Data Analysis: Geo-Spatial Visualization

![image](https://user-images.githubusercontent.com/44923999/205741474-6f1e333d-bc35-48b0-831d-4a8e38e31af9.png)

There are many, many options for visualizing and analyzing geo-spatial data.
This documentation details step-by-step instructions for **some** of these options.

| Option | Pros | Cons |
| ----- | ----- | ----- |
| **Power BI, Shape Map** | - | - |

_Note: This list of Pros and Cons is based on my research, experience and perception ONLY. Your answer to "why use Option X?" may be as simple as the fact that you favor that option._

This use case considers the following requirement statements:

* "We want to use geo-spatial visualization to help us better understand our data, but we do not know where to start"

### Step 1: Prepare Infrastructure

This solution requires the following resources:

* Data Explorer Samples (no need to instantiate anything new)
* [Power BI](https://powerbi.microsoft.com/en-us/desktop/)

### Step 2: Get Data

In this step, we will get sample data that will be used for all visualization exercises.

* Open Power BI Desktop

  <img src="https://user-images.githubusercontent.com/44923999/205939249-dbdd2005-e07c-4ba8-9480-9d2243a9a038.png" width="800" title="Snipped: December 6, 2022" />

* Click "**Get Data from another source**"

  <img src="https://user-images.githubusercontent.com/44923999/205939562-cabcc0c7-cdf5-42ee-86d4-7c3ce5ac09b0.png" width="600" title="Snipped: December 6, 2022" />

* On the "**Get Data**" pop-up, search for and select "**Azure Data Explorer (Kusto)**" and then click **Connect**

  <img src="https://user-images.githubusercontent.com/44923999/205969459-4dda392e-9994-4ec2-8c6b-fdcd9db907ee.png" width="600" title="Snipped: December 6, 2022" />

* On the "**Azure Data Explorer (Kusto)**" pop-up, enter Cluster value "**https://help.kusto.windows.net/**" and then click **OK**

  <img src="https://user-images.githubusercontent.com/44923999/205970278-d46aa816-40ed-446f-9d4f-492b77b9a5fc.png" width="600" title="Snipped: December 6, 2022" />

* On the **Navigator** pop-up, search for and select "**StormEvents**" and then click **Load**


## Power BI, Shape Map

Power BI, Maps
* Shape Maps - https://learn.microsoft.com/en-us/power-bi/visuals/desktop-shape-map
* Filled Maps - https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths?tabs=powerbi-desktop
ArcGIS Maps - https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualizations-arcgis

Power BI, Azure Maps
https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-get-started?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
Add a bar chart - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-bar-chart-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
Add a pie chart - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-pie-chart-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
Add a heat map - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-heat-map-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
GeoJSON - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-reference-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext

The location field
The Location field in the Azure Maps Power BI Visual can accept multiple values, such as country, region, state, city, street address and zip code. By providing multiple sources of location information in the Location field, you help to guarantee more accurate results and eliminate ambiguity that would prevent a specific location to be determined. For example, there are over 20 different cities in the United States named Franklin.
