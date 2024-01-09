# AI Search + OpenAI: User Interface
:warning: WORK-IN-PROGRESS :warning:

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7951bd4a-1806-427c-bf5f-23714f63b73a" width="1000" />

## Use Case
* "The AI Search and OpenAI demonstration apps are insufficient"
* "We would like to build a web application that allows us to query both AI Search and OpenAI (various configurations) at once"
* "We need a User Interface Template that we can customize for our various internal use cases"

## Proposed Solution
* **Code Application**: lorem ipsum
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
-----

:warning: RESUME HERE! :warning:

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/cae46981-77e1-4b41-aea3-d1cab8988400)

### _Layout.cshtml.css
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - WebApplication1</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/WebApplication1.styles.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-page="/Index">WebApplication1</a>
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

### Helper: AISearch.cs
```
using Azure;
using Azure.Search.Documents;
using Azure.Search.Documents.Models;
using System.Text.Json;

namespace WebApplication1.Helpers
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

### Helper: KeyVault.cs
```
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace WebApplication1.Helpers
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

### Helper: OpenAI.cs
```
using Azure;
using Azure.AI.OpenAI;

namespace WebApplication1.Helpers
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
                IndexName = keyvault.getSecret("AISearch-Name")+"-index",
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
