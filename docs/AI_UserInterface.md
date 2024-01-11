# AI Search + OpenAI: User Interface
:warning: WORK-IN-PROGRESS :warning:

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7951bd4a-1806-427c-bf5f-23714f63b73a" width="1000" />

## Use Case
* "The AI Search and OpenAI demonstration apps are insufficient"
* "We want a demonstration application that allows us to query both AI Search and OpenAI at once"
* "We want to see responses from more than one configuration of Open AI"
* "We need a User Interface Template that we can customize for our various internal use cases"

## Proposed Solution
* **Create Application**: lorem ipsum
* **Publish Application**: lorem ipsum

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [**AI Search**](https://azure.microsoft.com/en-us/products/search) index with Semantic Configuration

  _Note: I used the Tax Form index created in [DevOps: AI Deployment](https://github.com/richchapler/AzureSolutions/blob/main/docs/DevOps_AIDeployment.md)_

* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):

  * AISearch-IndexName
  * AISearch-Key
  * AISearch-Url
  * OpenAI-DeploymentName
  * OpenAI-Endpoint
  * OpenAI-Key

:warning: VALIDATE THIS LIST BEFORE PUBLISHING! :warning:

* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/)

-----

## Exercise 1: Create Application

In this exercise we will create an application.

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/138cf716-d18b-424f-a467-5cdeabcd4beb" width="600" title="Snipped January 9, 2024" />

On the "Create a new project**" form, search for and select "ASP .NET CoreWeb App (Razor Pages)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6e2b790f-8e37-4a26-8ed2-5e035ea9eda8" width="600" title="Snipped January 9, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c7878509-dba2-4dd8-ba27-81735fa0d81d" width="600" title="Snipped January 9, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fe93917f-d763-4026-86af-3ab3ddb22e3d" width="800" title="Snipped January 9, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2dbd5b36-79b8-42ec-aaae-c53d6ed114e7" width="800" title="Snipped January 9, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1662d9f3-56b5-4950-bf0a-db53bf6f0800" width="800" title="Snipped January 9, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Azure.Identity".

On the resulting pop-out, check the box next to your project and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6d312057-30b7-4579-b08a-1c8b1fa5b47a" width="300" title="Snipped January 9, 2024" />

When prompted, click "I Accept" on the "License Acceptance" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/306c0f70-0ae2-4b3f-970d-5a4e24137647" width="800" title="Snipped December 19, 2023" />

#### Additional Packages

Repeat this process for the following NuGet packages:

* Azure.AI.OpenAI (**1.0.0-beta.9**)
* Azure.Search.Documents
* Azure.Security.KeyVault.Secrets

-----

### Step 3: Add Helpers

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4310c7fb-41ff-4f34-b18b-e43771662e18" width="600" title="Snipped January 11, 2024" />

