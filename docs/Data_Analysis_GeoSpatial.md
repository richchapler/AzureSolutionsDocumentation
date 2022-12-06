# Data Analysis: Geo-Spatial Visualization

![image](https://user-images.githubusercontent.com/44923999/205741474-6f1e333d-bc35-48b0-831d-4a8e38e31af9.png)

There are many, many options for visualizing and analyzing geo-spatial data.
This documentation details step-by-step instructions for **some** of the Power BI-based options.

| Option | Description | Pros | Cons |
| ----- | ----- | ----- | ----- |
| **Shape Maps** | Compare regions using color | - Simple, Quick, Flexible | - Only Power BI Desktop<br>- Low Precision |
| **Filled Maps** | Compare regions using shading, tinting, or patterns | - | - |
| **ArcGIS Maps** | - | - | - |

_Note: This list of Pros and Cons is based on my research, experience and perception ONLY._

## Step 0: Understand Use Case
This use case considers the following requirement statements:

* "We want to use geo-spatial visualization to help us better understand our data, but we do not know where to start"

## Step 1: Prepare Infrastructure
This solution requires the following resources:

* Data Explorer Samples (no need to instantiate anything new)
* [Power BI](https://powerbi.microsoft.com/en-us/desktop/)

## Step 2: Get Data
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

  <img src="https://user-images.githubusercontent.com/44923999/205970985-7eb253d5-828f-4642-aa52-371be952cb7a.png" width="800" title="Snipped: December 6, 2022" />

* Click the **Data** icon in the left-hand navigation to review imported **StormEvents** data and confirm success

* To save changes, click **File** >> "**Save as**" and then complete the pop-up

## Step 3: Evaluate Shape Maps
Reference: https://learn.microsoft.com/en-us/power-bi/visuals/desktop-shape-map

In this step, we will get setup and evaluate Shape Maps.

* Click **File** >> "**Options and Settings**" >> **Options** >> "**Preview Features**"

  <img src="https://user-images.githubusercontent.com/44923999/205976019-55ac56fc-b74a-4a3f-9fb3-6e2384d9afab.png" width="600" title="Snipped: December 6, 2022" />

* Check the "**Shape map visual**" checkbox, click **OK** and then restart **Power BI Desktop**

* Open your saved file and then click the **Report** icon in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/205978728-4bdd4805-e990-4784-975f-db73d72035eb.png" width="800" title="Snipped: December 6, 2022" />

* Click the new "Shape map" icon in the **Visualizations** pane and resize the resulting visual in the main body (as desired)

  <img src="https://user-images.githubusercontent.com/44923999/205986949-1ca6170a-f034-4121-8e0d-4645e0d0bdb7.png" width="800" title="Snipped: December 6, 2022" />

* Drag-and-drop from the **Fields** pane to the **Visualizations** pane, including:

  | Bucket | Field |
  | ----- | ----- |
  | **Location** | **State** |
  | **Color saturation** | **Deaths Direct** |
  
  _Note from documentation: "The Location field in the Azure Maps Power BI Visual can accept multiple values, such as country, region, state, city, street address and zip code. By providing multiple sources of location information in the Location field, you help to guarantee more accurate results and eliminate ambiguity that would prevent a specific location to be determined. For example, there are over 20 different cities in the United States named Franklin."_
  
  <img src="https://user-images.githubusercontent.com/44923999/205988885-b90e04ca-fdcc-4c56-8dbe-6902f5d75620.png" width="800" title="Snipped: December 6, 2022" />

* Click the "Format your visual" tab on the **Visualizations** pane and explore settings {e.g., "**Zoom on selection**"}

* Rename the "**Page 1**" tab to "**Shape Map**" and save changes

## Step 4: Evaluate Filled Maps
Reference: https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths

In this step, we will get setup and evaluate Filled Maps.

* Click **File** >> "**Options and Settings**" >> **Options** >> **Security**

  <img src="https://user-images.githubusercontent.com/44923999/205995080-5ddff466-8874-4e65-8384-7be893654a47.png" width="600" title="Snipped: December 6, 2022" />

* Check the "**Use Map and Filled Map Visuals**" checkbox, click **OK** and then restart **Power BI Desktop**

* Open your saved file, and then click the **Report** icon in the left-hand navigation
* Right-click on the "**Shape Map**" tab, select "**Duplicate Page**" in the resulting drop-down, and then rename the new tab to "**Filled Map**"

  <img src="https://user-images.githubusercontent.com/44923999/205997326-9dc4b563-7c33-4070-93f2-e831656f66a0.png" width="800" title="Snipped: December 6, 2022" />

* Select the map visual and then click the "**Filled Map**" icon in the **Visualizations** pane

------------------------------------------------------------------------

# Notes

* Filled Maps - https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths?tabs=powerbi-desktop
* ArcGIS Maps - https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualizations-arcgis
* Power BI, Azure Maps - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-get-started?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
* Add a bar chart - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-bar-chart-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
* Add a pie chart - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-pie-chart-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
* Add a heat map - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-heat-map-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
* GeoJSON - https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-reference-layer?context=%2Fpower-bi%2Fcreate-reports%2Fcontext%2Fcontext
