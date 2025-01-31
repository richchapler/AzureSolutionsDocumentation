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

### Exercise 2: Vision Studio

Open [Azure AI | Vision Studio](https://portal.vision.cognitive.azure.com/) and browse the interface.

<img src="https://github.com/user-attachments/assets/46ec9526-8625-4103-912a-b23cb224c3ff" width="800" title="Snipped January 31, 2025" />

We will focus on the following topics:

- **Optical Character Recognition (OCR)** - Extract text from images, including handwriting and printed text
- **Spatial Analysis** - Detect people, movement, and zones in real-time video (used for surveillance, retail, etc.) ... **HARTIHA: IN SCOPE?**
- **Face** - Detect and analyze faces, including emotions and age estimation ... **HARTIHA: IN SCOPE?**
- Image Analysis
  - **Object Detection** - Identify objects and returns bounding boxes
  - **Caption Images** - Generate descriptions of images in natural language

### Exercise 2a: Vision Studio | Optical Character Recognition (OCR)

### **Exercise 2a: Vision Studio | Optical Character Recognition (OCR)**

<img src="https://github.com/user-attachments/assets/9a50e053-c04f-4fe3-846c-4dec2d6bbac0" width="800" title="Snipped January 31, 2025" />

Select **Optical Character Recognition (OCR)** from the left menu

---

### **Step 2: Upload an Image**
1. Click **"Try OCR"**
2. Upload an image containing text (handwritten or printed)
3. Click **Analyze**

---

### **Step 3: Review Extracted Text**
After processing, Vision Studio will display:
- **Extracted text from the image**
- **Confidence scores** for accuracy
- **JSON output** (expand to view full response)

---

### **Step 4: Compare with Python SDK (Optional)**
If needed, use **Python in VS Code** to replicate the results:

1. Create `ocr.py`  
2. Add the following:
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
       print(perform_ocr("test.jpg"))
   ```
3. Run in **PowerShell**:
   ```powershell
   python ocr.py
   ```
4. Compare the **Vision Studio JSON output** with the **Python script JSON output**.

---

### **Step 5: Discussion**
- What differences exist between **Vision Studio OCR output** and **Python SDK OCR output**?
- When would you **prefer the SDK** over **Vision Studio** (e.g., automation, real-time processing)?

---

### **Next Steps**
Proceed to **Exercise 2b: Vision Studio | Image Analysis (Object Detection & Captions)**.

---

Would you like to keep Python in this section, or remove it entirely and keep it **purely Vision Studio**? 🚀






---

## Module 1: Object Detection and OCR using Azure AI Vision

## Objective
- Extract text from an image using OCR
- Detect objects and their bounding boxes
- Generate image captions
- Return results in JSON format

## Prerequisites
- Module 0 completed
- Azure AI Vision API Key and Endpoint
- Sample images (provided or sourced by the user)

## Steps

1. Install Required Libraries
   ```sh
   pip install azure-ai-vision
   ```

2. Set Up API Key and Endpoint
   Modify `config.py`:
   ```python
   API_KEY = "your_azure_ai_vision_api_key"
   ENDPOINT = "your_azure_ai_vision_endpoint"
   ```

3. Perform OCR (Text Extraction)
   Create `ocr.py`:
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

   ```

4. Detect Objects and Extract Bounding Boxes
   Create `object_detection.py`:
   ```python
   def detect_objects(image_path):
       client = ImageAnalysisClient(endpoint=ENDPOINT, credential=AzureKeyCredential(API_KEY))
       with open(image_path, "rb") as image:
           result = client.analyze_image(image, visual_features=[VisualFeatures.OBJECTS])

       objects = [
           {"name": obj.name, "confidence": obj.confidence, "bounding_box": obj.bounding_box.as_dict()}
           for obj in result.objects
       ]
       return {"objects": objects}
   ```

5. Generate Image Captions
   Create `image_caption.py`:
   ```python
   def generate_caption(image_path):
       client = ImageAnalysisClient(endpoint=ENDPOINT, credential=AzureKeyCredential(API_KEY))
       with open(image_path, "rb") as image:
           result = client.analyze_image(image, visual_features=[VisualFeatures.CAPTION])

       return {"caption": result.caption.text if result.caption else "No caption generated"}
   ```

6. Combine Results and Output as JSON
   Create `main.py`:
   ```python
   import json
   from ocr import perform_ocr
   from object_detection import detect_objects
   from image_caption import generate_caption

   def analyze_image(image_path):
       results = {
           "ocr": perform_ocr(image_path),
           "objects": detect_objects(image_path),
           "caption": generate_caption(image_path),
       }
       return results

   if __name__ == "__main__":
       image_path = "test.jpg"  # Replace with actual image path
       analysis_results = analyze_image(image_path)
       print(json.dumps(analysis_results, indent=2))
   ```

## Expected Output (Example JSON)
```json
{
  "ocr": {
    "text": "Welcome to Azure AI Vision"
  },
  "objects": [
    {
      "name": "person",
      "confidence": 0.98,
      "bounding_box": {
        "x": 100,
        "y": 150,
        "width": 200,
        "height": 300
      }
    },
    {
      "name": "laptop",
      "confidence": 0.85,
      "bounding_box": {
        "x": 400,
        "y": 250,
        "width": 150,
        "height": 100
      }
    }
  ],
  "caption": "A person using a laptop in an office"
}
```
