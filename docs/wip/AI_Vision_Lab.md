# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [Azure AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)
* [Visual Studio Code](https://code.visualstudio.com/download) with Python extension
* [Python](https://www.python.org/) with "Add Python to PATH"

## Exercise 1: Prepare Visual Studio Code

Open Vision Studio Code >> Extensions, then search for and select Python from the "Extensions: Marketplace" pane.

<img src="https://github.com/user-attachments/assets/815273f0-a430-4e08-9056-d2fc91b4cec9" width="800" title="Snipped January 31, 2025" />

Confirm installation, then close. Restart Visual Studio Code.

In the toolbar, click View >> Terminal to open the terminal.

<img src="https://github.com/user-attachments/assets/eaf2dc75-80c4-406b-8dcd-675325c1bc75" width="800" title="Snipped January 31, 2025" />

Activate the virtual environment by executing the following PowerShell command:
```powershell
python -m venv venv
```

Ensure the latest bug fixes, security patches, and features by executing the following PowerShell command:
```powershell
python -m pip install --upgrade pip
```

Download and install the latest version of `azure-ai-vision-imageanalysis` by executing the following PowerShell command:
```powershell
pip install azure-ai-vision-imageanalysis
```

## Exercise 2: Optical Character Recognition (OCR)

### ...using Vision Studio 

Open [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and familiarize yourself with the interface and options.

<img src="https://github.com/user-attachments/assets/46ec9526-8625-4103-912a-b23cb224c3ff" width="800" title="Snipped January 31, 2025" />

We will focus on the following topics:

- **Optical Character Recognition (OCR)** - Extract text from images, including handwriting and printed text
- **Spatial Analysis** - Detect people, movement, and zones in real-time video (used for surveillance, retail, etc.) ... **HARTIHA: IN SCOPE?**
- **Face** - Detect and analyze faces, including emotions and age estimation ... **HARTIHA: IN SCOPE?**
- Image Analysis
  - **Object Detection** - Identify objects and returns bounding boxes
  - **Caption Images** - Generate descriptions of images in natural language

_NOTE: ON HOLD WHILE SUPPORT WORKS THROUGH VISION STUDIO UI ISSUES_

<img src="https://github.com/user-attachments/assets/9a50e053-c04f-4fe3-846c-4dec2d6bbac0" width="800" title="Snipped January 31, 2025" />

Select **Optical Character Recognition (OCR)** and then click "Extract text from images".

<img src="https://github.com/user-attachments/assets/fb90abc3-3c54-4f2c-a356-df292d1f3a77" width="800" title="Snipped January 31, 2025" />

#### **Step 1: Extract Text from an Image**
1. Click **"Try OCR"**
2. Upload an image containing printed or handwritten text
3. Click **Analyze**

#### **Step 2: Review the Results**
After processing, Vision Studio will display:
- **Extracted text**
- **Confidence scores**
- **JSON output** (expand to view full response)

### Optical Character Recognition (OCR) using Python 

#### **Step 1: Install Dependencies**
Ensure the required package is installed by running:

```powershell
pip install azure-ai-vision-imageanalysis
```

#### **Step 2: Set Up API Credentials**
Create a file called **`config.py`** and add:

```python
API_KEY = "your-azure-ai-vision-api-key"
ENDPOINT = "your-azure-ai-vision-endpoint"
```

#### **Step 3: Implement OCR in Python**
Create a file called **`ocr.py`** and add the following:

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
from config import API_KEY, ENDPOINT

def perform_ocr(image_path):
    client = ImageAnalysisClient(endpoint=ENDPOINT, credential=AzureKeyCredential(API_KEY))
    with open(image_path, "rb") as image:
        result = client.analyze_image(image, visual_features=[VisualFeatures.READ])

    return {"text": result.read.text if result.read else "No text detected"}

if __name__ == "__main__":
    image_path = "test.jpg"  # Replace with actual image path
    print(perform_ocr(image_path))
```

---

#### **Step 4: Run the Python Script**
Execute the script in **PowerShell**:

```powershell
python ocr.py
```
