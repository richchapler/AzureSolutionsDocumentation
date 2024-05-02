| Use Case: Lorem Ipsum |  
| --- |  
| **Actor(s)** <br> Jira, Logic App, User <br> <br> |  
| **Precondition(s)** <br> A trigger is set off in Jira <br> <br> |  
| **Basic Flow** <br> 1. Jira triggers the AI transformation <br> 2. Logic App sends a Teams chat to the user <br> 3. User sends back a message to Teams <br> 4. The message is processed <br> <br> |  
| **Alternative Flow(s)** <br> If the user does not respond to the Teams chat, a follow-up message is sent <br> <br> |  
| **Postcondition(s)** <br> The user's message has been processed and the issue in Jira is updated <br> <br> |  
| **Exception Flow(s)** <br> If the Logic App fails to send the Teams chat, an error is logged and the process is retried <br> <br> |  
| **Priority** <br> High - This use case directly impacts the Mean Time to Resolution <br> <br> |  
| **Frequency of Use** <br> This use case is triggered every time there is a new issue in Jira <br> <br> |  
| **Business Rule(s)** <br> The user must respond to the Teams chat within a certain time frame <br> <br> |  
| **Special Requirement(s)** <br> The Logic App must have the capability to send Teams chats <br> <br> |  
