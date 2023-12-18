# DevOps: AI Testing
⚠️WORK IN PROGRESS⚠️

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/daff9bd1-16d3-4c09-ba64-996c7ce15e3a" width="1000" />

## Use Case
* "We need to test hundreds of prompts with various configurations of AI Search and OpenAI"
* "We need to provide testers with a simple and friendly way to capture test cases and results"

## Proposed Solution
* **Customize DevOps**: Create a new process with a customized Test Case entity
* **Automate Processing**: Create a scheduled process that prepares new Test Cases for review
* **Confirm Success**: Demonstrate basic functionality

## Solution Requirements
This documentation assumes the following resources are ready for use:
* [**AI Search**](https://azure.microsoft.com/en-us/products/search) index with Semantic Configuration
  <br>_Note: I used the Tax Form index created in [DevOps: AI Deployment](https://github.com/richchapler/AzureSolutions/blob/main/docs/DevOps_AIDeployment.md)_
* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/) with organization and project
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * AISearch-IndexName
  * AISearch-Key
  * AISearch-SemanticConfiguration
  * AISearch-Url
  * DevOps-PersonalAccessToken
  * DevOps-Url
  * OpenAI-DeploymentName
  * OpenAI-Endpoint
  * OpenAI-Key
* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/) with **Azure development** workload and connected to your DevOps project

-----

## Exercise 1: Customize DevOps
In this exercise we will create a new process with a customized Test Case entity.

### Step 1: Create Inherited Process

Navigate to DevOps >> Organization Settings >> Boards >> Process and then click on "Basic".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/148d988e-2807-47f8-9e2f-279217fc67e9" width="800" title="Snipped: December 15, 2023" />

Click "create an inherited process" in the light blue header.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/10553d8e-e610-418b-adc6-abe2e80e219c" width="800" title="Snipped: December 15, 2023" />

Complete the form and click "Create process".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7792d59b-a084-4ed3-b0a1-9fe4406877b6" width="800" title="Snipped: December 15, 2023" />

Click to open "Test Case".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0c4ba82-d816-4aa3-bbdd-0f188ad8602b" width="800" title="Snipped: December 18, 2023" />

#### "Steps" tab

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3b4be34c-1c66-4713-8925-9d35ba0ecc76" width="800" title="Snipped: December 18, 2023" />

Rollover the "Deployment", click the ellipses, and then click "Hide from layout". Then, repeat for "Development".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/89a9729c-af39-4cd3-8184-c44ae7cec844" width="800" title="Snipped: December 18, 2023" />

Click "New field" and complete the resulting "Add a field to Test Case":

Prompt | Entry
:----- | :-----
**Create a field** | Selected
**Name** | UserMessage
**Type** | Text (multiple lines)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c3018fad-5acc-4cd3-8a02-24fe96c3f921" width="800" title="Snipped: December 18, 2023" />

On the "Layout" page, enter Label "User Message (aka Prompt)". Click "Add field" and then repeat for a new "SystemMessage" field.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e6e68496-fa13-455d-9a26-4109925f3f4e" width="800" title="Snipped: December 18, 2023" />

#### "Responses" tab

Click the eliipses on the "Summary" tab and then "Edit" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b66bc720-e344-40dd-a07b-2e64a4a616b0" width="800" title="Snipped: December 18, 2023" />

Enter name "Responses" and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9248f45b-06d5-4889-8a39-d33af8aa086a" width="800" title="Snipped: December 18, 2023" />

Hide the "Description" field and add two new fields: "Response_Simple" and "Response_Semantic"
_Note: "Simple" refers to "Keyword"_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/dd2b6ff4-95cd-4250-a744-6df9fe3b95f7" width="800" title="Snipped: December 18, 2023" />

In the third column, click "Add a field" and then complete the resulting "Add a field to Test Case":

Prompt | Entry
:----- | :-----
**Create a field** | Selected
**Name** | Response_Simple_Ranking
**Type** | Picklist (string) including: "1-Very Bad", "2-Bad", "3-Neutral", "4-Good", and "5-Very Good"

