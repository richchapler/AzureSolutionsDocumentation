# AI: Translator

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3f04c676-a006-4fbf-b2d2-52b01f7e64c1" width="1000" />

## Use Case
* "We operate in multiple countries and languages"
* "We have built a custom model supporting translation of company- and product-specific terms"
* "We want a web application that will support translation of input and documents using our custom model"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * Storage-ConnectionString
  * Storage-Endpoint
  * Translator-Key
  * DocumentTranslation-Endpoint
  * TextTranslation-Endpoint

* [Storage Account](Infrastructure_StorageAccount.md) with two containers, "source" and "target"

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d4291e30-134c-46e3-98a9-884a1fc7d1fb" width="600" title="Snipped July 12, 2024" />

On the "Create a new project" form, search for / select "ASP.NET Core Web App (Model-View-Controller)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/53119690-fac6-46d6-b9bc-17730c0c8f30" width="600" title="Snipped July 12, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1f2472fb-31af-4d91-9d52-220545aae624" width="600" title="Snipped July 12, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e67cdb10-d6de-4300-bec0-cd5fb6a7d8ab" width="800" title="Snipped July 12, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/79700c52-8221-4cc1-9d3c-259ec9973b56" width="800" title="Snipped July 12, 2024" />

Click **Tools** in the menu bar, expand "NuGet Package Manager" in the resulting menu and then click "Manage NuGet Packages for Solution...".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/38e6bc7d-2c91-4cb3-a4e7-5ab033294e8c" width="800" title="Snipped July 12, 2024" />

On the **Browse** tab of the "**NuGet - Solution**" page, search for and select "**AzureSolutions.Identity**".
<br>On the resulting pop-out, check the box next to your project and then click "**Install**".
<br>When prompted, click "**I Accept**" on the "**License Acceptance**" pop-up.
<br>When complete, close the "**NuGet - Solution**" tab.

Repeat this process for the following NuGet packages:
* Azure.AI.Translation.Document
* Azure.AI.Translation.Text
* Azure.Security.KeyVault.Secrets
* Azure.Storage.Blobs
* Newtonsoft.Json

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6497d1e8-b840-47c4-9299-5c0281e35c42" width="800" title="Snipped July 12, 2024" />

-----

### Step 3: Helper Classes

Right-click on the project, select "Add" >> "New folder" from the resulting dropdown, and enter name "Helpers".

#### Configuration.cs
Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "Config.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using Azure;
using Azure.AI.Translation.Document;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Storage.Blobs;

namespace AI_Translator.Helpers
{
    public static class Configuration
    {
        static readonly SecretClient KeyVault_Client = new(
                vaultUri: new Uri("https://hmckeyvault.vault.azure.net/"),
                credential: new DefaultAzureCredential(new DefaultAzureCredentialOptions()
                )
            );
        public static string Storage_Endpoint { get; private set; }
        public static string Storage_ConnectionString { get; private set; }
        public static BlobServiceClient Storage_Client { get; private set; }

        public static string Translator_Key { get; private set; }
        public static string DocumentTranslation_Endpoint { get; private set; }
        public static DocumentTranslationClient DocumentTranslation_Client { get; private set; }

        public static string TextTranslation_Endpoint { get; private set; }

        static Configuration()
        {
            try
            {
                Storage_Endpoint = GetSecret("Storage-Endpoint").Result;
                Storage_ConnectionString = GetSecret("Storage-ConnectionString").Result;
                Storage_Client = new BlobServiceClient(Storage_ConnectionString);

                Translator_Key = GetSecret("Translator-Key").Result;

                DocumentTranslation_Endpoint = GetSecret("DocumentTranslation-Endpoint").Result;
                DocumentTranslation_Client = new DocumentTranslationClient(
                   endpoint: new Uri(DocumentTranslation_Endpoint),
                   credential: new AzureKeyCredential(Translator_Key)
                    );

                TextTranslation_Endpoint = GetSecret("TextTranslation-Endpoint").Result;
            }
            catch (Exception ex) { throw new Exception($"Exception: {ex}", ex); }
        }

