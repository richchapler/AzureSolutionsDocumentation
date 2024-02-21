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

## Documentation Note

I used to spend a lot of time explaining code blocks, but I am skipping that part of my sharing process in favor of a recommendation to copy code blocks to Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/138cf716-d18b-424f-a467-5cdeabcd4beb" width="600" title="Snipped January 9, 2024" />

On the "Create a new project" form, search for and select "ASP .NET Core Web App (Razor Pages)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/226546ce-60de-4596-b7c0-b9a81f8c1e4f" width="600" title="Snipped February 15, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f2b564a2-7a54-4db6-a07c-08347254deb0" width="600" title="Snipped February 15, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6ae370f6-60c0-46e1-87a6-65a1fb066311" width="800" title="Snipped February 15, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9e41690c-0c77-4a61-bab8-95d6cea755c4" width="800" title="Snipped February 15, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c96b86db-db7e-462f-9af9-8dc230f184a0" width="800" title="Snipped February 21, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "AzureSolutions.Helpers".

_Note: This documentation uses [AzureSolution.Helpers](https://www.nuget.org/packages/AzureSolutions.Helpers/) v1.1.1_

On the resulting pop-out, check the box next to your  and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2a032967-9fe7-4c3e-b555-46f1c0db4ea1" width="300" title="Snipped February 15, 2024" />

When prompted, click "I Accept" on the "License Acceptance" pop-up.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6c50a96f-aba6-4feb-9b60-1a736c2834aa" width="800" title="Snipped February 15, 2023" />

-----

### Step 3: Helper Classes

_Note: Though most helper logic has been moved to the AzureSolutions.Helpers package, package-specific helper logic remains in this application_

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

#### AISearch.cs

Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "KeyVault.cs" on the resulting popup and click "Add". Replace the default code with:

```
using Azure.Search.Documents.Models;
using Microsoft.AspNetCore.SignalR;
using System.Data;
using System.Text.Json;

namespace AI_Interface.Helpers
{
    public class AISearch
    {
        public static readonly string[] FieldNames = ["CustomerID", "CompanyName"];

        public static async Task Query_AISearch(
            IHubCallerClients Clients,
            SearchQueryType AISearch_QueryType,
            string UserQuery
            )
        {
            var AISearch_Query = await AzureSolutions.Helpers.AISearch.Query.Execute(
                Constants.AISearch_Client,
                AISearch_QueryType,
                AISearch_Text: UserQuery,
                AISearch_SelectField: string.Join(",", AISearch.FieldNames)
                );

            /* ************************* AISearch_Query >> Data Table */

            DataTable dataTable = new();

            List<Dictionary<string, JsonElement>>? list = AISearch_Query.Response != null ? JsonSerializer.Deserialize<List<Dictionary<string, JsonElement>>>(AISearch_Query.Response) : null;

            if (list != null && list.Count > 0)
            {
                foreach (var key in list[0].Keys)
                {
                    dataTable.Columns.Add(key, typeof(string)); // Assumes all values are strings    
                }

                foreach (var item in list)
                {
                    DataRow row = dataTable.NewRow();

                    foreach (var key in item.Keys) { row[key] = item[key].ToString(); }

                    dataTable.Rows.Add(row);
                }
            }

            /* ************************* Column Headers >> Client */

            string tableId = $"tableAISearch_{AISearch_QueryType}";

            var columns = new List<string> { "Document", "Score", "SemanticSearch" };

            await Clients.Caller.SendAsync(tableId + "_SetHeaders", columns.ToArray());

            /* ************************* Data >> Client */

            if (dataTable.Rows != null && dataTable.Rows.Count > 0)
            {
                foreach (DataRow row in dataTable.Rows)
                {
                    var dataDictionary = new Dictionary<string, string>();

                    foreach (var column in columns)
                    {
                        if (dataTable.Columns.Contains(column)) { dataDictionary[column] = row[column]?.ToString() ?? "NULL"; }
                    }

                    await Clients.Caller.SendAsync(tableId + "_AddRows", dataDictionary, AISearch_Query.SecondsToProcess);
                }
            }
        }
    }
}
```

#### Constants.cs

Repeat the process to create "Constants.cs", then replace the default code with:

```
using Azure;
using Azure.AI.OpenAI;
using Azure.Identity;
using Azure.Search.Documents;
using Azure.Security.KeyVault.Secrets;

namespace AI_Interface.Helpers
{
    public static class Constants
    {
        /* ************************* KeyVault */

        private static readonly string KeyVault_Name = "YOUR_KEYVAULT_NAME";

        private static readonly SecretClient secretClient = new(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());

        private static string GetSecret(string secretName)
        {
            var secret = secretClient.GetSecret(secretName);
            return secret.Value.Value;
        }

        /* ************************* Constants */
        public static string AISearch_Name { get; } = GetSecret("AISearch-Name");
        public static string AISearch_Key { get; } = GetSecret("AISearch-Key");
        public static string AISearch_Index_Name { get; } = GetSecret("AISearch-Index-Name");
        public static string OpenAI_Name { get; } = GetSecret("OpenAI-Name");
        public static string OpenAI_Key { get; } = GetSecret("OpenAI-Key");
        public static string OpenAI_Deployment_Name { get; } = GetSecret("OpenAI-Deployment-Name");
        public static string AISearch_SemanticConfiguration_Name { get; } = GetSecret("AISearch-SemanticConfiguration-Name");

        /* ************************* Clients */

        public static SearchClient AISearch_Client
        {
            get
            {
                return new SearchClient(
                    endpoint: new Uri($"https://{AISearch_Name}.search.windows.net"),
                    indexName: AISearch_Index_Name,
                    credential: new AzureKeyCredential(AISearch_Key)
                );
            }
        }

        public static OpenAIClient OpenAI_Client
        {
            get
            {
                return new OpenAIClient(
                    endpoint: new Uri($"https://{OpenAI_Name}.openai.azure.com/"),
                    keyCredential: new AzureKeyCredential(OpenAI_Key)
                );
            }
        }
    }
}
```

#### OpenAI.cs

Repeat the process to create "OpenAI.cs", then replace the default code with:

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
            AzureCognitiveSearchQueryType AISearch_QueryType,
            string UserQuery,
            string SystemMessage
        )
        {
            try
            {
                Stopwatch s = new(); s.Start();

                var response = await AzureSolutions.Helpers.OpenAI.Prompt(
                    Constants.OpenAI_Client,
                    Constants.OpenAI_Deployment_Name,
                    Constants.AISearch_Name,
                    Constants.AISearch_Key,
                    Constants.AISearch_Index_Name,
                    AISearch_QueryType,
                    Constants.AISearch_SemanticConfiguration_Name,
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

Right-click on the project, select "Add" >> "Class" from the resulting dropdowns, enter name "Hub.cs" on the resulting popup then click "Add". Replace the default code with:

```
using AI_Interface.Helpers;
using Azure.AI.OpenAI;
using Azure.Search.Documents.Models;
using Microsoft.AspNetCore.SignalR;

namespace AI_Interface
{
    public class Hub() : Microsoft.AspNetCore.SignalR.Hub
    {
        public async Task ProcessQuery(
            string UserQuery,
            string SystemMessage,
            bool runAISearch_Simple = true,
            bool runAISearch_Full = true,
            bool runAISearch_Semantic = true,
            bool runOpenAI_Simple = true,
            bool runOpenAI_Semantic = true
        )
        {
            try
            {
                /* ************************* AISearch */

                if (runAISearch_Simple || runAISearch_Full || runAISearch_Semantic)
                {
                    await Clients.All.SendAsync("logMessage", $"Processing User Query '{UserQuery}' with AI Search");

                    if (runAISearch_Simple) { await AISearch.Query_AISearch(Clients, SearchQueryType.Simple, UserQuery); }

                    if (runAISearch_Full) { await AISearch.Query_AISearch(Clients, SearchQueryType.Full, UserQuery); }

                    if (runAISearch_Semantic) { await AISearch.Query_AISearch(Clients, SearchQueryType.Semantic, UserQuery); }
                }

                /* ************************* OpenAI */

                if (runOpenAI_Simple || runOpenAI_Semantic)
                {
                    await Clients.All.SendAsync("logMessage", $"Processing User Query '{UserQuery}' :: System Message '{SystemMessage}' with OpenAI");

                    if (runOpenAI_Simple)
                    {
                        var OpenAI_Prompt = await Helpers.OpenAI.Query(AISearch_QueryType: AzureCognitiveSearchQueryType.Simple, UserQuery, SystemMessage);

                        await Clients.Caller.SendAsync(
                            method: $"queryOpenAI_Simple",
                            arg1: new { response = OpenAI_Prompt.Response, elapsed = OpenAI_Prompt.SecondsToProcess });
                    }

                    if (runOpenAI_Semantic)
                    {
                        var OpenAI_Prompt = await Helpers.OpenAI.Query(AISearch_QueryType: AzureCognitiveSearchQueryType.Semantic, UserQuery, SystemMessage);

                        await Clients.Caller.SendAsync(
                            method: $"queryOpenAI_Semantic",
                            arg1: new { response = OpenAI_Prompt.Response, elapsed = OpenAI_Prompt.SecondsToProcess });
                    }
                }
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

Double-click to open "Program.cs". Replace the default code with:

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
app.MapHub<AI_Interface.Hub>("/Hub");

app.Run();
```

-----

### Step 5: Front-End

#### _Layout.cshtml

Expand "Pages" >> "Shared" and double-click to open "_Layout.cshtml". Replace the default code with:

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
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>

    <script src="~/js/runfirst.js" asp-append-version="true"></script>
    <script src="~/js/constants.js" asp-append-version="true"></script>
    <script src="~/js/submit.js" asp-append-version="true"></script>
    <script src="~/js/aisearch.js" asp-append-version="true"></script>
    <script src="~/js/openai.js" asp-append-version="true"></script>

    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

-----

#### Index.cshtml

Expand "Pages" and then double-click to open "Index.cshtml". Replace the default code with:

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
        <h4>Step 1: Select Types</h4>
        <label for="toggles">
            Click to strike-through / exclude types that you do not want to include in query processing.
        </label>
        <br />

        <div id="toggles">
            <h6 id="headerAISearch" style="cursor: pointer; display: inline;">AISearch >> </h6>
            <h6 id="headerAISearch_Simple" style="cursor: pointer; display: inline;">Simple</h6> |
            <h6 id="headerAISearch_Full" style="cursor: pointer; display: inline;">Full</h6> |
            <h6 id="headerAISearch_Semantic" style="cursor: pointer; display: inline;">Semantic</h6> ::
            <h6 id="headerOpenAI" style="cursor: pointer; display: inline;">OpenAI >> </h6>
            <h6 id="headerOpenAI_Simple" style="cursor: pointer; display: inline;">Simple</h6> |
            <h6 id="headerOpenAI_Semantic" style="cursor: pointer; display: inline;">Semantic</h6>
        </div>

        <h4>Step 2: Submit Query</h4>

        <div class="row">
            <div class="col">
                <label for="inputSystemMessage">Enter System Message (if applicable)</label>
                <input id="inputSystemMessage" type="text" placeholder="Optional, used for OpenAI only... functionality pending Azure Support Case" disabled style="width: 100%;" />
            </div>
            <div class="col">
                <label for="inputUserQuery">...then, enter User Query</label>
                <input id="inputUserQuery" type="text" placeholder="Type your query phrase and then press [Enter] to submit..." style="width: 100%;" />
            </div>
            <div class="col">
                <div>
                    <label for="buttonSubmit">...finally, click Submit</label>
                </div>
                <div>
                    <button id="buttonSubmit" type="button" class="btn btn-primary">Submit</button>
                </div>
            </div>
        </div>

        <h4>Step 3: Review Results</h4>
        <ul class="nav nav-tabs" id="myTab" role="tablist">
            <li class="nav-item">
                <a class="nav-link active" id="ai-search-tab" data-toggle="tab" href="#ai-search" role="tab" aria-controls="ai-search" aria-selected="true">AI Search</a>
            </li>
            <li class="nav-item">
                <a class="nav-link" id="open-ai-tab" data-toggle="tab" href="#open-ai" role="tab" aria-controls="open-ai" aria-selected="false">Open AI</a>
            </li>
        </ul>
        <div class="tab-content" id="myTabContent">
            <div class="tab-pane fade show active" id="ai-search" role="tabpanel" aria-labelledby="ai-search-tab">
                <div class="scrollable-container">
                    <table id="tableAISearch_Simple" class="table"></table>

                    <table id="tableAISearch_Full" class="table"></table>

                    <table id="tableAISearch_Semantic" class="table"></table>
                </div>
            </div>
            <div class="tab-pane fade" id="open-ai" role="tabpanel" aria-labelledby="open-ai-tab">
                <div class="scrollable-container">
                    <table id="tableOpenAI_Simple" class="table"></table>
                    <table id="tableOpenAI_Semantic" class="table"></table>
                </div>
            </div>
        </div>

        <div class="progress-bar" id="progressBar" style="display: none;">
            <div class="progress-bar-inner"></div>
        </div>

        <h4 id="headerLog">Log</h4>
        <textarea id="textareaLog" class="textarea" rows="4" readonly>@ViewData["messages"]</textarea>
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

Expand "wwwroot" >> "css" and then double-click to open "site.css". Replace the default code with:

```
html {
    font-size: 14px;
    position: relative;
    min-height: 100%;
}

h4 {
    margin: 20px 0 10px 0;
}

.container input[type="text"] {
    width: 100%;
    height: 30px;
}

.container button {
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
}  


.scrollable-container {
    height: 250px;
    overflow: auto;
    padding: 10px;
    background-color: whitesmoke;
}  

table {
    background-color: whitesmoke;
}

.form {
    display: flex;
    text-align: left;
}

.textarea {
    width: 100% !important;
    border: 1px solid lightgray;
    display: block;
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

.data-table tr:nth-child(even) {
    background-color: ghostwhite;
}

.data-table tr:nth-child(odd) {
    background-color: white;
}

#myTab {
    margin-top: 20px;
} 

.nav-tabs .nav-link {
    border: none;
}  

.nav-tabs .nav-link.active {
    background-color: whitesmoke;
}

label {
    margin-top: 0px;
    margin-bottom: 5px;
    font-size: 15px;
}
```

-----

### Step 6: Javascript

#### site.js

Expand "wwwroot" >> "js" and then delete "site.js".

-----

#### aisearch.js

Right click on "wwwroot" >> "js" and then add new file "aisearch.js". Replace the default code with:

```
const createHeaders = (columns, row) => {
    columns.forEach(header => {
        const th = document.createElement("th");
        th.textContent = header;
        row.appendChild(th);
    });
}

const setupTable = (type) => {
    const tableId = `tableAISearch_${type}`; const table = document.getElementById(tableId);

    const RUNorNOT = `AISearch_${type}_RunOrNot`;

    table.classList.add('data-table');

    connection.on(`${tableId}_SetHeaders`, (headers) => {
        if (appState.get(RUNorNOT) === RUN) {
            if (!table.tHead) {
                const thead = table.createTHead();
                let row = thead.insertRow();
                createHeaders(headers, row);
            }
        }
    });

    connection.on(`${tableId}_AddRows`, (dataDictionary, SecondsToProcess) => {
        if (appState.get(RUNorNOT) === RUN) {
            const row = table.insertRow();
            for (const key in dataDictionary) {
                const cell = row.insertCell();
                cell.textContent = dataDictionary[key];
            }

            let label = document.getElementById(`label${tableId}`);
            if (!label) {
                label = document.createElement('label');
                label.id = `label${tableId}`;
                label.textContent = `${type} (${SecondsToProcess} seconds)`;
                table.parentNode.insertBefore(label, table);
            }
        }
    });
}

["Simple", "Full", "Semantic"].forEach(setupTable);    
```

-----

#### constants.js

Right click on "wwwroot" >> "js" and then add new file "constants.js". Replace the default code with:

```
/* ************************* Toggles */

const setupAppState = (keys, defaultValue) => {
    const state = new Map();
    keys.forEach(key => state.set(key, defaultValue));
    return state;
}

const RUNorNOTs = [
    'AISearch_Simple_RunOrNot',
    'AISearch_Full_RunOrNot',
    'AISearch_Semantic_RunOrNot',
    'OpenAI_Simple_RunOrNot',
    'OpenAI_Semantic_RunOrNot'
];

const RUN = "Run", NOT = "Not";

const appState = setupAppState(RUNorNOTs, RUN);

const getRunOrNotValues = (keys, state, runValue) => keys.map(key => state.get(key) === runValue);

/* ************************* Elements */

const inputUserQuery = document.getElementById('inputUserQuery');
const inputSystemMessage = document.getElementById('inputSystemMessage');
const buttonSubmit = document.getElementById('buttonSubmit');
const progressBar = document.getElementById('progressBar');
const logArea = document.getElementById('textareaLog');

const clearElementContent = (element) => {
    while (element.firstChild) { element.removeChild(element.firstChild); }
};

/* ************************* Initial Setup, Configuration Headers */

const initialSetup_ConfigurationHeaders = (config) => {
    const id = `header${config}`; const runOrNot = `${config}_RunOrNot`;

    const header = document.getElementById(id);

    appState.set(runOrNot, RUN);

    header.addEventListener("click", () => {
        const currentState = appState.get(runOrNot);
        appState.set(runOrNot, currentState === RUN ? NOT : RUN);
        header.style.textDecoration = appState.get(runOrNot) === RUN ? "none" : "line-through";
    });
}

const configurations = [
    "AISearch_Simple",
    "AISearch_Full",
    "AISearch_Semantic",
    "OpenAI_Simple",
    "OpenAI_Semantic"
];

configurations.forEach(initialSetup_ConfigurationHeaders);  
```

-----

#### openai.js

Right click on "wwwroot" >> "js" and then add new file "openai.js". Replace the default code with:

```
const createRow = (labelText, data) => {
    const table = document.getElementById(`tableOpenAI_${labelText}`);
    const row = table.insertRow();
    const cell = row.insertCell();
    cell.style.width = '100%';
    cell.style.border = 'none';

    const createTextareaAndLabel = (cell, labelText, data) => {
        let textarea = document.getElementById(`textareaOpenAI_${labelText}`);
        let label = document.getElementById(`labelOpenAI_${labelText}`);

        if (textarea) {
            textarea.parentNode.removeChild(textarea);
        }

        if (label) {
            label.parentNode.removeChild(label);
        }

        textarea = document.createElement('textarea');
        textarea.id = `textareaOpenAI_${labelText}`;
        textarea.rows = '3';
        textarea.className = 'textarea';
        textarea.readOnly = true;
        textarea.textContent = data.response;
        cell.appendChild(textarea);

        label = document.createElement('label');
        label.id = `labelOpenAI_${labelText}`;
        label.textContent = `${labelText} (${data.elapsed} seconds)`;
        cell.appendChild(label);
    }

    createTextareaAndLabel(cell, labelText, data);
}

connection.on("queryOpenAI_Simple", data => createRow('Simple', data));
connection.on("queryOpenAI_Semantic", data => createRow('Semantic', data));
```

-----

#### runfirst.js

Right click on "wwwroot" >> "js" and then add new file "runfirst.js". Replace the default code with:

```
/* ************************* SignalR Connection (MUST BE FIRST!) */

const connection = new signalR.HubConnectionBuilder().withUrl("/Hub").build();

/* ************************* Logging (must come before user of logMessage, etc.) */

const logMessage = (message, source = 'Client-Side') => {
    const timestamp = new Date().toLocaleTimeString();
    logArea.value += `${timestamp}, ${source}... ${message}\n`;
    logArea.scrollTop = logArea.scrollHeight;
}

connection.on("logMessage", (message) => logMessage(message, 'Server-Side'));
```

-----

#### submit.js

Right click on "wwwroot" >> "js" and then add new file "submit.js". Replace the default code with:

```
const startConnection = async () => {
    if (connection.state === signalR.HubConnectionState.Disconnected) { await connection.start(); }
};

const stopConnection = async () => {
    if (connection.state === signalR.HubConnectionState.Connected) { await connection.stop(); }
};

const handleEvent = async (e) => {
    if (e.type === 'click' || (e.type === 'keydown' && e.key === 'Enter')) {
        if (buttonSubmit.innerText === 'Submit') {
            await startConnection();
            buttonSubmit.innerText = 'Stop';
            submit();
        }
        else {
            await stopConnection();
            buttonSubmit.innerText = 'Submit';
            progressBar.style.display = 'none';
        }
    }
};

buttonSubmit.addEventListener('click', handleEvent);
inputUserQuery.addEventListener('keydown', handleEvent);

const submit = (e) => {
    const tables = document.querySelectorAll('.data-table');
    tables.forEach(clearElementContent);
    const textareas = document.querySelectorAll('.textarea'); textareas.forEach(textarea => { textarea.value = ''; });

    progressBar.style.display = 'block';
    logMessage(`Sending User Query '${inputUserQuery.value}' :: System Message '${inputSystemMessage.value}' to Hub > ProcessQuery`);

    const runOrNotValues = getRunOrNotValues(RUNorNOTs, appState, RUN);

    connection.invoke("ProcessQuery", inputUserQuery.value, inputSystemMessage.value, ...runOrNotValues)
        .then(() => {
            progressBar.style.display = 'none';
            buttonSubmit.innerText = 'Submit';
        })
        .catch((err) => logMessage(err.toString()));  
};
```

-----

### Step 7: Confirm Success

Click "Debug" >> "Start Debugging" in the menu bar.

Enter a prompt and press the Enter key on your keyboard... allow time for processing and monitor progress in the messages logged at the bottom of the interface.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a1830c0f-dd44-434e-8958-7e61c453a1cb" width="800" title="Snipped February 21, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword, full, and semantic configurations).

-----

**Congratulations... you have successfully completed all steps**

-----
