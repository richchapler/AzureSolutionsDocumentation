# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Table of Contents  

- [Exercise 1: Prepare Environment](#exercise-1-prepare-environment)  
- [Exercise 2: Optical Character Recognition (OCR)](#exercise-2-optical-character-recognition-ocr)  
- [Exercise 3: Spatial Analysis](#exercise-3-spatial-analysis) 
- [Exercise 4: Face](#exercise-4-face) 

## Resource Requirements
- Cognitive Services
- Computer Vision

## Exercise 1: Prepare Environment
_Complete this exercise only if you intend to complete Pro-Code sub-exercises_

### Visual Studio Code
_Skip if you are using an Azure DevTestLab VM_

Install [Visual Studio Code](https://code.visualstudio.com/download)

<img src="https://github.com/user-attachments/assets/c5a8b922-259c-4add-9637-6f83d825e97b" width="800" title="Snipped February 3, 2025" />

After installation, open Visual Studio Code and navigate to the "Extensions: Marketplace" pane (`Ctrl+Shift+X`)

<img src="https://github.com/user-attachments/assets/815273f0-a430-4e08-9056-d2fc91b4cec9" width="800" title="Snipped January 31, 2025" />

Search for, select and install `Python`, then restart Visual Studio Code.

------------------------- -------------------------

### Python

Install the latest version of [Python.org](https://www.python.org/downloads/)

<img src="https://github.com/user-attachments/assets/1f103580-e5a3-46fc-82c7-2c91e700e75c" width="800" title="Snipped February 3, 2025" />

_Note: Select "Add Python to PATH" before clicking "Install Now"_

Open PowerShell and verify the installation:

```powershell
python --version
```

Expected output (version may vary):

```
Python 3.13.1
```

------------------------- -------------------------

#### Python Virtual Environment

Open Visual Studio Code and click View > Terminal to open the PowerShell terminal.

<img src="https://github.com/user-attachments/assets/e74ffba0-e515-42f9-b267-74631bef954f" width="800" title="Snipped February 3, 2025" />

Execute the following command to create and then navigate to a project directory:

```powershell
mkdir "$HOME\Documents\AI-Vision-Lab"
cd "$HOME\Documents\AI-Vision-Lab"
```

Execute the following command to create a virtual environment:

```powershell
python -m venv venv
```

Execute the following command to activate the virtual environment:

```powershell
venv\Scripts\Activate
```

After running this command, note that your prompt changes to include prefix `(venv)`, showing that it is working inside the virtual environment.

------------------------- -------------------------

### Jupyter

Open Visual Studio Code and click View > Terminal to open the PowerShell terminal.

<img src="https://github.com/user-attachments/assets/b86a2382-9dea-4dd4-81f9-41a191da00a3" width="800" title="Snipped February 5, 2025" />

Execute the following command to install Jupyter:

```powershell
pip install jupyter
```

Navigate to Extensions, then search for and select "Jupyter"

<img src="https://github.com/user-attachments/assets/3ac8dc62-cc5a-4264-992f-a809c0524c55" width="800" title="Snipped February 5, 2025" />

Click "Install".

<img src="https://github.com/user-attachments/assets/d618a771-041c-40a2-a351-6c63cc51d0bc" width="800" title="Snipped February 5, 2025" />

Test added functionality by clicking "File" >> "New File", then search for and select "Jupyter Notebook".

<img src="https://github.com/user-attachments/assets/066afd03-d3ff-435e-a651-ae6a28a57bc3" width="800" title="Snipped February 5, 2025" />

Click "View" >> "Command Palette", then search for and select "Python: Select Interpreter".

<img src="https://github.com/user-attachments/assets/bfd68ebc-350c-421e-9ec1-1f1f8fd3b35e" width="800" title="Snipped February 5, 2025" />

Select the "Recommended" interpreter.

<img src="https://github.com/user-attachments/assets/4abe9278-ecf8-49bd-aa00-330e06fed88b" width="800" title="Snipped February 5, 2025" />

------------------------- -------------------------

### Notebook

In Visual Studio Code, click "File" >> "New File", then search for and select "Jupyter Notebook". Save the file as `ocr.ipynb`.  

#### Environment Variables

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Environment Variables 
This cell checks if the `.env` file exists before attempting to load API credentials.  
If missing, it creates one and prompts you to update it.
```

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.

Click "+ Code" and paste the following code into the resulting cell:

```python
import os
from dotenv import load_dotenv

env_file = ".env"

if not os.path.isfile(env_file):
    with open(env_file, "w") as f:
        f.write("API_KEY=\nENDPOINT=\nIMAGE_PATH=\n")
    print(f".env file created at {os.path.abspath(env_file)}. Please update it with your API credentials and image path.")

load_dotenv()

API_KEY = os.getenv("API_KEY")
ENDPOINT = os.getenv("ENDPOINT")
IMAGE_PATH = os.getenv("IMAGE_PATH")

missing_messages = []

if not API_KEY:
    missing_messages.append("\n* Error: Missing API_KEY *\n"
                            "1. Navigate to https://portal.azure.com/, search for 'AI Vision', and select your deployed service\n"
                            "2. In the left navigation, select 'Resource Management' >> 'Keys and Endpoint'\n"
                            f"3. Paste the 'Key 1' value into {os.path.abspath(env_file).lower()} as API_KEY")

if not ENDPOINT:
    missing_messages.append("\n* Error: Missing ENDPOINT *\n"
                            "1. Navigate to https://portal.azure.com/, search for 'AI Vision', and select your deployed service\n"
                            "2. In the left navigation, select 'Resource Management' >> 'Keys and Endpoint'\n"
                            f"3. Paste the 'Endpoint' value into {os.path.abspath(env_file).lower()} as ENDPOINT")

if not IMAGE_PATH:
    missing_messages.append("\n* Error: Missing IMAGE_PATH *\n"
                            "1. Store a sample file {e.g., https://ocr.space/Content/Images/receipt-ocr-original.jpg} on your device {e.g, c:\\downloads\\receipt-ocr-original.jpg}\n"
                            f"2. Paste the local file path into {os.path.abspath(env_file).lower()} as IMAGE_PATH")

if missing_messages:
    print("\n".join(missing_messages))
    print(f"\nAfter correction, restart the kernel and re-execute this cell.")
```

The resulting `.env` file should look like:

```text
API_KEY={KEY 1}
ENDPOINT={Endpoint}
IMAGE_PATH={Local File Path}
```

Execute the code cell.

<img src="https://github.com/user-attachments/assets/f213b897-ff09-4b80-b235-64b22e8c60fd" width="800" title="Snipped February 5, 2025" />

------------------------- -------------------------

#### Dependencies  

Click "+ Markdown" and paste the following annotation into the resulting cell:  

```markdown
## Dependencies  
This cell checks for missing dependencies and installs them if necessary.
```

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.  

Click "+ Code" and paste the following code into the resulting cell:  

```python
import sys
import subprocess

required_packages = [
    "azure-ai-vision-imageanalysis",
    "python-dotenv",
    "opencv-python",
    "matplotlib",
    "requests"
]

missing_packages = [pkg for pkg in required_packages if subprocess.run([sys.executable, "-m", "pip", "show", pkg], capture_output=True, text=True).returncode != 0]

if missing_packages:
    print(f"Installing missing dependencies: {', '.join(missing_packages)}")
    subprocess.run([sys.executable, "-m", "pip", "install", *missing_packages])
else:
    print("All dependencies are already installed.")
```

Execute the code cell.

<img src="https://github.com/user-attachments/assets/a33e70c2-dbeb-417e-ab9a-aed24e32d6d8" width="800" title="Snipped February 5, 2025" />

------------------------- ------------------------- ------------------------- -------------------------

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

Review results on the "Detected attributes" and "JSON" tabs.

------------------------- -------------------------

### Pro Code  

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Exercise 2: Optical Character Recognition (OCR) 
Use Optical Character Recognition (OCR) on an image file (supported formats: JPEG, PNG, BMP, GIF, TIFF)
```  

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.

Click "+ Code" and paste the following code into the resulting cell:

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

if os.path.isfile(IMAGE_PATH):
    response = perform_ocr(IMAGE_PATH)
    print(response.json())
```

Execute the code cell.

<img src="https://github.com/user-attachments/assets/80f09c5c-1042-42f6-a9cb-6e196afde291" width="800" title="Snipped February 5, 2025" />

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

Review results on the "Detected attributes" and "JSON" tabs.

-------------------------

#### Liveness Detection

<img src="https://github.com/user-attachments/assets/d8f00fd3-c59c-4182-9ae9-f082bba05d84" width="800" title="Snipped February 18, 2025" />

Click "Face Liveness only" and then "Try Out".

<img src="https://github.com/user-attachments/assets/029518de-efc6-49ed-86c6-63c813bd6894" width="800" title="Snipped February 18, 2025" />

Click "Apply for access".

<img src="https://github.com/user-attachments/assets/3b7d6f98-a599-411e-8db3-72b62d1bb017" width="800" title="Snipped February 18, 2025" />

Sadly, approval is not instant... complete and submit the form and return when you have approved access.

-------------------------

#### Portrait Processing

<img src="https://github.com/user-attachments/assets/BLAH" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/BLAH" width="800" title="Snipped February 18, 2025" />

Interact with resulting functionality {e.g., "BLAH"}.

-------------------------

#### Photo ID Matching

<img src="https://github.com/user-attachments/assets/BLAH" width="800" title="Snipped February 18, 2025" />

Iteratively click the samples to the right of the "Drag and drop a file..." box.

<img src="https://github.com/user-attachments/assets/BLAH" width="800" title="Snipped February 18, 2025" />

Interact with resulting functionality {e.g., "BLAH"}.