        public async static Task<string> GetSecret(string secretName)
        {
            try
            {
                Response<KeyVaultSecret> secret = await KeyVault_Client.GetSecretAsync(secretName);
                return secret.Value.Value;
            }
            catch (Exception ex) { throw new Exception($"Failed to get secret '{secretName}' from Key Vault: {ex.Message}", ex); }
        }
    }
}
```

#### Storage.cs
Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "Storage.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Sas;

namespace AI_Translator.Helpers
{
    public class Storage
    {
        public Task<string> GenerateSasToken(BlobContainerClient bcc, string containerName)
        {
            var sasBuilder_Source = new BlobSasBuilder()
            {
                BlobContainerName = containerName,
                Resource = "c",
                ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
            };

            sasBuilder_Source.SetPermissions(BlobContainerSasPermissions.All);

            return Task.FromResult(bcc.GenerateSasUri(sasBuilder_Source).Query[1..]);
        }
    }
}
```

#### Translate.cs
Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "Translate.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using Azure.AI.Translation.Document;
using Azure.Storage.Blobs;
using Microsoft.AspNetCore.SignalR;
using Newtonsoft.Json;
using System.Text;

namespace AI_Translator.Helpers
{
    public class Translate
    {
        private readonly IHubContext<LogHub> _logger;
        private readonly string containerName = "target";
        private readonly BlobContainerClient bcc;

        public Translate(IHubContext<LogHub> hubContext)
        {
            _logger = hubContext;
            bcc = Configuration.Storage_Client.GetBlobContainerClient(containerName);
        }

        public async Task<string> File(IFormFile file, string url, string sourceLanguage, string targetLanguage)
        {
            string? urlTarget = null;
            DocumentTranslationOperation? operation = null;

            try
            {
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translating {file.FileName}");

                if (!bcc.CanGenerateSasUri) { throw new Exception($"{bcc.Name} can't generate a SAS token"); }
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"SAS token generation checked");

                string sasToken = await new Storage().GenerateSasToken(bcc, containerName);
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"SAS token generated");

                string fileNameWithoutExtension = Path.GetFileNameWithoutExtension(file.FileName).Replace(" ", "_");
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"File name without extension: {fileNameWithoutExtension}");

