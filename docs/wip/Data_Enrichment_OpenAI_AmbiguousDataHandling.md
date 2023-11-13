Scenario #1: SQL Server, AdventureWorks >> Cognitive Search >> OpenAI GPT-4, Keyword
* AI Search, Query String: "Large"... results: 1
    <br><br>
    ```
    {
      "@odata.context": "https://rchaplerss.search.windows.net/indexes('azuresql-index')/$metadata#docs(*)",
      "value": [
        {
          "@search.score": 2.70413,
          "ProductID": "842",
          "Name": "Touring-Panniers, Large",
          "ProductNumber": "PA-T100",
          "Color": "Grey",
          "StandardCost": "51.5625",
          "ListPrice": "125.0000",
          "Size": null,
          "Weight": null,
          "ProductCategoryID": 39,
          "ProductModelID": 120,
          "SellStartDate": "2006-07-01T00:00:00Z",
          "SellEndDate": "2007-06-30T00:00:00Z",
          "DiscontinuedDate": null,
          "ThumbnailPhotoFileName": "no_image_available_small.gif",
          "rowguid": "56334fff-91d4-495e-bf98-933bc1010f23",
          "ModifiedDate": "2008-03-11T10:01:36.827Z",
          "keyphrases": [
            "842"
          ]
        }
      ]
    }
    ```
* OpenAI Prompt: "What large products are there?"... results: 1
    <br><br>
    ```
    The retrieved document mentions a product called "Touring-Panniers, Large" with the product code "PA-T100". It is available in grey color.
    ```
