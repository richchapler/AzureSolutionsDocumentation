# How to monitor Blob Storage operations

## Use Case
* "We need to monitor Storage Account activities: browse, creates, edits, deletes, downloads, copies, data shares, size changes"
  * QUESTION: What are copies and shares?
* "Particularly, we need to know **who** was responsible for what operation"

## Options

| Option | Velocity | Operations | Endpoints | Concerns |
| :--- | :--- | :--- | :--- | :--- |
| [Blob Inventory](https://learn.microsoft.com/en-us/azure/storage/blobs/blob-inventory-how-to) | Daily | Not Applicable |
| [Change Feed](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-change-feed) | TBD | Creates<br>Deletes<br>Edits |
| [Storage Analytics](https://learn.microsoft.com/en-us/azure/storage/common/manage-storage-analytics-metrics)<br><sub>"Diagnostic Settings"</sub> | Minutes | Browse (ListBlobs)<br>Creates (PutBlob)<br>Deletes (DeleteBlob)<br>Downloads (GetBlob)<br>Edits (PutBlob+) | Log Analytics<br>Storage Account<br>Event Hub<br>Partner Solution
| [Event Handling](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview)<br><sub>"Events"</sub> | Seconds | Blob Created / Deleted / Renamed<br>Directory Created / Deleted / Renamed<br>Blob Tier Changed<br>more... |

None of the listed solutions...
* include canned alerts, reports, or analysis
* specifically identify the user making the change; closest is CallerIpAddress, AuthenticationHash, and UserAgentHeader 

Other solutions:
* Defender for Cloud does not enable the sort of file activity monitoring described by the use case
* Purview TBD

```
StorageBlobLogs
| order by TimeGenerated
| take 25
```

PowerShell Script for checking the size of a container:
```
foreach ($container in $containers) {
    $containerSize = (Get-AzStorageContainerStatistics -Name $container.Name -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName).ContainerSize
    $totalSize += $containerSize
}
$totalSize
```

**Is there auditing functionality that could be turned on to better see who is doing what?**

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
