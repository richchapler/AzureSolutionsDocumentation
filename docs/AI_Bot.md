# AI: Bot

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/49618b54-2b60-4047-b054-2f2bbc062ff1" width="1000" />

## Use Case
* "We want to surface our OpenAI solution as a bot in various channels {e.g., Teams, Skype, etc.)"
* "We must provide for custom bot handling of specific messages types {e.g., model type selection}"

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [AI Search](https://azure.microsoft.com/en-us/products/search) index with default Semantic Configuration
  * The documented configuration uses an index pointed at the Azure SQL AdventureWorks sample database

* [Application Service](https://learn.microsoft.com/en-us/azure/app-service/) and Hosting Plan
  * Once created (as part of exercise), must be granted "Key Vault Secrets User" role assignment on Key Vault

* [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases/tag/v4.5.2)

* [Bot Service](https://azure.microsoft.com/en-us/products/ai-services/ai-bot-service)

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) with the following [secrets](https://learn.microsoft.com/en-us/azure/key-vault/secrets):
  * AISearch-Name
  * AISearch-Key
  * AISearch-Index-Name
  * AISearch-SemanticConfiguration-Name
  * AISearch-SelectFields
  * OpenAI-Name
  * OpenAI-Key
  * OpenAI-Deployment-GPT

* [OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)

* [Visual Studio](https://visualstudio.microsoft.com/downloads/)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Bing Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

-----

### Step 1: Create Project

Open Visual Studio and click "Create a new project".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/d0f5043e-6668-44de-b05c-be3da3288f95" width="600" title="Snipped March 29, 2024" />

On the "Create a new project" form, search for and select "ASP.NET Core Web App (Model-View-Controller)", then click "Next".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/6dcdfa95-5232-4079-b7ff-7c3ba35abda8" width="600" title="Snipped March 29, 2024" />

Complete the "Configure your new project" form, then click "Next".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/13b9f0fb-60c2-4494-ba43-9ac530ff0e02" width="600" title="Snipped March 29, 2024" />

Complete the "Additional information" form, then click "Create".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/eb046821-afd1-427a-896e-350f0cfa0ac4" width="600" title="Snipped March 29, 2024" />

-----

### Step 2: Install NuGet

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/7f874628-7617-429b-bc73-de925a3b7f41" width="800" title="Snipped March 29, 2024" />

Click "Tools" in the menu bar, expand "NuGet Package Manager", then click "Manage NuGet Packages for Solution..."

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/a2feba33-ca9c-4b1e-a1ec-ad6df8d377c9" width="800" title="Snipped March 29, 2024" />

On the "Browse" tab of the "NuGet - Solution" page, search for and select "Microsoft.Bot.Builder.Integration.AspNet.Core".
<br>On the resulting pop-out, check the box next to your project and then click "Install".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/7f949040-a093-4b83-9112-2f96bb67d6f8" width="300" title="Snipped March 29, 2024" />

On the "License Acceptance" pop-up, click "I Accept".
<br><br>Repeat this process for "AzureSolutions.Helpers".

-----

### Step 3: Modify Logic

#### `Program.cs`
Replace the default logic with:

```csharp
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

#### `appsettings.json`

Replace the default logic with:
```json
{
  "MicrosoftAppType": "",
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "MicrosoftAppTenantId": "",
  "KeyVault_Name": ""
}
```

#### Properties >> `launchSettings.json`

Replace the default logic with:
```json
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

#### `wwwroot`

Delete existing contents of the `wwwroot` folder {e.g., `css`, `js`, etc.} and right-click to Add `default.html`.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>PromptBot</title>
</head>
<body>
    <h1>Your bot is ready!</h1>
    <p>http://localhost:3978/api/messages</p>
</body>
</html>
```

#### Controllers

Delete existing contents of the `Controllers` folder {e.g., `HomeController`, etc.} and right-click to Add `BotController.cs`.
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Integration.AspNet.Core;

namespace Microsoft.BotBuilderSamples.Controllers
{
    [Route("api/messages")]
    [ApiController]
    public class BotController : ControllerBase
    {
        private readonly IBotFrameworkHttpAdapter _adapter;
        private readonly IBot _bot;

        public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
        {
            _adapter = adapter;
            _bot = bot;
        }

        [HttpPost, HttpGet]
        public async Task PostAsync()
        {
            await _adapter.ProcessAsync(Request, Response, _bot);
        }
    }
}
```

#### `AdapterWithErrorHandler.cs`

Right-click to Add `AdapterWithErrorHandler.cs`
```csharp
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Bot.Connector.Authentication;

namespace Microsoft.BotBuilderSamples
{
    public class AdapterWithErrorHandler : CloudAdapter
    {
        public AdapterWithErrorHandler(BotFrameworkAuthentication auth, ILogger<IBotFrameworkHttpAdapter> logger)
            : base(auth, logger)
        {
            OnTurnError = async (turnContext, exception) =>
            {
                logger.LogError(exception, $"[OnTurnError] unhandled error : {exception.Message}");

                await turnContext.SendActivityAsync("The bot encountered an error or bug.");
                await turnContext.SendActivityAsync("To continue to run this bot, please fix the bot source code.");

                await turnContext.TraceActivityAsync("OnTurnError Trace", exception.Message, "https://www.botframework.com/schemas/error", "TurnError");
            };
        }
    }
}
```

#### Bots

Right click on the project and add a `Bots` folder. Repeat on the `Bots` folder to add `PromptBot.cs`.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace AI_Bot.Bots
{
    public class PromptBot : ActivityHandler
    {
        protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
        {
            try
            {
                var OpenAI_Prompt = await AzureSolutions.Helpers.OpenAI.Prompt.VectorSemantic(AI_Bot.Helpers.Config.oaic, AI_Bot.Helpers.Config.aisc,
                    UserQuery: turnContext.Activity.Text,
                    SystemMessage: null,
                    Temperature: 0.5f
                    );

                var replyText = $"Echo: {turnContext.Activity.Text}";
                await turnContext.SendActivityAsync(MessageFactory.Text(OpenAI_Prompt.Response), cancellationToken);
            }
            catch (Exception ex)
            {
                var exceptionMessage = $"Exception Message: {ex.Message}\n" +
                                       $"Exception StackTrace: {ex.StackTrace}\n" +
                                       $"Inner Exception: {ex.InnerException?.Message ?? "No inner exception"}";

                await turnContext.SendActivityAsync(MessageFactory.Text(exceptionMessage), cancellationToken);
            }

        }

        protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
        {
            var welcomeText = "Hello and welcome!";
            foreach (var member in membersAdded)
            {
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync(MessageFactory.Text(welcomeText, welcomeText), cancellationToken);
                }
            }
        }
    }
}

```

#### Helpers

Right click on the project and add a `Helpers` folder.

##### `Config.cs`

Right-click on the `Helpers` folder and add `Config.cs`.

```csharp
namespace AI_Bot.Helpers
{
    public static class Config
    {
        public static readonly AzureSolutions.Helpers.AISearch.Configuration aisc = AzureSolutions.Helpers.AISearch.Configuration.PrepareConfiguration(Helpers.KeyVault.KeyVault_Client);

        public static readonly AzureSolutions.Helpers.OpenAI.Configuration oaic = AzureSolutions.Helpers.OpenAI.Configuration.PrepareConfiguration(Helpers.KeyVault.KeyVault_Client);
    }
}
```

##### `KeyVault.cs`

Right-click on the `Helpers` folder and add `KeyVault.cs`.

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace AI_Bot.Helpers
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

        public static async Task<string> GetSecret(string secretName)
        {
            var secret = await KeyVault_Client.GetSecretAsync(secretName);
            return secret.Value.Value;
        }
    }
}
```

#### Miscellaneous

Delete `Models` and `Views` folders.

-----

### Step 4: Confirm Success

Click "Debug" >>  "Start Debugging".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/9887ce5e-b08f-49ba-8a8a-6fa83ffb186a" width="800" title="Snipped April 9, 2024" />

#### Bot Framework Emulator

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2617cc8f-8a3a-4f1d-a46a-5e10ff1ff9fb" width="800" title="Snipped March 29, 2024" />

On the main page of the Bot Framework Emulator, click "Open Bot".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/2265ed14-1d9b-4344-a52d-df1a07f2fbc8" width="800" title="Snipped March 29, 2024" />

Paste the "Bot URL" value and click "Connect"... neither "Microsoft App ID" or "Microsoft App password" are required.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/fa382d60-fe39-47eb-bcd4-2672aa9e460f" width="800" title="Snipped April 9, 2024" />

A new "Live Chat" tab will open and you can expect to see an opening "Hello and welcome!" message from PromptBot.

### Step 5: Publish Bot

In Visual Studio, right-click on the project name and click "Publish" in the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/86dfec81-4456-463a-8222-741f8721dc00" width="600" title="Snipped March 29, 2024" />

On the "Publish" popup, "Target" tab, click "Azure" and then "Next".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/cde2991a-7c0d-42cf-8087-1b7a454af14f" width="600" title="Snipped March 29, 2024" />

On the "Publish" popup, "Specific target" tab, click "Azure App Services (Windows)" and then "Next".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/66eae2eb-5889-4bc1-a948-baea3e1cbefd" width="450" title="Snipped March 29, 2024" />

On the "Publish" popup, "App Service" tab, click "new", then complete the resulting "App Services (Windows)" popup, click "Create".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/4e2b21c5-426c-463e-8fa4-2d80b85c8c68" width="600" title="Snipped March 29, 2024" />

Confirm "App Service" configuration, then click "Finish".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/012d1d8e-3aea-497f-a77c-5c5a90408e72" width="600" title="Snipped March 29, 2024" />

Confirm success on "Publish profile creation progress" and then click "Close".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/886263b5-1b6c-4417-856a-159771c6cd96" width="800" title="Snipped March 29, 2024" />

Back on the "AI_Bot: Publish" tab, click "Publish".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/57c29f7b-def9-4f3f-8974-e4824b49b033" width="800" title="Snipped April 9, 2024" />

### Step 6: Configure App

Navigate to the Azure Web App >> Configuration.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ed301755-3262-4e09-92f0-4bf73031e9cf" width="800" title="Snipped April 8, 2024" />

Click "+ New application setting".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/6e54af67-997d-4c6b-a020-87a811c6541b" width="800" title="Snipped April 8, 2024" />

Complete the resulting "Add/Edit application setting" popout for each of the following items:
* `KeyVault_Name`: YOUR KEYVAULT NAME
* `MicrosoftAppType`: `MultiTenant`
* `MicrosoftAppId`: YOUR CLIENT ID
* `MicrosoftAppPassword`: YOUR CLIENT SECRET
* `MicrosoftAppTenantId`: `common`

Click "Save" at the top of the "Configuration" page.

### Step 7: Configure Bot

Navigate to the Azure Bot >> Configuration.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/c6cbc18e-77dd-4ba2-b03e-93d3aa573b86" width="800" title="Snipped March 29, 2024" />

Paste the "Messaging endpoint" value from your published bot {e.g., "https://azuresolutions-aibot-asw.azurewebsites.net/api/messages"} and click "Apply".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ed3c7c08-93a8-4a93-87e0-96f24ea87ef6" width="800" title="Snipped March 29, 2024" />

Navigate to Azure Bot >> Channels, scroll through the list of "Available Channels" and click on "Microsoft Teams".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/c540e9a6-1e6c-45ba-b79c-493ce01dc4c4" width="800" title="Snipped March 29, 2024" />

Complete "Terms of Service" popup and click "Agree".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/5f27f86b-3ee1-4455-a893-b17b91d41347" width="800" title="Snipped March 29, 2024" />

On the "Messaging" tab, confirm selection of "Microsoft Teams Commercial..." and click "Apply" and then "Close".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/e543c86f-d8d0-43cd-a580-6b7f30ce9215" width="800" title="Snipped March 29, 2024" />

Click "Open in Teams".

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/d5507b4a-7536-45dd-81e7-21de43dc2885" width="800" title="Snipped March 29, 2024" />

Open Teams and confirm success.

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/178b935c-21fd-4afa-bb68-834552e42591" width="800" title="Snipped April 9, 2024" />

-----

**Congratulations... you have successfully completed all steps**

-----
