## Scenario #1: SQL Server, AdventureWorks >> Cognitive Search >> OpenAI GPT-4, Keyword

### AI Search, Query String: "Large"
Results: 1
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

### OpenAI Prompt: "what large products are there?"
Results: 1
```
The retrieved document mentions a product called "Touring-Panniers, Large" with the product code "PA-T100". It is available in grey color.
```
_Note: This answer demonstrates that OpenAI is using the "Color" column (name and data) in spite of not being prompted for that information._
_This answer suggests that OpenAI is not using the Size column, confused by use of "L" as an abbreviation for Large, or both_

#### Follow-up Question: "how much does it cost?"
```
The "Touring-Panniers, Large" product costs 125.0000, although the currency is not specified in the retrieved document.
```
_Note: This answer implies use and some understanding of column names... it is not clear why it chose List Price instead of Standard Cost (specifically when the quest was about cost)"_
  
#### Follow-up Question: "is that standard cost or list price?"
```
The retrieved document does not specify whether the price of 125.0000 for the "Touring-Panniers, Large" product is the standard cost or the list price.
```
_Note: This answer suggests that OpenAI cannot use column names, or that it is very literal in its interpretation of column names_
  
#### Follow-up Question: "is that StandardCost or ListPrice?"
```
The price of 125.0000 for the "Touring-Panniers, Large" product is the ListPrice, while the StandardCost is 51.5625, as per the retrieved document.
```
_Note: This answer seems to prove that OpenAI does use column names, specifically as spelled_








