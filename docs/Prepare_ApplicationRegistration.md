## Application Registration
_(aka "Application", "Client", "Service Principal")_<br><br>
_Notes_<br>
_* Minimize operational burden {e.g., maintenance of Client Secrets, system downtime, etc.} by using a User-Assigned **Managed Identity** instead of an Application Registration_
_* Consider using a multi-tenant Service Principal to support integration with third-party applications_

### Create with Azure Portal

* Navigate to **Azure Active Directory** and click on **App Registrations** in the Manage group of the navigation pane
* Click **+ New Registration**
* Complete the **Register an application** form

  <img src="https://user-images.githubusercontent.com/44923999/178037482-52960bbb-3b19-4950-9e44-646d98e9d3a4.png" width="600" title="Snipped: July 8, 2022" />
  
* Click **Register**

_Note: Consider generating a Key Vault Secret for both Client Identifier and Client Secret values_
