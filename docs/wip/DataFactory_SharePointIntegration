# Data Factory: SharePoint Integration

## Introduction

SharePoint in Microsoft 365 is a versatile platform that can serve as a home for various types of content and data. Beyond traditional lists, SharePoint offers:

- **Document Libraries:** Store and manage files (documents, images, media) along with associated metadata.
- **Site Pages & News:** Host communication and modern pages that can be updated or managed via APIs.
- **Folders & Metadata:** Organize data into folders and use managed metadata for categorization.
- **Custom SharePoint Apps & Web Parts:** Extend functionality by building custom solutions.
- **User Profiles & Site Collections:** Manage user data and perform administrative tasks across sites.
- **Teams Sites:** Often created as part of Microsoft Teams, these are group-connected sites optimized for collaboration.

### Scalability and Performance Considerations

- **List View Threshold:** Although a SharePoint list can contain millions of items, there is a default list view threshold of about 5,000 items. This threshold impacts how many items can be efficiently retrieved in a single query.
- **Best Practices:**  
  - Use indexed columns and filtered views to work around the threshold.
  - Partition large datasets or consider using dedicated databases (like Azure SQL Database) for high-volume or high-performance needs.
- **Use Cases:** SharePoint lists are ideal for lightweight, collaborative data storage. For scenarios requiring frequent high-volume queries or advanced performance, additional design strategies are recommended.

This guide focuses on using the built-in **SharePoint Online List** connector in Azure Data Factory (ADF) to interact with SharePoint lists in a scalable and manageable way.

---

## SharePoint Setup

When creating a SharePoint site (such as a Teams Site) in Microsoft 365, you’re establishing a group-connected site that supports robust collaboration features. Here’s how to set up your SharePoint environment:

1. **Using Microsoft Teams:**  
   - **Create a Team:** When you create a new team in Microsoft Teams, a corresponding SharePoint site (and associated Office 365 Group) is automatically provisioned.
   - **Access the Site:** In Teams, click on the Files tab and then “Open in SharePoint” to verify and configure your site.
  
