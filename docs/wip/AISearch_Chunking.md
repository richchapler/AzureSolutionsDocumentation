# AI Search, Chunking

Azure AI Search's SplitSkill is a cognitive skill used to break down large pieces of text into smaller chunks based on character count or other criteria. This enables efficient processing for downstream tasks such as embedding and vectorization, where smaller text segments improve search accuracy and relevancy by ensuring that the text is of manageable size for analysis and indexing. SplitSkill helps in creating structured chunks for enhanced search and retrieval in AI-powered search systems.

Here are the chunking rules and control options for Azure AI Search's SplitSkill:

* Maximum chunk size: You can specify the number of characters in each chunk. The default is usually 5,120 characters, but this can be customized
* Overlap: You can configure an overlap between chunks to preserve context between segments. For example, if set to 50 characters, the last 50 characters of one chunk will overlap with the first 50 characters of the next
* Split by sentence: SplitSkill can chunk data at sentence boundaries (using punctuation like periods, question marks, etc.) to avoid splitting within sentences. This helps maintain readability and improves the accuracy of downstream tasks like vectorization.
* Fallback chunking: If logical boundaries are not found (e.g., no sentences or paragraphs), chunking falls back to pure character count
* Text splitting control: You can control how the text is chunked by specifying either a hard limit on character count or a soft limit that favors natural breaks (like sentences)
* Minimum chunk size: You can define a minimum chunk size to avoid creating excessively small fragments

```json
{
  "skills": [
    {
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "name": "splitSkill",
      "description": "Split text into smaller chunks by sentence boundary.",
      "context": "/document/content",
      "outputs": [
        {
          "name": "textItems",
          "targetName": "chunks"
        }
      ],
      "defaultLanguageCode": "en",
      "textSplitMode": "sentences",
      "textSplitOptions": {
        "sentenceBoundary": true,
        "separatorPattern": "\\.",
        "fallbackToCharacterSplit": true,
        "minimumPageLength": 200
      }
    }
  ]
}
```

## Reference

* [SplitSkill Class](https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.search.models.splitskill?view=azure-dotnet-legacy)
* [SplitSkill.TextSplitMode Property](https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.search.models.splitskill.textsplitmode?view=azure-dotnet-legacy#microsoft-azure-search-models-splitskill-textsplitmode)
