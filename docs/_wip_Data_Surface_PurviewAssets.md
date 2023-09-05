Microsoft Purview >> "Export as CSV" result >> monthly pull to blob storage as CSV using Logic Apps?

To produce the report using the Purview UI:
* Click Data Estate Insights
* Click Assets

  ![image](https://github.com/richchapler/AzureSolutions/assets/44923999/dbf63ba0-3097-47e0-a4c4-a65600534d33)

* Click "View details"
* Select your collection on the resulting page

  ![image](https://github.com/richchapler/AzureSolutions/assets/44923999/3c3f44f7-a1cb-4d83-b941-519a7bf87e70)

* Click "Export as CSV" in the upper-right corner, then Export on the resulting pop-up

You can expect a result like:

```
"Asset ID","Asset name","Asset fully qualified name","Object type","Collection","Classification display name","Classification formal name","Asset data owner","Data owner email","Classification source","Reason for unclassified"
"d0b5cddd-501d-4509-8c01-e5923cba5a27","rchaplerdec2.westus3","https://rchaplerdec2.westus3.kusto.windows.net","Instances","rchaplerp","","","","","","Not applicable."
"11b9d5c0-7b10-4a08-9989-1e315297bd81","rchaplerded2","https://rchaplerdec2.westus3.kusto.windows.net/rchaplerded2","Databases","rchaplerp","","","","","","Not applicable."
```

Customer reports that the limit of 200 pages (* 50 items per page... i.e., 1000 items) is insufficient and wants to export the body of data to CSV.

SQL Server + Adventureworks + Purview permissions

```
-- In the master database, create a login for UserX
CREATE LOGIN [UserX] WITH PASSWORD = 'YourStrongPassword';

-- Create a user for UserX in the desired database
CREATE USER [UserX] FOR LOGIN [UserX];

-- Grant the desired role to UserX in the desired database
ALTER ROLE db_datareader ADD MEMBER [UserX];
ALTER ROLE db_datawriter ADD MEMBER [UserX];
```
