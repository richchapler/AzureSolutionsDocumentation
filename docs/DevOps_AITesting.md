# DevOps: AI Testing

⚠️WORK IN PROGRESS⚠️

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/daff9bd1-16d3-4c09-ba64-996c7ce15e3a" width="1000" />

## Use Case

* "We need to test hundreds of prompts using various solution configurations"
* "We need to provide testers with a simple and friendly way to capture test cases and results"

## Proposed Solution

* **Customize DevOps**: Create a new process with a customized Test Case entity
* **Automate Processing**: Create and publish a scheduled process that prepares new Test Cases for review
* **Confirm Success**: Demonstrate basic functionality

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [**AI Search**](https://azure.microsoft.com/en-us/products/search) index with Semantic Configuration

  _Note: I used the Tax Form index created in [DevOps: AI Deployment](https://github.com/richchapler/AzureSolutions/blob/main/docs/DevOps_AIDeployment.md)_

* [**DevOps**](https://azure.microsoft.com/en-us/products/devops/) with organization, project, and a Personal Access Token with "Read, write, & manage" permissions

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

Also prepare a CSV file with sample OpenAI prompts; example:

```
Work Item Type,Title,UserMessage
Test Case,Prompt,"How does Form 1040 provide a comprehensive view of an individual’s various sources of income, including wages, dividends, and capital gains?"
Test Case,Prompt,What insights can be drawn from the deductions section of Form 1040 about an individual’s financial obligations and contributions?
Test Case,Prompt,How does the information on a W2 form reflect an individual’s employment history and earnings for the year?
Test Case,Prompt,What can Form W4 tell us about an individual’s tax withholding preferences and their anticipated tax liability?
Test Case,Prompt,How does Form W9 provide information about an individual’s or entity’s contractual relationships and responsibilities?
```

_Note: My sample prompts are about tax forms since we're using the Tax Form index created in [DevOps: AI Deployment](https://github.com/richchapler/AzureSolutions/blob/main/docs/DevOps_AIDeployment.md)_

-----

## Exercise 1: Customize DevOps

In this exercise we will create a new process with a customized Test Case entity.

### Step 1: Create Process

Navigate to DevOps >> Organization Settings >> Boards >> Process and then click on "Basic".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/148d988e-2807-47f8-9e2f-279217fc67e9" width="800" title="Snipped: December 15, 2023" />

Click "create an inherited process" in the light blue header.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/10553d8e-e610-418b-adc6-abe2e80e219c" width="800" title="Snipped: December 15, 2023" />

Complete the form and click "Create process".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7792d59b-a084-4ed3-b0a1-9fe4406877b6" width="800" title="Snipped: December 15, 2023" />

### Step 2: Add Fields

Click to open "Test Case".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0c4ba82-d816-4aa3-bbdd-0f188ad8602b" width="800" title="Snipped: December 18, 2023" />

#### "Layout" >> "Steps"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3b4be34c-1c66-4713-8925-9d35ba0ecc76" width="800" title="Snipped: December 18, 2023" />

Rollover the "Deployment", click the ellipses, and then click "Hide from layout". Then, repeat for "Development".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/89a9729c-af39-4cd3-8184-c44ae7cec844" width="800" title="Snipped: December 18, 2023" />

Click "New field" and complete the resulting "Add a field to Test Case":

| Prompt             | Entry                 |
| :----------------- | :-------------------- |
| **Create a field** | Selected              |
| **Name**           | UserMessage           |
| **Type**           | Text (multiple lines) |

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c3018fad-5acc-4cd3-8a02-24fe96c3f921" width="800" title="Snipped: December 18, 2023" />

On the "Layout" page, enter Label "User Message (aka Prompt)". Click "Add field" and then repeat for a new "SystemMessage" field.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e6e68496-fa13-455d-9a26-4109925f3f4e" width="800" title="Snipped: December 18, 2023" />

#### "Layout" >> "Responses"

Click the eliipses on the "Summary" tab and then "Edit" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b66bc720-e344-40dd-a07b-2e64a4a616b0" width="800" title="Snipped: December 18, 2023" />

Enter name "Responses" and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9248f45b-06d5-4889-8a39-d33af8aa086a" width="800" title="Snipped: December 18, 2023" />

Hide the "Description" field and add two new fields: "Response_Simple" and "Response_Semantic"
_Note: "Simple" refers to "Keyword"_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/884a36ed-fe3b-46ae-9041-bd6c8157ae5d" width="800" title="Snipped: December 18, 2023" />

In the third column, click "Add a field" and then complete the resulting "Add a field to Test Case":

| Prompt             | Entry                                                        |
| :----------------- | :----------------------------------------------------------- |
| **Create a field** | Selected                                                     |
| **Name**           | Response_Simple_Ranking                                      |
| **Type**           | Picklist (string) including: "1-Very Bad", "2-Bad", "3-Neutral", "4-Good", and "5-Very Good" |

_Note: This field will be manually populated by testers as they evaluate automatically-generated responses_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8b231935-bddb-40ed-9278-3f1bda401c61" width="800" title="Snipped: December 18, 2023" />

On the "Layout" tab, enter Label "Response (keyword)" and then click "Add field". Repeat for "Response_Semantic_Ranking".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4f92d269-8d24-4669-855d-e534c3e003c6" width="800" title="Snipped: December 18, 2023" />

### Step 3: Modify States

Click on the "States" tab and then click "+ New state".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/91170166-161f-43b9-bb75-c89312e8e8a4" width="800" title="Snipped: December 18, 2023" />

Complete the "Add a state to Test Case" popup, including:

| Prompt             | Entry            |
| :----------------- | :--------------- |
| **Name**           | Ready for OpenAI |
| **State category** | Proposed         |

Click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/36e5aeb9-b45e-45ed-b9ae-b0c19d482333" width="800" title="Snipped: December 18, 2023" />

Rollover "Design", click the ellipses, and then click "Hide" in the dropdown.
<br>Click "+ New state" and complete the "Add a state to Test Case" popup, including:

| Prompt             | Entry           |
| :----------------- | :-------------- |
| **Name**           | Ready for Human |
| **State category** | In Progress     |

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ad81f183-503d-4255-8e37-63d5b5416320" width="800" title="Snipped: December 18, 2023" />

Click "Create" and then hide "In Progress" state "Ready".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d294e75f-f3d0-4457-b8e6-9d401a009c9e" width="800" title="Snipped: December 18, 2023" />

### Step 4: Prepare Project

From the Organization page, click "+ New project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0faee4db-70b8-4b9e-83db-a0954033b5c1" width="800" title="Snipped: December 18, 2023" />

Complete the resulting popout, including "Work item process" selection "AI Testing". Click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9b6b50d7-af77-45bb-8820-5005a3b1ba49" width="800" title="Snipped: December 18, 2023" />

Navigate to aiTesting >> Boards >> Work Items and then click "Import Work Items".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/123ba847-5de4-4e0c-a28b-9676f9303ebc" width="800" title="Snipped: December 18, 2023" />

Review imported Work Items.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0c64317a-2f72-49b4-9d96-41968f40a162" width="800" title="Snipped: December 18, 2023" />

Click "Save items".

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Automate Processing

In this exercise, we create and publish a scheduled process that prepares new Test Cases for review.

### Step 1: Create Visual Studio Project

Open Visual Studio and click "**Create a new project**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6f0351e0-670b-4dbc-9b84-1ceba120a2cd" width="600" title="Snipped: December 19, 2023" />

On the "**Create a new project**" form, search for and select "**Azure Functions**", then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6841ad94-cbc5-49f5-bc2d-0601e5918ffa" width="600" title="Snipped: December 19, 2023" />

Complete the "**Configure your new project**" form, then click "**Next**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/97f51669-f00a-4960-8a60-6226a9eb8243" width="600" title="Snipped: December 19, 2023" />

Complete the "**Additional information**" form, including:

| Prompt               | Entry                                 |
| :------------------- | :------------------------------------ |
| **Functions worker** | .NET 8.0 Isolated (Long Term Support) |
| **Function**         | Timer trigger                         |
| **Schedule**         | 0 */5 * * * *                         |

Click "**Create**".

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/693705ea-9706-4c04-9219-ea7b1058275e" width="800" title="Snipped: December 19, 2023" />

Click "Tools" in the menu bar, expand "NuGet Package Manager" and then click "Manage NuGet Packages for Solution..." in the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0bf233da-6648-4169-aaff-c92b05d00432" width="800" title="Snipped: December 19, 2023" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Azure.Identity". On the resulting pop-out, check the box next to your project and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a3225127-1683-4147-a8b1-02ddda6cc709" width="300" title="Snipped: December 19, 2023" />

When prompted, click "I Accept" on the "License Acceptance" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/306c0f70-0ae2-4b3f-970d-5a4e24137647" width="800" title="Snipped: December 19, 2023" />

#### Additional Packages

Repeat this process for the following NuGet packages:

* Azure.AI.OpenAI (**1.0.0-beta.9**)
* Azure.Security.KeyVault.Secrets
* Microsoft.TeamFoundationServer.Client

-----

### Step 3: Code Application

Rename "Function1.cs" to "processTestCases.cs". When prompted "Would you also like to perform a rename...", click "Yes".

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers". 

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d07757f0-5582-4361-b33c-f59eccda60af" width="600" title="Snipped: December 19, 2023" />

#### Helper Class: KeyVault

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup. Replace the default code with:

```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace processTestCases.Helpers
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

        public string getSecret(string secretName)
        {
            Response<KeyVaultSecret> responseKeyVaultSecret = client.GetSecret(secretName);
            return responseKeyVaultSecret.Value.Value;
        }
    }
}
```

**Logic Explained**:

1. **Namespace and Class Definition**
   The code defines a namespace `processTestCases.Helpers` and an internal class `KeyVault` within it. This class is used to interact with Azure Key Vault.

2. **Dependencies**
   The code uses three Azure libraries:
   - `Azure`: The main Azure SDK package.
   - `Azure.Identity`: Provides Azure Active Directory token authentication support across the Azure SDK.
   - `Azure.Security.KeyVault.Secrets`: Provides functionality to access and manage secrets in Azure Key Vault.

3. **Class Variables**
   The `KeyVault` class has one private variable `client` of type `SecretClient`. This client interacts with Azure Key Vault.

4. **Constructor**
   The constructor of the `KeyVault` class initializes the `client` with the URI of the Azure Key Vault and a `DefaultAzureCredential`. The `DefaultAzureCredential` is a type of Azure Identity that provides a simplified authentication experience to quickly start developing applications run in the Azure cloud.

5. **getSecret Method**
   The `getSecret` method takes a secret name as input, retrieves the secret from Azure Key Vault using the `client`, and returns the value of the secret.

Click "Save".

#### Helper Class: DevOps

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "DevOps.cs" on the resulting popup. Replace the default code with:

```
using Microsoft.TeamFoundation.WorkItemTracking.WebApi;
using Microsoft.TeamFoundation.WorkItemTracking.WebApi.Models;
using Microsoft.VisualStudio.Services.Common;
using Microsoft.VisualStudio.Services.WebApi;
using Microsoft.VisualStudio.Services.WebApi.Patch.Json;

namespace processTestCases.Helpers
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

**Logic Explained**:

This code is a helper class for interacting with Azure DevOps. Here's a breakdown of its logic:

1. **Initialization (`DevOps()`):**
   The constructor initializes a `WorkItemTrackingHttpClient` client using the Azure DevOps URL and a Personal Access Token (PAT) retrieved from Azure Key Vault.

2. **Fetching Test Cases (`getTestCases()`):**
   This method fetches 'Test Case' work items from Azure DevOps that are in the 'Ready for OpenAI' state. It uses the Work Item Query Language (WIQL) to form the query. If any matching work items are found, their details are fetched and returned as a list.

3. **Updating a Work Item (`updateWorkItem()`):**
   This method updates a specific work item in Azure DevOps. It takes several parameters including the work item ID, title, prompt, responses, and steps. It creates a `JsonPatchDocument` to hold the updates, which are then applied to the work item using the `UpdateWorkItemAsync()` method.

4. **Adding a Field to a Work Item (`addField()`):**
   This is a helper method used by `updateWorkItem()`. It adds a new field to the `JsonPatchDocument` for the work item. The field is specified by its path and value.

5. **Default Steps (`defaultSteps()`):**
   This method returns a string representing the default steps for a test case. These steps are formatted in XML.

The purpose of this class is to automate the process of fetching and updating test cases in Azure DevOps. It's designed to work with a specific workflow where test cases are initially set to the 'Ready for OpenAI' state, and then updated with responses from an AI model.

Click "Save".

#### Helper Class: OpenAI

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "OpenAI.cs" on the resulting popup. Replace the default code with:

```
using Azure;
using Azure.AI.OpenAI; /* pre-release NuGet Package: Azure.AI.OpenAI v1.0.0-beta.9 */

namespace processTestCases.Helpers
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

**Logic Explained**:

This is a C# class named `OpenAI` in the namespace `processTestCases.Helpers`. It uses the Azure.AI.OpenAI library to interact with the OpenAI API. Here's a breakdown of its functionality:

1. **Initialization**: The `OpenAI` constructor initializes an `OpenAIClient` with the endpoint and key retrieved from a `KeyVault` instance.

2. **Prompt Method**: This method accepts three parameters: `queryType`, `systemMessage`, and `userMessage`. It's an asynchronous method that returns a `Task<string>`, meaning it performs operations in the background and returns a string when it's done.

   - It first determines the type of Azure Cognitive Search query to use based on the `queryType` parameter.

   - It then creates an `AzureCognitiveSearchChatExtensionConfiguration` object, which is used to configure the Azure Cognitive Search extension. This includes setting the search endpoint, index name, query type, and other parameters.

   - A `ChatCompletionsOptions` object is created next. This object is used to configure the chat completion options, including the deployment name and Azure extensions options.

   - Two `ChatMessage` objects are added to the `ChatCompletionsOptions` object: one for the system message and one for the user message.

   - Finally, it calls the `GetChatCompletionsAsync` method on the `OpenAIClient` object, passing in the `ChatCompletionsOptions` object. This method sends a request to the OpenAI API and returns the content of the first choice from the response.

This class essentially serves as a helper for making requests to the OpenAI API, with specific configurations for Azure Cognitive Search. It's designed to be used in a larger application where the `Prompt` method would be called with specific parameters to generate a response from the OpenAI API. The response is then returned to the calling code.

Click "Save".

#### processTestCases.cs

Open "processTestCases.cs" and replace the default code with:

```
using Microsoft.Azure.Functions.Worker;
using processTestCases.Helpers;

namespace processTestCases
{
    public class processTestCases
    {
        [Function("processTestCases")]
        public async Task Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer)
        {
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
                    var responseSimple = await openai.Prompt(queryType: "Simple", systemMessage, userMessage);
                    var responseSemantic = await openai.Prompt(queryType: "Semantic", systemMessage, userMessage);

                    devops.updateWorkItem(
                        id,
                        title: $"Prompt: '{string.Join(" ", userMessage.Split(' ').Take(5))}...'",
                        userMessage,
                        responseSimple,
                        responseSemantic,
                        steps: devops.defaultSteps()
                        );
                }
            }
        }
    }
}
```

Logic Explained:

This is a piece of code written in C# for an Azure Function. The function is named `processTestCases` and it's triggered every 5 minutes (as indicated by the cron expression "0 */5 * * * *"). Here's a step-by-step explanation of the logic:

1. The function first creates instances of two classes: `DevOps` and `OpenAI`.

2. It then calls the `getTestCases` method from the `devops` instance to fetch test cases.

3. For each test case, it checks if the `Id` field has a value. If it does, it assigns the value to the `id` variable. If not, it skips the current iteration and moves to the next test case.

4. It then retrieves the `systemMessage` and `userMessage` from the test case fields.

5. If both `systemMessage` and `userMessage` are not null, it calls the `Prompt` method from the `openai` instance twice, once with the `queryType` set to "Simple" and once with it set to "Semantic". These calls return `responseSimple` and `responseSemantic` respectively.

6. Finally, it calls the `updateWorkItem` method from the `devops` instance, passing in the `id`, a title constructed from the `userMessage`, the `userMessage` itself, `responseSimple`, `responseSemantic`, and the result of calling `defaultSteps` from the `devops` instance.

In summary, this function is processing test cases by using an AI model to generate responses to prompts and updating the test cases with these responses. The processing is done every 5 minutes. The `DevOps` class is likely responsible for interacting with Azure DevOps to fetch and update work items (test cases), while the `OpenAI` class is likely responsible for interacting with the OpenAI API to generate responses to prompts. The specifics would depend on the implementation of these classes and methods.

Click "Save".

-----

### Step 4: Confirm Success








LOREM IPSUM

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