#### Helper Class: KeyVault

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/87a5af20-1942-4500-bb26-6cbe4c74e76b" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace AI_UserInterface.Helpers
{
    internal class KeyVault
    {
        private SecretClient client;

        public KeyVault()
        {
            client = new SecretClient(
                vaultUri: new Uri("https://KEYVAULTNAME.vault.azure.net/"),
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

#### Helper Class: AISearch

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d7ffc067-4bc0-4191-8bd3-c07510ecf5de" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

```
using Azure;
using Azure.Search.Documents;
using Azure.Search.Documents.Models;
using System.Text.Json;

namespace AI_UserInterface.Helpers
{
    internal class AISearch
    {
        private readonly KeyVault keyvault;

        public AISearch()
        {
            keyvault = new KeyVault();
        }

        public async Task<string> Prompt(string query)
        {
            try
            {
                SearchClient client = new(
                             endpoint: new Uri(keyvault.getSecret("AISearch-Url")),
                             indexName: $"{keyvault.getSecret("AISearch-Name")}-index",
                             credential: new AzureKeyCredential(keyvault.getSecret("AISearch-Key"))
                             );

                SearchResults<SearchDocument> response = await client.SearchAsync<SearchDocument>(query);

                List<SearchDocument> documents = new List<SearchDocument>();

                foreach (SearchResult<SearchDocument> result in response.GetResults())
                {
                    documents.Add(result.Document);
                }

                var options = new JsonSerializerOptions
                {
                    WriteIndented = true
                };

                return JsonSerializer.Serialize(documents, options);
            }
            catch (Exception ex)
            {
                return $"Error: {ex.Message}";
            }
        }
    }
}
```

#### Helper Class: OpenAI

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/503ef00f-8f06-4ad7-8eca-08d55cca6553" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

```
using Azure;
using Azure.AI.OpenAI;

namespace AI_UserInterface.Helpers
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
                IndexName = keyvault.getSecret("AISearch-Name") + "-index",
                QueryType = type,
                ShouldRestrictResultScope = true,
                DocumentCount = 5,
                SemanticConfiguration = keyvault.getSecret("AISearch-Name") + "-semanticconfiguration" /* Ignored when queryType != Semantic */
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

-----

### Step 4: Update Visual Elements

#### _Layout.cshtml

Expand "Pages" >> "Shared" and double-click to open "_Layout.cshtml".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e5a2bc69-ee94-485f-be3e-6ef2a1327f25" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - AI_UserInterface</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/AI_UserInterface.styles.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-page="/Index">AI_UserInterface</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

#### styles.css

Expand "wwwroot", right-click on "css", expand "Add" and then click "New Item".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0a195af1-054f-4e20-b9ba-63d173114b69" width="600" title="Snipped January 11, 2024" />

On the "Add New Item..." pop-up, expand "C#" >> "ASP.NET Core" >> "Web" >> "Content", click "Style Sheet" on the resulting options, enter Name "styles.css" and then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1eafa8fc-37e8-45ae-8ed4-8f4a2d0364d1" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

```
.form {
    display: flex;
    text-align: left;
}

.queryInput {
    width: 90%;
}

.queryButton {
    width: 10%;
}

.table {
    text-align: left;
}

.responseHeader {
    width: 15%;
}

.responseBody {
    width: 85%;
}

.responses {
    width: 100%;
    border: 1px solid black;
}

.messages {
    width: 100%;
    border: none;
}
```






-----

:warning: RESUME HERE! :warning:


-----
-----


![image](https://github.com/richchapler/AzureSolutions/assets/44923999/cae46981-77e1-4b41-aea3-d1cab8988400)


### Styles.css
```
.form {
    display: flex;
    text-align: left;
}

.queryInput {
    width: 90%;
}

.queryButton {
    width: 10%;
}

.table {
    text-align: left;
}

.responseHeader {
    width: 15%;
}

.responseBody {
    width: 85%;
}

.responses {
    width: 100%;
    border: 1px solid black;
}

.messages {
    width: 100%;
    border: none;
}
```




### Index.cshtml
```
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" href="~/css/styles.css" />
</head>
<body>
        <div class="container">
            <form method="post" class="form">
                <input type="text" name="userPrompt" class="queryInput" placeholder="Enter your query or prompt here..." />
                <input type="submit" value="Search" class="queryButton" />
            </form>
            <table class="table">
                <tbody>
                    <tr>
                        <td class="responseHeader">
                            <h5>AI Search</h5>
                        </td>
                        <td class="responseBody">
                        <textarea id="responseAISearch" rows=5" class="responses" readonly>@ViewData["responseAISearch"]</textarea>
                    </td>
                    </tr>
                    <tr>
                        <td class="responseHeader"><h5>OpenAI</h5>(keyword)</td>
                        <td class="responseBody">
                        <textarea id="responseOpenAI_Keyword" rows="5" class="responses" readonly">@ViewData["responseOpenAI_Keyword"]</textarea>
                    </td>
                    </tr>
                    <tr>
                        <td class="responseHeader"><h5>OpenAI</h5>(semantic)</td>
                        <td class="responseBody">
                        <textarea id="responseOpenAI_Semantic" rows="5" class="responses" readonly">@ViewData["responseOpenAI_Semantic"]</textarea>
                    </td>
                    </tr>
                </tbody>
            </table>

            <textarea id="messages" rows="4" class="messages" readonly>@ViewData["messages"]</textarea>
    </div>

    @* Libraries *@
    <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js"></script>

    @* Real-Time Server to Client-Side Messaging *@
    <script>
        const connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();
        connection.on("logMessage", function (message) { document.getElementById('messages').value += message; });
        connection.start().catch(function (err) { return console.error(err.toString()); });
    </script>
</body>
</html>
```

### Index.cshtml.cs
```
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.SignalR;
using WebApplication1.Helpers;

public class IndexModel : PageModel
{
    private readonly IHubContext<ChatHub> hc;

    [BindProperty]
    public string userPrompt { get; set; }

    public IndexModel(IHubContext<ChatHub> hubContext)
    {
        hc = hubContext;
        userPrompt = string.Empty;
    }

    public async Task<IActionResult> OnPostAsync()
    {
        await hc.Clients.All.SendAsync("logMessage", $"Processing Prompt: {string.Join(" ", userPrompt.Split(' ').Take(5))+"..."}\n");
        AISearch aisearch = new();
        OpenAI openai = new();

        await hc.Clients.All.SendAsync("logMessage", "* Querying AISearch...\n");
        ViewData["responseAISearch"] = await aisearch.Prompt(query: userPrompt);

        await hc.Clients.All.SendAsync("logMessage", "* Prompting OpenAI (keyword)...\n");
        ViewData["responseOpenAI_Keyword"] = await openai.Prompt(queryType: "Simple", systemMessage: "", userMessage: userPrompt);

        await hc.Clients.All.SendAsync("logMessage", "* Prompting OpenAI (semantic)...\n");
        ViewData["responseOpenAI_Semantic"] = await openai.Prompt(queryType: "Semantic", systemMessage: "", userMessage: userPrompt);

        return Page();
    }
}
```

### Helper: ChatHub.cs
```
using Microsoft.AspNetCore.SignalR;

public class ChatHub : Hub
{
    public async Task SendSystemMessage(string message)
    {
        await Clients.All.SendAsync("logMessage", message);
    }
}
```

### Helper: Program.cs
```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddSignalR();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();

app.MapHub<ChatHub>("/chatHub"); // Map your SignalR hub

app.Run();
```