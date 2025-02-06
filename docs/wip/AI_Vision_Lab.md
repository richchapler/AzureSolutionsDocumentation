# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Table of Contents  

- [Exercise 1: Prepare Environment](#exercise-1-prepare-environment)  
  - [Visual Studio Code](#visual-studio-code)  
  - [Python](#python)  
  - [Python Virtual Environment](#python-virtual-environment)  
  - [Jupyter](#jupyter)  
- [Exercise 2: Optical Character Recognition (OCR)](#exercise-2-optical-character-recognition-ocr)  
  - [Low Code](#low-code)  
    - [Extract text from an image](#extract-text-from-an-image)  
    - [Review the results](#review-the-results)  
  - [Pro Code](#pro-code)  
    - [Environment Variables](#environment-variables)  
    - [Dependencies](#dependencies)  
    - [OCR Function](#ocr-function)  
    - [Analyze Image](#analyze-image)  
    - [Analyze an Image from a URL](#analyze-an-image-from-a-url)  
    - [Display Image with OCR Results](#display-image-with-ocr-results)  
    - [Export OCR Results to a File](#export-ocr-results-to-a-file)  
- [Summary](#summary)  

## Exercise 1: Prepare Environment

### Visual Studio Code

Install [Visual Studio Code](https://code.visualstudio.com/download)

<img src="https://github.com/user-attachments/assets/c5a8b922-259c-4add-9637-6f83d825e97b" width="800" title="Snipped February 3, 2025" />

After installation, open Visual Studio Code and navigate to the "Extensions: Marketplace" pane (`Ctrl+Shift+X`)

<img src="https://github.com/user-attachments/assets/815273f0-a430-4e08-9056-d2fc91b4cec9" width="800" title="Snipped January 31, 2025" />

Search for, select and install `Python`, then restart Visual Studio Code.

---

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

---

### Python Virtual Environment

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

---

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

---

## Exercise 2: Optical Character Recognition (OCR)  

This exercise demonstrates how to use Azure AI Vision OCR to extract text from images. You will:  
- Use Vision Studio to analyze images with OCR  
- Use Jupyter Notebooks in Visual Studio Code for interactive coding  
- Analyze images using Azure AI Vision OCR via API  
- Retrieve text and confidence scores  

### Low Code  

_Note: This documentation assumes that [AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview) is ready for use_

Open [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and familiarize yourself with the interface and options.  

<img src="https://github.com/user-attachments/assets/46ec9526-8625-4103-912a-b23cb224c3ff" width="800" title="Snipped January 31, 2025" />  

This section covers:  

- Optical Character Recognition (OCR) – Extract text from images, including handwriting and printed text  
- Spatial Analysis – Detect people, movement, and zones in real-time video  
- Face – Detect and analyze faces, including emotions and age estimation  
- Image Analysis  
  - Object Detection – Identify objects and return bounding boxes  
  - Caption Images – Generate descriptions of images in natural language  

**Note: On hold while support works through Vision Studio UI issues**  

<img src="https://github.com/user-attachments/assets/9a50e053-c04f-4fe3-846c-4dec2d6bbac0" width="800" title="Snipped January 31, 2025" />  

Select Optical Character Recognition (OCR) and then click "Extract text from images".  

<img src="https://github.com/user-attachments/assets/fb90abc3-3c54-4f2c-a356-df292d1f3a77" width="800" title="Snipped January 31, 2025" />  

#### Extract text from an image  

1. Click "Try OCR"  
2. Upload an image containing printed or handwritten text  
3. Click "Analyze"  

#### Review the results  

After processing, Vision Studio will display:  

- Extracted text  
- Confidence scores  
- JSON output (expand to view full response)  

---

### Pro Code  

Click "File" >> "New File", then search for and select "Jupyter Notebook". Save the file as `ocr.ipynb`.  

### Environment Variables

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
    missing_messages.append("\n*** Error: Missing API_KEY ***\n"
                            "1. Navigate to https://portal.azure.com/, search for 'AI Vision', and select your deployed service\n"
                            "2. In the left navigation, select 'Resource Management' >> 'Keys and Endpoint'\n"
                            f"3. Paste the 'Key 1' value into {os.path.abspath(env_file).lower()} as API_KEY")

if not ENDPOINT:
    missing_messages.append("\n*** Error: Missing ENDPOINT ***\n"
                            "1. Navigate to https://portal.azure.com/, search for 'AI Vision', and select your deployed service\n"
                            "2. In the left navigation, select 'Resource Management' >> 'Keys and Endpoint'\n"
                            f"3. Paste the 'Endpoint' value into {os.path.abspath(env_file).lower()} as ENDPOINT")

if not IMAGE_PATH:
    missing_messages.append("\n*** Error: Missing IMAGE_PATH ***\n"
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

---

### Dependencies  

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

---

### OCR Function  

Click "+ Markdown" and paste the following annotation into the resulting cell:  

```markdown
## OCR Function  
Define `perform_ocr` function, which sends an image to Azure AI Vision for text extraction.
```

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.  

Click "+ Code" and paste the following code into the resulting cell:  

```python
import os
import requests

with open(IMAGE_PATH, "rb") as image_file:
    image_data = image_file.read()

headers = {
    "Ocp-Apim-Subscription-Key": API_KEY,
    "Content-Type": "application/octet-stream"
}

url = f"{ENDPOINT}/computervision/imageanalysis:analyze?api-version=2023-02-01-preview&features=read"
response = requests.post(url, headers=headers, data=image_data)
result = response.json()
print("API Response:", result)
```

Execute the code cell.

<img src="https://github.com/user-attachments/assets/6f48c7aa-d0ed-4b12-bbf8-2a46cf0f2a6e" width="800" title="Snipped February 5, 2025" />

---

### **Analyze Image**  

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Analyze Image  
Use Optical Character Recognition (OCR) on an image file (supported formats: JPEG, PNG, BMP, GIF, TIFF).  
```

Click the checkmark in the upper-right of the cell to "Stop Editing Cell" and render the markdown.

Click "+ Code" and paste the following code into the resulting cell:

```python
import os

if not IMAGE_PATH:
    print("*** Error: IMAGE_PATH is not set in the .env file ***")
elif not os.path.isfile(IMAGE_PATH):
    print(f"*** Error: File not found ***\n{IMAGE_PATH}")
else:
    extracted_text = perform_ocr(IMAGE_PATH)
    print("Extracted Text:\n", extracted_text)
```

Execute the code cell.

<img src="https://github.com/user-attachments/assets/b1ba0db4-fc9b-4762-9655-4306639be53b" width="800" title="Snipped February 5, 2025" />

Response should look like the following formatted and abbreviated JSON:

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
