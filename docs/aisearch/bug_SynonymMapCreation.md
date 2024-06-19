# Bug Report
## Issue with Programmatic Creation of Synonym Maps in Azure AI Search  
   
#### **Summary**  
When attempting to programmatically create a synonym map in Azure AI Search with a vectorizer in place, an error is encountered. The error indicates that the 'modelName' parameter is required in API version '2024-05-01-preview'.  
   
#### **Steps to Reproduce**  
1. Use the Azure SDK to create a synonym map in Azure AI Search.  
2. Ensure a vectorizer is configured in the search service.  
3. Attempt to create the synonym map with the specified vectorizer.  
   
#### **Expected Behavior**  
The synonym map should be created successfully without any errors.  
   
#### **Actual Behavior**  
The following error is encountered:  
```  
2024.06.19-09:19:20 - Synonym Map 'synonyms'... creating with SDK  Exception: Azure.RequestFailedException: The request is invalid. Details: definition : Error in Vectorizer 'myVectorizer' : 'modelName' parameter is required in API version '2024-05-01-preview'.  Status: 400 (Bad Request)  
Content:  {"error":{"code":"","message":"The request is invalid. Details: definition : Error in Vectorizer 'myVectorizer' : 'modelName' parameter is required in API version '2024-05-01-preview'."}}  
```  
   
#### **Environment**  
- **Azure AI Search Service Version**: 2024-05-01-preview  
- **SDK Version**: [Specify the SDK version you are using]  
- **Programming Language**: [Specify the language, e.g., C#, Python]  
- **Operating System**: [Specify the OS, e.g., Windows 10, macOS]  
   
#### **Additional Information**  
- The issue seems to be related to the 'modelName' parameter missing in the vectorizer configuration.  
- The error message suggests that the 'modelName' parameter is required for the specified API version.  
   
#### **Workaround**  
There is no known workaround at this time. The issue needs to be addressed to allow successful creation of synonym maps with vectorizers.  
   
#### **Attachments**  
- [Include any relevant code snippets, logs, or screenshots that can help in diagnosing the issue]  
   
#### **Priority**  
- [Specify the priority of the issue, e.g., High, Medium, Low]  
   
---  
   
Please review the above bug report and make any necessary adjustments before submitting it to the Azure AI Search support team.
