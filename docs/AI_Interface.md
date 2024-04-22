# AI Search + OpenAI: Web Application

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7951bd4a-1806-427c-bf5f-23714f63b73a" width="1000" />

## Use Case
* "The AI Search and OpenAI demonstration apps are insufficient"
* "We want a demonstration application that allows us to query both AI Search and OpenAI at once"
* "We want to see responses from more than one configuration of both AI Search and Open AI"
* "We need an easy way to evaluate time-to-respond"
* "We need a User Interface Template that we can customize for our various internal use cases"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [AI Search](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration
  * The documented configuration uses an index pointed at the Azure SQL AdventureWorks sample database

* [Application Service](https://learn.microsoft.com/en-us/azure/app-service/) and Hosting Plan
  * Once created (as part of exercise), must be granted "Key Vault Secrets User" role assignment on Key Vault

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):\
  * AISearch-Index-Name
  * AISearch-Key
  * AISearch-Name
  * AISearch-SelectFields
  * AISearch-SemanticConfiguration-Name
  * OpenAI-Deployment-Name
  * OpenAI-Key
  * OpenAI-Name

* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

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

#### Constants.cs

Repeat the process to create "Constants.cs", then replace the default code with:

```csharp
using Azure;
using Azure.AI.OpenAI;
using Azure.Search.Documents;

namespace AI_Interface.Helpers
{
    public static class Constants
    {
        /* ************************* Constants */
        public static string AISearch_Index_Name { get; } = KeyVault.GetSecret("AISearch-Index-Name").Result;
        public static string AISearch_Key { get; } = KeyVault.GetSecret("AISearch-Key").Result;
        public static string AISearch_Name { get; } = KeyVault.GetSecret("AISearch-Name").Result;
        public static string AISearch_SelectFields { get; } = KeyVault.GetSecret("AISearch-SelectFields").Result;
        public static string AISearch_SemanticConfiguration_Name { get; } = KeyVault.GetSecret("AISearch-SemanticConfiguration-Name").Result;
        public static string OpenAI_Deployment_Name { get; } = KeyVault.GetSecret("OpenAI-Deployment-Name").Result;
        public static string OpenAI_Key { get; } = KeyVault.GetSecret("OpenAI-Key").Result;
        public static string OpenAI_Name { get; } = KeyVault.GetSecret("OpenAI-Name").Result;

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

#### KeyVault.cs

Repeat the process to create "KeyVault.cs", then replace the default code with:

```csharp
using Azure;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace AI_Interface.Helpers
{
    public class KeyVault
    {
        private static readonly IConfiguration _configuration;
        public static string KeyVault_Name;
        public static readonly SecretClient KeyVault_Client;

        static KeyVault()
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appSettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile("appSettings.Development.json", optional: true, reloadOnChange: true)
                .AddEnvironmentVariables();
            _configuration = builder.Build();
            KeyVault_Name = _configuration.GetSection("KeyVault_Name").Value ?? "default";
            KeyVault_Client = new SecretClient(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());
        }

        public static readonly List<string> expectedSecrets = [
            "AISearch-Index-Name",
            "AISearch-Name",
            "AISearch-Key",
            "AISearch-SelectFields",
            "AISearch-SemanticConfiguration-Name",
            "OpenAI-Deployment-Name",
            "OpenAI-Key",
            "OpenAI-Name"
        ];

        //public static readonly SecretClient KeyVault_Client = new(new Uri($"https://{KeyVault_Name}.vault.azure.net"), new DefaultAzureCredential());

        public static async Task<string> GetSecret(string secretName)
        {
            var secret = await KeyVault_Client.GetSecretAsync(secretName);
            return secret.Value.Value;
        }

        public static async Task<List<Secret>> GetSecrets()
        {
            var secrets = new List<Secret>();

            foreach (var s in expectedSecrets)
            {
                try { secrets.Add(new Secret { Name = s, Value = await GetSecret(s) }); }
                catch (RequestFailedException) { secrets.Add(new Secret { Name = s, Value = null }); }
            }

            return secrets;
        }

        public class Secret
        {
            public string? Name { get; set; }
            public string? Value { get; set; }
        }
    }
}
```

-----

### Step 4: Back-End

#### Hub.cs

Right-click on the project, select "Add" >> "Class" from the resulting dropdowns, enter name "Hub.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
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
            string Filter,
            string SystemMessage,
            float Temperature,
            Dictionary<string, bool> runFlags
        )
        {
            try
            {
                await Clients.All.SendAsync("logMessage",
                    $"Processing User Query: {UserQuery} :: Filter: {Filter} :: System Message: {SystemMessage} :: Temperature: {Temperature}\n");

                /* ************************* OpenAI */

                var openAITypes = new Dictionary<string, string>
                {
                    { "OpenAI_Semantic", "Semantic" },
                    { "OpenAI_Simple", "Simple" }
                };

                foreach (var openAIType in openAITypes)
                {
                    if (runFlags[openAIType.Key])
                    {
                        //await Clients.All.SendAsync("logMessage", $"OpenAI >> User Query '{UserQuery}' :: Filter '{Filter}' :: System Message '{SystemMessage}' :: Temperature '{Temperature}' :: {openAIType.Value}");

                        var OpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt(
                            OpenAI_Client: Constants.OpenAI_Client,
                            OpenAI_Deployment_Name: Constants.OpenAI_Deployment_Name,
                            UserQuery,
                            SystemMessage,
                            Temperature,
                            AISearch_Name: openAIType.Value == null ? null : Constants.AISearch_Name,
                            AISearch_Key: openAIType.Value == null ? null : Constants.AISearch_Key,
                            AISearch_Index_Name: openAIType.Value == null ? null : Constants.AISearch_Index_Name,
                            AISearch_QueryType: openAIType.Value == "Simple" ? AzureSearchQueryType.Simple : AzureSearchQueryType.Semantic,
                            AISearch_SemanticConfiguration_Name: openAIType.Value == null ? null : Constants.AISearch_SemanticConfiguration_Name,
                            AISearch_Filter: Filter
                            );

                        await Clients.Caller.SendAsync(
                            method: openAIType.Value == null ? "displayEvaluation" : $"displayOpenAI_{openAIType.Value}",
                            arg1: new { response = OpenAI_Prompt.Response, citations = OpenAI_Prompt.Citations, stp = OpenAI_Prompt.SecondsToProcess }
                            );
                    }
                }

                /* ************************* AISearch */

                var searchTypes = new Dictionary<string, SearchQueryType>
                {
                    { "AISearch_Semantic", SearchQueryType.Semantic },
                    { "AISearch_Full", SearchQueryType.Full },
                    { "AISearch_Simple", SearchQueryType.Simple }
                };

                foreach (var searchType in searchTypes)
                {
                    if (runFlags[searchType.Key])
                    {
                        //await Clients.All.SendAsync("logMessage", $"AI Search >> Search Text: {UserQuery} :: Filter {Filter} :: Query Type: {searchType.Value}");

                        var Query_TopX = await AzureSolutions.Helpers.AISearch.Query.TopX(
                            AISearch_Client: Constants.AISearch_Client,
                            AISearch_QueryType: searchType.Value,
                            AISearch_SearchText: UserQuery,
                            AISearch_SelectFields: Constants.AISearch_SelectFields,
                            AISearch_Filter: Filter,
                            X: 10
                            );

                        await Clients.Caller.SendAsync("displayResults", Query_TopX.Response, searchType.Value.ToString(), UserQuery);
                    }
                }

                /* ************************* Generate Prompt Ideas */

                //await Clients.All.SendAsync("logMessage", $"OpenAI >> User Query 'Suggest the single best way to improve user prompt: '{UserQuery}' to help Open AI provide a meaningful response. Suggest an alternate version of the original user prompt that the user might adopt.' :: System Message 'You are an OpenAI Prompt Engineering specialist' :: Temperature '0.5'");

                var OpenAI_Prompt2 = await AzureSolutions.Helpers.OpenAI.Prompt(
                    OpenAI_Client: Constants.OpenAI_Client,
                    OpenAI_Deployment_Name: Constants.OpenAI_Deployment_Name,
                    UserQuery: "Suggest the single best way to improve user prompt: '" + UserQuery + "' to help Open AI provide a meaningful response. Suggest an alternate version of the original user prompt that the user might adopt.",
                    SystemMessage: "You are an OpenAI Prompt Engineering specialist",
                    Temperature: 0.5f
                );

                await Clients.Caller.SendAsync(
                    method: "displayEvaluation",
                    arg1: new { response = OpenAI_Prompt2.Response, citations = OpenAI_Prompt2.Citations, stp = OpenAI_Prompt2.SecondsToProcess }
                );

                //await Clients.All.SendAsync("logMessage", $"Evaluation Result: Query sent for evaluation");
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

```csharp
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

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Azure Solutions: @ViewData["Title"]</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.10.25/css/jquery.dataTables.css" asp-append-version="true">
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
                        <li class="nav-item">
                            <a class="nav-link btn btn-outline-secondary" style="color: black; border-left: 1px solid white; border-bottom: 1px solid white; border-right: 1px solid whitesmoke; border-top: 1px solid whitesmoke;" asp-area="" asp-page="/Index">OpenAI: @AI_Interface.Helpers.Constants.OpenAI_Name | AI Search:  @AI_Interface.Helpers.Constants.AISearch_Name</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link btn btn-outline-secondary" style="color: black; border-left: 1px solid white; border-bottom: 1px solid white; border-right: 1px solid whitesmoke; border-top: 1px solid whitesmoke;" asp-area="" asp-page="/KeyVault">KeyVault: @AI_Interface.Helpers.KeyVault.KeyVault_Name</a>
                        </li>
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

    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js"></script>
    <script type="text/javascript" charset="utf8" src="https://cdn.datatables.net/1.10.25/js/jquery.dataTables.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.0/papaparse.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.5.0/jszip.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.2/FileSaver.min.js"></script>

    <script src="~/js/runfirst.js" asp-append-version="true"></script>
    <script src="~/js/constants.js" asp-append-version="true"></script>

    <script src="~/js/interface.js" asp-append-version="true"></script>
    <script>window.onload = async function () { await prepareTabs(); };</script>

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

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "AI";
}

