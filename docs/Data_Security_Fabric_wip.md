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
[SQL granular permissions in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/sql-granular-permissions)

Execute the following logic to see granular permissions granted to you:
```
SELECT * FROM sys.fn_my_permissions(NULL, 'Database');
```

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/5e8cddec-02c6-4b8d-82f6-9743a2d95858)

-----

## Schema

Start by creating a schema that we can use for demonstration:

```
CREATE SCHEMA [rchaplerfw-s];
```

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/07eb8453-c705-4359-b6a6-e481b4db6ca7)

### Permissions


## Row-Level
_Documentation, Oct 2023: Row-level security is currently not supported. Dynamic data masking is currently not supported._
