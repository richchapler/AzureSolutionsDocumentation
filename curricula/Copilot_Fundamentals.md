# Copilot: Fundamentals

> This is a draft outline for a possible curriculum

# Copilot Studio

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Overview & Basics | Introduction to the interface and core concepts, defining agent purpose, personality, and instructions | Copilot Studio; Azure OpenAI; Low-code design |
| Designing Conversation Flows & Topics | Building multi-turn dialog flows with trigger phrases, branching, and context management | NLU intent matching; Dialog management; Power Virtual Agents |
| Prompting and Generative Responses | Crafting system/user prompts to guide the LLM’s tone and generate dynamic responses | Azure OpenAI GPT; Prompt engineering; Adaptive response generation |
| Adding Knowledge Sources for Context | Connecting documents, websites, or FAQs as knowledge bases to ground agent answers | Knowledge integration; Content indexing; Contextual Q\&A |
| Agent Flows for External Actions | Creating trigger-based workflows that let the agent call APIs or perform tasks during conversation | Power Automate agent flows; Connectors & plugins; Event triggers |

# Data Agent Capabilities (Conversational Data Access)

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Fabric Data Agents: Overview & Use Cases | Using data agents for natural-language Q\&A over organizational data (lakehouses, warehouses, Power BI) | Microsoft Fabric OneLake; Power BI semantic models; LLM-powered NL→Query |
| Setting Up a Fabric Data Agent | Configuring data connections and customizing agent instructions for enterprise data querying | Lakehouse/Warehouse connectivity; Data agent configuration; Custom context prompts |
| Querying Power BI Data with Natural Language | Connecting a Power BI dataset to the agent and asking plain-English questions for insights | Power BI integration; NL-to-DAX query generation; Results visualization |

# Integration Scenarios with Microsoft Products

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Copilot in Microsoft Teams | Publishing the agent as a Teams chatbot, using @mentions and adaptive cards for rich responses | Azure Bot Service; Teams App integration; Adaptive Cards |
| Copilot for Power BI Users | Embedding an AI assistant in reports/dashboards to answer questions and provide insights on demand | Power BI Service; Q\&A embedding; Power Automate integration |
| Copilot in Power Apps | Embedding chat interfaces in Canvas apps to guide users, fill forms, or trigger app actions | Power Apps Canvas; Embedded AI chat; Power Platform connectors |
| Outlook and Office 365 Integration | Conceptual extension of agents into email and calendar workflows via Microsoft Graph | Outlook Add-ins; Microsoft Graph API; 365 Copilot extensibility |
