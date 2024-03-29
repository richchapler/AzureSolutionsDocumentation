# AI Search + OpenAI: Bot

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/49618b54-2b60-4047-b054-2f2bbc062ff1" width="1000" />

⚠️ WORK-IN-PROGRESS ⚠️

## Use Case
* "We want to surface our OpenAI solution as a bot in various channels (Teams, Alexa, Facebook, etc.)

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [Bot Service](https://azure.microsoft.com/en-us/products/ai-services/ai-bot-service)

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)

## Documentation Note

I used to spend a lot of time explaining code blocks, but will skip that part of my sharing process in favor of a recommendation to copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d0f5043e-6668-44de-b05c-be3da3288f95" width="600" title="Snipped March 29, 2024" />

On the "Create a new project" form, search for and select "ASP.NET Core Web App (Model-View-Controller)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6dcdfa95-5232-4079-b7ff-7c3ba35abda8" width="600" title="Snipped March 29, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/13b9f0fb-60c2-4494-ba43-9ac530ff0e02" width="600" title="Snipped March 29, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eb046821-afd1-427a-896e-350f0cfa0ac4" width="600" title="Snipped March 29, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7f874628-7617-429b-bc73-de925a3b7f41" width="800" title="Snipped March 29, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a2feba33-ca9c-4b1e-a1ec-ad6df8d377c9" width="800" title="Snipped March 29, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Microsoft.Bot.Builder.Integration.AspNet.Core".
<br>On the resulting pop-out, check the box next to your project and then click "Install".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7f949040-a093-4b83-9112-2f96bb67d6f8" width="300" title="Snipped March 29, 2024" />

On the "License Acceptance" pop-up, click "I Accept".
<br><br>Repeat this process for "Microsoft.AspNetCore.Mvc.NewtonsoftJson".

-----

### Step 3: Update Logic

#### Program.cs

Replace the default logic with:
```
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.BotBuilderSamples;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHttpClient();
builder.Services.AddControllers().AddJsonOptions(options =>
{
    options.JsonSerializerOptions.WriteIndented = true;
});
builder.Services.AddSingleton<BotFrameworkAuthentication, ConfigurationBotFrameworkAuthentication>();
builder.Services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
builder.Services.AddTransient<IBot, AI_Bot.Bots.PromptBot>();

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
else { app.UseDeveloperExceptionPage(); }

app.UseDefaultFiles();
app.UseStaticFiles();
app.UseWebSockets();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

#### appsettings.json

Replace the default logic with:
```
{
  "MicrosoftAppType": "",
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "MicrosoftAppTenantId": ""
}
```

#### Properties >> launchSettings.json

Replace the default logic with:
```
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3978",
      "sslPort": 0
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    ".NET Core": {
      "commandName": "Project",
      "launchBrowser": true,
      "applicationUrl": "https://localhost:3979;http://localhost:3978",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

#### wwwroot

Delete existing contents of the `wwwroot` folder {e.g., `css`, `js`, etc.} and right-click to Add `default.html`.










Remove HomeController.cs and add BotController.cs
Add AdapterWithErrorHandler.cs

Remove everything in wwwroot and add default.html

Delete Models and Views




-----

### Step 7: Confirm Success

Click "Debug" >> "Start Debugging" in the menu bar.

Enter a prompt and press the Enter key on your keyboard... allow time for processing and monitor progress in the messages logged at the bottom of the interface.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac873cde-1ecd-4a05-b36a-32508e73f210" width="800" title="Snipped February 26, 2024" />

When processing is complete, you can expect to see responses from AI Search and OpenAI (both keyword, full, and semantic configurations).

-----

**Congratulations... you have successfully completed all steps**

-----
