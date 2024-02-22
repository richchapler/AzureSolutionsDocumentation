# Step-by-step instructions to configure Azure, code, and publish a Razor Pages web application  
   
This guide will help you create a web application that lists all blob containers in an Azure Storage Account using Managed Identity.   
  
## Step 1: Create a new Razor Pages web application in Visual Studio  
   
- Open Visual Studio and click on "Create a new project".  
- Choose "ASP.NET Core Web Application" and click "Next".  
- Enter your project name and location, then click "Create".  
- Choose ".NET Core" and "ASP.NET Core 3.1" (or later), select "Web Application", and click "Create".  
   
## Step 2: Install the Azure.Storage.Blobs and Azure.Identity NuGet packages  
   
- Right-click on your project in Solution Explorer and select "Manage NuGet Packages".  
- Click on "Browse", search for "Azure.Storage.Blobs", and click "Install".  
- Repeat the process for the "Azure.Identity" package.  
   
## Step 3: Update the IndexModel in Index.cshtml.cs  
   
- Open the Index.cshtml.cs file located in the Pages folder.  
- Replace the existing code with the following:  
   
```csharp  
using Azure.Identity;  
using Azure.Storage.Blobs;  
using Microsoft.AspNetCore.Mvc.RazorPages;  
using System;  
using System.Collections.Generic;  
using System.Threading.Tasks;  
   
namespace YourNamespace.Pages  
{  
    public class IndexModel : PageModel  
    {  
        public List<string> Containers { get; set; }  
  
        public async Task OnGetAsync()  
        {  
            var credential = new DefaultAzureCredential();  
            BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri("<your-storage-account-url>"), credential);  
            var blobContainers = blobServiceClient.GetBlobContainersAsync();  
            Containers = new List<string>();  
  
            await foreach (var blobContainer in blobContainers)  
            {  
                Containers.Add(blobContainer.Name);  
            }  
        }  
    }  
}  
```  
- Replace "YourNamespace" with the namespace of your project and "<your-storage-account-url>" with the URL of your Azure Storage Account.  
   
## Step 4: Update the Index view in Index.cshtml  
   
- Open the Index.cshtml file located in the Pages folder.  
- Replace the existing code with the following:  
   
```html  
@page  
@model YourNamespace.Pages.IndexModel  
   
<h2>Blob Containers</h2>  
   
<ul>  
    @foreach (var container in Model.Containers)  
    {  
        <li>@container</li>  
    }  
</ul>  
```  
- Replace "YourNamespace" with the namespace of your project.  
   
## Step 5: Run the application locally  
   
- Press F5 or click on the "Start Debugging" button.  
   
## Step 6: Create an App Service in Azure  
   
- In the Azure portal, click on "Create a resource".  
- Search for "App Service" and click "Create".  
- Fill in the details like subscription, resource group, name, runtime stack, region, etc. and click "Review + Create" then "Create".  
   
## Step 7: Enable Managed Identity for the App Service  
   
- In the Azure portal, navigate to your App Service.  
- In the left-hand menu, click on "Identity".  
- In the "System assigned" tab, switch the "Status" to "On" and click "Save".  
   
## Step 8: Grant the Managed Identity access to your Azure Storage Account  
   
- Navigate to your Azure Storage Account.  
- In the left-hand menu, click on "Access control (IAM)".  
- Click on "+ Add" > "Add role assignment".  
- Select the "Storage Blob Data Reader" role, select "User assigned managed identity" in the "Assign access to" dropdown, and select your App Service in the "User assigned managed identity" dropdown.  
- Click "Save".  
   
## Step 9: Publish the application to Azure  
   
- In Visual Studio, - Right-click on your project in Solution Explorer and select "Publish".  
- In the "Pick a publish target" window, select "Azure" and click "Next".  
- Choose "Windows" or "Linux" depending on your preference and click "Next".  
- In the "App Service" window, select your existing App Service and click "Finish".  
- Click on "Publish".  
  
## Step 10: Test the application  
  
- After the application is published, navigate to the URL of your Azure App Service. It should display a list of all blob containers in your Azure Storage Account.  
  
## Step 11: Register your application with Azure AD    
    
- In the Azure portal, go to "Azure Active Directory".    
- Click on "App registrations" and then "New registration".    
- Enter a name for your application, select the supported account types, and then enter the Redirect URI (the URL where users are sent after they are authenticated, which is typically the base URL of your app).    
- Click "Register".    
- After the application is registered, click on "Authentication" in the left-hand menu.  
- In the "Implicit grant and hybrid flows" section, check the box for "ID tokens".  
- Click on "Save" at the top of the page.  
- In the "Redirect URIs" section, click on "+ Add a platform".  
- Choose "Web" and then enter 'https://dmswa.azurewebsites.net/' and 'https://dmswa.azurewebsites.net/.auth/login/aad/callback' in the "Redirect URIs" field.  
- Click on "Configure" at the bottom of the page.  
  
## Step 12: Enable Authentication/Authorization for your App Service  
  
- In the Azure portal, go to your App Service.  
- In the left-hand menu, click on "Authentication".  
- Click on "+ Add identity provider".  
- Select "Microsoft" as the identity provider.  
- For "App registration type", select "Pick an existing app registration in this directory".  
- In the dropdown that appears, select the Azure AD app you registered earlier.  
- For "Client application requirement", select "Allow requests only from this application itself" if you want to restrict access to your app only. If you want to allow requests from specific client applications or any application, select the appropriate option.  
- For "Identity requirement", select "Allow requests from any identity" if you want to allow requests from any identity. If you want to restrict requests to specific identities, select the appropriate option.  
- For "App Service authentication settings", select "Require authentication" to ensure that requests to your app include information about the caller.  
- Click "Add".  
  
## Step 13: Assign the user (PersonX) to your application in Azure AD  
  
- In the Azure portal, go to "Azure Active Directory".  
- Click on "Enterprise applications", then select your application.  
- Click on "Users and groups", then "Add user".  
- In the "Users and groups" blade, click "None Selected" then find and select PersonX.  
- Click "Select", then "Assign".  
  
## Step 14: Test the application  
  
- After the application is published, navigate to the URL of your Azure App Service. PersonX will be prompted to log in with their Azure AD credentials. If the login is successful and they are assigned to the application, they will be able to use it. If not, they will see an error message.  
  
Remember, the code uses Managed Identity for authentication, which enhances the security of your application by eliminating the need to store sensitive information in your code. The DefaultAzureCredential class from the Azure SDK automatically uses the Managed Identity when your application is running on Azure. The Azure AD authentication ensures that only authorized users (PersonX in this case) can access your web application.  
