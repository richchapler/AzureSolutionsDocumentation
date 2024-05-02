| Use Case: Lorem Ipsum |  
| --- |  
| AI Transformation for Improved Resolution Time<br><br><sub>**Title**: Clear, concise title that reflects what the use case is about</sub> <br>   |  
| **Actor(s)** <br> <sub>Person or system that will be interacting with the system</sub> <br> Jira, Logic App, User <br> <br> |  
| **Precondition(s)** <br> <sub>State of the system before the use case starts</sub> <br> A trigger is set off in Jira <br> <br> |  
| **Basic Flow** <br> <sub>Sequence of events when everything goes as expected</sub> <br> 1. Jira triggers the AI transformation <br> 2. Logic App sends a Teams chat to the user <br> 3. User sends back a message to Teams <br> 4. The message is processed <br> <br> |  
| **Alternative Flow(s)** <br> <sub>Sequence of events when something does not go as expected</sub> <br> If the user does not respond to the Teams chat, a follow-up message is sent <br> <br> |  
| **Postcondition(s)** <br> <sub>State of the system after the use case ends</sub> <br> The user's message has been processed and the issue in Jira is updated <br> <br> |  
| **Exception Flow(s)** <br> <sub>Sequence of events when there are errors or exceptions</sub> <br> If the Logic App fails to send the Teams chat, an error is logged and the process is retried <br> <br> |  
| **Priority** <br> <sub>Importance of the use case in relation to other use cases</sub> <br> High - This use case directly impacts the Mean Time to Resolution <br> <br> |  
| **Frequency of Use** <br> <sub>How often the use case is expected to be performed</sub> <br> This use case is triggered every time there is a new issue in Jira <br> <br> |  
| **Business Rule(s)** <br> <sub>Any business rules that apply to the use case</sub> <br> The user must respond to the Teams chat within a certain time frame <br> <br> |  
| **Special Requirement(s)** <br> <sub>Any special requirements that apply to the use case</sub> <br> The Logic App must have the capability to send Teams chats <br> <br> |  
 
