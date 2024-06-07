# AI: Code Analysis

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/08ff9047-68f4-4829-b35e-7606d0305630" width="1000" />

## Use Case
* "We want to regularly snapshot GitHub Respository content"
* "Each snapshot should include documentation of specific code files {e.g., C#}, including enhancement suggestions"
* "We want to create a high-level README summary of the documented code"
* "Documentation should be template-based, so we can evolve over time, at scale"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [AI Search](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration

* GitHub [Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) and [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) with `repo` permissions

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * GitHub-PersonalAccessToken
  * OpenAI-Key
  * Storage-ConnectionString

* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)

* [Storage Account](Infrastructure_StorageAccount.md) with a container named "code"

## Documentation Note

Codeblock explanation is limited... I recommend that you copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Step 1: Prepare Project

Open Visual Studio and create a Console App project.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e2deb0e6-0b23-4f78-a4dc-64b7cf7d07fb" width="800" title="Snipped June 7, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/064d2f4b-6f1d-45bf-975e-e135849703de" width="800" title="Snipped June 7, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "AzureSolution_Helpers".
On the resulting pop-out, check the box next to your project and then click "Install".
Repeat this process for Octokit.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6fd5ad84-3cd3-4ef9-a1ae-f2bf06307309" width="800" title="Snipped June 7, 2024" />

-----

### Step 2: Add Templates

Right-click on the project and select "Add" >> "New Folder" from the resulting dropdown. Name the new folder "Templates".

#### codereview.md

Right-click on the "Templates" and select "Add" >> "New Item" from the resulting dropdown. Name the new file "codereview.md" and paste the following markdown:

```
Produce meaningful documentation including all of the following items in the following form:

-----
# {Name of File}

## Purpose  
Describe the overall purpose of the code. What is it designed to accomplish?

## Functionality  
Describe the specific functionality of the code. What does it do and how does it do it?

## Dependencies  
List any dependencies the code has. Are there any external libraries or packages it relies on?

## Inputs/Outputs  
Describe the inputs the code takes and the outputs it produces. What data does it work with?
   
## Enhancement Opportunities  
List a "top three" of things that would make the code better {e.g., faster, more modern, less expensive} 
```

#### nuget_readme.md

Right-click on the "Templates" and select "Add" >> "New Item" from the resulting dropdown. Name the new file "nuget_readme.md" and paste the following markdown:

```
Description

## Dependencies 
List Nuget Package / Version dependencies  

## Classes  
A full and sorted list of Key Vault Secrets referenced in the code.

## Configuration  
A full and sorted list of Key Vault Secrets referenced in the code.
```

-----

### Step 3: Modify Logic

#### Program.cs

Replace the default logic with:
```
using Azure.Storage.Blobs.Models;
using Octokit;
using System.Text;

namespace AI_Code
{
    class Program
    {
        static readonly HttpClient hc = new();
        static GitHubClient? GitHub_Client;
        static string? Path_Destination;

        static async Task Main()
        {
            /* ************************* User Input: KeyVault */

            var keyVaultName = AzureSolutions.Helpers.KeyVault.Name.Get("KeyVault_Name");
            if (keyVaultName == null) throw new ArgumentNullException(nameof(keyVaultName));

            var kvc = AzureSolutions.Helpers.KeyVault.Configuration.PrepareConfiguration(keyVaultName);

            if (kvc?.KeyVault_Client == null) throw new Exception("KeyVault Client is not initialized.");

            /* ************************* User Input: GitHub Repositories  */

            var ghc = AzureSolutions.Helpers.GitHub.Configuration.PrepareConfiguration(kvc.KeyVault_Client);

            if (ghc?.GitHub_Client == null) throw new Exception("GitHub client is not initialized.");

            GitHub_Client = ghc.GitHub_Client;

            var repos = await GitHub_Client.Repository.GetAllForCurrent();

            repos.Select((repo, index) => $"{index + 1}. {repo.Name}").ToList().ForEach(Console.WriteLine);

            Console.WriteLine("\nSelect Repository:");

            if (!int.TryParse(Console.ReadLine(), out int repoIndex) || repoIndex < 1 || repoIndex > repos.Count) { throw new ArgumentException("Invalid repository selection."); }
            var selectedRepo = repos[repoIndex - 1];

            /* ************************* Storage Account (destination) */

            var sc = AzureSolutions.Helpers.Storage.Configuration.PrepareConfiguration(kvc.KeyVault_Client);

            if (sc?.Storage_Client == null) throw new Exception("Storage Client is not initialized.");

            var container = sc.Storage_Client.GetBlobContainerClient("code") ?? throw new Exception("Container is null");

            Path_Destination = $"{selectedRepo.Name}/{DateTime.Now:yyyyMMdd-HHmm}";

            var directories = new Stack<string>(); directories.Push(""); /* ...Push("") represents root directory */

            Console.WriteLine($"\n************************* Copying files from repository '{selectedRepo.Name}' to storage account '{container?.AccountName}'...");

            while (directories.Count > 0)
            {
                var dir = directories.Pop();

                IEnumerable<RepositoryContent>? contents = null;

                contents = GitHub_Client is { }
                    ? string.IsNullOrEmpty(dir)
                        ? await GitHub_Client.Repository.Content.GetAllContents(selectedRepo.Owner.Login, selectedRepo.Name)
                        : await GitHub_Client.Repository.Content.GetAllContents(selectedRepo.Owner.Login, selectedRepo.Name, dir)
                    : throw new Exception("GitHub client is not initialized.");

                (contents ?? throw new Exception("Contents is null")).ToList().ForEach(async content =>
                {
                    if (content.Type == ContentType.Dir)
                    {
                        directories.Push(content.Path);
                    }
                    else if (content.Type == ContentType.File)
                    {
                        Console.WriteLine(content.Url);

                        using var stream = await hc.GetStreamAsync(content.DownloadUrl);

                        if (container == null) throw new Exception("Container is null");

                        await container.GetBlobClient($"{Path_Destination}/{content.Path}").UploadAsync(stream, true);
                    }
                });
            }

            Console.WriteLine("\n************************* Documenting C# Code...");

            var oaic = AzureSolutions.Helpers.OpenAI.Configuration.PrepareConfiguration(kvc.KeyVault_Client);

            var template_codereview = File.ReadAllText(Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "..\\..\\..\\Templates\\codereview.md")));

            var CodeReviews = new List<string>();

            if (container == null) throw new Exception("Container is null");

            var blobs = container.GetBlobs(BlobTraits.None, BlobStates.None, $"{Path_Destination}") ?? throw new Exception("Blobs is null");
            {
                var blobNames = blobs.Select(b => b.Name).ToList();

                foreach (var b in blobNames)
                    {
                        Console.WriteLine(b);
                        
                        if (b.Contains(".cs") && !b.Contains(".git"))
                        {
                            var r_bdi = await container.GetBlobClient(b).DownloadAsync();

                            using var streamReader = new StreamReader(r_bdi?.Value.Content ?? throw new Exception("Content is null"));

                            if (oaic == null) throw new Exception("OpenAI configuration is not initialized");

                            var OpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt.NoIndex(oaic,
                                UserQuery: await streamReader.ReadToEndAsync(),
                                SystemMessage: $"Use {b} as markdown file header.\n\n{template_codereview}",
                                Temperature: 0.25f);

                            using var Stream_Response = new MemoryStream(Encoding.UTF8.GetBytes(OpenAI_Prompt?.Response ?? string.Empty));

                            await container.GetBlobClient(b.Replace(".cs", ".md")).UploadAsync(Stream_Response, true);

                            CodeReviews.Add(OpenAI_Prompt?.Response ?? throw new Exception("Response is null"));
                        }
                }
            }

            Console.WriteLine("\n************************* Producing README.md...");

            var template_readme = File.ReadAllText(Path.GetFullPath(Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "..\\..\\..\\Templates\\nuget_readme.md")));

            var combinedOpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt.NoIndex(oaic ?? throw new Exception("OpenAI configuration is not initialized"),
                UserQuery: $"Produce README content that will be included with a NuGet package publication based on the code review content below:\n\n{string.Join("\n", CodeReviews)}",
                SystemMessage: $"Use the README template below:\n\n{template_readme}",
                Temperature: 0.0f);

            using var Stream_README = new MemoryStream(Encoding.UTF8.GetBytes(combinedOpenAI_Prompt?.Response ?? string.Empty));

            container.GetBlobClient($"{Path_Destination}/readme.md").Upload(Stream_README, overwrite: true);
        }
    }
}
```

