# How to secure Fabric Warehouse from top to bottom

## Warehouse
[Share your warehouse and manage permissions](https://learn.microsoft.com/en-us/fabric/data-warehouse/share-warehouse-manage-permissions)

Navigate to “My Workspace”, click the ellipses next to your warehouse, select “Manage permissions”.

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/eedeb6e0-6154-4ba8-9cb2-1a67d8ddb991)

Click "+ Add user" and complete the resulting pop-up form:
* Select user(s) or group(s)
* Do not add addtional permissions (we will grant granular permissions in later steps)

Click "Grant".

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/e455b831-8590-4d0a-81d1-540673cdaf28)

Navigate to your warehouse.

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/5b61e39b-b8d7-4bc1-b95d-e0fac6174d40)

 Click "T-SQL".

-----

## User
_Documentation, Oct 2023: `CREATE USER` cannot be explicitly executed currently. When GRANT or DENY is executed, the user is created automatically. The user will not be able to connect until sufficient workspace level rights are given._

### View My Permissions
```
SELECT * FROM sys.fn_my_permissions(NULL, 'Database');
```

-----

## Schema
[SQL granular permissions in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/sql-granular-permissions)

Start by creating a schema that we can use for demonstration:

```
CREATE SCHEMA [rchaplerfw-s];
```

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/07eb8453-c705-4359-b6a6-e481b4db6ca7)

### Permissions


-----

Microsoft Fabric supports various levels of security for data warehousing:

•	Workspace Roles: Workspace roles are used for development team collaboration within a workspace. Role assignment determines the actions available to the user and applies to all items within the workspace.
•	Item Permissions: Item permissions can be assigned directly to individual Warehouses. The user will receive the assigned permission on that single Warehouse.
•	Granular Security: More granular permissions are needed in some cases. To achieve this, standard T-SQL constructs can be used to provide specific permissions to users.
•	Object-Level Security: Object-level security is a security mechanism that controls access to specific database objects, such as tables, views, or procedures, based on user privileges or roles.
•	Row-Level Security: Row-level security is a database security feature that restricts access to individual rows or records within a database table based on specified criteria, such as user roles or attributes.
•	Column-Level Security: Column-level security allows you to restrict column access to protect sensitive data.

However, when I try some of these… example, how do I share a Warehouse with User X, I’m not seeing the things that they report should be there.
I’m reaching out internally to get answers. I’m assuming that I’m not seeing the options because it is a preview product, but I want to confirm.

Security for data warehousing - Microsoft Fabric | Microsoft Learn
Announcing: Column-Level &amp; Row-Level Security for Fabric Warehouse &amp; SQL Endpoint | Microsoft Fabric Blog | Microsoft Fabric
Get started securing your data in OneLake - Microsoft Fabric | Microsoft Learn
