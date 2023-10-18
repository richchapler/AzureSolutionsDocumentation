# How to secure Fabric Warehouse from top to bottom

## Warehouse Access
[Share your warehouse and manage permissions](https://learn.microsoft.com/en-us/fabric/data-warehouse/share-warehouse-manage-permissions)

Navigate to “My Workspace”, click the ellipses next to your warehouse, select “Share” and then complete the resulting popup

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/7674c341-4c5b-430f-9403-5d63e2add4d4)

**Test Result: User was able to see the warehouse, but not specific data or create a report since permissions have not been granted at that level.**

_Warehouse Access: Warehouse connectivity is dependent on being granted the Microsoft Fabric Read permission, at a minimum, for the Warehouse._
_Microsoft Fabric item permissions enable the ability to provide a user with SQL permissions, without needing to grant those permissions within SQL._

## Schema Access
_Note: User must have (at minimum) Read permission to the warehouse in order to connect to the database_



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