-----

### Step 4: Confirm Success

Click "Debug" >>  "Start Debugging".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac4453bf-ca14-42ae-8ed2-b0a3341d6d21" width="800" title="Snipped June 7, 2024" />

Messages in the console window will iterate through files in your GitHub repository... and these will correspond with the files being copied to your Azure Storage Account.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/76602c7d-1939-482a-bf1a-cd758f0d20f0" width="800" title="Snipped June 7, 2024" />

Navigate to the "code" container, browse and confirm success.
Identify and review a markdown file... for example, `index.md` describes helper class `index.cs`.

```markdown
# AzureSolutions/Helpers/AISearch/Index.cs

## Purpose  
This code is designed to interact with Azure's Search Index Client. It provides functionality to create and delete indexes, and list columns of an index. It is part of a larger project that uses Azure's AI Search capabilities.

## Functionality  
The code provides three main functions:

- `Client`: This function initializes and returns an instance of Azure's SearchIndexClient using the provided Azure Search name and key.
- `Create`: This function creates a new search index with the provided parameters. It supports creation via both Azure's API and SDK.
- `Delete`: This function deletes an existing search index.
- `ListColumns`: This function lists the columns of a given search index. It can filter the columns based on whether they are filterable or not.

## Dependencies  
This code depends on the Azure.Search.Documents.Indexes namespace from the Azure SDK for .NET. It specifically uses the SearchIndexClient, SearchIndex, SimpleField, SearchField, SearchableField, SearchSuggester, SemanticSearch, SemanticConfiguration, SemanticPrioritizedFields, SemanticField, and LexicalAnalyzerName classes.

## Inputs/Outputs  
The code primarily works with the following inputs:

- Azure Search name and key: These are used to authenticate and interact with Azure's Search service.
- Index name: The name of the search index to create, delete, or list columns for.
- Other parameters for index creation: These include the suggester name, semantic configuration name, and optionally a JSON definition.

The outputs of the code are:

- `Client`: Returns an instance of Azure's SearchIndexClient.
- `Create`: No return value, but creates a new search index in Azure.
- `Delete`: No return value, but deletes an existing search index in Azure.
- `ListColumns`: Returns a list of column names from the specified search index.

## Enhancement Opportunities  
1. Error handling: The code could be enhanced to handle more specific error scenarios and provide more detailed error messages.
2. Input validation: The code could include more robust input validation to ensure the provided parameters are valid before attempting to interact with Azure's Search service.
3. Asynchronous operations: The code could be enhanced to support asynchronous operations, allowing for better performance in scenarios where multiple indexes need to be created, deleted, or queried at once.
```

-----

**Congratulations... you have successfully completed this exercise**
