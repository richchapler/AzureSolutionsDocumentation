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

### Step 3: Helpers

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

### Step 4: Controllers

#### HomeController.cs
Right-click on the "Helpers" folder, select "Add" >> "Class" from the resulting dropdowns, enter name "Config.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using AI_Translator.Helpers;
using AI_Translator.Models;
using Microsoft.AspNetCore.Html;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.SignalR;
using System.Diagnostics;

namespace AI_Translator.Controllers
{
    public class HomeController(IHubContext<LogHub> hubContext) : Controller
    {
        private readonly IHubContext<LogHub> _logger = hubContext;

        public IActionResult Index() { return View(); }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error() { return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier }); }

        [HttpPost]
        public async Task<IActionResult> TranslateEnteredText(string text, string sourceLanguage, string targetLanguage)
        {
            await _logger.Clients.All.SendAsync("ReceiveMessage", $"<b>Translating Entered Text</b>");

            var translatedText = await new Translate(_logger).Input(text, sourceLanguage, targetLanguage);

            await _logger.Clients.All.SendAsync("ReceiveMessage", $"Translation Result: <b>{translatedText}</b>");

            return Ok();
        }

        [HttpPost]
        [RequestFormLimits(ValueLengthLimit = int.MaxValue, MultipartBodyLengthLimit = int.MaxValue)]
        public async Task<IActionResult> ProcessDroppedFiles(IFormCollection collection)
        {
            await _logger.Clients.All.SendAsync("ReceiveMessage", $"<b>Processing Dropped File</b>");

            string success = string.Empty; string error = string.Empty;

            try
            {
                if (collection["source_language"].Count > 0 && collection["target_language"].Count > 0)
                {
                    foreach (var file in collection.Files)
                    {
                        /* ************************* Upload */

                        var urlUploaded = await new Upload(_logger).File(file);

                        await _logger.Clients.All.SendAsync("ReceiveMessage", $"<a href='{urlUploaded}'>Upload Complete (click to download)</a>");

                        /* ************************* Translate */

                        var urlTranslated = await new Helpers.Translate(_logger).File(file, urlUploaded
                            , collection["source_language"].ToString()
                            , collection["target_language"].ToString()
                        );

                        if (urlTranslated != "Translation Failed")
                        {
                            await _logger.Clients.All.SendAsync("ReceiveMessage", $"<a href='{urlTranslated}'>Translation Complete (click to download)</a>");

                            /* ************************* Reformat */

                            var reformattedUrl = new Reformat(_logger).File(file, urlTranslated);
                            await _logger.Clients.All.SendAsync("ReceiveMessage", $"<a href='{reformattedUrl}'>Reformat Complete (click to download)</a>");
                        }
                    }
                }
            }
            catch (Exception ex) { await _logger.Clients.All.SendAsync("ReceiveMessage", $"Exception: {ex.Message}"); }

            return Json(new { message = new HtmlString(success), error });
        }
    }
}
```

-----

### Step 5: Back-End

#### Hub.cs
Right-click on the project, select "Add" >> "Class" from the resulting dropdowns, enter name "Hub.cs" on the resulting popup then click "Add". Replace the default code with:

```csharp
using Microsoft.AspNetCore.SignalR;

namespace AI_Translator
{
    public class LogHub : Hub
    {
        public async Task SendMessage(string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", message);
        }
    }
}
```

#### Program.cs
Double-click to open "Program.cs". Replace the default code with:

```csharp
using AI_Translator;
using Microsoft.AspNetCore.Server.Kestrel.Core;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

builder.Services.Configure<KestrelServerOptions>(options => { options.Limits.MaxRequestBodySize = 100 * 1024 * 1024; }); /* Upload max... Translator file size maximum is ~40MB */

builder.Services.AddSignalR();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapHub<LogHub>("/logHub");

app.Run();
```

-----

### Step 6: Front-End

#### _Layout.cshtml
Expand "Views" >> "Shared" and double-click to open "_Layout.cshtml". Replace the default code with:

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Translator</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    <link rel="stylesheet" href="~/Translator.styles.css" asp-append-version="true" />

    <style>
        #drop-area {
            border: 2px dashed #ccc;
            border-radius: 20px;
            width: 300px;
            height: 200px;
            text-align: center;
            line-height: 200px;
            margin: 0 auto;
        }
    </style>
</head>

<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container-fluid">
                <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">Translator</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between"> </div>
        </nav>
    </header>

    <div class="container"><main role="main" class="pb-3"> @RenderBody() </main></div>

    <footer class="border-top footer text-muted"></footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/microsoft-signalr/3.1.7/signalr.min.js"></script>

    <script src="~/js/runfirst.js" asp-append-version="true"></script>

    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

-----

#### Index.cshtml
Expand "Views" >> "Home" and then double-click to open "Index.cshtml". Replace the default code with:

```cshtml
@{
    ViewData["Title"] = "Home Page";
}

