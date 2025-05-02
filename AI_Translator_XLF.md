# AI, Translator: XLF Translation

### Use Case

The solution team wants to:
- **Eliminate dependency on external translation vendor services** by building an **internal Azure-based translation service**
- **Automate the translation** of XLF (XLIFF) files with **minimal manual intervention**
- **Preserve the full structure and formatting** of original documents for seamless re-import into publishing systems like **Articulate Storyline**
- **Enable multi-language translation** through a controlled, auditable API-driven process

### Initial Interaction
A user uploads an **XLF file** to an internal system (manual for now), and that file is **sent to Azure Translator** via API call for translation into one or more target languages.

### Need to Verify
- Azure Translator can accept XLF files without losing structure or embedded formatting
- The translated XLF maintains valid XML and can be reimported cleanly into original systems (e.g., Articulate Storyline)
- Any payload size limits (e.g., 5MB) or rate limits are documented and handled appropriately
- Costs are accurately estimable based on character count and Azure Translator pricing
- Potential need for pre-processing (chunking) or post-processing (rebuilding) if structure gets corrupted

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

## Proof-of-Concept

<!-- ------------------------- ------------------------- -->

### Create Translator

Login to Azure Portal, create a Translator instance named `{prefix}t`, Standard S1, with System-assigned Identity.

Navigate to Translator resource `{prefix}t` >> **Resource Management** >> **Keys and Endpoint**.

Copy `KEY 1` and `Document Translation Endpoint` values.

<!-- ------------------------- -->

#### Confirm Success

Open Azure Portal >> Cloud Shell and switch to PowerShell.

##### Set Translator Key
```powershell
$key = "<key>"
```

##### Set Translator Endpoint
```powershell
$endpoint = "https://{prefix}t.cognitiveservices.azure.com/"
```

##### Set Region
```powershell
$region = "westus"
```

##### Define API URI
```powershell
$uri = "$endpoint/translator/text/v3.0/translate?from=en&to=fr"
```

##### Create JSON Payload
```powershell
$body = '[{"Text": "Hello world"}]'
```

##### Construct Headers
```powershell
$headers = @{ "Ocp-Apim-Subscription-Key"=$key; "Ocp-Apim-Subscription-Region"=$region; "Content-Type"="application/json" }
```

##### Send Translation Request
```powershell
Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -Body $body
```

###### Expected Response
```plaintext
translations
------------
{@{text=Salut tout le monde; to=fr}}
```

<!-- ------------------------- ------------------------- -->

### Prepare Sample

Login to Azure Portal, create a Storage Account resource named `{prefix}s` and two containers: `xlf` and `xlf-translated`. 

On both containers, add role assignment `Storage Blob Data Contributor` to the Translator instance named `{prefix}t` and your credential, then upload sample file `sample.xlf`.

#### Confirm Version

Azure Translator only supports **XLIFF version 1.2**.

If you are unsure of the XLF version on your sample file, confirm by inspecting the `<xliff>` tag:
```powershell
(Get-Content "/home/rich/sample.xlf" -Raw) -match '<xliff[^>]+version="([^"]+)"' | Out-Null; $matches[1]
```

Confirm that the version is `1.2`

> The preceding PowerShell assumes that you have uploaded the `sample.xlf` file using **Manage Files**

<!-- ------------------------- ------------------------- -->

### Translate Sample

> Some of the steps below repeat steps from the **Confirm Success** section above

##### Set Translator Key
```powershell
$key = "<key>"
```

##### Set Translator Endpoint
```powershell
$endpoint = "https://{prefix}t.cognitiveservices.azure.com"
```

##### Set Region
```powershell
$region = "westus"
```

##### Set Storage Account Name
```powershell
$storageAccount = "{prefix}s"
```

##### Set Storage Account Container Name
```powershell
$container = "xlf"
```

##### Create Storage Context
```powershell
$context = New-AzStorageContext -StorageAccountName $storageAccount -UseConnectedAccount
```

##### Set Container URL
```powershell
$containerURL = "https://$storageAccount.blob.core.windows.net/$container"
```

##### Validate Container URL is Accessible
```powershell
Get-AzStorageBlob -Container $container -Context $context
```

###### Expected Response
```plaintext
   AccountName: {prefix}s, ContainerName: xlf                                                                               
                                                                                                                        
Name                 BlobType  Length          ContentType                    LastModified         AccessTier SnapshotTime                 IsDeleted  VersionId
----                 --------  ------          -----------                    ------------         ---------- ------------                 ---------  ---------
sample.xlf           BlockBlob 257133          application/octet-stream       2025-05-02 14:02:21Z Hot                                     False 
```

##### Set API Version
```powershell
$apiVersion = "2024-05-01"
```

##### Build Document Translation URI
```powershell
$uri = "$endpoint/translator/document/batches?api-version=$apiVersion"
```

##### Validate Translator Endpoint Is Responsive
```powershell
Invoke-RestMethod -Method Get -Uri $uri -Headers @{ "Ocp-Apim-Subscription-Key" = $key; "Ocp-Apim-Subscription-Region" = $region }
```

###### Expected Response
```json
value
-----
{@{id=5a7e020b-277f-4345-a9df-dbcfb1e84316; createdDateTimeUtc=5/2/2025 2:34:29 PM; lastActionDateTimeUtc=5/2/2025 2:34:29 PM; status=ValidationFailed; error=; summary=…
```

##### Construct Request Body
```powershell
$body = @{ inputs = @( @{ source = @{ sourceUrl = "https://$storageAccount.blob.core.windows.net/xlf"; storageType = "AzureBlob"; fileFilter = "*.xlf" }; targets = @( @{ targetUrl = "https://$storageAccount.blob.core.windows.net/xlf-translated"; storageType = "AzureBlob"; language = "fr" } ) } ) } | ConvertTo-Json -Depth 10
```

##### Submit Translation Job
```powershell
$postResponse = Invoke-WebRequest -Method Post -Uri $uri -Headers @{ "Ocp-Apim-Subscription-Key" = $key; "Ocp-Apim-Subscription-Region" = $region; "Content-Type" = "application/json" } -Body $body; $statusUrl = ($postResponse.Headers["Operation-Location"])[0]
```

##### Poll Job Status
```powershell
do { Start-Sleep -Seconds 5; $statusResponse = Invoke-RestMethod -Method Get -Uri $statusUrl -Headers @{ "Ocp-Apim-Subscription-Key" = $key; "Ocp-Apim-Subscription-Region" = $region }; Write-Host "Status:" $statusResponse.status } while ($statusResponse.status -in @("NotStarted","Running"))
```

###### Expected Response
```json
Status: Succeeded
```

##### Confirm Success

Navigate to storage account `{prefix}s` >> container `xlf-translated`, download the translated `sample.xlf` and confirm translation.