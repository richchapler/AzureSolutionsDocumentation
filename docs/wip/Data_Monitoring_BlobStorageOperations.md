# How to monitor Blob Storage operations

## Use Case
* "We need to monitor Storage Account activities: browse, creates, edits, deletes, downloads, copies, data shares, size changes"
  * QUESTION: What are copies and shares?
* "Particularly, we need to know **who** was responsible for what operation"

## Options

<table>
  <tr>
    <th align="left">Option</th>
    <th align="left">Pros</th>
    <th align="left">Cons</th>
  </tr>
  <tr valign="top">
    <td align="left"><a href="https://learn.microsoft.com/en-us/azure/storage/blobs/blob-inventory-how-to?tabs=azure-portal">Blob Inventory</a><br><sub>Storage Account >><br>Blob Inventory</sub></td>
    <td align="left"></td>
    <td align="left"><ul><li>No real-time alerts</li><li>No User detail</li></ul></td>
  </tr>
  <tr valign="top">
    <td align="left"><a href="https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-change-feed?tabs=azure-portal">Change Feed</a><br><sub>Storage Account >><br>Change Feed</sub></td>
    <td align="left"></td>
    <td align="left"><ul><li>Only creates, deletes, modifications</li></ul></td>
  </tr>
  <tr valign="top">
    <td align="left"><a href="https://learn.microsoft.com/en-us/azure/storage/common/manage-storage-analytics-metrics?tabs=azure-portal">Storage Analytics</a><br><sub>Storage Account >><br>Diagnostic Settings</sub></td>
    <td align="left"><ul><li>Confirmed Operations:<ul><li>Browse (ListBlobs)</li><li>Creates (PutBlob)</li><li>Deletes (DeleteBlob)</li><li>Downloads (GetBlob)</li><li>Edits (PutBlob+)</li></ul></li><li>Native "send to" Log Analytics, Storage Account, Event Hub, Partner Solution</li></ul></td>
    <td align="left"><ul><li>"completeness and timeliness... not guaranteed"</li><li>No real-time alerts</li><li>No User detail (only CallerIpAddress, AuthenticationHash, and UserAgentHeader)</li></ul></td>
  </tr>
  <tr valign="top">
    <td align="left"><a href="https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview">Event Handling</a><br><sub>Storage Account >><br>Events</sub></td>
    <td align="left"><ul><li>Real-time</li><li>Event Types<ul><li>Blob Created / Deleted / Renamed</li><li>Directory Created / Deleted / Renamed</li><li>Blob Tier Changed</li></ul></li></ul></td>
    <td align="left"><ul><li>No User detail</li><li>Will require development and maintenance</li></ul></td>
  </tr>
</table>






```
StorageBlobLogs
| order by TimeGenerated
| take 25
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
