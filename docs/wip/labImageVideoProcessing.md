# Lab Outline: Image and Video Processing with Azure AI Services

## Exercise 0: Setting Up Visual Studio Code for Python Development

## Objective
Ensure that participants have a properly configured environment to develop and run Python applications, particularly for integrating with the Azure AI Vision API.

## Prerequisites
- Windows, macOS, or Linux system
- Internet connection
- Azure Subscription (for API key and endpoint)

## Steps

1. Install Python
   - Download the latest version of Python from [python.org](https://www.python.org/)
   - Install Python, ensuring you check "Add Python to PATH"

2. Install Visual Studio Code (VS Code)
   - Download and install [VS Code](https://code.visualstudio.com/)
   - Open VS Code and install the Python extension:
     - Go to Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X` on macOS)
     - Search for "Python"
     - Click Install

3. Set Up a Virtual Environment
   - Open a new terminal in VS Code
   - Run:
     ```sh
     python -m venv venv
     ```
   - Activate the environment:
     - Windows: `venv\Scripts\activate`
     - macOS/Linux: `source venv/bin/activate`
   - Install required dependencies:
     ```sh
     pip install azure-ai-vision
     ```

4. Obtain Azure AI Vision API Key and Endpoint
   - Sign in to the [Azure Portal](https://portal.azure.com/)
   - Search for Cognitive Services
   - Create an AI Vision resource
   - Copy the API Key and Endpoint for use in the exercises

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
