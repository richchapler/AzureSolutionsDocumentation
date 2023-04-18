# Application Registration
_(aka "Application", "Client", "Security Principal", "Service Principal")_

<img src="https://user-images.githubusercontent.com/44923999/185750680-8bb8b009-3820-47de-8957-a206628516d7.png" width="1000" />

[Quickstart: Register an application with the Microsoft identity platform](https://learn.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app) and related documentation should serve as your primary source of information.

_Notes_<br>
_* Consider using a **System-Assigned Managed Identity** a **User-Assigned Managed Identity** rather than an Application Registration to minimize operational burden {e.g., maintenance of secrets, system downtime, etc.}_<br>
_* You might use a multi-tenant Service Principal to support integration with third-party applications that prompt only for Client Id and Client Secret_

## Create with PowerShell

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

<img src="https://user-images.githubusercontent.com/44923999/232788882-f8bc59c9-2886-4601-9432-c5188c1df872.png" width="800" title="Snipped: April 18, 2023" />

Modify, copy / paste, and then run the following command:

``` PowerShell
New-AzADServicePrincipal -DisplayName {SERVICE_PRINCIPAL_NAME}
```

You can expect a result like:

```
DisplayName Id                                   AppId
----------- --                                   -----
rchaplersp  64429f26-f1cf-4bd2-8d55-fea37532e180 8c57558e-0a36-41c4-bd98-6b7efeb23fab
```
