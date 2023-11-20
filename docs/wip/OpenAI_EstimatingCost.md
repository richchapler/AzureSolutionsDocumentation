# Open AI: Estimating Cost

## Pricing Calculator

Navigate to the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator).

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2853f78b-eb98-4a20-bef7-fe4e3511fa64" width="800" title="Snipped: November 20, 2023" />

Search for "Azure OpenAI Service" and then click "Add to estimate".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8ec80e82-2428-4600-ba16-802d0b4b0349" width="800" title="Snipped: November 20, 2023" />

Scroll down and complete the "Azure OpenAI Service" form, including:

Prompt | Description
:----- | :-----
Model Type | This refers to the type of AI model you want to use. Azure OpenAI Service offers different types of models, each with its own capabilities and use cases. For example, there are language models (like GPT-3.5-Turbo and GPT-4), base models (like Babbage-002 and Davinci-002), and fine-tuning models
Model | This is the specific model within the chosen type that you want to use. Each model has different performance characteristics and costs. For example, within the language models, you could choose between GPT-3.5-Turbo 4K, GPT-3.5-Turbo 16K, GPT-4 8K, and GPT-4 32K
Prompt | This refers to the number of tokens in the prompts you send to the model. A token is roughly equivalent to four characters for typical English text². You're charged per 1,000 tokens for the prompts you send to the model
Completion | This refers to the number of tokens in the completions (responses) you receive from the model. You're also charged per 1,000 tokens for the completions you receive from the model






## Reference
https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/manage-costs