                string extension = Path.GetExtension(file.FileName);
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"File extension: {extension}");

                string timestamp = DateTime.Now.ToString("yyyyMMdd-HHmm");
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Timestamp: {timestamp}");

                urlTarget = new Uri($"{Configuration.Storage_Endpoint}{containerName}/{fileNameWithoutExtension}_{timestamp}_{targetLanguage}{extension}?{sasToken}").ToString();
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Generated target URL");

                var input = new DocumentTranslationInput(
                    source: new TranslationSource(new Uri(url)) { LanguageCode = sourceLanguage },
                    targets: [new(new Uri(urlTarget), targetLanguage) { CategoryId = "c941a43f-8744-4408-b210-8f2f2bdb62ea-GENERAL" }]  /* CategoryId references Custom Model */
                )
                { StorageUriKind = StorageInputUriKind.File };
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"DocumentTranslationInput created with source URL and target URL");

                operation = await Configuration.DocumentTranslation_Client.StartTranslationAsync(input);
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translation operation started");

                while (!operation.HasCompleted)
                {
                    await Task.Delay(1000); // Wait for 1 second  
                    await operation.UpdateStatusAsync();

                    await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translation operation status: {operation.Status}");
                }
            }
            catch (Exception ex) { await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translate.cs Exception: {ex}"); }

            if (operation?.Status == DocumentTranslationStatus.Succeeded && urlTarget is not null) { return urlTarget; }
            else
            {
                if (operation != null)
                {
                    await foreach (var document in operation.GetValuesAsync())
                    {
                        if (document.Status == DocumentTranslationStatus.Failed)
                        { await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translation Failed: {document.Error.Message}"); }
                    }
                }
                return "Translation Failed";
            }
        }

        public async Task<string> Input(string textToTranslate, string sourceLanguage, string targetLanguage)
        {
            try
            {
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translating from {sourceLanguage} to {targetLanguage}");

                var requestBody = JsonConvert.SerializeObject(new object[] { new { Text = textToTranslate } });

                using var client = new HttpClient();
                using var request = new HttpRequestMessage();
                request.Method = HttpMethod.Post;

                request.RequestUri = new Uri($"{Configuration.TextTranslation_Endpoint}/translate?api-version=3.0&from={sourceLanguage}&to={targetLanguage}");

                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");

                request.Headers.Add("Ocp-Apim-Subscription-Key", Configuration.Translator_Key);
                request.Headers.Add("Ocp-Apim-Subscription-Region", "westus");

                HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);

                string result = await response.Content.ReadAsStringAsync();

                var translation = JsonConvert.DeserializeObject<dynamic>(result);

                if (translation is not null) return translation[0].translations[0].text; else return "Translation Failed";
            }

            catch (Exception ex) { await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translate.cs Exception: {ex}"); }

            return "Translation Failed";
        }
    }
}
```

#### Upload.cs
Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "Upload.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Microsoft.AspNetCore.SignalR;

namespace AI_Translator.Helpers
{
    public class Upload
    {
        public readonly IHubContext<LogHub> _logger;
        private readonly string containerName = "source";
        private readonly BlobContainerClient bcc;

        public Upload(IHubContext<LogHub> hubContext)
        {
            _logger = hubContext;
            bcc = Configuration.Storage_Client.GetBlobContainerClient(containerName);
            if (!bcc.CanGenerateSasUri) { _logger.Clients.All.SendAsync("ReceiveMessage", $"{bcc.Name} cannot generate a SAS token").Wait(); }
        }

        public async Task<string> File(IFormFile file)
        {
            try
            {
                await _logger.Clients.All.SendAsync("ReceiveMessage", $"Uploading {file.FileName}");
            
            string sasToken = await new Storage().GenerateSasToken(bcc, containerName);

            var urlSource = new Uri($"{Configuration.Storage_Endpoint}{containerName}/{file.FileName}?{sasToken}");

            var blob = Configuration.Storage_Client.GetBlobContainerClient(containerName).GetBlobClient(file.FileName);

            await blob.DeleteIfExistsAsync(DeleteSnapshotsOption.IncludeSnapshots);

                var lastReportedTime = DateTime.Now;
                var lastReportedProgress = 0.0;

                var uploadOptions = new BlobUploadOptions
                {
                    ProgressHandler = new Progress<long>(bytesUploaded =>
                    {
                        var percentUploaded = Math.Round((double)bytesUploaded / file.Length * 100, 1);
                        if ((DateTime.Now - lastReportedTime).TotalSeconds >= 1 && percentUploaded != lastReportedProgress)
                        {
                            lastReportedTime = DateTime.Now;
                            lastReportedProgress = percentUploaded;
                            _logger.Clients.All.SendAsync("ReceiveMessage", $"Upload operation status: {percentUploaded:F1}%").Wait();
                        }
                    })
                };

                await blob.UploadAsync(file.OpenReadStream(), uploadOptions, cancellationToken: CancellationToken.None);

                return urlSource.ToString();
            }
            catch (Exception ex)
            {
                throw new Exception($"Exception: {ex}", ex);
            }
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
using Microsoft.AspNetCore.SignalR;

namespace AI_Interface
{
    public class Hub : Microsoft.AspNetCore.SignalR.Hub
    {
        public async Task ProcessQuery(string UserQuery, string SystemMessage, float Temperature, string Filter, Dictionary<string, bool> runFlags)
        {
            try
            {
                await Clients.All.SendAsync("logMessage",
                    $"Processing User Query: {UserQuery} :: Filter: {Filter} :: System Message: {SystemMessage} :: Temperature: {Temperature}\n");

                /* ************************* OpenAI and AISearch */

                Task vectorsemanticTask = Task.CompletedTask, vectorsimpleTask = Task.CompletedTask, vectorTask = Task.CompletedTask, semanticTask = Task.CompletedTask, simpleTask = Task.CompletedTask;

                if (runFlags["OpenAI_VectorSemantic"])
                {
                    vectorsemanticTask = Task.Run(async () =>
                    {
                        var r = await AzureSolutions.Helpers.OpenAI.Prompt.VectorSemantic(Config.oaic, Config.aisc, UserQuery: UserQuery, SystemMessage: SystemMessage, Temperature: Temperature, AISearch_Filter: Filter);

                        await Clients.Caller.SendAsync(
                            method: $"displayOpenAI_{"VectorSemantic"}",
                            arg1: new { response = r.Response, citations = r.Citations, stp = r.SecondsToProcess }
                        );
                    });
                }

                if (runFlags["OpenAI_VectorSimple"])
                {
                    vectorsimpleTask = Task.Run(async () =>
                    {
                        var r = await AzureSolutions.Helpers.OpenAI.Prompt.VectorSimple(Config.oaic, Config.aisc, UserQuery: UserQuery, SystemMessage: SystemMessage, Temperature: Temperature, AISearch_Filter: Filter);

                        await Clients.Caller.SendAsync(
                            method: $"displayOpenAI_{"VectorSimple"}",
                            arg1: new { response = r.Response, citations = r.Citations, stp = r.SecondsToProcess }
                        );
                    });
                }

                if (runFlags["OpenAI_Vector"])
                {
                    vectorTask = Task.Run(async () =>
                    {
                        var r = await AzureSolutions.Helpers.OpenAI.Prompt.Vector(Config.oaic, Config.aisc, UserQuery: UserQuery, SystemMessage: SystemMessage, Temperature: Temperature, AISearch_Filter: Filter);

                        await Clients.Caller.SendAsync(
                            method: $"displayOpenAI_{"Vector"}",
                            arg1: new { response = r.Response, citations = r.Citations, stp = r.SecondsToProcess }
                        );
                    });
                }

                if (runFlags["OpenAI_Semantic"])
                {
                    semanticTask = Task.Run(async () =>
                    {
                        var r = await AzureSolutions.Helpers.OpenAI.Prompt.Semantic(Config.oaic, Config.aisc, UserQuery: UserQuery, SystemMessage: SystemMessage, Temperature: Temperature, AISearch_Filter: Filter);

                        await Clients.Caller.SendAsync(method: $"displayOpenAI_{"Semantic"}", arg1: new { response = r.Response, citations = r.Citations, stp = r.SecondsToProcess }
                        );
                    });
                }

                if (runFlags["OpenAI_Simple"])
                {
                    simpleTask = Task.Run(async () =>
                    {
                        await Clients.All.SendAsync("logMessage", $"Processing Simple\n");
                        
                        var r = await AzureSolutions.Helpers.OpenAI.Prompt.Simple(Config.oaic, Config.aisc, UserQuery: UserQuery, SystemMessage: SystemMessage, Temperature: Temperature, AISearch_Filter: Filter);

                        await Clients.All.SendAsync("logMessage", $"Simple: {r.Response}\n");

                        await Clients.Caller.SendAsync(
                            method: $"displayOpenAI_{"Simple"}",
                            arg1: new { response = r.Response, citations = r.Citations, stp = r.SecondsToProcess }
                        );
                    });
                }

                await Task.WhenAll(vectorsemanticTask, vectorsimpleTask, vectorTask, semanticTask, simpleTask);

                foreach (var expression in Config.Expressions)
                {
                    if (runFlags[expression.Key])
                    {
                        if (expression.Value.Item1 == "AISearch")
                        {
                            var AISearch_QueryResult =
                                expression.Value.Item2 == "Vector" ? await AzureSolutions.Helpers.AISearch.Query.Vector(Config.oaic, Config.aisc, AISearch_SearchText: UserQuery, AISearch_Filter: Filter, X: 10)

                                : expression.Value.Item2 == "Semantic" ? await AzureSolutions.Helpers.AISearch.Query.Semantic(Config.aisc, AISearch_SearchText: UserQuery, AISearch_Filter: Filter, X: 10)

                                : expression.Value.Item2 == "Simple" ? await AzureSolutions.Helpers.AISearch.Query.Simple(Config.aisc, AISearch_SearchText: UserQuery, AISearch_Filter: Filter, X: 10)

                                : null;

                            if (AISearch_QueryResult != null)
                            {
                                await Clients.All.SendAsync("logMessage", $"{expression.Value.Item2}: {AISearch_QueryResult.Response}\n");

                                await Clients.Caller.SendAsync("displayResults", AISearch_QueryResult.Response, expression.Value.Item2, UserQuery);
                            }
                        }
                    }
                }

                /* ************************* Generate Prompt Ideas */

                var OpenAI_PromptEvaluation = await AzureSolutions.Helpers.OpenAI.Prompt.NoIndex(Config.oaic,
                    UserQuery: "Suggest the single best way to improve user prompt: '" + UserQuery + "' to help Open AI provide a meaningful response. Suggest an alternate version of the original user prompt that the user might adopt.",
                    SystemMessage: "You are an OpenAI Prompt Engineering specialist",
                    Temperature: 0.5f
                );

                await Clients.Caller.SendAsync(
                    method: "displayEvaluation",
                    arg1: new
                    {
                        response = OpenAI_PromptEvaluation.Response,
                        citations = OpenAI_PromptEvaluation.Citations,
                        stp = OpenAI_PromptEvaluation.SecondsToProcess
                    }
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
                            <a class="nav-link btn btn-outline-secondary" style="color: black; border-left: 1px solid white; border-bottom: 1px solid white; border-right: 1px solid whitesmoke; border-top: 1px solid whitesmoke;" asp-area="" asp-page="/Index">OpenAI: @AI_Interface.Helpers.Config.oaic.OpenAI_Name | AI Search:  @AI_Interface.Helpers.Config.aisc.AISearch_Name</a>
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
            <script>@page
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

.container-style {
    border: 1px solid silver;
    box-shadow: 5px 5px 0px whitesmoke;
    padding: 10px;
    margin: 10px 0;
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

.fake-textarea {
    width: 100%;
    height: 90%;
    border: 1px solid #ccc;
    padding: 10px;
    overflow: auto;
    white-space: pre-wrap;
    word-wrap: break-word;
    background-color: white;
}

    .fake-textarea .highlight sup {
        border: 1px solid lightblue;
        background-color: aliceblue;
        padding-top: 5px;
        padding-right: 3px;
        padding-bottom: 6px;
        padding-left: 2px;
        display: inline-block;
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

.popup {
    display: none;
    position: fixed;
    bottom: 20px;
    right: 20px;
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
    height: 275px;
    overflow: auto;
    padding: 10px;
}

.subtext {
    font-size: 0.8em;
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

.toolbar-button {
    display: inline-block;
    vertical-align: middle;
    border: none;
    margin-right: 2px;
}

    .toolbar-button:hover {
        background-color: #f8f9fa;
    }

.toolbar-icon {
    width: 20px;
    height: 20px;
    margin-right: 5px;
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
connection.on("displayResults", (data, type, userQuery) => { displayResults(JSON.parse(data), type, userQuery); });

async function displayResults(data, type, userQuery) {

    logMessage(JSON.stringify(data, null, 2));    

    const tabPanel = document.getElementById(`tabPanel_AISearch_${type}`);

    async function createTableAndInitialize() {

        /* ************************* Create table */

        const tableId = `tableAISearch_${type}`;
        const table = document.createElement('table');
        table.id = tableId;
        table.classList.add("data-table");

        /* ************************* Create headers */

        const thead = table.createTHead();
        const headerRow = thead.insertRow();
        const headers = [...Object.keys(data[0])];  
        headers.map(header => {
            const th = document.createElement('th');
            th.textContent = header;
            return headerRow.appendChild(th);
        });

        /* ************************* Create tbody */

        const tbody = document.createElement('tbody');
        table.appendChild(tbody);

        /* ************************* Add rows */

        for (const item of data) {
            const row = tbody.insertRow();
            for (const header of headers) {
                const cell = row.insertCell();
                if (header in item) { cell.textContent = item[header]; }
            }  
        }

        tabPanel.insertAdjacentElement('beforeend', table);
        return tableId;
    }

    const tableId = await createTableAndInitialize();

    /* ************************* Initialize DataTable */
    $(function () {
        $(`#${tableId}`).DataTable({
            pageLength: 3,
            columnDefs: [{
                targets: '_all',
                render: function (data, type, row) {
                    if (type === 'display' && data.length > 25) {
                        const span = document.createElement('span');
                        span.title = data;
                        span.textContent = `${data.substr(0, 22)}...`;
                        return span.outerHTML;
                    } else { return data; }
                }
            }],
            autoWidth: true
        });
    });
}
```

-----

#### constants.js

Right click on "wwwroot" >> "js" and then add new file "constants.js". Replace the default code with:

```js
const ProcessingStart_Constants = new Date();  

