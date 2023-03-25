# Data Enrichment: Open AI (WiP)

<img src="https://user-images.githubusercontent.com/44923999/227210296-1540091a-e156-41d9-9cfd-278246c311f1.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "I want to learn about Azure OpenAI with a simple use case"
* "I want to enrich data with compelling content ready for consumption by end-users"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [cluster](Infrastructure_DataExplorer_Cluster.md), [database](Infrastructure_DataExplorer_Database.md), and [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard) with "Owner" IAC permissions and "**Database User**" permissions granted to the Synapse System-Assigned Managed Identity
* [**OpenAI**](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview) with [deployment model](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal)
* [**Postman**](https://www.postman.com/product/workspaces/) with a workspace and collection
* [**Synapse**](Infrastructure_Synapse.md)

## Proposed Solution

Concept: Enrich Data Explorer's sample dataset, "StormEvents" with cost data for similar, recent events.<br>
_{i.e., "these three similar storm events had X damages that cost $Y to repair"}_

This documentation will address solution requirements in two exercises:

* Exercise 1: Prepare Prompt
* Exercise 2: Create Data Flow

-----

## Exercise 1: Prepare Prompt
In this exercise, we will iteratively generate a prompt that can be used with an OpenAI DaVinci model.

Navigate to your Postman workspace and collection,<br>
_{e.g., https://web.postman.co/workspace/My-Workspace~00000000-0000-0000-0000-000000000000}_

Click "+" to open a new tab and rename to "OpenAI_DaVinci".
Enter the following values:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select "**POST**"
**Enter URL or paste text** | Modify and paste:<br>`https://rchaplerai.openai.azure.com/openai/deployments/rchapleraimd/completions?api-version=2022-12-01`

Navigate to the "**Headers**" tab.

<img src="https://user-images.githubusercontent.com/44923999/227531164-07b77089-5142-4db1-8d3e-05a5b2a79381.png" width="800" title="Snipped: March 24, 2023" />

Enter the following values:

Key | Value
:----- | :-----
**api-key** | your OpenAI Key
**Content-Type** | application/json

<img src="https://user-images.githubusercontent.com/44923999/227530522-a2adbc66-42f6-4102-ad21-acb8d5fb39fb.png" width="800" title="Snipped: March 24, 2023" />

Now we begin crafting our request...

### Attempt #1, Basic Request

Navigate to the "**Body**" tab and paste the following JSON:

```
{
    "prompt":"storm events",
    "max_tokens":1000,
    "temperature":0.10
}
```

Logic Explained:
* `prompt`... the query for which to generate a completion; our strategy will be to start with something very simple and hone in later steps
* `max_tokens`... establishes context length
* `temperature`... closer to 0 for greater accuracy and closer to 1 for more creativity

Click "**Send**".

<img src="https://user-images.githubusercontent.com/44923999/227566583-85bceb5f-83ea-4ad8-a7d2-9df6b3056a50.png" width="800" title="Snipped: March 24, 2023" />

We can expect a response like this:

```
{
    "id": "cmpl-6xdR7jo4K0ZT9bRKmzV72YmTSPN6X",
    "object": "text_completion",
    "created": 1679670965,
    "model": "text-davinci-003",
    "choices": [
        {
            "text": ", and the effects of climate change on stormwater management.\n\nThe course will also cover the design of stormwater management systems, including green infrastructure, and the use of computer models to simulate stormwater runoff. Students will learn how to design and implement stormwater management systems that are cost-effective, efficient, and environmentally friendly. In addition, students will gain an understanding of the regulatory requirements for stormwater management and the importance of public engagement in the process.",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 93,
        "prompt_tokens": 2,
        "total_tokens": 95
    }
}
```

This start is a good test of the API, but not very specific to the concept.<br>
We need to develop our understanding of source data that we might use to enhance our prompt.

### Attempt #2, Parameter Preparation

Navigate to your Data Explorer database, then Query in the navigation pane.
Enter `StormEvents | take 1` and then click "**Run**".<br>

<img src="https://user-images.githubusercontent.com/44923999/227569221-98668b58-0089-4884-bfec-ff762b17b52c.png" width="800" title="Snipped: March 24, 2023" />

A quick review of the data provides some interesting options:

Column | Data
:----- | :-----
**Start Time** | 2007-01-01T00:00:00Z
**State** | NORTH CAROLINA
**Event Type** | Thunderstorm Wind

With this sort of data we might develop a prompt like:

`List three recent examples of **NORTH CAROLINA** storm events similar to the **Thunderstorm Wind** storm event that began **2007-01-01T00:00:00Z**`

Return to Postman, navigate to the "**Body**" tab, update the "**prompt**" value in the JSON and click "**Send**".

<img src="https://user-images.githubusercontent.com/44923999/227571968-4c3c096b-5e11-4c33-8ede-51c411da8e6a.png" width="800" title="Snipped: March 24, 2023" />

This time, the response is more specific and informative.

```
{
    "id": "cmpl-6xdiHSdO5nXSHhbULeET3szA9C9tx",
    "object": "text_completion",
    "created": 1679672029,
    "model": "text-davinci-003",
    "choices": [
        {
            "text": "\n\n1. Tropical Storm Isaias (2020): This storm caused widespread wind damage and power outages across North Carolina in August 2020.\n\n2. Winter Storm Diego (2018): This storm brought heavy snow and ice to North Carolina in December 2018, causing numerous power outages and hazardous travel conditions.\n\n3. Hurricane Florence (2018): This storm caused extensive flooding and wind damage across the state in September 2018, resulting in numerous fatalities and billions of dollars in damage.",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 98,
        "prompt_tokens": 34,
        "total_tokens": 132
    }
}
```

The response lacks cost data (which we need for the original concept), so let's try again...

### Attempt #3, Concept Alignment

Expand the prompt to include cost and contributors:

`List three recent examples of NORTH CAROLINA storm events similar to the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z.
**Include the cost of each example along with cost contributors.**`

Return to Postman, navigate to the "**Body**" tab, update the "**prompt**" value in the JSON and click "**Send**".

<img src="https://user-images.githubusercontent.com/44923999/227618447-d6051167-bd9b-483d-bf98-741aa164fb5c.png" width="800" title="Snipped: March 24, 2023" />

Now the response includes cost information.

```
{
    "id": "cmpl-6xdpkH53KdkQBf126qdVqkPrY8ViP",
    "object": "text_completion",
    "created": 1679672492,
    "model": "text-davinci-003",
    "choices": [
        {
            "text": "\n\n1. Tropical Storm Michael (2018): Cost: $1.5 billion; Contributors: Flooding, wind damage, and power outages.\n\n2. Hurricane Florence (2018): Cost: $17 billion; Contributors: Flooding, wind damage, and power outages.\n\n3. Hurricane Dorian (2019): Cost: $3.5 billion; Contributors: Flooding, wind damage, and power outages.",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 93,
        "prompt_tokens": 46,
        "total_tokens": 139
    }
}
```

Some challenges, though:
* The results include different events than Attempt #2
* Results seem more extreme {e.g., Tropical Storm costing $1.5b vs. "Thunderstorm Wind"}
* Factors seem very generic

### Attempt #4, Response Normalization

Going back to the available Data Explorer data, we find column "EpisodeNarrative" which provides additional detail for comparison, and can be used in place of some of the more granular columns.

`List the top three storm events similar in scope to 'A small cluster of thunderstorms moved rapidly across the foothills and piedmont of western North Carolina, producing scattered wind damage.', sorted descending by cost, and including a description of cost components.`

What's changed...
* `top three... sorted descending by cost`... I gave specific instruction regarding what examples matter
* `similar in scope to`... I provided a qualifier and switched from granular columns to a single descriptive column "EpisodeNarrative"
* `a description of cost components`... I was more clear how to present contributors to cost

And, a better result:

```
{
    "id": "cmpl-6xhJaHRiBqV0hjLFr42bLLrWYPUHE",
    "object": "text_completion",
    "created": 1679685874,
    "model": "text-davinci-003",
    "choices": [
        {
            "text": "\n\n1. The 2018 Western North Carolina Windstorm: This storm caused widespread wind damage across the foothills and piedmont of western North Carolina, resulting in an estimated $50 million in damages. The storm caused downed trees, power outages, and structural damage to homes and businesses.\n\n2. The 2016 Western North Carolina Windstorm: This storm caused widespread wind damage across the foothills and piedmont of western North Carolina, resulting in an estimated $30 million in damages. The storm caused downed trees, power outages, and structural damage to homes and businesses.\n\n3. The 2014 Western North Carolina Windstorm: This storm caused widespread wind damage across the foothills and piedmont of western North Carolina, resulting in an estimated $20 million in damages. The storm caused downed trees, power outages, and structural damage to homes and businesses.",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 177,
        "prompt_tokens": 50,
        "total_tokens": 227
    }
}
```

Prompt iteration does not end here, but we have done enough for this exercise.

-----

## Exercise 2: Create Data Flow
In this exercise, we will bake our prompt into a Data Flow, parameterize from Data Explorer, StormEvents sample data and insert the result into a sink dataset.

Our prompt, parameterized:

```List the top three storm events similar in scope to '{EpisodeNarrative}', sorted descending by cost, and including a description of cost components.```

Lorem Ipsum

-----

## Reference

* [OpenAI Service REST API Reference](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference)
* [.ingest inline command (push)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-ingestion/ingest-inline)
