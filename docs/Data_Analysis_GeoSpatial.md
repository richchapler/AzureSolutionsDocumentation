# Data Analysis: Geo-Spatial Visualization

![image](https://user-images.githubusercontent.com/44923999/205741474-6f1e333d-bc35-48b0-831d-4a8e38e31af9.png)

There are many, many options for visualizing and analyzing geo-spatial data.
This documentation details step-by-step instructions for **some** of the Power BI-based options.

| Option | Description | Pros | Cons |
| ----- | ----- | ----- | ----- |
| **Shape Maps** | Compare locations {e.g., state} using color | - Simple, Quick, Flexible | - No GPS coordinate support |
| **Filled Maps** | Precise spatial comparison with gradient , etc. | - Supports GPS coordinates | |
| **Azure Maps** | Layer charts, etc. on top of map | - Many Visualization Options | |
| - Bar Chart | - | - | - |
| - Pie Chart | - | - | - |
| - Reference | Upload and overlay GeoJSON-based content | - | Does not apply GeoJSON properties {e.g., color} |

_Note: This list of Pros and Cons is based on my research, experience and perception ONLY._

## Understand Use Case
This use case considers the following requirement statements:

* "We want to use geo-spatial visualization to help us better understand our data, but we do not know where to start"

## Prepare Infrastructure
This solution requires the following resources:

* Data Explorer sample data (new instantiation unnecessary)
* [Power BI](https://powerbi.microsoft.com/en-us/desktop/)

## Exercise 1: Get Data
In this exercise, we will get sample data that will be used for all visualization exercises.

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

## Exercise 2: Evaluate Shape Maps
_https://learn.microsoft.com/en-us/power-bi/visuals/desktop-shape-map_

In this exercise, we will get setup and evaluate Power BI, Shape Maps.

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

* Click the "**Format your visual**" tab on the **Visualizations** pane and explore settings {e.g., "**Zoom on selection**"}

* Rename the "**Page 1**" tab to "**Shape Map**" and save changes

## Exercise 3: Evaluate Filled Maps
_https://learn.microsoft.com/en-us/power-bi/visuals/power-bi-visualization-filled-maps-choropleths_

In this exercise, we will get setup and evaluate Power BI, Filled Maps.

* Click **File** >> "**Options and Settings**" >> **Options** >> **Security**

  <img src="https://user-images.githubusercontent.com/44923999/205995080-5ddff466-8874-4e65-8384-7be893654a47.png" width="600" title="Snipped: December 6, 2022" />

* Check the "**Use Map and Filled Map Visuals**" checkbox, click **OK** and then restart **Power BI Desktop**

* Open your saved file, and then click the **Report** icon in the left-hand navigation
* Right-click on the "**Shape Map**" tab, select "**Duplicate Page**" in the resulting drop-down, and then rename the new tab to "**Filled Map**"

  <img src="https://user-images.githubusercontent.com/44923999/205997326-9dc4b563-7c33-4070-93f2-e831656f66a0.png" width="800" title="Snipped: December 6, 2022" />

* Select the map visual and then click the "**Filled Map**" icon in the **Visualizations** pane

  <img src="https://user-images.githubusercontent.com/44923999/206021877-cb9770c9-8a17-4d1a-9126-1dcad95df428.png" width="800" title="Snipped: December 6, 2022" />

* Click the "**Format your visual**" tab on the **Visualizations** pane and explore settings {e.g., **Style**, "**Zoom buttons**", etc.}
* Expand "**Fill colors**" >> **Colors** and then click the "**fx**" button

  <img src="https://user-images.githubusercontent.com/44923999/206022199-86c21fec-9989-4050-96fb-6072a158727f.png" width="600" title="Snipped: December 6, 2022" />

* Complete the resulting "**Default color - Fill colors**" pop-up, including:

  | Prompt | Entry |
  | ----- | ----- |
  | **What field should we base this on?** | Select "**DeathsDirect**" |
  | **Summarization** | Select "**Sum**" |

* Click **OK**

  <img src="https://user-images.githubusercontent.com/44923999/206022900-6c96ff06-33f6-4272-8d15-4470ada209f5.png" width="800" title="Snipped: December 6, 2022" />

#### Save and Publish

* Click **File** >> **Publish** >> "**Publish to Power BI**"
* On the resulting "**Publish to Power BI**" pop-up, select "**My workspace**" and then click **Select** (Save / Replace when prompted)

  <img src="https://user-images.githubusercontent.com/44923999/206024469-e19221fc-b8f4-4096-acd9-2a16ed2675ea.png" width="800" title="Snipped: December 6, 2022" />

## Exercise 4: Evaluate Azure Maps
_https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-get-started_

In this exercise, we will get setup and evaluate Power BI, Azure Maps.

* Click **File** >> "**Options and Settings**" >> **Options** >> "**Preview Features**"

  <img src="https://user-images.githubusercontent.com/44923999/206030992-eab86670-7849-4937-8537-a7fa3f1e0903.png" width="600" title="Snipped: December 6, 2022" />

* Check the "**Azure map visual**" checkbox, click **OK** and then restart **Power BI Desktop**

* Open your saved file, and then click the **Report** icon in the left-hand navigation
* Right-click on the "**Filled Map**" tab, select "**Duplicate Page**" in the resulting drop-down, and then rename the new tab to "**Azure Map**"

  <img src="https://user-images.githubusercontent.com/44923999/206033779-ccc3b3d1-a13f-4613-b943-9b0b849f4afc.png" width="800" title="Snipped: December 6, 2022" />

* Select the map visual and then click the "**Azure Map**" icon in the **Visualizations** pane

### Exercise 4a: Bar Chart Layer
_https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-bar-chart-layer_

  <img src="https://user-images.githubusercontent.com/44923999/206034982-ced01298-e96b-46f3-8080-6897609c5ec2.png" width="800" title="Snipped: December 6, 2022" />

* Click the "**Format your visual**" tab on the **Visualizations** pane
* Set "**Bubble layer**" to **Off** and "**Bar chart layer**" to **On**
* Explore additional visualization settings {e.g., Size, Shape, Colors, etc.}

#### Save and Publish

* Click **File** >> **Publish** >> "**Publish to Power BI**"
* On the resulting "**Publish to Power BI**" pop-up, select "**My workspace**" and then click **Select** (Save / Replace when prompted)

  <img src="https://user-images.githubusercontent.com/44923999/206225403-4d470706-0692-419a-820f-30b1b5333fa0.png" width="800" title="Snipped: December 7, 2022" />

### Exercise 4b: Pie Chart Layer
_https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-pie-chart-layer_

  <img src="https://user-images.githubusercontent.com/44923999/206219840-a9fc5f6b-de5d-4a3b-ae48-b506c9aa12d7.png" width="800" title="Snipped: December 7, 2022" />

* Click the "**Add data to your visual**" tab on the **Visualizations** pane
  * Drag the **EventType** field to the **Legend** bucket
* Click the "**Format your visual**" tab on the **Visualizations** pane
  * Set "**Bubble layer**" to **On** and "**Bar chart layer**" to **Off**
  * Explore additional visualization settings {e.g., Size, Shape, Colors, etc.}

#### Save and Publish

* Click **File** >> **Publish** >> "**Publish to Power BI**"
* On the resulting "**Publish to Power BI**" pop-up, select "**My workspace**" and then click **Select** (Save / Replace when prompted)

  <img src="https://user-images.githubusercontent.com/44923999/206226233-4970d95e-090b-4979-a3ff-27a26fe64e38.png" width="800" title="Snipped: December 7, 2022" />

### Exercise 4c: Reference Layer (GeoJSON)
_https://learn.microsoft.com/en-us/azure/azure-maps/power-bi-visual-add-reference-layer_

#### Prepare GeoJSON File

_https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function_

* Navigate to [Azure Data Explorer, Samples Database](https://dataexplorer.azure.com/clusters/help/databases/Samples)

  <img src="https://user-images.githubusercontent.com/44923999/206757950-bef53759-1990-4aa2-b112-c05f22b68efc.png" width="800" title="Snipped: December 9, 2022" />

* **Run** the following KQL:

  ```
  StormEvents
  | summarize DeathsDirect = sum(DeathsDirect) by h3 = geo_point_to_h3cell(EndLon,EndLat,2)
  | where DeathsDirect > 0
  | extend properties = pack_all()
  | project feature = bag_pack("type", "Feature", "geometry",geo_h3cell_to_polygon(h3), "properties", properties)
  | summarize features = make_list(feature)
  | project bag_pack("type", "FeatureCollection", "features", features)
  ```

Logic Explained:

  * `summarize` and `where` ... providing for filtration of groupings with zero-valued measures
  * `geo_point_to_h3cell` ... calculates the [H3](https://www.uber.com/blog/h3/) value for a given longitude, latitude, and resolution<br>_https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function_<br>_Note: S2 (http://s2geometry.io/) is another grouping method that might be considered_

  * `pack_all` ... creates a dynamic object with related data from all columns<br>_https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/packallfunction_
  * `bag_pack` ... creates a dynamic object from a list of keys and values<br>_https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/packfunction_
  * `geo_h3cell_to_polygon` ... calculates the polygon from a given H3 value<br>_https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-h3cell-to-polygon-function_


Sample Result (abridged):

    ```
    {
      "type": "FeatureCollection",
      "features": [
        {
          "type": "Feature",
          "geometry": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  "-80.698564844719911",
                  "26.062694311272747"
                ],
                [
                  "-79.211732382045582",
                  "26.761491776435669"
                ],
                [
                  "-79.057435773237245",
                  "28.3728922138514"
                ],
                [
                  "-80.425374695204042",
                  "29.321821309161468"
                ],
                [
                  "-81.9664941624314",
                  "28.63180650194559"
                ],
                [
                  "-82.08430112269555",
                  "26.984013476507922"
                ],
                [
                  "-80.698564844719911",
                  "26.062694311272747"
                ]
              ]
            ]
          },
          "properties": {
            "h3": "8244affffffffff",
            "DeathsDirect": 2
          }
        }
      ]
    }
    ```

  * Copy the result, paste into a text editor {e.g., Notepad, Visual Studio Code, etc.}, and save the file to your Desktop as "sample.json"

    <img src="https://user-images.githubusercontent.com/44923999/207388243-d3be3bc6-3beb-43d2-b294-f7d5a174b6ce.png" width="800" title="Snipped: December 13, 2022" />

  * To confirm success, drag-and-drop the new GeoJSON file on the map at: https://samples.azuremaps.com/geospatial-files/drag-and-drop-geojson-file-onto-map
  
    <img src="https://user-images.githubusercontent.com/44923999/207389620-6d869f4e-5bea-4547-98ad-20c42762c437.png" width="800" title="Snipped: December 13, 2022" />
