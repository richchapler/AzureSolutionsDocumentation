# How to monitor Blob Storage operations

## Use Case
* "We need to monitor Storage Account activities: creations, modifications, deletes, downloads, copies, data shares, size changes"
* "Particularly, we need to know who was responsible for what operation"

## Options

Option | Pros | Cons
:----- | :----- | :-----
[Storage Account > Change Feed](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-change-feed?tabs=azure-portal) | Lorem | Only creates, deletes, modifications

-----

Open the Storage Account and navigate to **Monitoring** >> **Logs**.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/85fe757c-4b5e-4dcd-9826-f906d871523b" width="800" title="Snipped: November 7, 2023" />

Lorem Ipsum

-----

## Blob Creation

Log Analytics

```
  AzureDiagnostics
  | where TimeGenerated > ago(1h)
  | where Category == "StorageWrite"
  | where OperationName == "PutBlob"
  | project TimeGenerated, AccountName, Uri = split(URL_s, "?")[0]
```


## Blob Modification

## Blob Deletion

## Blob Download

## Blob Copy

## Blob Share

## Blob Re-Size
