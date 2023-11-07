# How to monitor Blob Storage operations

## Use Case
* "We need to monitor Storage Account activities: creations, modifications, deletes, downloads, copies, data shares, size changes"
* "Particularly, we need to know who was responsible for what operation"

## Options

Option | Notes | Pros | Cons
:----- | :----- | :----- | :-----
[Storage Account >> Change Feed](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-change-feed?tabs=azure-portal) | Lorem | Lorem | Only creates, deletes, modifications
[Storage Account >> Blob Inventory](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-inventory-how-to?tabs=azure-portal) | Created rule... waiting to see report | Lorem | No real-time alerts
[Azure Storage Analytics](https://learn.microsoft.com/en-us/azure/storage/common/manage-storage-analytics-metrics?tabs=azure-portal) | Storage Account >> Diagnostic Settings | - Confirmed Operations:<br>--- Creates (PutBlob)<br>--- Downloads (GetBlob)<br>- Native "send to" Log Analytics, Storage Account, Event Hub, Partner Solution | - "completeness and timeliness... not guaranteed"<br>- No real-time alerts<br>- User not included (only CallerIpAddress)
Azure Monitor | Lorem | Lorem | Focuses on performance, capacity, and availability... not operations like create
Query Logs? | Lorem | Lorem | Lorem
Function or Logic App + Event Grid | Lorem | Lorem | Will require development and maintenance

-----

[Enable and manage Azure Storage Analytics logs...](https://learn.microsoft.com/en-us/azure/storage/common/manage-storage-analytics-logs)
[Azure Storage analytics logging](https://learn.microsoft.com/en-us/azure/storage/common/storage-analytics-logging)

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