/* ************************* UI Elements */

const buttonClear = document.getElementById('buttonClear');
const buttonDownload = document.getElementById('buttonDownload');
const buttonLog = document.getElementById('buttonLog');
const buttonSave = document.getElementById('buttonSave');

const inputUserQuery = document.getElementById('inputUserQuery');
const inputSystemMessage = document.getElementById('inputSystemMessage');
const inputTemperature = document.getElementById('inputTemperature');

const progressBar = document.getElementById('progressBar');
const textareaLog = document.getElementById('textareaLog');

const clearElementContent = (element) => { while (element.firstChild) { element.removeChild(element.firstChild); } };

logMessage(`constants.js processed in ${((new Date() - ProcessingStart_Constants) / 1000).toFixed(3)}s`);  
```

-----

#### interface.js

Right click on "wwwroot" >> "js" and then add new file "interface.js". Replace the default code with:

```js
const prepareTabs = async () => {

    /* Start seconds-to-process timer */
    const ProcessingStart_Interface = new Date();

    /* ************************* Get (and empty) index.cshtml containers */
    const resultTabs = $("#resultTabs"), resultTabContent = $("#resultTabContent");
    resultTabs.empty();

    /* ************************* Define tab configurations */
    const configurations = [
        { parent: "OpenAI", children: ["VectorSemantic", "VectorSimple", "Vector", "Semantic", "Simple"] },
        { parent: "AISearch", children: ["Vector", "Semantic", "Simple"] },
        { parent: "Ideas", children: ["Prompt"] }
    ];

    /* ************************* Loop through configurations */
    configurations.forEach(function (config, parentIndex) {  

        /* ************************* Create and append parent tab */
        const P = config.parent; const p = P.toLowerCase();
        resultTabs.append(`<li class="nav-parent">${P}</li>`);

        /* ************************* Loop through children */
        config.children.forEach(function (child, childIndex) {  

            /* ************************* Create and append child tab */
            const C = child, PC = `${P}-${C}`, P_C = `${P}_${C}`, c = C.toLowerCase(), pc = `${p}-${c}`;

            const childTab = $(`<li class="nav-item ${P}-subtab ${parentIndex === 0 && childIndex === 0 ? 'active' : ''}"><a class="nav-link result-tab" id="${PC}-tab" data-toggle="tab" href="#${pc}">${C}</a></li>`);  
            /* the specific data-toggle, href and lowercase appear to be necessary for Bootstrap tab functionality */

            resultTabs.append(childTab);

            /* ************************* Get "strike-through or not" from LocalStorage */
            const isActive = localStorage.getItem(P_C); // logMessage(`${P_C} isActive ${isActive}`);

            /* ************************* Get link element from child tab and set strike-through based on current state */
            const link = $(childTab).find('a'); link.css('text-decoration', isActive === '1' ? 'none' : 'line-through');

            /* ************************* Create a new tab panel for the child tab */
            const childTabPanel = $(`<div id="${pc}" class="tab-pane fade ${parentIndex === 0 && childIndex === 0 ? 'show active' : ''}"><div id="tabPanel_${P_C}" class="scrollable-container"></div></div>`);  
            /* first div: the specific id, lowercase, and class necessary for Bootstrap tab functionality, second div: this is the container actually used to display results */

            /* ************************* Append the new tab panel to the tab content container */
            resultTabContent.append(childTabPanel);

            /* ************************* Add right-click event listener to the child tab */
            childTab.on('contextmenu', (e) => {

                $(childTab).find('a').trigger('click');  

                /* ************************* Prevent default right-click action */
                e.preventDefault();

                /* ************************* Toggle state in LocalStorage based on its current value */
                localStorage.setItem(P_C, localStorage.getItem(P_C) === '1' ? '0' : '1');

                /* ************************* Get link element from child tab and set strike-through based on updated state */
                const link = $(childTab).find('a'); link.css('text-decoration', localStorage.getItem(P_C) === '1' ? 'none' : 'line-through');

                /* ************************* Get and empty the tab panel element */
                const tabPanel = $(`#tabPanel_${P_C}`); tabPanel.empty();

                /* ************************* If the tab is deselected, show a message in the tab panel */
                if (localStorage.getItem(P_C) === '0') { tabPanel.append(`<p>${PC} de-selected</p>`); }

            });
        });
    });

    logMessage(`interface.js processed in ${((new Date() - ProcessingStart_Interface) / 1000).toFixed(3)}s`);
};  
```

-----

#### openai.js

Right click on "wwwroot" >> "js" and then add new file "openai.js". Replace the default code with:

```js
connection.on(`displayEvaluation`, data => displayQueryEvaluation(data));