_Note: This field will be manually populated by testers as they evaluate automatically-generated responses_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b231935-bddb-40ed-9278-3f1bda401c61" width="800" title="Snipped: December 18, 2023" />

On the "Layout" tab, enter Label "Response (keyword)" and then click "Add field". Repeat for "Response_Semantic_Ranking".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4f92d269-8d24-4669-855d-e534c3e003c6" width="800" title="Snipped: December 18, 2023" />

### Step 2: Create Project

From the Organization page, click "+ New project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0faee4db-70b8-4b9e-83db-a0954033b5c1" width="800" title="Snipped: December 18, 2023" />

Complete the resulting popout, including "Work item process" selection "AI Testing". Click "Create".











-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Automate Processing
In this exercise, we will test prompts programmatically interact with OpenAI + AI Search index:

### Step 1: Create Visual Studio Project
In this exercise, we will use the AI Search Development Kit (SDK) to create a data source, index, skillset, and indexer.

Open Visual Studio and click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/317959b5-dfd7-4c97-af0c-0578f9e89429" width="600" title="Snipped: October 10, 2023" />

On the "**Create a new project**" form, search for and select "**Console App**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/db8c2898-b607-4441-8b1e-4f4f3dbd56b4" width="600" title="Snipped: October 10, 2023" />

Complete the "**Configure your new project**" form, then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2408d491-ba3b-4ba7-9d84-02caf1dab54d" width="600" title="Snipped: October 10, 2023" />

Complete the "**Additional information**" form, then click "**Create**".

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/464851d5-30c0-4b72-87d5-cb95658d919d" width="800" title="Snipped: October 11, 2023" />

Click **Tools** in the menu bar, expand "**NuGet Package Manager**" in the resulting menu and then click "**Manage NuGet Packages for Solution...**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0b3bc3a-e6af-47ff-8ed2-8c0d0340e44e" width="800" title="Snipped: October 11, 2023" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**Azure.Search.Documents**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b56a92c-594a-4a18-afaa-24a4872ac73b" width="300" title="Snipped: October 11, 2023" />

When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d9906b3b-848d-4807-bc0c-441daf502865" width="800" title="Snipped: October 11, 2023" />

**Additional Packages**

Repeat this process for the following NuGet packages:

* Azure.Identity
* Azure.Security.KeyVault.Secrets

Close the "**NuGet - Solution**" tab.

-----

### Step 3: Code Application

Replace the default code on the "**Program.cs**" tab with the following C#:



### Helper Class: DevOps

