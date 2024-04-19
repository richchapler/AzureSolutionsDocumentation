# AI and at-scale Code Analysis

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/08ff9047-68f4-4829-b35e-7606d0305630" width="1000" />

## Use Case
* "We want to use AI to better understand legacy code"
* "We want to use AI to suggest code optimizations, **at scale**"

## Proposed Solution
This solution will address requirements in four exercises:

* Exercise 1: Chat with Legacy Code
* Exercise 2: Iterative Code Suggestions
* 
## Solution Requirements

This documentation assumes the following resources are ready for use:

* [AI Search](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration

* GitHub [Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) and [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * BlobServiceClient-ConnectionString
  * OpenAI-Key
  * GitHub-PersonalAccessToken

* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)

* [Storage Account](Infrastructure_StorageAccount.md) with a container named "code"

## Documentation Note

Codeblock explanation is limited... I recommend that you copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

## Exercise 1: Chat with Legacy Code

### Step 1: Create Project

Open Visual Studio and create a Console App project.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0659d99b-82f5-4d29-8326-e4a78af08818" width="800" title="Snipped April 16, 2024" />

### Step 2: Install Nuget

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eb2cb824-3951-488f-813e-7bf9d655e41b" width="800" title="Snipped April 16, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Octokit".
On the resulting pop-out, check the box next to your project and then click "Install".
Repeat this process for:
* Azure.Security.KeyVault.Secrets
* Azure.Storage.Blobs
* AzureSolutions.Helpers

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/26207ff5-ae91-4df6-a3d3-fcf12b73d7d8" width="800" title="Snipped April 16, 2024" />

-----

### Step 3: Modify Logic

#### Program.cs

Replace the default logic with:
```
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Storage.Blobs;
using Octokit;
using System.Text;

class Program
{
    static readonly string KeyVault_Name = "azuresolutions";
    static readonly SecretClient KeyVault_Client = new(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());

    static readonly HttpClient hc = new();

    static async Task Main()
    {
        BlobServiceClient bsc = new(connectionString: await GetSecret("BlobServiceClient-ConnectionString"));
        var blobContainerClient = bsc.GetBlobContainerClient("code");

        GitHubClient ghc = new(new ProductHeaderValue("Lorem")) { Credentials = new Credentials(await GetSecret("GitHub-PersonalAccessToken")) };

        var repos = await ghc.Repository.GetAllForCurrent();

        var currentDateTime = DateTime.Now.ToString("yyyyMMdd-HHmmss");

        foreach (var repo in repos)
        {
            var directories = new Stack<string>();
            directories.Push("");

            while (directories.Count > 0)
            {
                var dir = directories.Pop();
                IReadOnlyList<RepositoryContent> contents;
                if (string.IsNullOrEmpty(dir))
                {
                    contents = await ghc.Repository.Content.GetAllContents(repo.Owner.Login, repo.Name);
                }
                else
                {
                    contents = await ghc.Repository.Content.GetAllContents(repo.Owner.Login, repo.Name, dir);
                }

                foreach (var content in contents)
                {
                    if (content.Type == ContentType.Dir)
                    {
                        directories.Push(content.Path);
                    }
                    else if (content.Type == ContentType.File)
                    {
                        Console.WriteLine($"Url: {content.Url}");

                        var file = await hc.GetByteArrayAsync(content.DownloadUrl);
                        var fileContent = Encoding.UTF8.GetString(file);

                        var blobName = $"{currentDateTime}/{repo.Name}/{content.Path}";
                        var blobClient = blobContainerClient.GetBlobClient(blobName);

                        using var stream = new MemoryStream(file);

                        await blobClient.UploadAsync(stream, true);
                    }
                }
            }
        }
    }

    public static async Task<string> GetSecret(string secretName)
    {
        var secret = await KeyVault_Client.GetSecretAsync(secretName);
        return secret.Value.Value;
    }
}
```

-----

### Step 4: Confirm Success

Click "Debug" >>  "Start Debugging".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/345f39aa-5b36-4bc4-924a-f558dd311a60" width="800" title="Snipped April 16, 2024" />

Messages in the console window will iterate through files in your GitHub repository... and these will correspond with the files being copied to your Azure Storage Account.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fedcf39e-886e-4d14-ab24-318a09156d01" width="800" title="Snipped April 16, 2024" />

Navigate to the "code" container and then the dated folder created by the console app. Browse and confirm success. 

-----

### Step 5: Create Index

Navigate to AI Search.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/27eeda13-84ef-4ae9-8879-592e60268475" width="800" title="Snipped April 16, 2024" />

Click "Import Data".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3b7317e9-8bfa-4b77-a11b-374b1bf7de4b" width="800" title="Snipped April 17, 2024" />

Select Data Source "Azure Blob Storage".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2dcb1d8d-541d-4099-9382-ff2896ded255" width="800" title="Snipped April 17, 2024" />

Complete the "Connect to your data" tab/form, pointing at the "code" repository in Azure Storage, then click "Next: Add cognitive skills".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0969c04d-abc9-40f6-bdfd-decfd576b350" width="800" title="Snipped April 17, 2024" />

Click "Skip to: Customize target index".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/84f26b2a-fa97-4f9f-adfe-7c257da115dd" width="800" title="Snipped April 17, 2024" />

Enter an index name, check the header checkboxes for "Retrievable" and "Searchable", then click "Next: Create an indexer".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2c4df56c-48ac-4592-8938-d86b4bacda7e" width="800" title="Snipped April 17, 2024" />

Enter an indexer name, then click "Submit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f8a1fa00-88b6-4cc5-9a18-f08ba26f0503" width="800" title="Snipped April 17, 2024" />

Navigate to "Search Management" >> "Indexers" and confirm successful index processing.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b683039-f6ab-4a99-89ad-bb6c5400ed43" width="800" title="Snipped April 17, 2024" />

Navigate to "Search Management" >> "Index", select the new index and on the resulting page, click "Search" to see results.

-----

### Step 6: Prompt OpenAI

Navigate to OpenAI.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/496b6093-9873-43cc-9290-d90154a00dc9" width="800" title="Snipped April 17, 2024" />

Click "Go to Azure OpenAI Studio" >> "Playground" >> "Chat".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ee9041db-f0d4-444d-a395-a7bd188a7a2d" width="800" title="Snipped April 17, 2024" />

On the "Setup" panel, "Add your data" tab, click "+ Add a data source".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/df5f3ea3-54cc-482c-bc00-1e0fbff77e8a" width="800" title="Snipped April 17, 2024" />

Complete the "Select or add data source" form, pointing at the previously-created AI Search Index, and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d4c94c2b-6229-403d-a839-7465d5da3192" width="800" title="Snipped April 17, 2024" />

Complete the "Data management" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/531842d9-8f06-478c-93e8-4d78b1b8cc23" width="800" title="Snipped April 17, 2024" />

Confirm values on the "Review and finish" page, then click "Save and close".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f9721f68-4ef8-4892-9426-8f2baf0189e9" width="800" title="Snipped April 17, 2024" />

Try out prompts you might use to query and better understand your legacy codebase.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise #2: Iterative Code Suggestions

### Step 1: Update Project

Return to the Visual Studio project and update the code:

```
using Azure;
using Azure.AI.OpenAI;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Storage.Blobs;
using Octokit;
using System.Text;

class Program
{
    static readonly string KeyVault_Name = "azuresolutions";
    static readonly SecretClient KeyVault_Client = new(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());

    static readonly HttpClient hc = new();

    static async Task Main()
    {
        BlobServiceClient bsc = new(connectionString: await GetSecret("BlobServiceClient-ConnectionString"));
        var blobContainerClient = bsc.GetBlobContainerClient("code");

        GitHubClient ghc = new(new ProductHeaderValue("Lorem")) { Credentials = new Credentials(await GetSecret("GitHub-PersonalAccessToken")) };

        OpenAIClient oaic = new(
           endpoint: new Uri($"https://{await GetSecret("OpenAI-Name")}.openai.azure.com/"),
           keyCredential: new AzureKeyCredential(await GetSecret("OpenAI-Key"))
       );

        var repos = await ghc.Repository.GetAllForCurrent();

        var currentDateTime = DateTime.Now.ToString("yyyyMMdd-HHmmss");

        foreach (var repo in repos)
        {
            var directories = new Stack<string>();
            directories.Push("");

            while (directories.Count > 0)
            {
                var dir = directories.Pop();
                IReadOnlyList<RepositoryContent> contents;
                if (string.IsNullOrEmpty(dir))
                {
                    contents = await ghc.Repository.Content.GetAllContents(repo.Owner.Login, repo.Name);
                }
                else
                {
                    contents = await ghc.Repository.Content.GetAllContents(repo.Owner.Login, repo.Name, dir);
                }

                foreach (var content in contents)
                {
                    if (content.Type == ContentType.Dir)
                    {
                        directories.Push(content.Path);
                    }
                    else if (content.Type == ContentType.File)
                    {
                        var file = await hc.GetByteArrayAsync(content.DownloadUrl);
                        var fileContent = Encoding.UTF8.GetString(file);

                        using var stream = new MemoryStream(file);

                        /* Write to Azure Storage */

                        // Console.WriteLine($"Copying {content.Url}");

                        // var blobName = $"{currentDateTime}/{repo.Name}/{content.Path}";
                        // var blobClient = blobContainerClient.GetBlobClient(blobName);

                        // await blobClient.UploadAsync(stream, true);

                        /* Prompt OpenAI */

                        if (content.Name.EndsWith(".cs"))
                        {
                            var OpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt(
                                       OpenAI_Client: oaic,
                                       OpenAI_Deployment_Name: "gpt-4-32k",
                                       UserQuery: $"What does this code do?\n\n{fileContent}",
                                       SystemMessage: "You are a code expert",
                                       Temperature: 0.0f
                                        );

                            Console.WriteLine($"Code:\n{fileContent}\nWhat it does...\n{OpenAI_Prompt.Response}");

                            Console.ReadLine();
                        }
                    }
                }
            }
        }
    }

    public static async Task<string> GetSecret(string secretName)
    {
        var secret = await KeyVault_Client.GetSecretAsync(secretName);
        return secret.Value.Value;
    }
}
```

Notes:
* Additions to: `using...`, `OpenAIClient...`, commenting of prior `blob` logic, and addition of `Prompt OpenAI` logic
* `UserQuery` includes the simple question `What does this code do?`
* Response is only written to the console... ultimately you might send this through to your DevOps system, a database, etc.


-----

### Step 2: Confirm Success

Click "Debug" >>  "Start Debugging".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1a37e026-08b0-4793-ab28-2cb1955f3955" width="800" title="Snipped April 19, 2024" />

Messages in the console window will iterate through C# files in your GitHub repository, ask Open AI "what does this code do?", and then surface response.

-----

**Congratulations... you have successfully completed this exercise**
