## Exercise 1: Create Custom Connector

### 1. General
Scheme: HTTPS
Host: rchaplerss.search.windows.net
Base URL: /

### 2. Security
Authentication Type: API Key
Parameter Label: api-key
Parameter Name: api-key
Parameter Location: Header

### 3. Definition
#### General
Click "+ New action"
Summary: Query
Operation ID: Query

#### Request
Click "+ Import from sample"
* URL: https://rchaplerss.search.windows.net/indexes/rchaplerss-index/docs?search=*&api-version=2020-06-30
* Headers: Content-Type application/json
* Verb: GET

##### Query Parameters
* search
  * Default value: *
  * Is required? No
  * Visibility: none
* api-version
  * Default value: 2020-06-30
  * Is required: Yes
  * Visibility: internal
* Content-Type
  * Default value: application/json
  * Is required: Yes
  * Visibility: none

Turn on "Swagger editor".
* Authorize with api-key, then Close
* Click "GET"
* Click "Try it out"
* Click "Execute"

Turn off "Swagger editor".

Click "+ Add default response".
* Headers: Content-Type application/json
* Body
  ```
  {
    "@odata.context": "string",
    "@search.answers": [
      "string"
    ],
    "value": [
      {
        "@search.score": 0,
        "@search.rerankerScore": 0,
        "@search.captions": [
          {
            "text": "string",
            "highlights": "string"
          }
        ],
        "id": "string",
        "metadata_author": "string",
        "metadata_content_type": "string",
        "metadata_creation_date": "string",
        "metadata_language": "string",
        "metadata_storage_content_type": "string",
        "metadata_storage_file_extension": "string",
        "metadata_storage_last_modified": "string",
        "metadata_storage_name": "string",
        "metadata_storage_path": "string",
        "metadata_storage_size": 0,
        "metadata_title": "string",
        "content": "string",
        "keyphrases_content": [
          {}
        ],
        "text": [
          {}
        ],
        "keyphrases_text": [
          {}
        ],
        "chunk_id": "string",
        "parent_id": "string",
        "chunk": "string",
        "vector": [
          "string"
        ]
      }
    ]
  }
  ```

Click "Import".

Click "default default" box to confirm import.

Click "Create connector".

## Reference

* [Tutorial: Query an Azure AI Search index from Power Apps](https://learn.microsoft.com/en-us/azure/search/search-howto-powerapps)
