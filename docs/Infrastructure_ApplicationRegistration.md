## Application Registration
_(aka "Application", "Client", "Service Principal")_<br>

![image](https://user-images.githubusercontent.com/44923999/185750680-8bb8b009-3820-47de-8957-a206628516d7.png)

_Notes_<br>
_* Consider using a **System-Assigned Managed Identity** a **User-Assigned Managed Identity** rather than an Application Registration to minimize operational burden {e.g., maintenance of secrets, system downtime, etc.}_<br>
_* You might use a multi-tenant Service Principal to support integration with third-party applications that prompt only for Client Id and Client Secret_

### Create with Azure Portal

Complete the following steps:

* Navigate to "**Azure Active Directory**" and click on "**App Registrations**" in the Manage group of the navigation pane
* Click "**+ New Registration**"
* Complete the "**Register an application**" form

  <img src="https://user-images.githubusercontent.com/44923999/178037482-52960bbb-3b19-4950-9e44-646d98e9d3a4.png" width="600" title="Snipped: July 8, 2022" />
  
* Click **Register**

_Note: Consider generating a Key Vault Secret for any Client Identifier and Client Secret values_
