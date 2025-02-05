# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Exercise 1: Compute Gallery  

This section guides you through setting up an Azure Compute Gallery to manage and version your virtual machine images. You will:  
- Create an Azure Compute Gallery  
- Capture a virtual machine image  
- Share or export the image for portability  

### Create Compute Gallery  

Open the [Azure Portal](https://portal.azure.com/), then search for and select "Azure compute galleries".

<img src="https://github.com/user-attachments/assets/4a5bc580-1867-4c5f-9813-d4c28f81714b" width="800" title="Snipped February 5, 2025" />

Click "+ Create".

<img src="https://github.com/user-attachments/assets/5e31f45e-2b73-4881-a872-b8abe528a0ae" width="800" title="Snipped February 5, 2025" />

Complete the form on the "Basics" tab and click "Next: Sharing method >".

<img src="https://github.com/user-attachments/assets/678ebbd5-4511-4528-baf5-b90afb1579dd" width="800" title="Snipped February 5, 2025" />

Confirm settings on the "Sharing" tab and click "Review + create".
<br>⚠️_Note: We will probably want to select "RBAC + share to public community gallery"_⚠️

<img src="https://github.com/user-attachments/assets/8cc43646-3f71-4b3e-89a3-448c1df187bc" width="800" title="Snipped February 5, 2025" />

Click "Create".

<img src="https://github.com/user-attachments/assets/c5455f0c-d893-4080-8635-e5cf8cd0d66e" width="800" title="Snipped February 5, 2025" />

Click "Go to resource".

---

### Create Virtual Machine  

Open the [Azure Portal](https://portal.azure.com/), then search for and select "Virtual machines".

<img src="https://github.com/user-attachments/assets/88e923c3-dead-4e20-ae49-8d44c642a7ad" width="800" title="Snipped February 5, 2025" />

Click "+ Create".

<img src="https://github.com/user-attachments/assets/04295a2c-57f1-4876-8f74-580ad67eabf5" width="800" title="Snipped February 5, 2025" />




1. Navigate to Azure Virtual Machines and click + Create  
2. Configure the virtual machine:  
   - Image: Choose Windows or Linux  
   - Size: Select an appropriate VM size  
   - Authentication: Choose Password or SSH key  
3. Click Review + Create → Create  
4. After deployment, install the required tools inside the VM  

---

### Step 3: Capture and Version the VM Image  

1. Prepare the VM for Imaging:  
   - Connect to the VM via RDP (Windows) or SSH (Linux)  
   - Run Sysprep (Windows) or waagent -deprovision+user (Linux)  
   - Shut down the VM  

2. Capture the Image:  
   - Navigate to Azure Compute Gallery  
   - Click + Create Image Definition  
   - Set a Name, OS Type, and VM Generation  
   - Click Create  

3. Version the Image:  
   - Select the created Image Definition  
   - Click + Create Image Version  
   - Choose the source VM and specify the version (e.g., `1.0.0`)  
   - Click Create  

---

### Step 4: Share or Export the Image  

#### Share within Azure  
1. Navigate to Compute Gallery → Select the image  
2. Click Sharing → Add users, groups, or external tenants  
3. Click Save  

#### Export as a VHD  
1. Navigate to the Managed Disk of the VM  
2. Click Export → Generate URL  
3. Download the VHD file for use outside Azure  

## Exercise 2: Prepare Environment

This documentation assumes the following Azure resources are ready for use:
* [AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)

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

### Visual Studio Code

Install [Visual Studio Code](https://code.visualstudio.com/download)

<img src="https://github.com/user-attachments/assets/c5a8b922-259c-4add-9637-6f83d825e97b" width="800" title="Snipped February 3, 2025" />

After installation, open Visual Studio Code and navigate to the "Extensions: Marketplace" pane (`Ctrl+Shift+X`)

<img src="https://github.com/user-attachments/assets/815273f0-a430-4e08-9056-d2fc91b4cec9" width="800" title="Snipped January 31, 2025" />

Search for, select and install `Python`, then restart Visual Studio Code.

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

After running this command, note that your prompt changes to include prefix  `(venv)` showing that it working instide the virtual environment.

### Dependencies  

Execute the following command to upgrade `pip`:  
```powershell
python -m pip install --upgrade pip
```  

Execute the following command to install the Azure AI Vision SDK:  
```powershell
pip install azure-ai-vision-imageanalysis
```  

Execute the following command to install `python-dotenv` for secure API key storage:  
```powershell
pip install python-dotenv
```

### API Credentials  

Execute the following command to create a `.env` file in the project directory:  
```powershell
New-Item -Path .env -ItemType File
```  

Execute the following command to open the `.env` file in Notepad:
```powershell
notepad .env
```

Modify and paste the following content into your `.env` file:
```text
API_KEY=<your-azure-ai-vision-api-key>
ENDPOINT=<your-azure-ai-vision-endpoint>
```  

Execute the following command to create a `config.py` file in the project directory:  
```powershell
New-Item -Path config.py -ItemType File
```

Execute the following command to open the `config.py` file in Notepad:
```powershell
notepad config.py
```

Paste the following content into your `config.py` file:
```python
from dotenv import load_dotenv
import os

load_dotenv()

API_KEY = os.getenv("API_KEY")
ENDPOINT = os.getenv("ENDPOINT")
```

_Note: Visual Studio Code + Python will be used throughout exercises in this documentation_

## Exercise 3: Optical Character Recognition (OCR)

### ...using Vision Studio 

Open [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and familiarize yourself with the interface and options.

<img src="https://github.com/user-attachments/assets/46ec9526-8625-4103-912a-b23cb224c3ff" width="800" title="Snipped January 31, 2025" />

We will focus on the following topics:

- Optical Character Recognition (OCR) - Extract text from images, including handwriting and printed text
- Spatial Analysis - Detect people, movement, and zones in real-time video (used for surveillance, retail, etc.) ... HARTIHA: IN SCOPE?
- Face - Detect and analyze faces, including emotions and age estimation ... HARTIHA: IN SCOPE?
- Image Analysis
  - Object Detection - Identify objects and returns bounding boxes
  - Caption Images - Generate descriptions of images in natural language

_NOTE: ON HOLD WHILE SUPPORT WORKS THROUGH VISION STUDIO UI ISSUES_

<img src="https://github.com/user-attachments/assets/9a50e053-c04f-4fe3-846c-4dec2d6bbac0" width="800" title="Snipped January 31, 2025" />

Select Optical Character Recognition (OCR) and then click "Extract text from images".

<img src="https://github.com/user-attachments/assets/fb90abc3-3c54-4f2c-a356-df292d1f3a77" width="800" title="Snipped January 31, 2025" />

#### Step 1: Extract Text from an Image
1. Click "Try OCR"
2. Upload an image containing printed or handwritten text
3. Click Analyze

#### Step 2: Review the Results
After processing, Vision Studio will display:
- Extracted text
- Confidence scores
- JSON output (expand to view full response)

---

### ...using Python  

Open Visual Studio Code and click File > New Text File.  

<img src="https://github.com/user-attachments/assets/dbf375c2-b3ee-4275-80f0-0a4cdc61047e" width="800" title="Snipped February 3, 2025" />  

Click "Select a language", then search for and select `Python`.  

### Implement OCR in Python  

Execute the following command to create an `ocr.py` file:  

```powershell
New-Item -Path ocr.py -ItemType File
```

Execute the following command to open `ocr.py` in Notepad:  

```powershell
notepad ocr.py
```

Modify and paste the following content into `ocr.py`:  

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

### Run the Python Script  

Execute the following command in PowerShell to run the OCR script:  

```powershell
python ocr.py
```