const buildTabPanel = (type, data) => {

    //logMessage(JSON.stringify(data, null, 2));    

    /* ************************* Get and empty tabpanel */
    const tabPanel = document.getElementById(`tabPanel_OpenAI_${type}`);
    tabPanel.innerHTML = '';

    /* ************************* Create label */

    label = document.createElement('label');
    label.id = `labelOpenAI_${type}`;
    label.textContent = `Time-to-Process: ${data.stp.toFixed(1)} seconds`;
    tabPanel.appendChild(label);

    /* ************************* Create textarea */

    let textarea = document.createElement('div');
    textarea.id = `textareaOpenAI_${type}`;
    textarea.className = 'fake-textarea';
    textarea.readOnly = true;

    textarea.innerHTML = data.response.replace(/\[doc(\d+)\]/g, function (match, p1) {
        return '<span class="highlight" title="' + data.citations[p1-1] + '"><sup>' + p1 + '</sup></span>';
    });

    tabPanel.appendChild(textarea);  
}

["VectorSemantic", "VectorSimple", "Vector", "Semantic", "Simple"].forEach(type => { connection.on(`displayOpenAI_${type}`, data => buildTabPanel(type, data)); });

const displayQueryEvaluation = (data) => {
    const tabPanel = document.getElementById('tabPanel_Ideas_Prompt');

    const label = document.createElement('label');
    label.textContent = "Suggestions";
    tabPanel.appendChild(label);

    const textarea = document.createElement('textarea');
    textarea.id = 'evaluationResult';
    textarea.rows = '9';
    textarea.className = 'textarea';
    textarea.readOnly = true;
    textarea.style.width = '100%';
    textarea.textContent = data.response;
    tabPanel.appendChild(textarea);
}
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
    textareaLog.value += `${timestamp}, ${source}... ${message}\n`;
    textareaLog.scrollTop = textareaLog.scrollHeight;
}