```
using Microsoft.TeamFoundation.WorkItemTracking.WebApi;
using Microsoft.TeamFoundation.WorkItemTracking.WebApi.Models;
using Microsoft.VisualStudio.Services.Common;
using Microsoft.VisualStudio.Services.WebApi;
using Microsoft.VisualStudio.Services.WebApi.Patch.Json;

namespace ConsoleApp1.Helpers
{
    internal class DevOps
    {
        KeyVault keyvault = new();

        private WorkItemTrackingHttpClient client;

        public DevOps()
        {

            client = new VssConnection(
                baseUrl: new Uri(keyvault.getSecret("DevOps-Url")),
                credentials: new VssBasicCredential(
                    userName: string.Empty,
                    password: keyvault.getSecret("DevOps-PersonalAccessToken") /* DevOps Personal Access Token with "Read, write, & manage" permissions */
                    )
                ).GetClient<WorkItemTrackingHttpClient>();            
        }

        public async Task<List<WorkItem>> getTestCases()
        {
            Wiql q = new()
            {
                Query = @"SELECT [System.Id], [System.State], [Custom.SystemMessage], [Custom.Prompt]
                            FROM workitems
                            WHERE [System.WorkItemType] = 'Test Case' AND [System.State] = 'Ready for OpenAI'"
            };

            WorkItemQueryResult wiqr = await client.QueryByWiqlAsync(q);

            List<WorkItem> testCases = new();

            if (wiqr != null && wiqr.WorkItems.Any())
            {
                int[] ids = wiqr.WorkItems.Select(item => item.Id).ToArray();
                testCases = await client.GetWorkItemsAsync(ids);
            }

            return testCases;
        }

        public void updateWorkItem(int id, string title, string prompt, string responseSimple, string responseSemantic, string steps)
        {
            Microsoft.VisualStudio.Services.WebApi.Patch.Json.JsonPatchDocument devopsWorkItem_Definition = new();
            addField(devopsWorkItem_Definition, "/fields/System.Title", title);
            addField(devopsWorkItem_Definition, "/fields/System.State", "Ready for Human");
            addField(devopsWorkItem_Definition, "/fields/Prompt", prompt);
            addField(devopsWorkItem_Definition, "/fields/Response_Simple", responseSimple);
            addField(devopsWorkItem_Definition, "/fields/Response_Simple_Ranking", "3-Neutral");
            addField(devopsWorkItem_Definition, "/fields/Response_Semantic", responseSemantic);
            addField(devopsWorkItem_Definition, "/fields/Response_Semantic_Ranking", "3-Neutral");
            addField(devopsWorkItem_Definition, "/fields/Microsoft.VSTS.TCM.Steps", steps);

            var workItem = client.UpdateWorkItemAsync(devopsWorkItem_Definition, id).Result;
        }

        private void addField(Microsoft.VisualStudio.Services.WebApi.Patch.Json.JsonPatchDocument devopsWorkItem_Definition, string path, string value)
        {
            devopsWorkItem_Definition.Add(
                new JsonPatchOperation
                {
                    Operation = Microsoft.VisualStudio.Services.WebApi.Patch.Operation.Add,
                    Path = path,
                    Value = value
                }
            );
        }

        public string defaultSteps()
        {
            return @"
            <steps id='0' last='3'>
                <step id='2' type='ValidateStep'>
                    <parameterizedString isformatted='true'>Review User Message (aka Prompt) and System Message</parameterizedString>
                    <parameterizedString isformatted='true'>Confirm that messages are well-formed</parameterizedString>
                    <description/>
                </step>
                <step id='3' type='ValidateStep'>
                    <parameterizedString isformatted='true'>Evaluate Responses</parameterizedString>
                    <parameterizedString isformatted='true'>Rank Responses from each model</parameterizedString>
                    <description/>
                </step>
            </steps>";
        }
    }
}
```

### Helper Class: KeyVault

```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace ConsoleApp1.Helpers
{
    internal class KeyVault
    {
        private SecretClient client;

        public KeyVault()
        {
            client = new SecretClient(
                vaultUri: new Uri("https://rchaplerkv.vault.azure.net/"),
                credential: new DefaultAzureCredential()
                );
        }

        public KeyVaultSecret getSecret(string secretName)
        {
            Response<KeyVaultSecret> responseKeyVaultSecret = client.GetSecret(secretName);
            return responseKeyVaultSecret.Value;
        }
    }
}
```

### Helper Class: OpenAI

