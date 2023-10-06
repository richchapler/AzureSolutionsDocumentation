Goal is to create a Cognitive Search index using C#, then capture in DevOps

## Nuget
* Azure.Search.Documents

```
using Azure;
using Azure.Search.Documents;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;
using System;

public class Program
{
    public static void Main(string[] args)
    {
        string serviceName = "rchaplerss";
        string adminApiKey = "bUChVfJmk9sWfy68DTY2Y1hAZCCflQ6vALyHCekAVkAzSeAvrFVm";
        string dataSourceName = "rchaplerss-datasource";
        string indexName = "rchaplerss-index";
        string indexerName = "rchaplerss-indexer";
        string skillsetName = "rchaplerss-skillset";

        var serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
        var credential = new AzureKeyCredential(adminApiKey);

        var indexClient = new SearchIndexClient(serviceEndpoint, credential);
        var indexerClient = new SearchIndexerClient(serviceEndpoint, credential);

        /* ************************* Data Source */

        var sidsc = new SearchIndexerDataSourceConnection(
            dataSourceName,
            SearchIndexerDataSourceType.AzureBlob,
            "DefaultEndpointsProtocol=https;AccountName=rchaplers;AccountKey=Pma3aYFLn1KYN3vlhljUwzdxNgfapnk8c4JVcmkGhFCynieXJE4opphYc6W1L4hC1dU13szYyyff+AStT2ZPhw==;EndpointSuffix=core.windows.net",
            new SearchIndexerDataContainer("forms")
            );

        indexerClient.DeleteIndexer(indexerName); /* must be deleted before data source connection */
        indexerClient.DeleteDataSourceConnection(dataSourceName);

        indexerClient.CreateDataSourceConnection(sidsc);

        /* ************************* Index */

        SearchIndex index = new SearchIndex(indexName)
        {
            Fields =
            {
                new SimpleField("id", SearchFieldDataType.String) { IsKey = true },
                new SearchableField("content") { AnalyzerName = LexicalAnalyzerName.EnMicrosoft }
            }
        };

        indexClient.CreateOrUpdateIndex(index);

        /* ************************* Skillset */
        //Skillset skillset = new Skillset(skillsetName, new List<Skill>());
        //indexerClient.CreateOrUpdateSkillset(skillset);

        /* ************************* Indexer */
        //SearchIndexer indexer = new SearchIndexer(indexerName, dataSourceName, indexName)
        //{
        //    SkillsetName = skillsetName
        //};
        //indexerClient.CreateOrUpdateIndexer(indexer);
    }
}


```