connection.on("logMessage", (message) => logMessage(message, 'Server-Side'));
```

-----

#### submit.js

Right click on "wwwroot" >> "js" and then add new file "submit.js". Replace the default code with:

```js
/* ************************* Handle Events from listeners: 1) "Submit" button click or 2) "Enter" key down */

const HandleEvent = async (e) => {
    if (e.type === 'click' || (e.type === 'keydown' && e.key === 'Enter' && !e.shiftKey)) {

        e.preventDefault(); /* Prevents the addition of a newline in the text area */

        let submitIcon = document.querySelector('#submitIcon'); // Assuming the id of your img tag is 'submitIcon'  

        if (submitIcon.src.includes('send.svg')) {
            if (connection.state === signalR.HubConnectionState.Disconnected) { await connection.start(); }
            submitIcon.src = '../images/stop.svg';
            submitIcon.title = "Stop";

            await processQuery();
            $('#tab_Ideas_Prompt').trigger('click');
        }
        else {

            if (connection.state === signalR.HubConnectionState.Connected) { await connection.stop(); }
            submitIcon.src = '../images/send.svg';
            submitIcon.title = "Send";
            progressBar.style.display = 'none';
        }
    }
};

/* ***** Event Listeners... MUST follow HandleEvent */

inputUserQuery.addEventListener('keydown', HandleEvent);
submitIcon.addEventListener('click', HandleEvent);

