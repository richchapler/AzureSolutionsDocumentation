# KeyVault

## How-To: List Soft-Deleted KeyVaults
```
az keyvault list-deleted --subscription {SUBSCRIPTION_ID}
```

## How-To: Purge a Soft-Deleted KeyVault
```
az keyvault purge --subscription {SUBSCRIPTION_ID} --name {KEYVAULT_NAME}
```

## Permissions: List and Create Secrets (even if you are already a KeyVault Owner)... RBAC Role: "Key Vault Administrator"
