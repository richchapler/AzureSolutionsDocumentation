# AI Search, Chunking

Azure AI Search's SplitSkill is a cognitive skill used to break down large pieces of text into smaller chunks based on character count or other criteria. This enables efficient processing for downstream tasks such as embedding and vectorization, where smaller text segments improve search accuracy and relevancy by ensuring that the text is of manageable size for analysis and indexing. SplitSkill helps in creating structured chunks for enhanced search and retrieval in AI-powered search systems.

Here are the chunking rules and control options for Azure AI Search's SplitSkill:

* Maximum chunk size: You can specify the number of characters in each chunk. The default is usually 5,120 characters, but this can be customized
* Overlap: You can configure an overlap between chunks to preserve context between segments. For example, if set to 50 characters, the last 50 characters of one chunk will overlap with the first 50 characters of the next
* Split by sentence or paragraph: SplitSkill can chunk data at logical boundaries like sentences or paragraphs, preventing mid-sentence breaks. This can be configured depending on how you want the chunks to be created.
* Fallback chunking: If logical boundaries are not found (e.g., no sentences or paragraphs), chunking falls back to pure character count
* Text splitting control: You can control how the text is chunked by specifying either a hard limit on character count or a soft limit that favors natural breaks (like sentences)
* Minimum chunk size: You can define a minimum chunk size to avoid creating excessively small fragments

```json
{
  "skills": [
    {
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "name": "splitSkill",
      "description": "Split large text into smaller chunks by character count with overlap.",
      "context": "/document/content",
      "outputs": [
        {
          "name": "textItems",
          "targetName": "chunks"
        }
      ],
      "defaultLanguageCode": "en",
      "textSplitMode": "pages",
      "maximumPageLength": 5120,
      "overlap": 50,
      "textSplitOptions": {
        "sentenceBoundary": true,
        "paragraphBoundary": true,
        "fallbackToCharacterSplit": true,
        "minimumPageLength": 200
      }
    }
  ]
}
```
