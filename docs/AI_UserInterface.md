# AI Search + OpenAI: User Interface

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7951bd4a-1806-427c-bf5f-23714f63b73a" width="1000" />

## Use Case
* "The AI Search and OpenAI demonstration apps are insufficient"
* "We want a demonstration application that allows us to query both AI Search and OpenAI at once"
* "We want to see responses from more than one configuration of both AI Search and Open AI"
* "We need an easy way to evaluate time-to-respond"
* "We need a User Interface Template that we can customize for our various internal use cases"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [**AI Search**](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration

* [**Application Service**](https://learn.microsoft.com/en-us/azure/app-service/)

* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):

  * AISearch-Index-Name
  * AISearch-Key
  * AISearch-Name
  * AISearch-SemanticConfiguration-Name
  * OpenAI-Deployment-Name
  * OpenAI-Name
  * OpenAI-Key

* [**Visual Studio**](https://visualstudio.microsoft.com/downloads/)

-----

## Exercise 1: Create Application

In this exercise we will create a web application.

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/138cf716-d18b-424f-a467-5cdeabcd4beb" width="600" title="Snipped January 9, 2024" />

On the "Create a new project**" form, search for and select "ASP .NET CoreWeb App (Razor Pages)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/226546ce-60de-4596-b7c0-b9a81f8c1e4f" width="600" title="Snipped February 15, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f2b564a2-7a54-4db6-a07c-08347254deb0" width="600" title="Snipped February 15, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6ae370f6-60c0-46e1-87a6-65a1fb066311" width="800" title="Snipped February 15, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9e41690c-0c77-4a61-bab8-95d6cea755c4" width="800" title="Snipped February 15, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/dd2f9cc3-9b68-435a-9804-0d8c6f18d94e" width="800" title="Snipped February 15, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "AzureSolutions.Helpers".

On the resulting pop-out, check the box next to your  and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2a032967-9fe7-4c3e-b555-46f1c0db4ea1" width="300" title="Snipped February 15, 2024" />

When prompted, click "I Accept" on the "License Acceptance" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6c50a96f-aba6-4feb-9b60-1a736c2834aa" width="800" title="Snipped February 15, 2023" />

-----

### Step 3: Helper Classes

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4310c7fb-41ff-4f34-b18b-e43771662e18" width="600" title="Snipped January 11, 2024" />

#### KeyVault.cs

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

#### AISearch.cs

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2df5c3a5-2498-4565-a389-ee91114e610e" width="800" title="Snipped January 24, 2024" />

Replace the default code with:

```
using Azure;
using Azure.Search.Documents;
using Azure.Search.Documents.Models;
using System.Text.Json;

namespace AI_UserInterface.Helpers
{
    public class AISearch
    {
        private readonly KeyVault keyvault;

        public AISearch()
        {
            keyvault = new KeyVault();
        }

        public async Task<string> Query(string prompt, string type)
        {
            try
            {
                SearchClient client = new(
                    endpoint: new Uri(keyvault.getSecret("AISearch-Url")),
                    indexName: keyvault.getSecret("AISearch-IndexName"),
                    credential: new AzureKeyCredential(keyvault.getSecret("AISearch-Key"))
                    );

                SearchOptions so = new();
                switch (type)
                {
                    case "Simple": so.QueryType = SearchQueryType.Simple; break;
                    case "Full": so.QueryType = SearchQueryType.Full; break;
                    case "Semantic": so.QueryType = SearchQueryType.Semantic; break;
                }

                SearchResults<SearchDocument> response = await client.SearchAsync<SearchDocument>(prompt, so);

                List<SearchResult<SearchDocument>> documents = new List<SearchResult<SearchDocument>>();

                foreach (SearchResult<SearchDocument> result in response.GetResults()) { documents.Add(result); }

                return JsonSerializer.Serialize(documents, new JsonSerializerOptions { WriteIndented = true });
            }
            catch (Exception ex)
            {
                return $"Exception: {ex.Message}";
            }
        }
    }
}
```

#### OpenAI.cs

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, and enter name "KeyVault.cs" on the resulting popup.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/efd9871f-ad02-4273-a5ee-d6b2bae39674" width="800" title="Snipped January 24, 2024" />

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

        public async Task<string> Prompt(string type, string userMessage, string systemMessage = "")
        {
            var acsqt = AzureCognitiveSearchQueryType.Simple; /* set as a default, need to evaluate whether this is desirable */

            switch (type)
            {
                case "Simple": acsqt = AzureCognitiveSearchQueryType.Simple; break;
                case "Semantic": acsqt = AzureCognitiveSearchQueryType.Semantic; break;
            }

            AzureCognitiveSearchChatExtensionConfiguration config = new()
            {
                SearchEndpoint = new Uri(keyvault.getSecret("AISearch-Url")),
                IndexName = keyvault.getSecret("AISearch-IndexName"),
                QueryType = acsqt,
                ShouldRestrictResultScope = true,
                DocumentCount = 5,
                SemanticConfiguration = keyvault.getSecret("AISearch-SemanticConfigurationName") /* Ignored when type != Semantic */
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

### Step 4: Front-End

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

-----

### Step 5: Back-End

#### SignalHub.cs

Right-click on the project, select "Add" >> "Class" from the resulting dropdowns, enter name "SignalHub.cs" on the resulting popup then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a7963763-f63b-4971-94e6-c132a2b1079c" width="800" title="Snipped January 24, 2024" />

Replace the default code with:

```
using AI_UserInterface.Helpers;
using Microsoft.AspNetCore.SignalR;
using System.Diagnostics;
using System.Text.Json;

public class SignalHub : Hub
{
    public async Task ProcessPrompt(string prompt)
    {
        try
        {
            /* ************************* Query AISearch */

            await queryAISearch("Simple", "queryAISearch_Simple", prompt);
            await queryAISearch("Full", "queryAISearch_Full", prompt);
            await queryAISearch("Semantic", "queryAISearch_Semantic", prompt);

            /* ************************* Query OpenAI */

            await queryOpenAI("Simple", "queryOpenAI_Simple", prompt);
            await queryOpenAI("Semantic", "queryOpenAI_Semantic", prompt);
        }
        catch (Exception ex)
        {
            await Clients.All.SendAsync("logMessage", $"Exception: {ex}\n");
        }
    }

    private async Task queryAISearch(string type, string method, string prompt)
    {
        Stopwatch s = new(); s.Start();

        await Clients.Caller.SendAsync(method, arg1: new { response = "Processing...", elapsed = ">0" });

        AISearch ais = new();

        var responseAISearch = await ais.Query(prompt, type);

        s.Stop();

        string response = "";
        List<Dictionary<string, JsonElement>>? list = new();

        try
        {
            list = JsonSerializer.Deserialize<List<Dictionary<string, JsonElement>>>(responseAISearch);
        }
        catch (JsonException ex)
        {
            await Clients.All.SendAsync("logMessage", $"Failed to deserialize responseAISearch: {ex.Message}. responseAISearch: {responseAISearch}");
        }

        var maxScorePerName = list
            .Select(item => new
            {
                Name = item["Document"].ToString(),
                Score = item.ContainsKey("Score") ? ConvertJsonElementToDouble(item["Score"]) : 0.0,
                RerankerScore = item.ContainsKey("SemanticSearch") && item["SemanticSearch"].ValueKind == JsonValueKind.Object && item["SemanticSearch"].GetProperty("RerankerScore").ValueKind == JsonValueKind.Number ? ConvertJsonElementToDouble(item["SemanticSearch"].GetProperty("RerankerScore")) : 0.0
            })
            .GroupBy(item => JsonSerializer.Deserialize<Dictionary<string, JsonElement>>(item.Name)["metadata_storage_name"].ToString())
            .Select(group => new
            {
                Name = group.Key,
                MaxScore = group.Max(item => item.Score),
                MaxRerankerScore = group.Max(item => item.RerankerScore)
            })
            .OrderByDescending(item => item.MaxScore)  
            .ThenByDescending(item => item.MaxRerankerScore)  
            .Take(3);   // Take the top 3  

        foreach (var item in maxScorePerName)
        {
            string name = item.Name.Length > 30 ? item.Name.Substring(0, 30) + "..." : item.Name;
            string score = item.MaxScore.ToString("F1");
            string rerankerscore = item.MaxRerankerScore.ToString("F1");
            response += $"- {name}\n  (max score: {score}, rerankerscore: {rerankerscore})\n";
        }

        await Clients.Caller.SendAsync(method, arg1: new { response = response, json = responseAISearch, elapsed = s.Elapsed.TotalSeconds.ToString("F1") }
        );
    }

    public static double ConvertJsonElementToDouble(JsonElement element)
    {
        return element.ValueKind == JsonValueKind.Number ? element.GetDouble() : (double)0;
    }

    private async Task queryOpenAI(string type, string method, string prompt)
    {
        Stopwatch s = new(); s.Start();

        OpenAI oai = new();

        await Clients.Caller.SendAsync(method, arg1: new { response = "Processing...", elapsed = "..." });
        var responseOpenAI = await oai.Prompt(type, systemMessage: "", userMessage: prompt);

        s.Stop();

        await Clients.Caller.SendAsync(
            method,
            arg1: new { response = responseOpenAI, elapsed = Math.Round(s.Elapsed.TotalSeconds, 1).ToString("F1") }
        );
    }


    public async Task LogMessage(string message)
    {
        await Clients.All.SendAsync("logMessage", message);
    }
}
```

#### Program.cs

Double-click to open "Program.cs".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac05f41a-0f85-4bdf-bcf3-6b35af4518cf" width="800" title="Snipped January 11, 2024" />

Replace the default code with:

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

-----

### Step 6: Model-View-Controller (MVC)

#### Index.cshtml

Expand "Pages" and then double-click to open "Index.cshtml".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1c54110d-a774-49dc-b84d-2590d89d694b" width="800" title="Snipped January 24, 2024" />

Replace the default code with:

```
@page "{handler?}"
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<!DOCTYPE html>
<html>
<body>
    <div class="container">
        <form method="post" class="form">
            <input type="text" class="queryInput" id="userPrompt" name="userPrompt" placeholder="Enter your query or prompt here..." />
            <button type="button" class="queryButton" onclick="processPrompt()">Search</button>
        </form>
        <table class="table">
            <tbody>
                <tr class="row">
                    <td class="responseHeader"><h5>AI Search</h5></td>
                    <td class="responseBody">
                        <table class="table" style="width:100%;">
                            <tr>
                                <td style="width: 33%;border: none;">
                                    <textarea id="textareaAISearch_Simple" class="textareas" rows="12" readonly>@ViewData["textareaAISearch_Simple"]</textarea>
                                    <label id="labelAISearch_Simple">Simple</label>
                                </td>
                                <td style="width: 33%;border: none;">
                                    <textarea id="textareaAISearch_Full" class="textareas" rows="12" readonly>@ViewData["textareaAISearch_Full"]</textarea>
                                    <label id="labelAISearch_Full">Full</label>
                                </td>
                                <td style="width: 33%;border: none;">
                                    <textarea id="textareaAISearch_Semantic" class="textareas" rows="12" readonly>@ViewData["textareaAISearch_Semantic"]</textarea>
                                    <label id="labelAISearch_Semantic">Semantic</label>
                                </td>
                            </tr>
                        </table>
                    </td>
                </tr>
                <tr class="row">
                    <td class="responseHeader"><h5>OpenAI</h5></td>
                    <td class="responseBody">
                        <table class="table" style="width:100%;">
                            <tr>
                                <td style="width: 100%;border: none;">
                                    <textarea id="textareaOpenAI_Simple" rows="5" class="textareas" readonly>@ViewData["textareaOpenAI_Simple"]</textarea>
                                    <label id="labelOpenAI_Simple">Simple</label>
                                </td>
                            </tr>
                            <tr>
                                <td style="width: 100%;border: none;">
                                    <textarea id="textareaOpenAI_Semantic" rows="5" class="textareas" readonly>@ViewData["textareaOpenAI_Semantic"]</textarea>
                                    <label id="labelOpenAI_Semantic">Semantic</label>
                                </td>
                            </tr>
                        </table>
                    </td>
                </tr>
                <tr class="row">
                    <td class="responseHeader"><h5>Log</h5></td>
                    <td class="responseBody">
                        <textarea id="messages" rows="5" class="textareas" readonly>@ViewData["messages"]</textarea>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <style>
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

        .responseBody {
            width: 85%;
            border: none;
        }

        .responseHeader {
            width: 15%;
            border: none;
        }

        .textareas {
            width: 100% !important;
            border: 1px solid black;
            display: block;
            font-family: Consolas;
            font-size: 11px;
        }

        .row {
            border-bottom: 1px solid lightgray !important;
        }

        .table {
            text-align: left;
        }
    </style>

    @* Libraries *@
    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js" "></script>

    @* Server >> Client-Side Messaging without Refresh *@
    <script>
        const connection = new signalR.HubConnectionBuilder().withUrl("/signalHub").build();

        connection.on("logMessage", function (message) { document.getElementById('messages').value += message; });

        connection.on("queryAISearch_Simple", function (data) {
            document.getElementById('textareaAISearch_Simple').value = data.response + (data.json !== undefined ? "\n-------------------------\n" + data.json : "");
            document.getElementById('labelAISearch_Simple').innerText = "Simple (" + data.elapsed + " seconds)";
        });

        connection.on("queryAISearch_Full", function (data) {
            document.getElementById('textareaAISearch_Full').value = data.response + (data.json !== undefined ? "\n-------------------------\n" + data.json : "");
            document.getElementById('labelAISearch_Full').innerText = "Full (" + data.elapsed + " seconds)";
        });

        connection.on("queryAISearch_Semantic", function (data) {
            document.getElementById('textareaAISearch_Semantic').value = data.response + (data.json !== undefined ? "\n-------------------------\n" + data.json : "");
            document.getElementById('labelAISearch_Semantic').innerText = "Semantic (" + data.elapsed + " seconds)";
        });

        connection.on("queryOpenAI_Simple", function (data) {
            document.getElementById('textareaOpenAI_Simple').value = data.response;
            document.getElementById('labelOpenAI_Simple').innerText = "Simple (" + data.elapsed + " seconds)";
        });

        connection.on("queryOpenAI_Semantic", function (data) {
            document.getElementById('textareaOpenAI_Semantic').value = data.response;
            document.getElementById('labelOpenAI_Semantic').innerText = "Semantic (" + data.elapsed + " seconds)";
        });

        connection.start()

        function processPrompt() {
            document.getElementById('textareaAISearch_Simple').value = "Pending...";
            document.getElementById('textareaAISearch_Full').value = "Pending...";
            document.getElementById('textareaAISearch_Semantic').value = "Pending...";
            document.getElementById('textareaOpenAI_Simple').value = "Pending...";
            document.getElementById('textareaOpenAI_Semantic').value = "Pending...";

            let userPrompt = document.getElementById('userPrompt').value;

            connection.invoke("ProcessPrompt", userPrompt).catch(function (err) { return console.error(err.toString()); });
        }
    </script>
</body>
</html>
```

-----

### Step 7: Confirm Success

Click "Debug" >> "Start Debugging" in the menu bar.

Enter a prompt and click "Search"... allow time for processing and monitor progress in the messages logged at the bottom of the interface.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2448cb50-ba16-4ea3-8c08-04f9c74786ef" width="800" title="Snipped January 24, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword and semantic configurations).

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Publish Application

Right-click on the project and select "Publish..." from the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6a2f5899-3475-4ee3-a35d-9a06efa8398f" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "Target" tab, confirm selection of "Azure" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1a8d84ed-49ad-44d9-9acf-0b21182299e8" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "Specific target" tab, select "Azure App Service (Windows)" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6e56c98a-d253-45b1-8b75-3e69a3c4cbfd" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "App Service" tab, click "Create a new instance".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7dc1423e-9d0f-49b9-a5f9-7f0cee728e6e" width="400" title="Snipped January 11, 2024" />

Complete the "App Service (Windows" >> "Create new" pop-up form and then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b9fb47e3-2409-42a9-b547-b8c1ae4dfc05" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "App Service" tab, confirm selection of the new App Service and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2bcd9da9-a647-4c9f-9b31-06a1c6a76677" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "Deployment type" tab, confirm selection of "Publish..." and then click "Finish".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/67301903-6b0c-430f-91ac-8876f41f3c85" width="600" title="Snipped January 11, 2024" />

On the "Publish" pop-up, "Finish" tab, confirm success and then click "Close".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a0699015-b693-4636-867e-1508263943de" width="800" title="Snipped January 11, 2024" />

On the "...Publish" tab, click the "Publish" button and monitor progress.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4de6c06e-899b-4d3a-8487-44fd763ffc9e" width="800" title="Snipped January 11, 2024" />

Once successfully published, Visual Studio will launch your new web application in a browser.

-----

**Congratulations... you have successfully completed all exercises**

-----