```
using Azure;
using Azure.AI.OpenAI; /* pre-release NuGet Package: Azure.AI.OpenAI v1.0.0-beta.9 */

namespace ConsoleApp1.Helpers
{
    internal class OpenAI
    {
        KeyVault keyvault = new();

        private OpenAIClient client;

        public OpenAI()
        {
            client = new OpenAIClient(
                endpoint: new Uri(keyvault.getSecret("OpenAI-Endpoint")),
                keyCredential: new AzureKeyCredential(keyvault.getSecret("OpenAI-Key"))
            );
        }

        public async Task<string> Prompt(string queryType, string systemMessage, string userMessage)
        {
            AzureCognitiveSearchQueryType type;

            switch (queryType.ToLower())
            {
                case "simple": type = AzureCognitiveSearchQueryType.Simple; break;
                case "semantic": type = AzureCognitiveSearchQueryType.Semantic; break;
                default: throw new ArgumentException($"Invalid query type: {queryType}");
            }

            AzureCognitiveSearchChatExtensionConfiguration config = new()
            {
                SearchEndpoint = new Uri(keyvault.getSecret("AISearch-Url")),
                IndexName = keyvault.getSecret("AISearch-IndexName"),
                QueryType = type,
                ShouldRestrictResultScope = true,
                DocumentCount = 5,
                SemanticConfiguration = keyvault.getSecret("AISearch-SemanticConfiguration") /* Ignored when queryType != Semantic */
            };
            config.SetSearchKey(searchKey: keyvault.getSecret("AISearch-Key"));

            ChatCompletionsOptions cco = new()
            {
                DeploymentName = keyvault.getSecret("OpenAI-DeploymentName"),
                AzureExtensionsOptions = new AzureChatExtensionsOptions() { Extensions = { config } }
            };

            cco.Messages.Add(new ChatMessage(ChatRole.System, systemMessage));
            cco.Messages.Add(new ChatMessage(ChatRole.User, userMessage));

            var response = await client.GetChatCompletionsAsync(cco);
            return response.Value.Choices[0].Message.Content;
        }
    }
}
```

### Program.cs

```
using ConsoleApp1.Helpers;

DevOps devops = new(); OpenAI openai = new();

var testCases = await devops.getTestCases();

foreach (var testCase in testCases)
{
    int id;
    if (testCase.Id.HasValue) { id = testCase.Id.Value; } else { continue; }

    string? systemMessage = testCase.Fields["Custom.SystemMessage"].ToString();
    string? userMessage = testCase.Fields["Custom.Prompt"].ToString();

    if (systemMessage != null && userMessage != null)
    {
        string title = $"Prompt: '{string.Join(" ", userMessage.Split(' ').Take(5))}...'";
        Console.WriteLine(title);

        var responseSimple = await openai.Prompt(queryType: "Simple", systemMessage, userMessage);
        var responseSemantic = await openai.Prompt(queryType: "Semantic", systemMessage, userMessage);

        devops.updateWorkItem(
            id,
            title,
            userMessage,
            responseSimple,
            responseSemantic,
            steps: devops.defaultSteps()
            );
    }
}
```

Logic Explained:
1. **SearchIndexer Creation**: A `SearchIndexer` named `indexer` is created with a specified indexer name, data source name, and index name. The indexer is configured with specific parameters and a skillset name
2. **IndexingParametersConfiguration**: The `IndexingParametersConfiguration` is set to `BlobIndexerImageAction.GenerateNormalizedImagePerPage`, which means the indexer will perform Optical Character Recognition (OCR) on images in blobs and generate a normalized image per page
3. **IsDisabled**: The indexer is initially disabled (`IsDisabled = true`) to prevent it from auto-processing after creation... re-enable by modifying Indexer >> "Indexer Definition (JSON)"
4. **OutputFieldMappings**: These mappings define how the output of a skill is mapped to a field in an index schema:
   <br>`text`... from `OcrSkill` and `/document/normalized_images/*/text`
   <br>`keyphrases`... from `KeyPhraseExtractionSkill` and `/document/content/keyphrases`
   <br>`myColumn`... from `WebApiSkill` and `/document/content/myColumn` (custom skillset)
6. **DeleteIndexer** and **CreateIndexer**: The existing indexer with the same name is deleted if it exists, and then the new indexer is created.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 3: Confirm Sucess
In this exercise, we will lorem ipsum.

### Step 1: Lorem Ipsum

-----

**Congratulations... you have successfully completed all exercises**

-----

Reference:
* https://learn.microsoft.com/en-us/azure/ai-services/openai/
* https://github.com/Azure/azure-sdk-for-net/blob/Azure.AI.OpenAI_1.0.0-beta.9/sdk/openai/Azure.AI.OpenAI/README.md
* https://learn.microsoft.com/en-us/azure/devops/boards/queries/import-work-items-from-csv?view=azure-devops
