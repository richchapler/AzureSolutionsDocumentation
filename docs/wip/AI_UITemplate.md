# ASP.NET Core Web App (Razor Pages)

Here's a simple guide to help you create a web app using Visual Studio, C#, Azure, and AI Search:  
   
1. **Setting up Visual Studio & C#:**  
   - You need to have the latest version of Visual Studio installed. You can download it from the official Microsoft website.  
   - Make sure to install the ASP.NET and web development workload during the Visual Studio installation.  
   - Create a new ASP.NET Core Web App (Model-View-Controller) project. ASP.NET Core is the modern, cross-platform framework for building apps with .NET and C#.  
   
2. **Creating the web app:**  
   - Build your basic web application following MVC (Model-View-Controller) architecture. Your query to AI Search will be made from the Controller, and the response will be sent back to the View for display.  
   
3. **Integrating AI Search:**  
   - To integrate AI Search, you'll need to use the Azure Cognitive Search client library for .NET.  
   - Add the Azure.Search.Documents package to your project.  
   - Create an instance of `SearchClient` for connecting to the search index.  
   - Use `SearchClient.Search` method to send a search query and get the response.  
   
4. **Publishing to Azure:**  
   - Create an Azure App Service from the Azure portal. This will host your web app.  
   - From Visual Studio, right-click on your project and select Publish.  
   - Select 'Azure' as the target and 'Azure App Service' as the specific target.  
   - Follow the prompts to select your subscription, the existing App Service you created, and publish.  
   
Remember to store any sensitive data like your Azure Cognitive Search service name, index name, and query API key securely, such as in Azure Key Vault or at least in your appsettings.json configuration file.  
   
This is a very simplified guide. Each of these steps can have more detail depending on your exact requirements, so feel free to ask for clarification on any point.














-----

# PowerApps (delete?)
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
