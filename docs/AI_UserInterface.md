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
  * AISearch-SelectFields
  * AISearch-SemanticConfiguration-Name
  * OpenAI-Deployment-Name
  * OpenAI-Key
  * OpenAI-Name

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

_Note: This documentation uses [AzureSolution.Helpers](https://www.nuget.org/packages/AzureSolutions.Helpers/) v1.1.3_

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

namespace AI_Interface.Helpers
{
    public class AISearch
    {
        private static readonly Dictionary<SearchQueryType, string> _responses = [];
        public static IReadOnlyDictionary<SearchQueryType, string> Responses => _responses;

        public static async Task Query_AISearch(IHubCallerClients HubClient, SearchQueryType AISearch_QueryType, string UserQuery)
        {
            await HubClient.All.SendAsync("logMessage", $"Processing User Query '{UserQuery}' :: Configuration '{AISearch_QueryType}' with AISearch");

            var AISearch_Query = await AzureSolutions.Helpers.AISearch.Query.Execute(
                Constants.AISearch_Client,
                AISearch_QueryType,
                AISearch_Text: UserQuery,
                AISearch_SelectField: Constants.AISearch_SelectFields
                );

            if (AISearch_Query.Response != null) { _responses[AISearch_QueryType] = AISearch_Query.Response; }
            await HubClient.Caller.SendAsync("displayResults", AISearch_Query.Response, AISearch_QueryType.ToString());
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

        private static readonly string KeyVault_Name = "dmsk";

        private static readonly SecretClient secretClient = new(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());

        private static string GetSecret(string secretName)
        {
            var secret = secretClient.GetSecret(secretName);
            return secret.Value.Value;
        }

        /* ************************* Constants */
        public static string AISearch_Index_Name { get; } = GetSecret("AISearch-Index-Name");
        public static string AISearch_Key { get; } = GetSecret("AISearch-Key");
        public static string AISearch_Name { get; } = GetSecret("AISearch-Name");
        public static string AISearch_SelectFields { get; } = GetSecret("AISearch-SelectFields");
        public static string AISearch_SemanticConfiguration_Name { get; } = GetSecret("AISearch-SemanticConfiguration-Name");
        public static string OpenAI_Deployment_Name { get; } = GetSecret("OpenAI-Deployment-Name");
        public static string OpenAI_Key { get; } = GetSecret("OpenAI-Key");
        public static string OpenAI_Name { get; } = GetSecret("OpenAI-Name");

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
using Microsoft.AspNetCore.SignalR;

namespace AI_Interface.Helpers
{
    public class OpenAI
    {
        public static async Task Query(
            IHubCallerClients HubClient,
            AzureCognitiveSearchQueryType AISearch_QueryType,
            string UserQuery,
            string SystemMessage
        )
        {
            await HubClient.All.SendAsync("logMessage", $"Processing User Query '{UserQuery}' :: System Message '{SystemMessage}' :: Configuration '{AISearch_QueryType}' with OpenAI");

            var OpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt(
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

            await HubClient.Caller.SendAsync(
                method: $"displayOpenAI_{AISearch_QueryType}",
                arg1: new { response = OpenAI_Prompt.Response, stp = OpenAI_Prompt.SecondsToProcess }
                );
        }
    }

    public class OpenAIResult
    {
        public string? Response { get; set; }
        public double SecondsToProcess { get; set; }
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
                /* ************************* OpenAI */

                if (runOpenAI_Simple || runOpenAI_Semantic)
                {
                    if (runOpenAI_Simple) { await Helpers.OpenAI.Query(Clients, AISearch_QueryType: AzureCognitiveSearchQueryType.Simple, UserQuery, SystemMessage); }

                    if (runOpenAI_Semantic) { await Helpers.OpenAI.Query(Clients, AISearch_QueryType: AzureCognitiveSearchQueryType.Semantic, UserQuery, SystemMessage); }
                }

                /* ************************* AISearch */

                if (runAISearch_Simple || runAISearch_Full || runAISearch_Semantic)
                {
                    await Clients.All.SendAsync("logMessage", $"Processing User Query '{UserQuery}' with AI Search");

                    if (runAISearch_Simple) { await AISearch.Query_AISearch(Clients, SearchQueryType.Simple, UserQuery); }

                    if (runAISearch_Full) { await AISearch.Query_AISearch(Clients, SearchQueryType.Full, UserQuery); }

                    if (runAISearch_Semantic) { await AISearch.Query_AISearch(Clients, SearchQueryType.Semantic, UserQuery); }
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
                <a class="navbar-brand" asp-area="" asp-page="/Index">Azure Solutions</a>
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
        <h4>Step 1: Submit Query</h4>

        <div class="row">
            <div class="col">
                <label for="inputUserQuery">User Query</label>
                <textarea id="inputUserQuery" rows="2" placeholder="Type your query phrase and then press [Enter] to submit..." style="width: 100%;"></textarea>
            </div>
        </div>
        <div class="row">
            <div class="col">
                <label for="inputSystemMessage">System Message</label>
                <textarea id="inputSystemMessage" disabled rows="2" placeholder="Optional, used for OpenAI only" style="width: 100%;"></textarea>
            </div>
        </div>
        <div id="toggles">
            <span id="headerOpenAI" style="cursor: pointer; display: inline;">OpenAI >> </span>
            <span id="headerOpenAI_Simple" style="cursor: pointer; display: inline;">Simple</span> |
            <span id="headerOpenAI_Semantic" style="cursor: pointer; display: inline;">Semantic</span> ::
            <span id="headerAISearch" style="cursor: pointer; display: inline;">AISearch >> </span>
            <span id="headerAISearch_Simple" style="cursor: pointer; display: inline;">Simple</span> |
            <span id="headerAISearch_Full" style="cursor: pointer; display: inline;">Full</span> |
            <span id="headerAISearch_Semantic" style="cursor: pointer; display: inline;">Semantic</span>
        </div>
        <div class="row">
            <div class="col">
                <div>
                    <button id="buttonSubmit" type="button" class="btn btn-primary">Submit</button>
                </div>
            </div>
        </div>
    </div>


    <h4>Step 2: Review Results</h4>

    <ul id="resultTabs" class="nav nav-tabs" role="tablist"></ul>

    <div class="tab-content" id="myTabContent">

        <div id="aisearch-simple" class="tab-pane fade">
            <div id="tabPanel_AISearch_Simple" class="scrollable-container"></div>
        </div>

        <div id="aisearch-full" class="tab-pane fade">
            <div id="tabPanel_AISearch_Full" class="scrollable-container"></div>
        </div>

        <div id="aisearch-semantic" class="tab-pane fade">
            <div id="tabPanel_AISearch_Semantic" class="scrollable-container"></div>
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

Expand "Pages" >> "Index.cshtml" and then double-click to open "Index.cshtml.cs". Replace the default code with:

```
using AI_Interface.Helpers;
using Azure.Search.Documents.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using System.Data;
using System.Text;
using System.Text.Json;

namespace AI_Interface.Pages
{
    public class IndexModel : PageModel
    {
        public IActionResult OnGetDownloadJson(SearchQueryType queryType)
        {
            return File(
                fileContents: Encoding.UTF8.GetBytes(AISearch.Responses[queryType]),
                contentType: "application/json",
                fileDownloadName: $"AISearch_{queryType}_{DateTime.Now:yyyyMMdd_HHmm}.json"
                );
        }

        public IActionResult OnGetDownloadCSV(SearchQueryType queryType)
        {
            var jsonData = AISearch.Responses[queryType];
            var document = JsonDocument.Parse(jsonData);
            if (document == null) { return BadRequest("Invalid data"); }

            DataTable dt = new();
            dt.Columns.Add("Score");
            dt.Columns.Add("RerankerScore");

            var firstElement = document.RootElement.EnumerateArray().First();
            var documentPropertiesForColumns = firstElement.GetProperty("Document").EnumerateObject();
            foreach (var prop in documentPropertiesForColumns) { dt.Columns.Add(prop.Name); }

            foreach (JsonElement element in document.RootElement.EnumerateArray())
            {
                DataRow dr = dt.NewRow();
                dr["Score"] = element.GetProperty("Score").GetDouble();

                if (element.GetProperty("SemanticSearch").TryGetProperty("RerankerScore", out JsonElement rerankerScoreElement)
                    && rerankerScoreElement.ValueKind == JsonValueKind.Number)
                {
                    dr["RerankerScore"] = rerankerScoreElement.GetDouble();
                }
                else { dr["RerankerScore"] = (double?)null; }

                var documentPropertiesForRow = element.GetProperty("Document").EnumerateObject();
                foreach (var prop in documentPropertiesForRow) { dr[prop.Name] = prop.Value.GetString() ?? string.Empty; }
                dt.Rows.Add(dr);
            }

            StringBuilder sb = new();
            var columnNames = dt.Columns.Cast<DataColumn>().Select(column => column.ColumnName);
            sb.AppendLine(string.Join(",", columnNames));

            foreach (DataRow row in dt.Rows)
            {
                var fields = row.ItemArray.Select(field => field?.ToString() ?? string.Empty);
                sb.AppendLine(string.Join(",", fields));
            }

            var csvData = sb.ToString();

            return File(
                fileContents: Encoding.UTF8.GetBytes(csvData),
                contentType: "text/csv",
                fileDownloadName: $"AISearch_{queryType}_{DateTime.Now:yyyyMMdd_HHmm}.csv"
            );
        }
    }
}
```

-----

#### site.css

Expand "wwwroot" >> "css" and then double-click to open "site.css". Replace the default code with:

```
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

.container button {
    align-items: center;
    display: flex;
    height: 30px;
    justify-content: center;
}

.container input[type="text"] {
    height: 30px;
    width: 100%;
}

.data-table {
    width: 100%;
}

    .data-table tr:nth-child(even) {
        background-color: ghostwhite;
    }

    .data-table tr:nth-child(odd) {
        background-color: white;
    }

.form {
    display: flex;
    text-align: left;
}

h4 {
    margin: 20px 0 10px 0;
}

html {
    font-size: 14px;
    min-height: 100%;
    position: relative;
}

#myTab {
    margin-top: 20px;
}

.label {
    font-size: 15px;
    margin-bottom: 5px;
    margin-top: 0px;
}

.nav-parent {
    font-size: 20px;
    margin-right: 10px;
    margin-left: 10px;
}

.nav-tabs .nav-link {
    border: none;
}

    .nav-tabs .nav-link.active {
        background-color: whitesmoke;
    }

.progress-bar {
    background-color: whitesmoke;
    border-radius: 10px;
    height: 5px;
    overflow: hidden;
    padding: 1px;
    width: 100%;
}

.progress-bar-inner {
    animation: progress 2s ease-in-out infinite;
    background-color: blue;
    height: 100%;
    width: 50%;
}

.scrollable-container {
    background-color: whitesmoke;
    height: 250px;
    overflow: auto;
    padding: 10px;
}

.table {
    background-color: whitesmoke;
}

.textarea {
    border: 1px solid lightgray;
    display: block;
    width: 100% !important;
}

.tabpanel {
    background-color: whitesmoke;
    height: 250px;
    padding: 10px;
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
connection.on("displayResults", (data, type) => { displayResults(JSON.parse(data), type); });

function displayResults(data, type) {

    //logMessage(JSON.stringify(data, null, 2));

    /* ************************* Get tabPanel */

    const tabPanel = document.getElementById(`tabPanel_AISearch_${type}`);

    /* ************************* Create table */

    const tableId = `tableAISearch_${type}`;
    const table = document.createElement('table');
    table.id = tableId;
    table.classList.add("data-table");
    tabPanel.appendChild(table);

    /* ************************* Create headers */

    const thead = table.createTHead(); let headerRow = thead.insertRow();

    let headers = ['Score', 'RerankerScore'].concat(Object.keys(data[0].Document));
    headers.forEach(header => {
        const th = document.createElement("th");
        th.textContent = header;
        headerRow.appendChild(th);
    });

    /* ************************* Add rows */

    data.forEach(item => {
        const row = table.insertRow();
        headers.forEach(header => {
            const cell = row.insertCell();
            if (header in item) { cell.textContent = item[header]; }
            else if (header in item.Document) { cell.textContent = item.Document[header]; }
            else if (header === 'RerankerScore' && item.SemanticSearch && 'RerankerScore' in item.SemanticSearch) { cell.textContent = item.SemanticSearch.RerankerScore; }
        });
    });

    /* ************************* Create "JSON Download" and "CSV Download" links */

    const downloadLinkJson = document.createElement('a');
    downloadLinkJson.href = `${window.location.origin}/?queryType=${type.toLowerCase()}&handler=DownloadJson&file=${type.toLowerCase()}.json`;
    downloadLinkJson.textContent = 'JSON';

    const downloadLinkCsv = document.createElement('a');
    downloadLinkCsv.href = `${window.location.origin}/?queryType=${type.toLowerCase()}&handler=DownloadCSV&file=${type.toLowerCase()}.csv`;
    downloadLinkCsv.textContent = 'CSV';

    const downloadText = document.createTextNode('Download ');
    const spaceText = document.createTextNode(' ');
    const lineBreak = document.createElement('br');

    tabPanel.appendChild(lineBreak);
    tabPanel.appendChild(downloadText);
    tabPanel.appendChild(downloadLinkJson);
    tabPanel.appendChild(spaceText);
    tabPanel.appendChild(downloadLinkCsv);  
}  
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
const buildTabPanel = (type, data) => {

    /* ************************* Get and empty tabpanel */
    const tabPanel = document.getElementById(`tabPanel_OpenAI_${type}`);
    tabPanel.innerHTML = '';

    /* ************************* Create label */

    label = document.createElement('label');
    label.id = `labelOpenAI_${type}`;
    label.textContent = `Time-to-Process: ${data.stp.toFixed(1)} seconds`;
    tabPanel.appendChild(label);

    /* ************************* Create textarea */

    textarea = document.createElement('textarea');
    textarea.id = `textareaOpenAI_${type}`;
    textarea.rows = '9';
    textarea.className = 'textarea';
    textarea.readOnly = true;
    textarea.textContent = data.response;
    tabPanel.appendChild(textarea);
}

["Simple", "Semantic"].forEach(type => { connection.on(`displayOpenAI_${type}`, data => buildTabPanel(type, data)); });  

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
    message = message.replace(/[\r\n]+/g, ' '); // replace carriage returns and newlines with a space  
    logArea.value += `${timestamp}, ${source}... ${message}\n`;
    logArea.scrollTop = logArea.scrollHeight;
}  

connection.on("logMessage", (message) => logMessage(message, 'Server-Side'));
```

-----

#### submit.js

Right click on "wwwroot" >> "js" and then add new file "submit.js". Replace the default code with:

```
/* ************************* Handle Events from listeners (below): 1) Submit button clicked, or 2) Enter key pressed */

const HandleEvent = async (e) => {

    if (e.type === 'click' || (e.type === 'keydown' && e.key === 'Enter')) {

        if (buttonSubmit.innerText === 'Submit') {

            if (connection.state === signalR.HubConnectionState.Disconnected) { await connection.start(); }

            buttonSubmit.innerText = 'Stop';

            await prepareTabs();

            await prepareTabPanels();

            await processQuery();
        }
        else {

            if (connection.state === signalR.HubConnectionState.Connected) { await connection.stop(); }

            buttonSubmit.innerText = 'Submit';

            progressBar.style.display = 'none';
        }
    }
};

/* ***** Event Listeners... MUST follow HandleEvent */

buttonSubmit.addEventListener('click', HandleEvent);
inputUserQuery.addEventListener('keydown', HandleEvent);

/* ************************* Prepare Tabs */

const prepareTabs = async () => {

    var configurations = [
        { parent: "OpenAI", children: ["Simple", "Semantic"] },
        { parent: "AISearch", children: ["Simple", "Full", "Semantic"] }
    ];

    var tabList = $("#resultTabs");
    tabList.empty();

    configurations.forEach(function (config, index) {

        var parentTab = $('<li class="nav-parent">' + config.parent + '</li>');

        tabList.append(parentTab);

        config.children.forEach(function (child) {

            var childTab = $(`<li class="nav-item ${config.parent}-subtab"><a class="nav-link" id="${config.parent}-${child}-tab"
                data-toggle="tab" href="#${config.parent.toLowerCase()}-${child.toLowerCase()}">  
                ${child}</a></li>`);
            /* the specific data-toggle, href and lowercase appear to be necessary for Bootstrap tab functionality */

            tabList.append(childTab);
        });
    });
};

/* ************************* Prepare Tab Panels */

const prepareTabPanels = async () => {

    var configurations = [
        { parent: "OpenAI", children: ["Simple", "Semantic"] },
        { parent: "AISearch", children: ["Simple", "Full", "Semantic"] }
    ];

    var tabContent = $("#myTabContent");
    configurations.forEach(function (config, index) {
        config.children.forEach(function (child) {

            if (config.parent === "OpenAI") {

                var childPane = $(
                    `<div id="${config.parent.toLowerCase()}-${child.toLowerCase()}" class="tab-pane fade">  
                        <div id="tabPanel_${config.parent}_${child}" class="scrollable-container"></div></div>`
                );
                /* first div: the specific id, lowercase, and class appear to be necessary for Bootstrap tab functionality */
                /* second div: this is the container actually used to display results */

                tabContent.append(childPane);
            }
        });
    });
};


/* ************************* Process Query */

const processQuery = async (e) => {

    logMessage(`Sending User Query '${inputUserQuery.value}' :: System Message '${inputSystemMessage.value}' to Hub > ProcessQuery`);

    document.querySelectorAll('.data-table').forEach(clearElementContent); /* Clear previous results from tables */

    document.querySelectorAll('.textarea').forEach(textarea => { textarea.value = ''; }); /* Clear previous results from text areas */

    progressBar.style.display = 'block'; /* Reset progress bar */

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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac873cde-1ecd-4a05-b36a-32508e73f210" width="800" title="Snipped February 26, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword, full, and semantic configurations).

-----

**Congratulations... you have successfully completed all steps**

-----
