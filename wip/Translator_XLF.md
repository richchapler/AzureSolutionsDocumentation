# Translator: XLF Translation

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

Login to Azure Portal, create and then navigate to a Translator resource named `{prefix}t`.

Navigate to **Resource Management** >> **Keys and Endpoint**.

Copy `KEY 1` and `Document Translation Endpoint` values.

<!-- ------------------------- -->

#### Confirm Success

Open Azure Portal >> Cloud Shell and switch to PowerShell.

##### Set Translator key
```powershell
$key = "<key>"
```

##### Set Translator "Document Translation" endpoint
```powershell
$endpoint = "https://{prefix}t.cognitiveservices.azure.com/"
```

##### Specify Translator region
```powershell
$region = "westus"
```

##### Define full API URI for text translation
```powershell
$uri = "$endpoint/translator/text/v3.0/translate?from=en&to=fr"
```

##### Create JSON body with text to translate
```powershell
$body = '[{"Text": "Hello world"}]'
```

##### Construct HTTP headers
```powershell
$headers = @{ "Ocp-Apim-Subscription-Key"=$key; "Ocp-Apim-Subscription-Region"=$region; "Content-Type"="application/json" }
```

##### Send translation request to Azure Translator
```powershell
Invoke-RestMethod -Method Post -Uri $uri -Headers $headers -Body $body
```

##### Expected Response
```plaintext
translations
------------
{@{text=Salut tout le monde; to=fr}}
```

<!-- ------------------------- ------------------------- -->

### Prepare Sample

Login to Azure Portal, create a Storage Account resource named `{prefix}s` and a container named `xlf`. 

Assign necessary firewall and permissions (e.g., `Storage Blob Data Contributor` container `xlf`).

Upload sample file `sample.xlf`.

#### Confirm Version

Azure Translator only supports **XLIFF version 1.2**.

##### Check the version by inspecting the `<xliff>` tag
```powershell
(Get-Content "/home/rich/sample.xlf" -Raw) -match '<xliff[^>]+version="([^"]+)"' | Out-Null; $matches[1]
```

Confirm that the version is `1.2`

<!-- ------------------------- ------------------------- -->

### Translate Sample

> Some of the steps below repeat steps from the **Confirm Success** section above

##### Set Translator key
```powershell
$key = "<key>"
```

##### Set Translator "Document Translation" endpoint
```powershell
$endpoint = "https://westus.api.cognitive.microsofttranslator.com"
```

##### Set Translator region
```powershell
$region = "westus"
```

##### Set Storage Account name
```powershell
$storageAccount = "{prefix}s"
```

##### Set Blob container name
```powershell
$container = "xlf"
```

##### Create Storage Context using Entra ID
```powershell
$context = New-AzStorageContext -StorageAccountName $storageAccount -UseConnectedAccount
```

##### Construct container URL
```powershell
$containerURL = "https://$storageAccount.blob.core.windows.net/$container"
```

##### Validate container URL is accessible using RBAC
```powershell
Get-AzStorageBlob -Container $container -Context $context
```

##### Validate the Translator endpoint path resolves
```powershell
Invoke-WebRequest -Uri "$endpoint/translator/document/batch/v1.0/batches" -Method Options
```


> BLOCKED BY INABILITY TO USE THE `Microsoft.DocumentTranslation` Resource Provider




##### Validate that the Translator key and region match the endpoint
```powershell
$headers = @{ "Ocp-Apim-Subscription-Key"=$key; "Ocp-Apim-Subscription-Region"=$region; "Content-Type"="application/json" }
```

##### Build translation job body
```powershell
$body = @{inputs=@(@{source=@{containerURL=$containerURL};targets=@(@{containerURL=$containerURL;language="fr"})})} | ConvertTo-Json -Depth 10
```

##### Validate job body structure
```powershell
$body
```

##### Submit the translation job
```powershell
$response = Invoke-RestMethod -Method Post -Uri "$endpoint/translator/document/batch/v1.0/batches" -Headers $headers -Body $body
```

##### Output the translation job ID
```powershell
$response.id
```

##### Output the initial job status
```powershell
$response.status
```