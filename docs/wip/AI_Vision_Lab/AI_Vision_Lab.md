# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Table of Contents  

- [Exercise 1: Prepare Environment](#exercise-1-prepare-environment)  
- [Exercise 2: Optical Character Recognition (OCR)](#exercise-2-optical-character-recognition-ocr)  
- [Exercise 3: Spatial Analysis](#exercise-3-spatial-analysis)  
- [Exercise 4: Face](#exercise-4-face)  
- [Exercise 5: Image Analysis](#exercise-5-image-analysis) 

## Resource Requirements
In "East US" region (or other region that supports AI Vision resources), instantiate:

- AI Services 
- Computer Vision

------------------------- ------------------------- -------------------------

## Exercise 1: Prepare Environment
_Complete this exercise only if you intend to complete Pro-Code exercises_

Start with a pre-configured virtual machine and add the following artifacts:

* [PowerShell](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/PowerShell.html)
* [Visual Studio Code](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode.html) with [Jupyter](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode_Jupyter.html)
* [Python (including Virtual Environment)](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/Python.html) with the [dotenv module](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/Python_DotEnvModule.html)

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 2: Optical Character Recognition (OCR)  

This exercise demonstrates how to use Azure AI Vision OCR to extract text from images.

### 2.1 Extract Text from Images

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and log in with your Azure credentials.

<img src="https://github.com/user-attachments/assets/f008f817-51b8-4a7c-b230-19803782bde7" width="800" title="Snipped February 14, 2025" />

Click on the "Optical character recognition" tab.

#### 2.1.1 Low Code

<img src="https://github.com/user-attachments/assets/9d70618d-2d50-4ebc-81a8-55a9bb8be4bc" width="800" title="Snipped February 14, 2025" />

Check "I acknowledge...", click "Browse for a file" and then click "Please select a resource".

<img src="https://github.com/user-attachments/assets/3da8778e-7b78-4ab7-9aa7-5b9e5fe15edc" width="800" title="Snipped February 14, 2025" />

Complete the "Select an AzureResource" form and then click "Confirm".

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/d50659f8-4762-4c5f-a6da-95505f2645be" width="800" title="Snipped February 14, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

------------------------- -------------------------

### 2.1.2 Pro Code

#### Step 1: Notebook

In Visual Studio Code, click File > New File, then search for and select Jupyter Notebook.

Save the new file as `ai_vision.ipynb`.

-------------------------

#### Step 2: `.env` File

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## `.env` File
Create (if necessary) and prompt for update with API credentials and image path.
```

Render the Markdown cell by clicking the checkmark in the upper-right controls.

Click ""+ Code"" and paste the following code into the new cell:

```python
import os

env_file = ".env"

if not os.path.isfile(env_file):
    with open(env_file, "w") as f:
        f.write("API_KEY=\nENDPOINT=\nIMAGE_PATH=\n")
    print(f".env file created at {os.path.abspath(env_file)}. Please update it with your API credentials and image path.")
else:
    print(f".env file found at {os.path.abspath(env_file)}. Please verify its contents.")
```

Execute cell; expected output:

```
.env file created at c:\Users\{user}\.env. Please update it with your API credentials and image path.
```

Navigate to the folder and open the `.env` file with your preferred text editor. Update it with actual values; for example:

```text
API_KEY={Computer Vision KEY}
ENDPOINT=https://prefixcv.cognitiveservices.azure.com/
IMAGEPATH_OCR=C:\temp\ocr.jpg
```

*Note: Consider downloading sample images from Vision Studio*

Save the file after updating.

-------------------------

#### Step 3: Load Environment Variables

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Load Environment Variables
This cell loads environment variables from the `.env` file.
```

Render the Markdown cell by clicking the checkmark in the upper-right controls.

Click ""+ Code"" and paste the following code into the new cell:

```python
import os
from dotenv import load_dotenv

env_file = ".env"
load_dotenv(env_file)

API_KEY = os.getenv("API_KEY")
ENDPOINT = os.getenv("ENDPOINT")
IMAGEPATH_OCR = os.getenv("IMAGEPATH_OCR")
```

Execute cell to ensure that variables are correctly loaded.

-------------------------

#### Step 4: Pro Code

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Optical Character Recognition (OCR)
Extract Text from Images
```

Render the Markdown cell by clicking the checkmark in the upper-right controls.

Click ""+ Code"" and paste the following code into the new cell:

```python
import os
import requests

def perform_ocr(image_path):
    with open(image_path, "rb") as f:
        image_data = f.read()
    url = f"{ENDPOINT.rstrip('/')}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=read"
    headers = {
        "Ocp-Apim-Subscription-Key": API_KEY,
        "Content-Type": "application/octet-stream"
    }
    try:
        response = requests.post(url, headers=headers, data=image_data)
        response.raise_for_status()
        return response
    except requests.exceptions.RequestException as e:
        print("OCR error:", e)
        return None

if IMAGEPATH_OCR and os.path.isfile(IMAGEPATH_OCR):
    res = perform_ocr(IMAGEPATH_OCR)
    if res:
        print(res.json())
else:
    print("IMAGEPATH_OCR is not defined or the file does not exist.")
```

Execute cell and review the returned JSON result.

------------------------- -------------------------

#### Expected Result  
_Note: JSON formatted and abbreviated_

```json
{
  "readResult": {
    "stringIndexType": "TextElements",
    "content": "NATIONAL IDENTITY CARD\nBIOMETRICS\nNAME: John Doe\nSEX: Male\nHAIR: Brown\nHEIGHT: 5'10\"\nWEIGHT: 165\nBORN: 13 Dec 1981\nDRIVERS LICENSE: XXT55340H59\n1003A2- 107624-( *- 102) 440u 28878976-(tf. 7 -- i9#3.c)",
    "pages": [
      {
        "height": 3528.0,
        "width": 5300.0,
        "angle": 0.1194,
        "pageNumber": 1,
        "words": [
          {
            "content": "NATIONAL",
            "boundingBox": [1456.0, 909.0, 1932.0, 906.0, 1930.0, 1010.0, 1453.0, 1012.0],
            "confidence": 0.995,
            "span": {"offset": 0, "length": 8}
          },
          ...
        ]
      }
    ],
    "styles": [],
    "modelVersion": "2022-04-30"
  },
  "modelVersion": "2023-02-01-preview",
  "metadata": {"width": 5300, "height": 3527}
}
```

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 3: Spatial Analysis  

This exercise demonstrates how to use Azure AI Vision Spatial Analysis to detect people and movement in an image.

### 3.1 Video Retrieval and Summary

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and log in with your Azure credentials.

<img src="https://github.com/user-attachments/assets/8be99f3d-8a1c-46fc-9013-6d70be7db5c0" width="800" title="Snipped March 14, 2025" />

Click on the "Spatial analysis" tab.

-------------------------

#### 3.1.1 Low Code

<img src="https://github.com/user-attachments/assets/3fd31e11-15ea-4b81-a1be-7c208b566c2b" width="800" title="Snipped February 14, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/a8a9125e-7b11-4252-bfbe-da386c2c8a80" width="800" title="Snipped February 14, 2025" />

Interact with resulting functionality (e.g., "Locate a frame in the video").

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 4: Face

This exercise demonstrates how to use Azure AI Vision Face to detect and analyze human faces in images.

### 4.1 Detect Faces in an Image

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and log in with your Azure credentials.

<img src="https://github.com/user-attachments/assets/ced5f919-80b9-40ad-83dd-61da99015b68" width="800" title="Snipped February 18, 2025" />

Click on the "Face" tab.

-------------------------

#### 4.1.1 Low Code

<img src="https://github.com/user-attachments/assets/d78b53a5-aca9-40aa-8589-c1c7d0baa362" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/8074b06b-9b85-4235-8992-abbe7942ce04" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

-------------------------

#### 4.2 Liveness Detection

<img src="https://github.com/user-attachments/assets/d8f00fd3-c59c-4182-9ae9-f082bba05d84" width="800" title="Snipped February 18, 2025" />

Click "Face Liveness only" and then "Try Out".

<img src="https://github.com/user-attachments/assets/029518de-efc6-49ed-86c6-63c813bd6894" width="800" title="Snipped February 18, 2025" />

Click "Apply for access".

<img src="https://github.com/user-attachments/assets/3b7d6f98-a599-411e-8db3-72b62d1bb017" width="800" title="Snipped February 18, 2025" />

Complete and submit the form.

<img src="https://github.com/user-attachments/assets/b0d52333-bcb5-4227-849c-eace366d3fa6" width="800" title="Snipped February 18, 2025" />

The "Azure AI Gating Team" will email and promise to respond in ~10 business days.

-------------------------

#### 4.3 Portrait Processing

<img src="https://github.com/user-attachments/assets/9f4ed2f0-dd8f-4c93-af7c-37c190b14df5" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/ea5b5082-3af4-4b79-8635-14447d980f1b" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs and review the generated portrait.

-------------------------

#### 4.4 Photo ID Matching

<img src="https://github.com/user-attachments/assets/d8273d24-dd8f-4f8b-9585-dd4fc1ac4180" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box and compare to the Camera Preview.

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 5: Image Analysis

This exercise demonstrates how to use Azure AI Vision Image Analysis to extract meaningful insights from images, including object detection, caption generation, and scene understanding.

Navigate to [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and log in with your Azure credentials.

<img src="https://github.com/user-attachments/assets/f80fe268-afa1-498a-a428-770f8690c05b" width="800" title="Snipped February 18, 2025" />

Click on the "Image analysis" tab.

------------------------- ------------------------- -------------------------

### 5.1 Search Photos with Image Retrieval

#### 5.1.1 Low Code

<img src="https://github.com/user-attachments/assets/340088a0-c508-46a1-bb03-278d74c121f0" width="800" title="Snipped February 18, 2025" />

Iteratively click the sample image sets, select a retrieval query and click Search.

<img src="https://github.com/user-attachments/assets/687cc074-f25f-4c59-80fd-650c80d16312" width="800" title="Snipped February 18, 2025" />

Consider trying your own image.

------------------------- -------------------------

#### 5.1.2 Pro Code

##### Step 1: Environment Variables

Open the `.env` file with your preferred text editor and append:

```text
IMAGEPATH_SEARCH=C:\temp\search.jpg
```

*Note: Consider downloading sample images from Vision Studio*

Save the file after updating.

Return to the `ai_vision.ipynb` notebook, "Load Environment Variables" code cell and append:

```python
IMAGEPATH_SEARCH = os.getenv("IMAGEPATH_SEARCH")
```

Execute cell to ensure that variables are correctly loaded.

-------------------------

##### Step 2: Pro Code

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
##  Image Analysis
Search Photos with Image Retrieval
```

Render the Markdown cell then click "+ Code" and paste the following code into the new cell:

```python
import os
import requests

def search_photos_with_image_retrieval(image_path):
    with open(image_path, "rb") as image_file:
        image_data = image_file.read()

    headers = {
        "Ocp-Apim-Subscription-Key": API_KEY,
        "Content-Type": "application/octet-stream"
    }

    # Use the "tags" feature to extract descriptive keywords for image retrieval.
    url = f"{ENDPOINT}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=tags"
    
    response = requests.post(url, headers=headers, data=image_data)
    return response

if IMAGEPATH_SEARCH and os.path.isfile(IMAGEPATH_SEARCH):
    response = search_photos_with_image_retrieval(IMAGEPATH_SEARCH)
    print(response.json())
else:
    print("IMAGEPATH_SEARCH is not defined or the file does not exist. Please check your .env file.")
```

Execute cell and review the returned JSON result.

------------------------- -------------------------

###### Expected Result  
_Note: JSON formatted and abbreviated_

```json
{
  "modelVersion": "2023-02-01-preview",
  "metadata": {
    "width": 398,
    "height": 340
  },
  "tagsResult": {
    "values": [
      {
        "name": "outdoor",
        "confidence": 0.9897938966751099
      },
      {
        "name": "tractor",
        "confidence": 0.986690878868103
      },
      {
        "name": "agriculture",
        "confidence": 0.9778040051460266
      },
      {
        "name": "farm",
        "confidence": 0.9688516855239868
      },
      {
        "name": "cash crop",
        "confidence": 0.9292412996292114
      },
      {
        "name": "agricultural machinery",
        "confidence": 0.9211045503616333
      },
      {
        "name": "plantation",
        "confidence": 0.9173258543014526
      },
      {
        "name": "crop",
        "confidence": 0.9134097099304199
      },
      {
        "name": "farmworker",
        "confidence": 0.8990883231163025
      },
      {
        "name": "soil",
        "confidence": 0.8958316445350647
      },
      {
        "name": "plant",
        "confidence": 0.8954752683639526
      },
      {
        "name": "plough",
        "confidence": 0.8804078102111816
      },
      {
        "name": "farmer",
        "confidence": 0.8800090551376343
      },
      {
        "name": "grass",
        "confidence": 0.8694881200790405
      },
      {
        "name": "mountain",
        "confidence": 0.8089953660964966
      },
      {
        "name": "ground",
        "confidence": 0.7287822365760803
      },
      {
        "name": "field",
        "confidence": 0.6489622592926025
      },
      {
        "name": "farm machine",
        "confidence": 0.5761915445327759
      }
    ]
  }
}
```

------------------------- ------------------------- -------------------------

### 5.2 Add Dense Captions to Images

<img src="https://github.com/user-attachments/assets/1c4581fe-36dc-4501-821d-c05390591c18" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/fc0dd38d-1da9-4ae5-b457-88cb45ba8d02" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

------------------------- ------------------------- -------------------------

### 5.3 Add Captions to Images

<img src="https://github.com/user-attachments/assets/df89523d-e20b-4509-b3cb-091fff6f6405" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/2a1bdd9f-aff9-412d-bb28-6a15e40a7204" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

------------------------- ------------------------- -------------------------

### 5.4 Detect Common Objects in Images

Not documented...

------------------------- ------------------------- -------------------------

### 5.5 Extract Common Tags from Images

<img src="https://github.com/user-attachments/assets/bd4656ac-989c-4f38-840f-5074db0b76bf" width="800" title="Snipped February 18, 2025" />

Choose a model, choose a language, and then iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/b865ebbf-6596-40ba-990d-271057af81f8" width="800" title="Snipped February 18, 2025" />

Review results on the "Detected attributes" / "JSON" tabs.

------------------------- ------------------------- -------------------------

### 5.6 Create Smart-Cropped Images

<img src="https://github.com/user-attachments/assets/c76c1094-341d-4fa7-8d2b-dd74a94dcef6" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/89d14157-7a61-4af0-a36f-55980aa8485c" width="800" title="Snipped February 18, 2025" />

Review results on the "Cropped image" tab and adjust aspect ratio to taste.
