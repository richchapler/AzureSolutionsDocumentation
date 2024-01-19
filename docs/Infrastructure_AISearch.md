# AI Search
_(aka "Cognitive Search")_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/39276a5d-db29-475c-a075-bd61949ba80b" width="1000" />

[Microsoft Learn: Azure AI Search documentation](https://learn.microsoft.com/en-us/azure/search/) should serve as your primary source of information.

## Frequently-Asked Questions

* How do I selectively delete documents from an AI Search Index?
  * Reference: [Add, Update or Delete Documents (Azure AI Search REST API)](https://learn.microsoft.com/en-us/rest/api/searchservice/addupdate-or-delete-documents)
  * It is typically unnecessary to re-run the indexer after removing a document

* Are document updates detected by the periodically running schedule?
  * Reference: [Schedule an indexer in Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-howto-schedule-indexers?tabs=portal)
  * Yes, document updates are picked up by the periodically running indexer schedule but **changes will only be reflected in the search index after the indexer run is complete**
  * "Prerequisite: Change detection in the data source... Azure Storage and SharePoint have built-in change detection. Other data sources, such as Azure SQL and Azure Cosmos DB must be enabled manually."

* What is the best combination of Semantic, Vector, and Scoring Profile configuration?
  * Answer, of course is "it depends", to help decide, however:
    * **Semantic Search**: Great for **understanding the intent behind a user's query**, and can be useful for **finding relevant results even when the exact keywords are not present** in the document. It also helps with synonym handling.
    * **Vector Search**: Particularly effective when dealing with **large amounts of high-dimensional data**. It's great for use-cases like image recognition, recommendation systems, and natural language processing where semantic understanding and context are important.
    * **Scoring Profiles**: These are **useful for fine-tuning the ranking of the search results based on specific business needs**. They can take into consideration various factors like freshness of the data, popularity, or any other field in the data.
