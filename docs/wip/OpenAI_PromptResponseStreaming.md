From https://learn.microsoft.com/en-us/azure/ai-services/openai/use-your-data-quickstart?tabs=command-line%2Cpython-new&pivots=programming-language-csharp

Used with \.nuget\packages\azure.ai.openai\2.0.0-beta.5\

```csharp
#pragma warning disable AOAI001 

using Azure;
using Azure.AI.OpenAI;
using Azure.AI.OpenAI.Chat;
using OpenAI.Chat;

class Program
{
    static async Task Main()
    {
        string azureOpenAIEndpoint = "https://azuresolutions-openai-canadaeast.openai.azure.com/";
        string azureOpenAIKey = "KEY";
        string deploymentName = "gpt-4-32k";
        string searchEndpoint = "https://azuresolutions-aisearch.search.windows.net";
        string searchKey = "KEY";
        string searchIndex = "index";

        AzureOpenAIClient azureClient = new(new Uri(azureOpenAIEndpoint), new AzureKeyCredential(azureOpenAIKey));
        ChatClient chatClient = azureClient.GetChatClient(deploymentName);
        ChatCompletionOptions options = new();
        options.AddDataSource(new AzureSearchChatDataSource()
        {
            Endpoint = new Uri(searchEndpoint),
            IndexName = searchIndex,
            Authentication = DataSourceAuthentication.FromApiKey(searchKey),
        });

        Console.Write("Enter your question: "); string? userQuestion = Console.ReadLine();

        var chatUpdates = chatClient.CompleteChatStreamingAsync([new UserChatMessage(userQuestion)], options);

        AzureChatMessageContext? onYourDataContext = null;

        await foreach (var chatUpdate in chatUpdates)
        {
            if (chatUpdate.Role.HasValue)
            {
                Console.WriteLine($"{chatUpdate.Role}: ");
            }
            foreach (var contentPart in chatUpdate.ContentUpdate)
            {
                Console.Write(contentPart.Text);
            }
            onYourDataContext ??= chatUpdate.GetAzureMessageContext();
        }
        Console.WriteLine();
        if (onYourDataContext?.Intent is not null)
        {
            Console.WriteLine($"Intent: {onYourDataContext.Intent}");
        }
        foreach (AzureChatCitation citation in onYourDataContext?.Citations ?? [])
        {
            Console.Write($"Citation: {citation.Content}");
        }
    }
}
```