@using (Html.BeginForm("UploadFiles", "Home", FormMethod.Post, new { enctype = "multipart/form-data", id = "uploadForm" }))
{
    <div>
        <div>
            <select name="source_language" id="source_language" style="display: inline-block;">
                <option value="" disabled selected>Select language</option>
                <option value="en">English</option>
                <option value="ja">Japanese</option>
                <option value="es">Spanish</option>
                <option value="fr-ca">French (Canada)</option>
                <option value="zh-Hans">Chinese (Simplified)</option>
            </select>
            >>
            <select name="target_language" id="target_language" style="display: inline-block;">
                <option value="" disabled selected>Select language</option>
                <option value="en">English</option>
                <option value="ja">Japanese</option>
                <option value="es">Spanish</option>
                <option value="fr-ca">French (Canada)</option>
                <option value="zh-Hans">Chinese (Simplified)</option>
            </select>
        </div>

        <div style="display: flex; align-items: flex-start;">

            <div id="leftSide" style="width: 50%; height: 75vh; display: flex; flex-direction: column;">
                <div style="display: flex; flex-direction: column; position: relative;" padding: 5px;>

                    <textarea id="text-entry" placeholder="[Enter] text to translate" style="height: 75vh; overflow: auto;"></textarea>

                    <div id="drop-area" style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 75%; height: 25%; border: 1px dashed black; background-color: whitesmoke; display: flex; justify-content: center; align-items: center;">
                        Drag-and-drop files here
                    </div>

                </div>
            </div>

            <div id="rightSide" style="width: 50%; height: 75vh; display: flex; flex-direction: column;">
                <div class="message-table-wrapper">
                    <table id="divMessages" class="message-table"></table>
                </div>
            </div>

        </div>
    </div>

    <style>
        .message-table-wrapper {
            width: 100%;
            height: 100vh;
            border: 1px solid black;
            overflow: auto;
        }

        .message-table {
            width: 100%;
        }
            .message-table tr:nth-child(even) {
                background: whitesmoke;
            }

            .message-table tr:nth-child(odd) {
                background: white;
            }
    </style>
}

@section Scripts {
    <script type="text/javascript">
        $(document).ready(function () {

            $('#text-entry').on('keypress', function (e) {
                if (e.which == 13) {  // Enter key
                    e.preventDefault();  // Prevents the default action to be triggered (newline)

                    var text = $(this).val().trim();
                    var sourceLanguage = $('#source_language').val();
                    var targetLanguage = $('#target_language').val();

                    if (!sourceLanguage || !targetLanguage) {
                        var timestamp = new Date().toLocaleTimeString();
                        $('#divMessages').append(timestamp + ': Please select both source and target languages.\n\n');
                        return;
                    }

                    $.ajax({
                        url: '/Home/TranslateEnteredText',
                        type: 'post',
                        data: { text: text, sourceLanguage: sourceLanguage, targetLanguage: targetLanguage },
                        success: function (response) { $('#divMessages').append(response + '\n'); },
                        fail: function (jqXHR, textStatus, errorThrown) { $('#divMessages').append('Error: ' + textStatus + ' ' + errorThrown + '\n'); }
                    });
                }
            });

            $('#drop-area').on({
                dragover: function (e) {
                    e.preventDefault();
                    e.stopPropagation();
                },
                dragleave: function (e) {
                    e.preventDefault();
                    e.stopPropagation();
                },
                drop: function (e) {
                    if (e.originalEvent.dataTransfer && e.originalEvent.dataTransfer.files.length) {
                        e.preventDefault();
                        e.stopPropagation();
                        $('#divMessages').html(''); // Clear the messages div
                        upload(e.originalEvent.dataTransfer.files);
                        sendFiles();
                    }
                }
            });

            const formData = new FormData();

            function sendFiles() {
                $.ajax({
                    url: '/Home/ProcessDroppedFiles',
                    type: 'post',
                    data: formData,
                    processData: false,
                    contentType: false,
                    cache: false,
                }).always(function () { formData.delete('file'); });
            }

            function upload(files) {
                formData.append('source_language', $('#source_language').val());
                formData.append('target_language', $('#target_language').val());

                for (let i = 0; i < files.length; i++) {
                    $('#divMessages').append('<b>' + files[i].name + '</b> (' + (files[i].size / 1024 / 1024).toFixed(2) + 'MB)<br>');
                    formData.append('file', files[i]);
                }
            }
        });
    </script>
}
```

-----

#### site.css

Expand "wwwroot" >> "css" and then double-click to open "site.css". Replace the default code with:

```css
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
```

-----

### Step 7: Javascript

#### site.js

Expand "wwwroot" >> "js" and then delete "site.js".

-----

#### runfirst.js

Right click on "wwwroot" >> "js" and then add new file "runfirst.js". Replace the default code with:

```js
/* ************************* SignalR Connection (MUST BE FIRST!) */

const connection = new signalR.HubConnectionBuilder()
    .withUrl("/logHub")
    .build();

connection.on("ReceiveMessage", function (message) {
    const textarea = document.getElementById("divMessages");
    let row = textarea.insertRow(0);
    let cell = row.insertCell(0);
    cell.innerHTML = `${new Date().toLocaleTimeString()}: ${message}`;
});

connection.start().catch(function (err) {
    return console.error(err.toString());
});
```

-----

### Step 8: Confirm Success
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
