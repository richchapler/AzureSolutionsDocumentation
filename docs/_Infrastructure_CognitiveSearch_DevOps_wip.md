Goal is to create a Cognitive Search index using C#, then capture in DevOps

## Nuget
* Azure.Search.Documents

```
using Azure;
using Azure.Search.Documents.Indexes;
using Azure.Search.Documents.Indexes.Models;

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
                new SimpleField("metadata_storage_content_type", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_storage_size", SearchFieldDataType.Int64) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_storage_last_modified", SearchFieldDataType.DateTimeOffset) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_storage_content_md5", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_storage_name", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_storage_path", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false },
                new SimpleField("metadata_storage_file_extension", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_content_type", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_language", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_author", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_title", SearchFieldDataType.String) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SimpleField("metadata_creation_date", SearchFieldDataType.DateTimeOffset) { IsFilterable = false, IsSortable = false, IsFacetable = false, IsHidden = true },
                new SearchableField("content") { IsFilterable = false, IsSortable = false, IsFacetable = false, AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("merged_content") { AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("organizations", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("text", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("layoutText", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("imageTags", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene },
                new SearchableField("imageCaption", collection: true) { AnalyzerName = LexicalAnalyzerName.StandardLucene }
            }
        };

        indexClient.DeleteIndex(index);

        indexClient.CreateOrUpdateIndex(index);

        /* ************************* Skillset */

        var skills = new List<SearchIndexerSkill>
        {
            new ShaperSkill(
                inputs: new List<InputFieldMappingEntry>
                {
                    new InputFieldMappingEntry("metadata_storage_content_type") { Source = "/document/metadata_storage_content_type" },
                    new InputFieldMappingEntry("metadata_storage_size") { Source = "/document/metadata_storage_size" },
                    new InputFieldMappingEntry("metadata_storage_last_modified") { Source = "/document/metadata_storage_last_modified" }
                },
                outputs: new List<OutputFieldMappingEntry>
                {
                    new OutputFieldMappingEntry("output") { TargetName = "objectprojection" }

                }
            )
            {
                Context = "/document"
            }
        };

        var skillset = new SearchIndexerSkillset(skillsetName, skills);

        indexerClient.DeleteSkillset(skillset);

        indexerClient.CreateSkillset(skillset);

        /* ************************* Indexer */

        FieldMapping fieldMapping = new FieldMapping("/document/merged_content/organizations")
        {
            TargetFieldName = "organizations"
        };

        SearchIndexer indexer = new SearchIndexer(indexerName, dataSourceName, indexName)
        {
            FieldMappings = { fieldMapping },
            SkillsetName = skillsetName
        };

        indexerClient.DeleteIndexer(indexer);

        indexerClient.CreateIndexer(indexer);
    }
}
```
