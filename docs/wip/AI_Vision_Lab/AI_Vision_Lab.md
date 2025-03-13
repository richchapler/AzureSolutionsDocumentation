# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Table of Contents  

- [Exercise 1: Prepare Environment](#exercise-1-prepare-environment)  
- [Exercise 2: Optical Character Recognition (OCR)](#exercise-2-optical-character-recognition-ocr)  
- [Exercise 3: Spatial Analysis](#exercise-3-spatial-analysis) 
- [Exercise 4: Face](#exercise-4-face) 
- [Exercise 5: Image Analysis](#exercise-5-image-analysis) 

## Resource Requirements
- AI Services (in East US region)
- Computer Vision

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 1: Prepare Environment
_Complete this exercise only if you intend to complete Pro-Code exercises_

Start with a pre-configured virtual machine and add the following artifacts:

* [PowerShell](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/PowerShell.html)
* [Visual Studio Code](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode.html) with [Jupyter](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode_Jupyter.html)
* [Python (including Virtual Environment)](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/Python.html)

------------------------- ------------------------- -------------------------

## Exercise 2: Optical Character Recognition (OCR)  

This exercise demonstrates how to use Azure AI Vision OCR to extract text from images. 

### Low Code

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and login with your Azure credentials.

<img src="https://github.com/user-attachments/assets/f008f817-51b8-4a7c-b230-19803782bde7" width="800" title="Snipped February 14, 2025" />

Click on the "Optical character recognition" tab.

#### Extract Text from Images

<img src="https://github.com/user-attachments/assets/9d70618d-2d50-4ebc-81a8-55a9bb8be4bc" width="800" title="Snipped February 14, 2025" />

Check "I acknowledge...", click "Browse for a file" and then click "Please select a resource".

<img src="https://github.com/user-attachments/assets/3da8778e-7b78-4ab7-9aa7-5b9e5fe15edc" width="800" title="Snipped February 14, 2025" />

Complete the "Select an AzureResource" form and then click "Confirm".

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/d50659f8-4762-4c5f-a6da-95505f2645be" width="800" title="Snipped February 14, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

------------------------- -------------------------

### Pro Code

**Step 1: Ensure Your Notebook is Open**

Before proceeding, make sure you have a Jupyter Notebook open in Visual Studio Code:

1. In Visual Studio Code, click **File** > **New File**.
2. Search for and select **Jupyter Notebook**.
3. Save the file as `ocr.ipynb`.

-------------------------

**Step 2: Load Environment Variables**

**+ Markdown Cell**

Paste the following annotation into a new Markdown cell:

```markdown
## Load Environment Variables
This cell loads your API credentials and image path from the `.env` file. Make sure your `.env` file contains the following variables:
- API_KEY
- ENDPOINT
- IMAGE_PATH
```

Render the Markdown cell.

**+ Code Cell**

Next, create a new Code cell and paste the following code:

```python
import os
from dotenv import load_dotenv

env_file = ".env"
load_dotenv(env_file)

API_KEY = os.getenv("API_KEY")
ENDPOINT = os.getenv("ENDPOINT")
IMAGE_PATH = os.getenv("IMAGE_PATH")

# Optionally, print the variables to verify they are loaded
print("API_KEY:", API_KEY)
print("ENDPOINT:", ENDPOINT)
print("IMAGE_PATH:", IMAGE_PATH)
```

Run the cell to ensure that `API_KEY`, `ENDPOINT`, and `IMAGE_PATH` are correctly loaded.

-------------------------

**Step 3: Add the OCR Annotation**

Click **+ Markdown** and paste the following annotation into the resulting cell:

```markdown
## Exercise 2: Optical Character Recognition (OCR) 
Use Optical Character Recognition (OCR) on an image file (supported formats: JPEG, PNG, BMP, GIF, TIFF)
```

Render the Markdown cell.

-------------------------

**Step 4: Add the OCR Code**

Click **+ Code** and paste the following code into the new cell:

```python
import os
import requests

def perform_ocr(image_path):
    with open(image_path, "rb") as image_file:
        image_data = image_file.read()

    headers = {
        "Ocp-Apim-Subscription-Key": API_KEY,
        "Content-Type": "application/octet-stream"
    }

    url = f"{ENDPOINT}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=read"
    response = requests.post(url, headers=headers, data=image_data)
    return response

if IMAGE_PATH and os.path.isfile(IMAGE_PATH):
    response = perform_ocr(IMAGE_PATH)
    print(response.json())
else:
    print("IMAGE_PATH is not defined or the file does not exist. Please check your .env file.")
```

Run the cell to send your image (as specified by `IMAGE_PATH` in your `.env` file) to the Azure AI Vision endpoint, and review the returned JSON for OCR results.

------------------------- -------------------------

#### Expected Response
_Note: JSON formatted and abbreviated for convenience_

```json
{
  "readResult": {
    "stringIndexType": "TextElements",
    "content": "Walmart\nSave money. Live better.\n( 330 ) 339 - 3991\nMANAGER DIANA EARNEST\n231 BLUEBELL DR SW\nNEW PHILADELPHIA OH 44663\nST# 02115 OP# 009044 TE# 44 TR# 01301\n...\nSUBTOTAL 93.62\nTAX 1 6.750% 4.59\nTOTAL 98.21\nVISA TEND 98.21\n...\n07/28/17\n02:39:48\nCHANGE DUE 0.00\n# ITEMS SOLD 25\n...",
    "pages": [
      {
        "height": 1373.0,
        "width": 554.0,
        "angle": 0.0,
        "pageNumber": 1,
        "words": [
          {"content": "Walmart", "boundingBox": [...], "confidence": 0.996, "span": {"offset": 0, "length": 7}},
          {"content": "Save", "boundingBox": [...], "confidence": 0.994, "span": {"offset": 8, "length": 4}},
          ...
          {"content": "TOTAL", "boundingBox": [...], "confidence": 0.998, "span": {"offset": 1023, "length": 5}},
          {"content": "98.21", "boundingBox": [...], "confidence": 0.994, "span": {"offset": 1029, "length": 5}}
        ],
        "spans": [{"offset": 0, "length": 1421}],
        "lines": [
          {"content": "Walmart", "boundingBox": [...], "spans": [{"offset": 0, "length": 7}]},
          {"content": "Save money. Live better.", "boundingBox": [...], "spans": [{"offset": 8, "length": 24}]},
          ...
          {"content": "TOTAL", "boundingBox": [...], "spans": [{"offset": 1023, "length": 5}]},
          {"content": "98.21", "boundingBox": [...], "spans": [{"offset": 1029, "length": 5}]}
        ]
      }
    ],
    "styles": [],
    "modelVersion": "2022-04-30"
  },
  "modelVersion": "2023-02-01-preview",
  "metadata": {"width": 554, "height": 1373}
}
```

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 3: Spatial Analysis  

This exercise demonstrates how to use Azure AI Vision Spatial Analysis to detect people and movement in an image.

### Low Code

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and login with your Azure credentials.

<img src="https://github.com/user-attachments/assets/ba6405d5-9959-47c6-bf99-b86461672a45" width="800" title="Snipped February 14, 2025" />

Click on the "Spatial analysis" tab.

-------------------------

#### Video Retrieval and Summary

<img src="https://github.com/user-attachments/assets/3fd31e11-15ea-4b81-a1be-7c208b566c2b" width="800" title="Snipped February 14, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/a8a9125e-7b11-4252-bfbe-da386c2c8a80" width="800" title="Snipped February 14, 2025" />

Interact with resulting functionality {e.g., "Locate a frame in the video"}.

-------------------------

#### Count People in an Area

<img src="https://github.com/user-attachments/assets/5f3d30bc-38a6-4608-94ec-edc4f4629e43" width="800" title="Snipped February 14, 2025" />

⚠️ ERRORING OUT... PENDING WITH SUPPORT ⚠️

-------------------------

#### Detect when People Cross a Line

<img src="https://github.com/user-attachments/assets/22d02513-a10d-4c4e-83ca-ec1fda026628" width="800" title="Snipped February 18, 2025" />

⚠️ ERRORING OUT... PENDING WITH SUPPORT ⚠️

-------------------------

#### Detect when People Enter / Exit a Zone

<img src="https://github.com/user-attachments/assets/5288dc72-2cb5-4ad0-aabd-d5d249c9f424" width="800" title="Snipped February 18, 2025" />

⚠️ ERRORING OUT... PENDING WITH SUPPORT ⚠️

-------------------------

#### Monitor Social Distancing

<img src="https://github.com/user-attachments/assets/43add9f6-137c-4ca9-b374-f98d3c5232d1" width="800" title="Snipped February 18, 2025" />

⚠️ ERRORING OUT... PENDING WITH SUPPORT ⚠️

------------------------- -------------------------

## ProCode

Click "+ Markdown" and paste the following annotation into the resulting cell:  

```markdown
## Exercise 3: Spatial Analysis
Use Spatial Analysis to detect people and movement in an image.
```

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.  

Click "+ Code" and paste the following code into the resulting cell:  

```python
import os
import requests

def perform_spatial_analysis(image_path):
    with open(image_path, "rb") as image_file:
        image_data = image_file.read()

    headers = {
        "Ocp-Apim-Subscription-Key": API_KEY,
        "Content-Type": "application/octet-stream"
    }

    url = f"{ENDPOINT}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=people"
    response = requests.post(url, headers=headers, data=image_data)
    return response

if os.path.isfile(IMAGE_PATH):
    response = perform_spatial_analysis(IMAGE_PATH)
    print(response.json())
```

LOREM IPSUM

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 4: Face

This exercise demonstrates how to use Azure AI Vision Face to detect and analyze human faces in images.

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and login with your Azure credentials.

<img src="https://github.com/user-attachments/assets/ced5f919-80b9-40ad-83dd-61da99015b68" width="800" title="Snipped February 18, 2025" />

Click on the "Face" tab.

-------------------------

#### Detect Faces in an Image

<img src="https://github.com/user-attachments/assets/d78b53a5-aca9-40aa-8589-c1c7d0baa362" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/8074b06b-9b85-4235-8992-abbe7942ce04" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### Liveness Detection

<img src="https://github.com/user-attachments/assets/d8f00fd3-c59c-4182-9ae9-f082bba05d84" width="800" title="Snipped February 18, 2025" />

Click "Face Liveness only" and then "Try Out".

<img src="https://github.com/user-attachments/assets/029518de-efc6-49ed-86c6-63c813bd6894" width="800" title="Snipped February 18, 2025" />

Click "Apply for access".

<img src="https://github.com/user-attachments/assets/3b7d6f98-a599-411e-8db3-72b62d1bb017" width="800" title="Snipped February 18, 2025" />

Complete and submit the form.

<img src="https://github.com/user-attachments/assets/b0d52333-bcb5-4227-849c-eace366d3fa6" width="800" title="Snipped February 18, 2025" />

The "Azure AI Gating Team" will email and promises to respond in ~10 business days.

⚠️ WAITING FOR APPROVED ACCESS ⚠️

-------------------------

#### Portrait Processing

<img src="https://github.com/user-attachments/assets/9f4ed2f0-dda8-4c93-af7c-37c190b14df5" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/ea5b5082-3af4-4b79-8635-14447d980f1b" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs and review the generated portrait.

-------------------------

#### Photo ID Matching

<img src="https://github.com/user-attachments/assets/d8273d24-dd8f-4f8b-9585-dd4fc1ac4180" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box and compare to the Camera Preview.

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 5: Image Analysis

This exercise demonstrates how to use Azure AI Vision Image Analysis to extract meaningful insights from images, including object detection, caption generation, and scene understanding.

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and login with your Azure credentials.

<img src="https://github.com/user-attachments/assets/f80fe268-afa1-498a-a428-770f8690c05b" width="800" title="Snipped February 18, 2025" />

Click on the "Image analysis" tab.

------------------------- -------------------------

#### Recognize Products on Shelves

_Note (Feb 18, 2025): "In order to run this demo the resource must belong to these regions: East US, West Europe, West US 2"_

##### Low Code

<img src="https://github.com/user-attachments/assets/d6c18b07-86f3-4466-87fe-20ac5f7059dc" width="800" title="Snipped February 18, 2025" />

Iteratively try models:

- **Prebuilt product vs. gap model** – Detects products on shelves and identifies empty gaps for inventory tracking.  
- **Sample custom model** – Demonstrates a custom-trained product recognition model for specific use cases.  
- **Train your own model** – Allows users to train a model with their own dataset for tailored product detection.

##### Prebuilt Product vs. Gap Model

<img src="https://github.com/user-attachments/assets/1c722fed-a358-4232-942c-c30bf2ce30c8" width="800" title="Snipped February 18, 2025" />

On the "Detected products" tab, note that only two values are surfaced: 1) product and 2) gap

##### Sample Custom Model with Product Names

<img src="https://github.com/user-attachments/assets/a210b12e-daf3-49fa-9601-02779df0b807" width="800" title="Snipped February 18, 2025" />

On the "Detected products" tab, note various product names.

-------------------------

##### Pro Code

**Click "+ Markdown"** in your Jupyter Notebook and paste the following annotation:

```markdown
## Recognize Products on Shelves (Pro Code)
Use Azure AI Vision's object detection (and optionally custom models) to identify products on shelves and detect empty gaps.
```

After rendering the markdown, **click "+ Code"** and paste the following snippet:

```python
import os
import requests

def recognize_products_on_shelves(image_path):
    with open(image_path, "rb") as image_file:
        image_data = image_file.read()

    headers = {
        "Ocp-Apim-Subscription-Key": API_KEY,
        "Content-Type": "application/octet-stream"
    }

    # For general product detection, use "objects".
    # For specialized "product vs gap" scenarios, you may need a custom model or specialized feature.
    url = f"{ENDPOINT}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=objects"

    response = requests.post(url, headers=headers, data=image_data)
    return response

if os.path.isfile(IMAGE_PATH):
    response = recognize_products_on_shelves(IMAGE_PATH)
    print(response.json())
```

1. **Run** the code cell to send your image (defined by `IMAGE_PATH` in your `.env` file) to the Azure AI Vision endpoint.
2. **Review** the returned JSON to inspect the bounding boxes and labels for the detected products.
3. **Optional**: If you have a custom model, append `&modelVersion=<YourModelName>` to the URL as needed. Refer to the [Azure AI Vision documentation](https://learn.microsoft.com/azure/cognitive-services/computer-vision/) for further details.

-------------------------

#### Customize Models with Images

LOREM IPSUM

-------------------------

#### Search Photos with Image Retrieval

<img src="https://github.com/user-attachments/assets/340088a0-c508-46a1-bb03-278d74c121f0" width="800" title="Snipped February 18, 2025" />

Iteratively click the sample image sets, select a retrieval queries and click Search.

<img src="https://github.com/user-attachments/assets/687cc074-f25f-4c59-80fd-650c80d16312" width="800" title="Snipped February 18, 2025" />

Consider trying your own image.

-------------------------

#### Add Dense Captions to Images

<img src="https://github.com/user-attachments/assets/1c4581fe-36dc-4501-821d-c05390591c18" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/fc0dd38d-1da9-4ae5-b457-88cb45ba8d02" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### Remove Backgrounds from Images

<img src="https://github.com/user-attachments/assets/4d58e8d8-53d9-437f-b026-e8ebfdbaea14" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/372865dc-7aa5-4b63-b091-07ac0d98f86b" width="800" title="Snipped February 18, 2025" />

Review results on the "Remove backgrounds from images" / "Foreground matting" tabs.

-------------------------

#### Add Captions to Images

<img src="https://github.com/user-attachments/assets/df89523d-e20b-4509-b3cb-091fff6f6405" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/2a1bdd9f-aff9-412d-bb28-6a15e40a7204" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### Detect Common Objects in Images

LOREM IPSUM

-------------------------

#### Extract Common Tags from Images

<img src="https://github.com/user-attachments/assets/bd4656ac-989c-4f38-840f-5074db0b76bf" width="800" title="Snipped February 18, 2025" />

Choose a model, choose a language, and then iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/b865ebbf-6596-40ba-990d-271057af81f8" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### Detect Sensitive Content in Images

<img src="https://github.com/user-attachments/assets/91b53833-7fa1-4c9d-8147-3d01487ab3e4" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/89aac916-f5e7-44e4-b653-ad4a26caa8fc" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### Create Smart-Cropped Images

<img src="https://github.com/user-attachments/assets/c76c1094-341d-4fa7-8d2b-dd74a94dcef6" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/89d14157-7a61-4af0-a36f-55980aa8485c" width="800" title="Snipped February 18, 2025" />

Review results on the "Cropped image" tab and adjust aspect ratio to taste.



