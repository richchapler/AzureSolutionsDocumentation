# AI and at-scale Code Analysis

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a74be382-8c5b-4b17-80f6-bc8c0232f68e" width="1000" />

## Use Case
* "We want to use AI to better understand legacy code"
* "We want to use AI to suggest code optimizations, at scale"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [AI Search](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration

* GitHub [Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository) and [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * BlobServiceClient-ConnectionString
  * OpenAI-Key
  * GitHub-PersonalAccessToken

* [Storage Account](Infrastructure_StorageAccount.md)

## Documentation Note

Codeblock explanation is limited... I recommend that you copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Exercise 1: Chat with Codebase

### Step 1: Create Project

Open Visual Studio and create a Console App project.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0659d99b-82f5-4d29-8326-e4a78af08818" width="800" title="Snipped April 16, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eb2cb824-3951-488f-813e-7bf9d655e41b" width="800" title="Snipped April 16, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Octokit".
On the resulting pop-out, check the box next to your project and then click "Install".
Repeat this process for:
* Azure.Security.KeyVault.Secrets
* Azure.Storage.Blobs
* AzureSolutions.Helpers
