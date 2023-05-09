# Data Surface: GeoSpatial Visualization with Azure Maps (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0596e57b-78be-413e-96c5-828db0288d6a" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to analyze real-time, streaming GPS coordinates"
* "We want to take advantage of advanced mapping features"
* "We tried Power BI and it doesn't keep up with the size of our dataset"

## Prerequisites
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [**Storage Account**](Infrastructure_StorageAccount.md)

-----

## Exercise 1: Prepare Environment
In this exercise, we will LOREM IPSUM.

### Step X: Configure Storage Account >> Capabilities >> Static Website

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/53cc40a5-00a7-43dc-b671-99b59111ca41" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1503ae03-5e1c-43ca-b861-f3fa4816e8af" width="800" title="Snipped: May 9, 2023" />

#### `404.html`

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Azure Maps Demo</title>
	<link rel="shortcut icon" href="./img/favicon.ico"/>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
</head>
<body>
    <h1>Sorry, something is wrong.</h1>
</body>
</html>
```

#### `index.html`

LOREM IPSUM

COPY THESE TO THE `$web` folder?

`https://rchaplers.z1.web.core.windows.net/`

### Step 1: Create App Registration

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0dc8746c-1a8d-4626-a282-66e4818016ca" width="800" title="Snipped: May 9, 2023" />

### Step 2: App Registration >> Authentication >> Platform Configurations

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/12a79f01-dedf-4972-9132-37d1357a46f4" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e98df21b-41ee-4c25-ad80-8517039b1bd0" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2fba70c6-da41-4d69-b817-9e98383ca234" width="800" title="Snipped: May 9, 2023" />

#### `logout.html`

```
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Azure Maps Demo</title>
	<link rel="shortcut icon" href="./img/favicon.ico"/>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
</head>
<body>
    <h1>Thank you and have a good day!</h1>
</body>
</html>
```
### Step 3: App Registration >> Certificates & secrets >> New Client Secret

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e69d9091-9ba7-4760-8334-eca6674725ce" width="800" title="Snipped: May 9, 2023" />

`NOR8Q~T-wfnwTaR0nT9.g6B5hvAkEpaAIewhJdvR`

### Step 4: App Registration >> API Permissions >> Add a Permission

#### Add Permission, Azure Data Explorer

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3147ad1f-70e4-43e0-8fea-913fc47e69e3" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/526a4bb9-f1e3-4899-9e2c-6a6e686e3b95" width="800" title="Snipped: May 9, 2023" />

Click "**Add permissions**".

#### Add Permission, Azure Maps

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2ffc0be2-300d-49a5-9726-66cf72a216f1" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/30a114ee-c847-4f9a-8dc5-73b3c9f4cbcb" width="800" title="Snipped: May 9, 2023" />

Click "**Add permissions**".

#### Grant admin consent for Default Directory ... **to do this, you require admin permissions on the subscription**

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6d206fc4-646c-46f8-86e6-b0f4256e427e" width="800" title="Snipped: May 9, 2023" />

## Exercise 2: Modify `index.html`
In this exercise, we will LOREM IPSUM.

## Step 1: Modify `index.html`

LOREM IPSUM

## Step 2: Upload `index.html` (and other files)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/018cb2ee-98ff-4979-a0da-926a5477efb5" width="800" title="Snipped: May 9, 2023" />

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eeecfa42-5575-4be1-b9cb-a0b1a8c38340" width="800" title="Snipped: May 9, 2023" />

`https://rchaplers.blob.core.windows.net/$web/index.html?sp=r&st=2023-05-09T17:34:13Z&se=2023-05-10T01:34:13Z&spr=https&sv=2022-11-02&sr=b&sig=znVLccBUmTo4bAg9CYAocnD15Mo%2BBskTour7r0YYIeQ%3D`

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/98b67862-e68f-49cb-a349-5f3bd8411955" width="800" title="Snipped: May 9, 2023" />

LOREM IPSUM


-----

## Reference

* Azure Maps
  * [Azure Maps Samples](https://samples.azuremaps.com/)
