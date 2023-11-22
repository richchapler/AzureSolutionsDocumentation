# Open AI: Estimating Cost

## Pricing Calculator

Navigate to the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator).

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2853f78b-eb98-4a20-bef7-fe4e3511fa64" width="800" title="Snipped: November 20, 2023" />

Search for "Azure OpenAI Service" and then click "Add to estimate".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8ec80e82-2428-4600-ba16-802d0b4b0349" width="800" title="Snipped: November 20, 2023" />

Scroll down and complete the "Azure OpenAI Service" form, including:

Prompt | Description
:----- | :-----
Model Type | * **Language Models** {e.g., GPT-4-8K}: used to add natural language capabilities, build conversational interfaces, identify key terms and phrases, understand sentiments, and more<br><br>* **Image Models** {e.g., Dall-E}: used for image classification, object detection, and more<br><br>* **Embedding Models** {e.g., Ada}: used to convert text into numerical vectors that capture the semantic meaning of the words or sentences
Prompt | Refers to the number of tokens {i.e., four characters of typical English text} in the prompts you send to the model 
Completion | refers to the number of tokens in the completions (responses) you receive from the model

### Token
The prompt `Write the Python to truncate a decimal into an integer` is 9 tokens long:
1. "Write"
2. " the"
3. " Python"
4. " to"
5. " truncate"
6. " a"
7. " decimal"
8. " into"
9. " an integer"

Each word or whitespace-separated character sequence is considered a token.

_Note: this is a simplified explanation and the actual tokenization process can be more complex, especially for languages other than English_

### Completion
The number of completions would depend on the maximum token limit set for the model's response. For example, if you set a maximum limit of 100 tokens for the model's response, and your prompt is 9 tokens long, then the model has 91 tokens left for the completion.

_Note: both the prompt and the completion contribute to the total number of tokens used in a call. If the total token count exceeds the model's maximum limit, the call will fail. Also, very long generations are more likely to be cut off._

## Reference
* [Plan to manage costs for Azure OpenAI Service](https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/manage-costs)
* [What are tokens and how to count them?](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them)