2. **Directly in SharePoint:**  
   - **Go to the SharePoint Admin Center:**  
     - Navigate to [admin.microsoft.com](https://admin.microsoft.com) and sign in with your administrator account.
     - In the left-hand navigation, expand **Admin centers** and click on **SharePoint**.
   - **Create a Site:**  
     - Click on **Create**.
     - Choose **Team site** (which is connected to Microsoft 365 Groups) as your site type.
     - Follow the prompts to name your site, assign owners, and configure privacy settings.
  
3. **Configuration and Customization:**  
   - Customize the site with web parts, lists, and libraries to meet your collaboration and data storage needs.
   - **Important:** If you plan to use the **SharePoint Online List** connector, ensure you have a list in your site where your data is stored. You can create a new list or use an existing one.

---

## Application Registration Setup

If your organization cannot grant admin consent for application permissions, you must configure your integration to use delegated permissions. Delegated permissions allow the application to act on behalf of a signed-in user, leveraging that user’s permissions. Follow these steps:

1. **Sign In to the Azure Portal:**  
   - Navigate to [https://portal.azure.com](https://portal.azure.com) and sign in with your account. Note that you do not need Global Administrator rights for delegated permissions, but the user must have sufficient access to SharePoint.

2. **Navigate to Azure Active Directory:**  
   - In the left-hand menu, select **Azure Active Directory**.

3. **Locate Your App Registration:**  
   - Click on **App registrations**.
   - Find and select the application registered for your SharePoint integration.

4. **Configure API Permissions for Delegated Access:**  
   - Within your app’s settings, click on **API permissions**.
   - Click **Add a permission**.
   - Under **Microsoft APIs**, select **SharePoint**.
   - Choose **Delegated Permissions**.  
   - Select the minimal permissions required (for example, **Sites.Read.All** if you only need to read data, or **Sites.FullControl.All** if full control is necessary).  
   - Click **Add permissions**.
   
5. **User Consent and Sign-In:**  
   - With delegated permissions, individual users will be prompted to consent when they sign in.  
   - If the organization restricts user consent, you may need to request that your tenant administrator adjust the consent settings in Azure AD so that users can grant consent for this app.
   
6. **Adjust Your Integration to Use Delegated Authentication:**  
   - Update your Azure Data Factory (ADF) linked service configuration to use an authentication flow that supports delegated permissions. This usually means integrating with a user sign-in process (such as OAuth2’s authorization code flow).
   - Ensure that the user signing in has the necessary SharePoint permissions on the target site.

By following these steps, you configure your application to use delegated permissions, allowing it to operate under the context of a signed-in user when admin consent for application permissions is not possible. This approach is well-suited for scenarios where interactive user sign-in is acceptable.

---

## Part 1: Pulling Data from SharePoint Using the SharePoint Online List Connector

### Step 1: Create the Linked Service in ADF

1. **Access ADF Studio:**  
   Open your Azure Data Factory instance in ADF Studio.
   
2. **Navigate to Linked Services:**  
   - Click on the **Manage** tab.
   - Under **Linked services**, click **New**.
   
3. **Select the Connector:**  
   - Choose **SharePoint Online List** from the list of connectors.
   
4. **Configure the Linked Service:**  
   - **Name:** Enter a descriptive name (e.g., `LS_SharePointOnlineList`).
   - **Site URL:** Provide the URL for your SharePoint Online site (e.g., `https://<your-tenant>.sharepoint.com/sites/<your-teams-site>`).
   - **Authentication:**  
     - For application permissions, use the client ID and client secret from your Azure AD app registration.
     - Ensure the configuration reflects the use of application permissions (client credentials flow).
     - Make sure the account or app registration has the required permissions on your SharePoint list.
   
5. **Test Connection:**  
   - Click **Test connection** to confirm that ADF can connect to your SharePoint site.
   - Once successful, click **Create**.

---

## Part 2: Creating a Dataset for the SharePoint List

### Step 1: Set Up a New Dataset

1. **Create Dataset:**  
   In the **Author** section, create a new dataset.
   
2. **Choose Data Source:**  
   - Select **SharePoint Online List** as your connector.
   
3. **Configure Dataset:**  
   - **Linked Service:** Associate the dataset with the linked service you created (`LS_SharePointOnlineList`).
   - **List Name/Endpoint:** Specify the SharePoint list name or its endpoint. For example:  
     ```
     web/lists/getbytitle('YourListName')/items
     ```
   - **Schema Mapping:** Adjust the dataset schema to match your SharePoint list fields if necessary.

---

## Part 3: Creating a Pipeline to Copy Data from SharePoint

### Step 1: Build the Pipeline

1. **Pipeline Creation:**  
   In the **Author** section, create a new pipeline.
   
2. **Add Copy Data Activity:**  
   - Drag a **Copy Data** activity onto the canvas.
   
3. **Configure the Source:**  
   - Set the source to the dataset you just created.
   - Optionally add query parameters or filters to work within the list view threshold (using indexed columns or filtered views).
   
4. **Configure the Sink:**  
   - Choose your destination (e.g., Azure Blob Storage, SQL Database).
   - Map the source columns to the sink schema appropriately.
   
5. **Validate & Run:**  
   - Validate the pipeline.
   - Run a debug session to ensure data is being extracted correctly.
   - Publish your pipeline once testing is complete.

---

## Additional Considerations

- **Write Operations:**  
  While the SharePoint Online List connector is primarily designed for reading list data, writing data may have limitations. For advanced write operations, consider using ADF’s **Web Activity** to call SharePoint’s REST API.

- **Security:**  
  Use Azure Key Vault to securely store credentials and reference them in your linked services.

- **Monitoring & Troubleshooting:**  
  Utilize the **Monitor** tab in ADF to track pipeline runs and diagnose any issues. Consider implementing retry policies or error-handling steps for robust operation.

- **Handling Large Lists:**  
  If your list is approaching the list view threshold, ensure that your dataset and queries are optimized (using filters, indexed columns, or pagination) to maintain performance.
