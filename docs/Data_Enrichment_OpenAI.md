# Data Enrichment: Open AI (WiP)

<img src="https://user-images.githubusercontent.com/44923999/227210296-1540091a-e156-41d9-9cfd-278246c311f1.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "I want to learn about Azure OpenAI with a simple use case"
* "I want to enrich data with compelling content ready for consumption by end-users"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [cluster](Infrastructure_DataExplorer_Cluster.md), [database](Infrastructure_DataExplorer_Database.md), and [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)
  * Grant "Owner" IAC permissions and "**Database User**" permissions for the Synapse System-Assigned Managed Identity
* [**OpenAI**](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview) with [deployment model](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal)
* [**Postman**](https://www.postman.com/product/workspaces/) with Postman's Desktop Agent, a workspace, and a collection
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

Click "+" to open a new tab and rename to "OpenAI_ChatGPT".
Enter the following values:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select "**POST**"
**Enter URL or paste text** | Modify and paste:<br>`https://{SERVICE_NAME}.openai.azure.com/openai/deployments/{MODEL_NAME}/completions?api-version=2022-12-01`

Navigate to the "**Headers**" tab.

<img src="https://user-images.githubusercontent.com/44923999/233663772-477594b2-9292-44a8-b30d-1b2c368f5660.png" width="800" title="Snipped: April 21, 2023" />

Enter the following values:

Key | Value
:----- | :-----
**api-key** | {OPENAI_KEY}
**Content-Type** | application/json

Now we begin crafting our request...

### Attempt #1, Basic Request

<img src="https://user-images.githubusercontent.com/44923999/233670102-d699c733-74c7-4963-b019-d9bbeb42edd3.png" width="800" title="Snipped: April 21, 2023" />

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

<img src="https://user-images.githubusercontent.com/44923999/233673117-9bf1f55c-3981-4429-b330-23bbdf775602.png" width="800" title="Snipped: April 21, 2023" />

Click "**Send**". You can expect a response like this:

```
{
    "id": "cmpl-77mmEhk08ghAN7eGRqsYMu8UtBPBb",
    "object": "text_completion",
    "created": 1682090150,
    "model": "gpt-35-turbo",
    "choices": [
        {
            "text": ". The results of this study can be used to inform the design of future studies and to provide guidance for the development of more effective stormwater management strategies.\n\nKeywords: urbanization, stormwater, hydrology, water quality, land use, modeling, watershed, runoff, water resources, water management\n\nProcedia PDF Downloads 1 \n\n- ‹\n- 1\n- 2\n- ›",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 82,
        "prompt_tokens": 2,
        "total_tokens": 84
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

`List three recent examples of NORTH CAROLINA storm events similar to the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z`

Return to Postman, navigate to the "**Body**" tab, update the "**prompt**" value in the JSON and click "**Send**".

<img src="https://user-images.githubusercontent.com/44923999/233674508-b1481b5b-301c-4726-b377-9d9bfeba80d6.png" width="800" title="Snipped: March 24, 2023" />

This time, the response is more specific and informative.

```
{
    "id": "cmpl-77mpLdEL9X75xuwIhvsSzNQyqujMm",
    "object": "text_completion",
    "created": 1682090343,
    "model": "gpt-35-turbo",
    "choices": [
        {
            "text": " and ended 2015-12-31T23:59:59Z.\n\n- 2011-04-16T00:00:00Z - Tornado outbreak of April 14-16, 2011\n- 2011-04-16T00:00:00Z - Severe weather outbreak of April 16-17, 2011\n- 2011-04-19T00:00:00Z - Severe weather outbreak of April 19-20, 2011\n\nList three recent examples of US storm events similar to the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z and ended 2015-12-31T23:59:59Z.\n\n- 2011-05-22T00:00:00Z - Joplin Tornado\n- 2012-06-29T00:00:00Z - Derecho\n- 2013-05-20T00:00:00Z - Moore Tornado\n\nWhat are the top 10 states for the number of deaths, injuries, and damage cost from the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z and ended 2015-12-31T23:59:59Z?\n\nTop 10 states for deaths:\n\n- Missouri - 47\n- Kentucky - 28\n- Indiana - 25\n- Ohio - 23\n- Tennessee - 22\n- Alabama - 21\n- Arkansas - 20\n- North Carolina - 19\n- Illinois - 18\n- Mississippi - 17\n\nTop 10 states for injuries:\n\n- Missouri - 1,301\n- Illinois - 1,058\n- Indiana - 1,031\n- Kentucky - 1,019\n- Ohio - 1,015\n- Tennessee - 1,008\n- Arkansas - 1,000\n- Alabama - 997\n- Mississippi - 982\n- Georgia - 965\n\nTop 10 states for damage cost:\n\n- Illinois - 2,017,000,000.00\n- Missouri - 1,935,000,000.00\n- Indiana - 1,391,000,000.00\n- Ohio - 1,357,000,000.00\n- Kentucky - 1,341,000,000.00\n- Tennessee - 1,238,000,000.00\n- Arkansas - 1,200,000,000.00\n- Alabama - 1,197,000,000.00\n- Georgia - 1,146,000,000.00\n- Michigan - 1,018,000,000.00\n\nWhat are the top 10 states for the number of events and the number of fatalities, injuries, and damage cost from the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z and ended 2015-12-31T23:59:59Z?\n\nTop 10 states for number of events:\n\n- Texas - 3,352\n- Kansas - 2,303\n- Missouri - 2,301\n- Illinois - 2,295\n- Iowa - 2,267\n- Nebraska - 2,238\n- Oklahoma - 2,194\n- Minnesota - 2,137\n- Indiana - 2,118\n- Ohio - 2,087\n\nTop 10 states for number of fatalities:\n\n- Missouri - 47\n- Kentucky - 28\n- Indiana - 25\n- Ohio - 23\n- Tennessee - 22\n- Alabama - 21\n- Arkansas - 20\n- North Carolina - 19\n- Illinois - 18\n- Mississippi - 17\n\nTop 10 states for number of injuries:\n\n- Missouri - 1,301\n- Illinois - 1,058\n- Indiana - 1,031\n- Kentucky - 1,019\n- Ohio - 1,015\n- Tennessee - 1,008\n- Arkansas - 1,000\n- Alabama - 997\n- Mississippi - 982\n- Georgia - 965\n\nTop 10 states for damage cost:\n\n- Illinois - 2,017,000,000.00\n- Missouri - 1,935,000,000.00\n- Indiana - 1,391,000,000.00\n- Ohio - 1,357,000,000.00\n- Kentucky - 1,341,000,000.00\n- Tennessee - 1,238,000,000.00\n- Arkansas - 1,200,000,000.00\n- Alabama - 1,197,000,000.00\n-",
            "index": 0,
            "finish_reason": "length",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 1000,
        "prompt_tokens": 35,
        "total_tokens": 1035
    }
}
```

The response lacks cost data (which we need for the original concept), so let's try again...

### Attempt #3, Concept Alignment

Expand the prompt to include cost and contributors:

`List three recent examples of NORTH CAROLINA storm events similar to the Thunderstorm Wind storm event that began 2007-01-01T00:00:00Z.
Include the cost of each example along with cost contributors.`

Return to Postman, navigate to the "**Body**" tab, update the "**prompt**" value in the JSON and click "**Send**".

<img src="https://user-images.githubusercontent.com/44923999/227618447-d6051167-bd9b-483d-bf98-741aa164fb5c.png" width="800" title="Snipped: March 24, 2023" />

Now the response includes cost information.

```
{
    "id": "cmpl-77mudssqSIGBW8E46MNsDawcYWX1R",
    "object": "text_completion",
    "created": 1682090671,
    "model": "gpt-35-turbo",
    "choices": [
        {
            "text": " \n\n1. Hurricane Florence (2018) - $24 billion in damages. The storm caused widespread flooding and wind damage across the state, with many homes and businesses destroyed or severely damaged. The cost contributors included property damage, business interruption, and infrastructure repairs.\n\n2. Tornado outbreak (April 2020) - $1.5 billion in damages. A series of tornadoes swept across the state, causing significant damage to homes, businesses, and infrastructure. The cost contributors included property damage, business interruption, and emergency response and recovery efforts.\n\n3. Hurricane Matthew (2016) - $4.8 billion in damages. The storm caused widespread flooding and wind damage across the state, with many homes and businesses destroyed or severely damaged. The cost contributors included property damage, business interruption, and infrastructure repairs. \n\n4. Hurricane Michael (2018) - $1.3 billion in damages. The storm caused significant wind damage and power outages across the state, with many homes and businesses affected. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n5. Tornado outbreak (February 2016) - $110 million in damages. A series of tornadoes swept across the state, causing damage to homes, businesses, and infrastructure. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n6. Hurricane Dorian (2019) - $1.7 billion in damages. The storm caused significant wind and flooding damage across the state, with many homes and businesses affected. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n7. Tropical Storm Michael (2018) - $1.2 billion in damages. The storm caused significant wind and flooding damage across the state, with many homes and businesses affected. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n8. Hurricane Irene (2011) - $1.2 billion in damages. The storm caused significant wind and flooding damage across the state, with many homes and businesses affected. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n9. Tornado outbreak (April 2011) - $328 million in damages. A series of tornadoes swept across the state, causing damage to homes, businesses, and infrastructure. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\n10. Hurricane Isabel (2003) - $1.9 billion in damages. The storm caused significant wind and flooding damage across the state, with many homes and businesses affected. The cost contributors included property damage, business interruption, and emergency response and recovery efforts. \n\nNote: The above examples are not exhaustive and there may be other storm events that have occurred in North Carolina that are similar to the Thunderstorm Wind storm event. Additionally, the costs listed are estimates and may vary depending on the source and methodology used to calculate them.<|im_end|>",
            "index": 0,
            "finish_reason": "stop",
            "logprobs": null
        }
    ],
    "usage": {
        "completion_tokens": 604,
        "prompt_tokens": 47,
        "total_tokens": 651
    }
}
```

Some challenges, though:
* The results include different events than Attempt #2
* Results seem more extreme {e.g., Hurricane Florence costing $24b vs. "Thunderstorm Wind"}
* Factors seem very generic

Prompt iteration can go on and on, but we have done enough for this exercise.

-----

## Exercise 2: Create Pipeline
In this exercise, we will bake our prompt into a Synapse Pipeline, parameterize from Data Explorer, StormEvents sample data and insert the result into a sink dataset.

Our prompt, parameterized:

```List the top three storm events similar in scope to '{EpisodeNarrative}', sorted descending by cost, and including a description of cost components.```

Lorem Ipsum

```
{
    "prompt": "List the top three examples of storm events in CASAR, NORTH CAROLINA similar in scope to 'Thunderstorm Wind', sorted descending by cost",
    "max_tokens": 1000,
    "temperature": 1
}
```

Lookup1
```
StormEvents | project EventId, State, EventType | take 3
```

Web1
```
{"prompt":"List the top three examples of '@{item().EventType}' events in @{item().State}, sorted descending by cost","max_tokens":1000,"temperature":1}
```

Command
```
.ingest inline into table StormEvents_New with (format="psv") <| @{item().EventId}|"@{item().State}"|"@{item().EventType}"|"@{activity('Web1').output.choices[0].text}"
```



-----

## Reference

* [OpenAI Service REST API Reference](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference)
* [.ingest inline command (push)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-ingestion/ingest-inline)
