# Lab: AI Vision

<img src="https://github.com/user-attachments/assets/2158f09e-207e-4b86-b45d-5d381090f8d2" width="1000" />

## Exercise 1: Prepare Environment

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

### **Dependencies**  

Execute the following command to upgrade `pip`:  

```powershell
python -m pip install --upgrade pip
```

Execute the following command to install the required dependencies:  

```powershell
pip install azure-ai-vision-imageanalysis python-dotenv opencv-python matplotlib requests
```

- `azure-ai-vision-imageanalysis` – Azure AI Vision SDK for OCR  
- `python-dotenv` – Secure storage of API credentials in a `.env` file  
- `opencv-python` – Image processing library (used to display images)  
- `matplotlib` – Library for image visualization  
- `requests` – HTTP library for downloading images from URLs  

Run the command to install all dependencies.

---

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

---

## Exercise 2: Optical Character Recognition (OCR)  

This exercise demonstrates how to use Azure AI Vision OCR to extract text from images. You will:  
- Use Vision Studio to analyze images with OCR  
- Use Jupyter Notebooks in Visual Studio Code for interactive coding  
- Analyze images using Azure AI Vision OCR via API  
- Retrieve text and confidence scores  

### Low Code  

This documentation assumes the following Azure resources are ready for use:
* [AI Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/overview)

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

<img src="https://github.com/user-attachments/assets/4a5bc580-1867-4c5f-9813-d4c28f81714b" width="800" title="Snipped February 5, 2025" />

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Load API Credentials  
This cell loads the API key and endpoint from the `.env` file to authenticate with Azure AI services.
```

Click "+ Code" and paste the following code into the resulting cell:

```python
from dotenv import load_dotenv
import os

load_dotenv()

API_KEY = os.getenv("API_KEY")
ENDPOINT = os.getenv("ENDPOINT")

print(f"API Key: {'Loaded' if API_KEY else 'Missing'}")
print(f"Endpoint: {'Loaded' if ENDPOINT else 'Missing'}")
```

Click "Run All" to test






Run the cell (`Shift+Enter`) to confirm the API key and endpoint are loaded.  

#### Install required dependencies  

In the next cell, install the Azure AI Vision SDK if not already installed:  

```python
!pip install azure-ai-vision-imageanalysis
```

Run the cell to install.  

#### Define OCR function  

In the next cell, add:  

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential

client = ImageAnalysisClient(endpoint=ENDPOINT, credential=AzureKeyCredential(API_KEY))

def perform_ocr(image_path):
    with open(image_path, "rb") as image:
        result = client.analyze_image(image, visual_features=[VisualFeatures.READ])

    if result.read and result.read.text:
        return result.read.text
    return "No text detected"

print("OCR function ready")
```

Run the cell to load the function.  

#### Upload and analyze an image  

##### Local image  

In a new cell, add:  

```python
image_path = "test.jpg"  # Replace with your image path

extracted_text = perform_ocr(image_path)

print("Extracted Text:\n", extracted_text)
```

Run the cell and verify the extracted text.  

##### Web image  

To analyze an image from a URL instead, add:  

```python
import requests

def download_image(url, save_path="downloaded_image.jpg"):
    response = requests.get(url)
    if response.status_code == 200:
        with open(save_path, "wb") as file:
            file.write(response.content)
        return save_path
    return None

image_url = "https://example.com/sample.jpg"  # Replace with actual image URL

image_path = download_image(image_url)
if image_path:
    extracted_text = perform_ocr(image_path)
    print("Extracted Text:\n", extracted_text)
else:
    print("Failed to download image")
```

Run the cell to analyze text from a web image.  

#### Display image with OCR results  

```python
import matplotlib.pyplot as plt
import cv2

def display_image(image_path, extracted_text):
    img = cv2.imread(image_path)
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    plt.figure(figsize=(8, 6))
    plt.imshow(img)
    plt.axis("off")
    plt.title("Extracted Text:\n" + extracted_text, fontsize=12)
    plt.show()

display_image(image_path, extracted_text)
```

Run the cell to display the image with recognized text.  

#### Export OCR results to a file  

```python
output_file = "ocr_results.txt"

with open(output_file, "w", encoding="utf-8") as file:
    file.write(extracted_text)

print(f"OCR results saved to {output_file}")
```

Run the cell to save results.  

---

## Summary  

You have successfully:  
- Used Vision Studio for OCR  
- Set up Jupyter Notebooks in Visual Studio Code  
- Loaded API credentials securely  
- Used Azure AI Vision OCR to extract text from images  
- Displayed and saved extracted text  
