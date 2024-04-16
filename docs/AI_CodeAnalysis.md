# AI and at-scale Code Analysis

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a74be382-8c5b-4b17-80f6-bc8c0232f68e" width="1000" />

## Use Case
* "We want to use AI to better understand legacy code"
* "We want to use AI to suggest code optimizations, **at scale**"

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

### Exercise 1: Chat with Legacy Code

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







LOREM IPSUM
-----

**Congratulations... you have successfully completed this exercise**

-----
