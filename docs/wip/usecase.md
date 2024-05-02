| Use Case: Improved Mean Time-to-Resolution |  
:-----
| This use case describes the process of using AI to improve the resolution time of issues. It involves the interaction between Jira, a Logic App, and a User. <br> <br> <sub>**Description**: Clear, concise description that reflects what the use case is about</sub> <br> |    
| Jira, Logic App, User <br> <br> <sub>**Actor(s)**: Person or system that will be interacting with the system</sub> <br> |  
| A trigger is set off in Jira <br> <br> <sub>**Precondition(s)**: State of the system before the use case starts</sub> <br> |  
| 1. Jira triggers the AI transformation <br> 2. Logic App sends a Teams chat to the user <br> 3. User sends back a message to Teams <br> 4. The message is processed <br> <br> <sub>**Basic Flow**: Sequence of events when everything goes as expected</sub> <br> |  
| If the user does not respond to the Teams chat, a follow-up message is sent <br> <br> <sub>**Alternative Flow(s)**: Sequence of events when something does not go as expected</sub> <br> |  
| The user's message has been processed and the issue in Jira is updated <br> <br> <sub>**Postcondition(s)**: State of the system after the use case ends</sub> <br> |  
| If the Logic App fails to send the Teams chat, an error is logged and the process is retried <br> <br> <sub>**Exception Flow(s)**: Sequence of events when there are errors or exceptions</sub> <br> |  
| High - This use case directly impacts the Mean Time to Resolution <br> <br> <sub>**Priority**: Importance of the use case in relation to other use cases</sub> <br> |  
| This use case is triggered every time there is a new issue in Jira <br> <br> <sub>**Frequency of Use**: How often the use case is expected to be performed</sub> <br> |  
| The user must respond to the Teams chat within a certain time frame <br> <br> <sub>**Business Rule(s)**: Any business rules that apply to the use case</sub> <br> |  
| The Logic App must have the capability to send Teams chats <br> <br> <sub>**Special Requirement(s)**: Any special requirements that apply to the use case</sub> <br> |  
