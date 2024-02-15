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
  * My current configuration uses an index pointed at the Azure SQL AdventureWorks sample database

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

_Note: This documentation uses [AzureSolution.Helpers](https://www.nuget.org/packages/AzureSolutions.Helpers/) v1.1.0_

On the resulting pop-out, check the box next to your  and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2a032967-9fe7-4c3e-b555-46f1c0db4ea1" width="300" title="Snipped February 15, 2024" />

When prompted, click "I Accept" on the "License Acceptance" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6c50a96f-aba6-4feb-9b60-1a736c2834aa" width="800" title="Snipped February 15, 2023" />

-----

### Step 3: Helper Classes

_Note: Though most helper logic has been moved to the AzureSolutions.Helpers package, package-specific helper logic remains in this application_

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

#### AISearch.cs

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "KeyVault.cs" on the resulting popup and click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/562e6636-7f0f-482f-a52e-02e7f78b850c" width="600" title="Snipped February 15, 2024" />

Replace the default code with:

```
using Azure.Search.Documents.Models;
using AzureSolutions.Helpers;
using System.Diagnostics;
using System.Text.Json;
using System.Collections.Generic;
using System.Linq;

namespace AI_Interface.Helpers
{
    public class AISearch_Result
    {
        public string? Response { get; set; }
        public double SecondsToProcess { get; set; }
    }

    public class AISearch
    {
        public static async Task<AISearch_Result> Query(
            Azure.Search.Documents.SearchClient AISearch_Client,
            SearchQueryType AISearch_QueryType,
            string AISearch_Text,
            string AISearch_SelectField
            )
        {
            try
            {
                Stopwatch s = new(); s.Start();

                var response = await AzureSolutions.Helpers.AISearch.Common.Query(
                    AISearch_Client,
                    AISearch_QueryType,
                    AISearch_Text,
                    AISearch_SelectField
                    );

                s.Stop();

                return new AISearch_Result { Response = response, SecondsToProcess = Math.Round(s.Elapsed.TotalSeconds, 1) };
            }
            catch (Exception ex)
            {
                Log.Write(message: $"Exception: {ex.Message}", type: "Error");
                return new AISearch_Result { Response = $"Exception: {ex.Message}", SecondsToProcess = 0 };
            }
        }

        public static class ScoreHelper
        {
            public static IEnumerable<dynamic> CalculateScores(List<Dictionary<string, JsonElement>> list, string fieldName)
            {
                return list
                    .Select(item =>
                    {
                        item.TryGetValue("Score", out JsonElement scoreValue);
                        double score = ConvertJsonElementToDouble(scoreValue);

                        double rerankerScore = 0.0;
                        if (item.TryGetValue("SemanticSearch", out JsonElement semanticSearch) && semanticSearch.ValueKind == JsonValueKind.Object)
                        {
                            semanticSearch.TryGetProperty("RerankerScore", out JsonElement rerankerScoreValue);
                            if (rerankerScoreValue.ValueKind == JsonValueKind.Number)
                            {
                                rerankerScore = ConvertJsonElementToDouble(rerankerScoreValue);
                            }
                        }

                        return new
                        {
                            Name = item["Document"].ToString(),
                            Score = score,
                            RerankerScore = rerankerScore
                        };
                    })
                    .GroupBy(item =>
                    {
                        var itemDictionary = JsonSerializer.Deserialize<Dictionary<string, JsonElement>>(item.Name);
                        string FieldName_AISearchPivot = string.Empty;
                        if (itemDictionary != null && itemDictionary.TryGetValue(fieldName, out JsonElement storageName))
                        {
                            FieldName_AISearchPivot = storageName.ToString();
                        }
                        return FieldName_AISearchPivot;
                    })
                    .Select(group => new
                    {
                        Name = group.Key,
                        MaxScore = group.Max(item => item.Score),
                        MaxRerankerScore = group.Max(item => item.RerankerScore),
                        AvgScore = group.Average(item => item.Score),
                        AvgRerankerScore = group.Average(item => item.RerankerScore)
                    });
            }

            private static double ConvertJsonElementToDouble(JsonElement element)
            {
                return element.ValueKind == JsonValueKind.Number ? element.GetDouble() : (double)0;
            }
        }
    }
}
```

#### OpenAI.cs

Repaat the process to create "OpenAI.cs", then replace the default code with:

```
using Azure.AI.OpenAI;
using AzureSolutions.Helpers;
using System.Diagnostics;

namespace AI_Interface.Helpers
{
    public class OpenAIResult
    {
        public string? Response { get; set; }
        public double SecondsToProcess { get; set; }
    }

    public class OpenAI
    {
        public static async Task<OpenAIResult> Query(
            OpenAIClient OpenAI_Client,
            string OpenAI_Deployment_Name,
            string AISearch_Name,
            string AISearch_Key,
            string AISearch_Index_Name,
            AzureCognitiveSearchQueryType AISearch_QueryType,
            string AISearch_SemanticConfiguration_Name,
            string UserQuery,
            string SystemMessage
        )
        {
            try
            {
                Stopwatch s = new(); s.Start();

                var response = await AzureSolutions.Helpers.OpenAI.Prompt(
                    OpenAI_Client,
                    OpenAI_Deployment_Name,
                    AISearch_Name,
                    AISearch_Key,
                    AISearch_Index_Name,
                    AISearch_QueryType,
                    AISearch_SemanticConfiguration_Name,
                    UserQuery,
                    SystemMessage
                    );

                s.Stop();

                return new OpenAIResult { Response = response, SecondsToProcess = Math.Round(s.Elapsed.TotalSeconds, 1) };
            }
            catch (Exception ex)
            {
                Log.Write(message: $"Exception: {ex.Message}", type: "Error");
                return new OpenAIResult { Response = $"Exception: {ex.Message}", SecondsToProcess = 0 };
            }
        }
    }
}
```

-----

### Step 4: Back-End

#### Hub.cs

Right-click on the project, select "Add" >> "Class" from the resulting dropdowns, enter name "Hub.cs" on the resulting popup then click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/396bd000-a069-423a-912b-8002f2e45e72" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
using AI_Interface.Helpers;
using Azure;
using Azure.AI.OpenAI;
using Azure.Search.Documents;
using Azure.Search.Documents.Models;
using Azure.Security.KeyVault.Secrets;
using AzureSolutions.Helpers;
using Microsoft.AspNetCore.SignalR;
using System.Text.Json;

namespace AI_Interface
{
    public class Hub(SecretClient sc) : Microsoft.AspNetCore.SignalR.Hub
    {
        private readonly SecretClient KeyVault_Client = sc;
        private SearchClient? AISearch_Client;
        private OpenAIClient? OpenAI_Client;
        private string? AISearch_Name, AISearch_Key, AISearch_Index_Name, AISearch_SemanticConfiguration_Name;
        private string? OpenAI_Name, OpenAI_Key, OpenAI_Deployment_Name;
        private readonly string fieldName = "CompanyName"; /* the field that AISearch will use for results... "metadata_storage_name" for blob storage */

        public async Task ProcessPrompt(string prompt)
        {
            try
            {
                /* ************************* Constants */

                AISearch_Name = KeyVault.GetSecret(KeyVault_Client, "AISearch-Name");
                AISearch_Key = KeyVault.GetSecret(KeyVault_Client, "AISearch-Key");
                AISearch_Index_Name = KeyVault.GetSecret(KeyVault_Client, "AISearch-Index-Name");
                AISearch_SemanticConfiguration_Name = KeyVault.GetSecret(KeyVault_Client, "AISearch-SemanticConfiguration-Name");
                OpenAI_Name = KeyVault.GetSecret(KeyVault_Client, "OpenAI-Name");
                OpenAI_Key = KeyVault.GetSecret(KeyVault_Client, "OpenAI-Key");
                OpenAI_Deployment_Name = KeyVault.GetSecret(KeyVault_Client, "OpenAI-Deployment-Name");

                /* ************************* Clients */

                AISearch_Client = new SearchClient(
                    endpoint: new Uri($"https://{AISearch_Name}.search.windows.net"),
                    indexName: AISearch_Index_Name,
                    credential: new AzureKeyCredential(AISearch_Key)
                    );

                OpenAI_Client = new OpenAIClient(
                    endpoint: new Uri($"https://{OpenAI_Name}.openai.azure.com/"),
                    keyCredential: new AzureKeyCredential(OpenAI_Key)
                    );

                /* ************************* Queries */

                await Query_AISearch(AISearch_QueryType: SearchQueryType.Simple, prompt);

                await Query_AISearch(AISearch_QueryType: SearchQueryType.Full, prompt);

                await Query_AISearch(AISearch_QueryType: SearchQueryType.Semantic, prompt);

                await Query_OpenAI(AISearch_QueryType: AzureCognitiveSearchQueryType.Simple, UserQuery: prompt);

                await Query_OpenAI(AISearch_QueryType: AzureCognitiveSearchQueryType.Semantic, UserQuery: prompt);
            }
            catch (Exception ex) { await Clients.All.SendAsync("logMessage", $"Exception: {ex}\n"); }
        }

        private async Task Query_AISearch(SearchQueryType AISearch_QueryType, string prompt)
        {
            if (AISearch_Client == null) { throw new InvalidOperationException("AISearch_Client is null"); }

            var AISearch_Query = await AI_Interface.Helpers.AISearch.Query(
                 AISearch_Client,
                 AISearch_QueryType,
                 AISearch_Text: prompt,
                 AISearch_SelectField: fieldName
             );

            if (AISearch_Query.Response == null) { throw new InvalidOperationException("AISearch_Query.Response is null"); }

            List<Dictionary<string, JsonElement>>? list = JsonSerializer.Deserialize<List<Dictionary<string, JsonElement>>>(AISearch_Query.Response);

            if (list != null)
            {
                foreach (var item in AISearch.ScoreHelper.CalculateScores(list, fieldName))
                {
                    await Clients.Caller.SendAsync("tableAISearchResults_AddRows", new
                    {
                        column0 = AISearch_QueryType.ToString() + " (" + AISearch_Query.SecondsToProcess.ToString("F1") + " seconds)",
                        column1 = item.Name,
                        column2 = item.MaxScore.ToString("F1"),
                        column3 = item.AvgScore.ToString("F1"),
                        column4 = item.MaxRerankerScore.ToString("F1"),
                        column5 = item.AvgRerankerScore.ToString("F1")
                    });
                }
            }
            else
            { throw new InvalidOperationException("list is null"); }
        }

        private async Task Query_OpenAI(AzureCognitiveSearchQueryType AISearch_QueryType, string UserQuery)
        {
            try
            {
                string ClientSide_Script = $"queryOpenAI_{AISearch_QueryType}";

                if (OpenAI_Client == null) { throw new InvalidOperationException("OpenAI_Client is null"); }
                if (OpenAI_Deployment_Name == null) { throw new InvalidOperationException("OpenAI_Deployment_Name is null"); }
                if (AISearch_Name == null) { throw new InvalidOperationException("AISearch_Name is null"); }
                if (AISearch_Key == null) { throw new InvalidOperationException("AISearch_Key is null"); }
                if (AISearch_Index_Name == null) { throw new InvalidOperationException("AISearch_Index_Name is null"); }
                if (AISearch_SemanticConfiguration_Name == null) { throw new InvalidOperationException("AISearch_SemanticConfiguration_Name is null"); }

                var OpenAI_Prompt = await Helpers.OpenAI.Query(
                    OpenAI_Client,
                    OpenAI_Deployment_Name,
                    AISearch_Name,
                    AISearch_Key,
                    AISearch_Index_Name,
                    AISearch_QueryType,
                    AISearch_SemanticConfiguration_Name,
                    UserQuery,
                    SystemMessage: ""
                    );

                await Clients.Caller.SendAsync(method: ClientSide_Script,
                    arg1: new { response = OpenAI_Prompt.Response, elapsed = OpenAI_Prompt.SecondsToProcess } /* resolves to "data" on client-side */
                );
            }
            catch (Exception ex) { await Clients.All.SendAsync("logMessage", $"Exception: {ex}\n"); }
        }

        public async Task LogMessage(string message)
        {
            await Clients.All.SendAsync("logMessage", message);
        }
    }
}
```

#### Program.cs

Double-click to open "Program.cs".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/cffe711b-8c20-47f0-9bea-874e842838a4" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddSignalR();

/* ************************* Singletons (registered for the entire application) */

string KeyVault_Name = "{YOUR KEY VAULT NAME}";
builder.Services.AddSingleton(x =>
{
    var secretClient = new SecretClient(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());
    return secretClient;
});

/* ************************* */

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
app.MapHub<AI_Interface.Hub>("/Hub");

app.Run();
```

-----

### Step 5: Front-End

#### _Layout.cshtml

Expand "Pages" >> "Shared" and double-click to open "_Layout.cshtml".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fdb3c74c-6070-46c5-badf-703c4f517f09" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - AI_Interface</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-page="/Index">AI_Interface</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                     @*    <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-page="/Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-page="/Privacy">Privacy</a>
                        </li> *@
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

    <footer class="border-top footer text-muted">
        <div class="container">
            @* &copy; 2024 - AI_Interface - <a asp-area="" asp-page="/Privacy">Privacy</a> *@
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

-----

#### Index.cshtml

Expand "Pages" and then double-click to open "Index.cshtml".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9ac136e5-ccb1-4bc1-8f7e-84b427ee9a3e" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<!DOCTYPE html>
<html>
<body>
    <div class="container">
        <input type="text" id="userPrompt" placeholder="Type your query phrase and then press [Enter] to submit..." />
        <div class="progress-bar" id="progressBar" style="display: none;"><div class="progress-bar-inner"></div></div>
        <h5>AI Search</h5>
        <table class="table" name="tableAISearchResults"></table>
        <h5>OpenAI</h5>
        <table class="table" name="tableOpenAIResults"></table>
    </div>

    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js"></script>
</body>
</html>
```

-----

#### Index.cshtml.cs

Expand "Pages" >> "Index.cshtml" and then double-click to open "Index.cshtml.cs".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/06a19926-3585-40a7-9357-6594d49f6ece" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace AI_Interface.Pages
{
    public class IndexModel : PageModel
    {
    }
}
```

-----

#### site.css

Expand "wwwroot" >> "css" and then double-click to open "site.css".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b35281bf-a468-4253-bdb5-55a4f6db92d4" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
html {
    font-size: 14px;
}

@media (min-width: 768px) {
    html {
        font-size: 16px;
    }
}

.btn:focus, .btn:active:focus, .btn-link.nav-link:focus, .form-control:focus, .form-check-input:focus {
    box-shadow: 0 0 0 0.1rem white, 0 0 0 0.25rem #258cfb;
}

html {
    position: relative;
    min-height: 100%;
}

body {
    margin-bottom: 60px;
}

.form {
    display: flex;
    text-align: left;
}

.textarea {
    width: 100% !important;
    border: 1px solid black;
    display: block;
}
/*
.row {
    border-bottom: 1px solid lightgray !important;
}
*/
.table {
    text-align: left;
    width: 100%;
    border: none;
}

.progress-bar {
    width: 100%;
    height: 5px;
    background-color: whitesmoke;
    border-radius: 10px;
    overflow: hidden;
    padding: 1px;
}

.progress-bar-inner {
    height: 100%;
    width: 50%;
    background-color: blue;
    animation: progress 2s ease-in-out infinite;
}

@keyframes progress {
    0% {
        transform: translateX(-100%);
    }

    50% {
        transform: translateX(0);
    }

    100% {
        transform: translateX(100%);
    }
}

.container input[type="text"] {
    width: 100%;
}

h5 {
    margin-top: 20px;
}
```

-----

#### site.js

Expand "wwwroot" >> "css" and then double-click to open "site.css".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8ce8aacc-0a47-4679-a907-64c058812d3d" width="800" title="Snipped February 15, 2024" />

Replace the default code with:

```
const connection = new signalR.HubConnectionBuilder().withUrl("/Hub").build();

connection.on("logMessage", function (message) {
    document.getElementById('messages').value += message;
});

/* ************************* tableOpenAIResults */

function createRow(labelText) {
    let row = document.querySelector('table[name="tableOpenAIResults"]').insertRow();
    let cell = row.insertCell();
    cell.style.width = '100%';
    cell.style.border = 'none';

    let textarea = document.createElement('textarea');
    textarea.id = 'textareaOpenAI_' + labelText;
    textarea.rows = '5';
    textarea.className = 'textarea';
    textarea.readOnly = true;
    cell.appendChild(textarea);

    let label = document.createElement('label');
    label.id = 'labelOpenAI_' + labelText;
    label.textContent = labelText;
    cell.appendChild(label);
}

connection.on("queryOpenAI_Simple", function (data) {
    createRow('Simple');
    document.getElementById("textareaOpenAI_Simple").value = data.response;
    document.getElementById("labelOpenAI_Simple").innerText = "Simple (" + data.elapsed + " seconds)";
});

connection.on("queryOpenAI_Semantic", function (data) {
    createRow('Semantic');
    document.getElementById("textareaOpenAI_Semantic").value = data.response;
    document.getElementById("labelOpenAI_Semantic").innerText = "Semantic (" + data.elapsed + " seconds)";
    var progressBar = document.getElementById('progressBar');
    progressBar.style.display = 'none';
});

/* ************************* tableAISearchResults */

connection.on("tableAISearchResults_AddRows", function (data) {
    let table = document.querySelector('table[name="tableAISearchResults"]');

    if (!table.tHead) {
        let thead = table.createTHead();
        let row = thead.insertRow();
        let headers = ["Type", "Name", "Max Score", "Average Score", "Max Re-Ranker Score", "Average Re-Ranker Score"];
        for (let i = 0; i < headers.length; i++) {
            let th = document.createElement("th");
            let text = document.createTextNode(headers[i]);
            th.appendChild(text);
            row.appendChild(th);
        }
    }

    let row = table.insertRow();
    let cell0 = row.insertCell(); cell0.textContent = data.column0;
    let cell1 = row.insertCell(); cell1.textContent = data.column1;
    let cell2 = row.insertCell(); cell2.textContent = data.column2;
    let cell3 = row.insertCell(); cell3.textContent = data.column3;
    let cell4 = row.insertCell(); cell4.textContent = data.column4;
    let cell5 = row.insertCell(); cell5.textContent = data.column5;
});

/* ************************* EventListener... user presses [Enter] after entering query */

document.getElementById('userPrompt').addEventListener('keydown', function (e) {
    if (e.key === 'Enter') {
        e.preventDefault();
        var progressBar = document.getElementById('progressBar');
        progressBar.style.display = 'block';
        connection.invoke("ProcessPrompt", document.getElementById('userPrompt').value)
            .catch(function (err) { return console.error(err.toString()); });
    }
});

connection.start();    
```




LOREM IPSUM






-----

### Step 7: Confirm Success

Click "Debug" >> "Start Debugging" in the menu bar.

Enter a prompt and click "Search"... allow time for processing and monitor progress in the messages logged at the bottom of the interface.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2448cb50-ba16-4ea3-8c08-04f9c74786ef" width="800" title="Snipped January 24, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword and semantic configurations).

-----

**Congratulations... you have successfully completed all exercises**

-----
