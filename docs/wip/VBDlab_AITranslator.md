# Lab: AI Translator

<img src="https://github.com/user-attachments/assets/96481bac-c772-4dae-98c3-139bfd04f9db" width="1000" />

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/create-translator-resource)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!



Lab: Text Translation with Azure AI Translator
Objective
Students will use Azure Translator to translate text between languages, explore language detection, and make API requests using C#.
________________________________________
Prerequisites
1.	Azure Setup:
o	An active Azure subscription.
o	A Translator resource created in the Azure portal.
o	Obtain the API endpoint and key from the Azure portal.
2.	Development Environment:
o	Visual Studio (Community or Enterprise Edition) installed.
o	Azure SDK installed for .NET (via NuGet).
o	Postman (optional, for testing API requests).
________________________________________
Steps
Step 1: Create an Azure Translator Resource
1.	Log in to the Azure Portal.
2.	Search for "Translator" in the search bar and select "Translator (Cognitive Services)."
3.	Click Create and configure the resource with the following: 
o	Subscription: Choose your active subscription.
o	Resource Group: Create a new or use an existing one.
o	Name: Enter a unique name for your Translator resource.
o	Pricing Tier: Choose the free tier for this lab.
o	Region: Select the closest region.
4.	Once created, navigate to the resource and copy the Key and Endpoint.
________________________________________
Step 2: Set Up Your Project in Visual Studio
1.	Open Visual Studio and create a new Console App (.NET) project.
2.	Name the project AzureTranslatorDemo.
3.	Add the Azure.AI.Translation.Text package via NuGet: 
o	Go to Tools > NuGet Package Manager > Manage NuGet Packages for Solution.
o	Search for Azure.AI.Translation.Text and install it.
________________________________________
Step 3: Write the Translation Code
1.	Replace the Program.cs content with the following code:
using System;
using System.Net.Http;
using System.Text.Json;
using System.Text;

class Program
{
    private const string Endpoint = "https://<your-resource-name>.cognitiveservices.azure.com/translator/text/v3.0/";
    private const string SubscriptionKey = "<your-key>";

    static async System.Threading.Tasks.Task Main(string[] args)
    {
        Console.WriteLine("Enter the text to translate:");
        string textToTranslate = Console.ReadLine();
        string targetLanguage = "fr"; // Translate to French

        string route = $"translate?api-version=3.0&to={targetLanguage}";

        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", SubscriptionKey);
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Region", "your-region"); // Replace with your resource region

            object[] body = new object[] { new { Text = textToTranslate } };
            string requestBody = JsonSerializer.Serialize(body);

            using (HttpRequestMessage request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(Endpoint + route);
                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.SendAsync(request);
                string result = await response.Content.ReadAsStringAsync();

                Console.WriteLine("\nTranslation Result:");
                Console.WriteLine(result);
            }
        }
    }
}
2.	Replace <your-resource-name> and <your-key> with your Azure Translator endpoint and API key.
________________________________________
Step 4: Run and Test the Application
1.	Press F5 to build and run the application.
2.	Input a sentence in English (e.g., "Hello, how are you?") and observe the translated output in French.
3.	Modify the targetLanguage variable to try different languages (e.g., "es" for Spanish, "de" for German).
________________________________________
Bonus Activity
•	Modify the code to detect the language of the input text automatically by calling the detect endpoint.
•	Experiment with translating text into multiple languages simultaneously by appending additional &to=<language-code> parameters in the route.
________________________________________
Expected Output
•	Input: Hello, how are you?
•	Output: Bonjour, comment ça va ?
Would you like me to provide additional tasks, such as handling errors or improving the interface?