/* ************************* Process Query */

const processQuery = async (e) => {

    logMessage(`Sending User Query "${inputUserQuery.value}" :: AISearch_Filter "${inputFilter.value}" :: System Message "${inputSystemMessage.value}" :: Temperature ${inputTemperature.value}`);

    document.querySelectorAll('[id^="tabPanel_"]').forEach(tabPanel => { tabPanel.innerHTML = ''; });

    progressBar.style.display = 'block'; /* Reset progress bar */

    const runFlags = {
        "AISearch_Semantic": false,
        "AISearch_Full": false,
        "AISearch_Simple": false,
        "OpenAI_Semantic": false,
        "OpenAI_Simple": false
    };  

    Object.keys(runFlags).forEach(key => { runFlags[key] = localStorage.getItem(key) === '1'; });  

    connection.invoke("ProcessQuery", inputUserQuery.value, inputFilter.value, inputSystemMessage.value, parseFloat(inputTemperature.value), runFlags)

        .then(() => {
            progressBar.style.display = 'none';
            $('.result-tab:first').tab('show'); /* Show first tab panel */
            submitIcon.src = '../images/send.svg';
            submitIcon.title = "Send";
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

### Step 8: Publish Application

In Visual Studio >> Solution Explorer, right-click on the project name and select "Publish" from the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6324c434-9560-4e25-ad15-ab89c698f04f" width="600" title="Snipped April 22, 2024" />

On the "Publish" popup, "Target" tab, select "Azure" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/73935164-0dc7-4034-977f-fe3838d8a135" width="600" title="Snipped April 22, 2024" />

On the "Publish" popup, "Specific target" tab, select "Azure App Service (Windows)" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/62936cd5-6fc7-4a78-8c28-d3f4ac96f184" width="600" title="Snipped April 22, 2024" />

On the "Publish" popup, "App Service" tab, select App Service and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/297c5213-e196-421b-a73c-18e18cd3b908" width="600" title="Snipped April 22, 2024" />

On the "Publish" popup, "Deployment type" tab, select "Publish (generates pubxml file)" and then click "Finish".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/684cca9a-4b78-47dc-a2a2-4b4e9b801e88" width="600" title="Snipped April 22, 2024" />

Click "Close".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e8542029-4c67-4723-8660-2dc16728ff03" width="800" title="Snipped April 22, 2024" />

On the "...Publish" tab, click "Publish".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e4be1d51-80bb-4559-880c-017c7a0d5ab5" width="800" title="Snipped April 22, 2024" />

When publication is complete, your browser will open to the published web application.

-----

### Step 9: Secure Application

Go to Azure Portal >> App Service >> Settings >> Authentication.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2c1630ea-03ba-4dec-877c-3129aac203cb" width="800" title="Snipped April 23, 2024" />

Click "Add identity provider".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/64688615-12a9-42f6-8a08-c1e85048e0cd" width="800" title="Snipped April 23, 2024" />

Complete the "Add an identity provider" form.

Prompt | Entry
:----- | :-----
Identity Provider | Microsoft
Choose a tenant... | Workforce configuration (current tenant)
App registration type | Provide the details of an existing app registration
Application (client) ID | Name of your application registration
Client secret | Secret value for your application registration
Tenant requirement | Use default restrictions based on issuer

Click "Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/70109dfc-95e5-43fe-bf5a-6defbc573e46" width="800" title="Snipped April 23, 2024" />

Individual Permissions... possible configuration
* Enterprise Application >> Properties >> Assignment Required? >> yes

-----

**Congratulations... you have successfully completed all steps**

-----
