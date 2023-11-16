# AI: Improve Prompt Response Quality

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fe54b808-31eb-4932-a561-892f53854750" width="1000" />

## Use Case
* "We believe that our source data includes ambiguities that, resolved, would result in better OpenAI response to prompts"

## Proposed Solution
* Stage Resources: Create AI Search Index and Open AI Deployment
* Test Prompt / Response: Evaluate prompt quality
* Implement Enhancements: Lorem

## Solution Requirements
* [AI Search](https://azure.microsoft.com/en-us/products/search)
* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)
* [SQL Server](https://learn.microsoft.com/en-us/azure/azure-sql) and [Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview?view=azuresql) with [AdventureWorks sample data](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)
* [Visual Studio](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload installed

-----
-----

## Exercise 1: Stage Resources
In this exercise, we will import AdventureWorks sample data into AI Search and then use the index in the OpenAI Chat Playground.
<br>_Note: the instructions below are for creating a minimum viable index {i.e., no bells-and-whistles}_

### Step 1: Create AI Search Index
Navigate to AI Search > "Overview", click "Import data" and on the resulting page, select "Azure SQL Database" from the "Data Source" dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b67ee133-abc4-441d-bb38-3eb2ba2ffca9" width="800" title="Snipped: November 16, 2023" />

Click the "Choose an existing connection" link and select the SQL Database that will be the source.
<br>Complete the "Import data" form and then click "Test connection".
<br>Once the connection is validated, select the "[SalesLT].[Product]" table from the added "Table/View" dropdown.
<br>Click "Next: Add cognitive skills (Optional)".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d467d9c9-4a10-4479-b383-89a561be3b22" width="800" title="Snipped: November 16, 2023" />

Click "Skip to: Customize target index".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0377231d-3f77-451b-b7c5-a6bb1586d046" width="800" title="Snipped: November 16, 2023" />

Check the top "Retrievable" and "Searchable" boxes (which will check all boxes for all fields).
<br>Click "Next: Create an indexer".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4e908d25-3801-42c6-b0bb-07b3290eef04" width="800" title="Snipped: November 16, 2023" />

Click "Submit".

#### Confirm Success

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/248f073f-b490-47b0-b561-a7aa25e2173d" width="800" title="Snipped: November 16, 2023" />

Navigate to the new index and confirm success.

### Step 2: Create OpenAI Deployment
Navigate to OpenAI Studio > "Chat playground".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc174b2b-9db3-4757-b426-69b9bc65ff37" width="800" title="Snipped: November 16, 2023" />

Navigate to "Add your data..." and then click "Add a data source" on the "Assistant setup" pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/65fe3909-2010-49a1-ba6f-2772bd25b5bd" width="800" title="Snipped: November 16, 2023" />

Complete the "Add data" forms, then click "Save and close".

#### Confirm Success
Enter a prompt in the "Chat session" pane.
<br>_Note: In Exercise 2, we will focus on ambiguous Product Size data, so I started with: `Describe a large product`_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eeb2fcff-b5f5-4728-bd93-ebbc752b77b6" width="800" title="Snipped: November 16, 2023" />

Confirm response; for example:

```
The retrieved document provides information about a large product, specifically a "Touring-Panniers" with the model number "PA-T100". It is grey in color. The product has a price of 125.00 units, though it's not clear what currency this is in. There's also a number 51.5625 associated with the product, but without further context, it's unclear what this number represents.
```

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 2: Test Prompt / Response
In this exercise, we will evaluate prompt response (expected vs. actual).

### Step 1: Source Data, SQL
Navigate to the SQL Database >> Query Editor and login.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6ba15e7d-84fd-4ec9-b1c3-b9631518d60e" width="800" title="Snipped: November 16, 2023" />

Run the following T-SQL query:
```
SELECT TOP 10 ProductID, Name, Color, Size
FROM [SalesLT].[Product]
```

You will note that the results do not include anything specifically called out as "Large", which aligns with the limited response we saw from OpenAI earlier.
<br>There is, however, a column named Size and rows that contain the value "L", which we can assume stands for "Large"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/afa55b23-efe1-4656-8fec-806c62cae098" width="800" title="Snipped: November 16, 2023" />

Run the following T-SQL query:
```
SELECT TOP 10 ProductID, Name, Color, Size
FROM [SalesLT].[Product]
WHERE [Name] like '%Large%'
```

If we specifically query for rows with "Large" in the Name column, we see the single value that surfaced earlier in OpenAI.

### Step 2: Source Data, AI Search
Navigate to the AI Search index.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/200d99cd-1afd-4ed9-bdf1-6a949fb352d2" width="800" title="Snipped: November 16, 2023" />

On the "Search explorer" tab, enter Query Phrase `Large` and press "Search".

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

### OpenAI Prompt: Starter Question
Example: "what large products are there?"
```
The retrieved document mentions a product called "Touring-Panniers, Large" with the product code "PA-T100". It is available in grey color.
```
_Note: This answer demonstrates that OpenAI is using the "Color" column (name and data) in spite of not being prompted for that information._
<br>_This answer suggests that OpenAI is not using the "Size" column, confused by use of "L" as an abbreviation for Large, or both_

<br><br>
#### Follow-up: Can I ask a question that ambiguously reference columns that should be available?
Example: "how much does it cost?"
```
The "Touring-Panniers, Large" product costs 125.0000, although the currency is not specified in the retrieved document.
```
_Note: This answer implies use and some understanding of column names... it choose a column that corresponds to the request, but is not clear why it chose List Price instead of Standard Cost (specifically when the question was about cost)"_

<br><br>
#### Follow-up: Does more specificity help it reference columns?
Example: "is that standard cost or list price?"
```
The retrieved document does not specify whether the price of 125.0000 for the "Touring-Panniers, Large" product is the standard cost or the list price.
```
_Note: This answer suggests that OpenAI cannot use column names, OR that it is very literal in its interpretation of column names {e.g., spacing in column references confuses response}_

<br><br>
#### Follow-up: What if I am VERY specific {i.e., using the column names directly from the database schema}?
Example: "is that StandardCost or ListPrice?"
```
The price of 125.00 units for the "Touring-Panniers, Large" product is the ListPrice, while the StandardCost is 51.5625 units.
```
_Note: This answer seems to prove that OpenAI does use column names, specifically as spelled_

<br><br>
#### Follow-up: Does capitalization matter?
Example: "is that standardcost or listprice?"
```
The price of 125.00 units for the "Touring-Panniers, Large" product is the ListPrice, while the StandardCost is 51.5625 units.
```
_Note: Capitalization does not appear to matter_

<br><br>
#### Follow-up: Does Semantic Configuration help?
I ran through all the previous prompts after re-configuring for use of Semantic Search; example:

Example: "what large products are there?"
```
The requested information is not found in the retrieved data. Please try another query or topic.
```
_Note: Semantic does not return the same response as Keyword_

<br><br>
#### Follow-up: Use Case Driven Request

Example: "write a formal product description for a touring-panniers"
```
The Touring-Panniers is a large, high-quality product designed for the discerning traveler. It carries the product code PA-T100 and comes in a sleek grey color. The cost price of the Touring-Panniers is 51.5625, and it retails at a price of 125.0000, offering excellent value for its robust design and functionality.
```

## EXERCISE 3: ADD SYNONYMMAP TO EKE MORE INFORMATION FOR "LARGE" FROM SIZE="L"

Lorem Ipsum