<!DOCTYPE html>
<html>
<body>
    <partial name="_Toolbar" />

    <div class="container">

        <div class="container-style">

            @* Input: User Query *@
            <div style="position: relative;">
                <textarea id="inputUserQuery" rows="2" placeholder="Type your query phrase and then press [Enter] to submit..." style="width: 100%; padding-right: 30px;"></textarea>
                <img id="submitIcon" src="~/images/send.svg" style="position: absolute; bottom: 10px; right: 10px; width: 20px; height: 20px;" title="Send">
            </div>

            @* Input: AISearch_Filter *@
            <div class="row">
                <div class="col">
                    <label for="inputFilter">Filter</label>
                    <input id="inputFilter" type="text" placeholder="[OPTIONAL] Enter AI Search filter {e.g., ColumnX eq '12345'}" style="width: 100%;">
                </div>
            </div>

            @* Input: System Message *@
            <div class="row">
                <div class="col">
                    <label for="inputSystemMessage">System Message</label>
                    <textarea id="inputSystemMessage" rows="1" placeholder="[OPTIONAL] Used for OpenAI only" style="width: 100%;"></textarea>
                </div>
            </div>
            <script>
                document.addEventListener('DOMContentLoaded', (event) => { document.getElementById('inputSystemMessage').value = localStorage.getItem('systemMessage'); });
                /* Pull saved setting from localStorage, if possible */
            </script>

            @* Input: Temperature *@
            <div class="row">
                <div class="col">
                    <label for="inputTemperature">Temperature</label>
                    <input id="inputTemperature" type="range" min="0" max="1" step="0.01" value="0.5" style="width: 100%;">
                    <div style="display: flex; justify-content: space-between; white-space: nowrap;">
                        <span class="subtext">More Precise</span>
                        <span class="subtext">More Creative</span>
                    </div>
                </div>
            </div>
            <script>
                document.addEventListener('DOMContentLoaded', (event) => { document.getElementById('inputTemperature').value = localStorage.getItem('temperature'); });
                /* Pull saved setting from localStorage, if possible */
            </script>
        </div>

        @* Input: Result Tabs and Panels *@
        <div class="container-style">
            <ul id="resultTabs" class="nav nav-tabs" role="tablist"></ul>
            <div id="resultTabContent" class="tab-content"></div>
            <div class="progress-bar" id="progressBar" style="display: none;"><div class="progress-bar-inner"></div></div>
        </div>

        @* Log and Popup *@
        <textarea id="textareaLog" class="textarea" rows="4" style="display: none; width: 100%;" readonly>@ViewData["messages"]</textarea>
        <div id="popup" class="popup"></div>

    </div>

</body>
</html>
```

-----

#### Index.cshtml.cs

Expand "Pages" >> "Index.cshtml" and then double-click to open "Index.cshtml.cs". Replace the default code with:

```csharp
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

```css
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

```js
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

```js
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

```js
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

```js
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

```js
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

Make sure that you have a temporary value in `appsettings.json` for `KeyVault_Name`.
Click "Debug" >> "Start Debugging" in the menu bar.
Enter a prompt and press the Enter key on your keyboard... allow time for processing and monitor progress in the messages logged at the bottom of the interface.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0e7d6516-b0b4-40fc-be94-152ddb2eafd0" width="800" title="Snipped April 9, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword, full, and semantic configurations).

-----

**Congratulations... you have successfully completed all steps**

-----
