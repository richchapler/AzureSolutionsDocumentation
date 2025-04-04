# Lab: AI Speech

<img src="https://github.com/user-attachments/assets/lorem" width="1000" />

## Introduction

Azure AI Speech is a cloud-based service from Microsoft that leverages advanced algorithms to process and understand spoken language, extracting valuable insights and enabling dynamic communication solutions. It encompasses a range of capabilities—from real-time speech-to-text transcription and speaker recognition to lifelike text-to-speech synthesis and multilingual translation—all designed to solve real-world problems quickly and effectively.

## Sections

### Prepare Resources  
* Set up the necessary Azure services  
* Configure your environment to securely manage API credentials using environment variables

### Optical Character Recognition (OCR)
* Learn how to extract text from images  
* Analyze and interpret the JSON output to understand how text is detected and structured

------------------------- -------------------------

## Exercise 1: Prepare Resources  
_Complete this exercise only if you intend to complete Pro-Code exercises_

### Azure
In "East US" region (or other region that supports AI Speech resources), instantiate:

* AI Services
* Application Registration (with Client Secret)
* Computer Speech
* Open AI
* Storage Account (general purpose v1)
* Video Indexer

-------------------------

### Development Environment
Prepare a machine with the following artifacts:

* [PowerShell](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/PowerShell.html)
* [Visual Studio Code](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode.html) with [Jupyter](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/VisualStudioCode_Jupyter.html)
* [Python (including Virtual Environment)](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/Python.html) with the [dotenv module](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/Python_DotEnvModule.html)

-------------------------

#### Azure Identity Module

Open Visual Studio Code, navigate to View >> Terminal, then execute the following command:

```powershell
pip install azure-identity
```

Once installed, you'll be able to import `DefaultAzureCredential` in your code to obtain access tokens via your application registration credentials (rather than using keys).

-------------------------

#### Python Notebook

Open Visual Studio Code and create a new Jupyter Notebook by selecting File > New File, then searching for and selecting Jupyter Notebook. Save the new file as `ai_speech.ipynb` in `c:\myProject`.

-------------------------

#### Environment Variables

Create and initialize the `.env` file from within your notebook:

Add a Markdown cell with the following annotation:
```markdown
## Create `.env` File  
Create (if necessary) and prompt for update with API credentials and image path.
```

Then add a Code cell and paste the following code:
```python
import os
env_file = ".env"
if not os.path.isfile(env_file):
    open(env_file, "w").close()
    print(f".env file created at {os.path.abspath(env_file)}")
else:
    print(f".env file found at {os.path.abspath(env_file)}")
```

Execute the cell. The output should indicate whether the `.env` file was created or already exists.

Open the `.env` file and append the following lines:
```text
COMPUTER_VISION_API_KEY={Computer Speech, KEY 1}
COMPUTER_VISION_ENDPOINT=https://{prefix}cv.cognitiveservices.azure.com/

VIDEO_INDEXER_ACCOUNT_ID={Video Indexer, Account ID}
VIDEO_INDEXER_LOCATION={Video Indexer, Location}
VIDEO_INDEXER_TOKEN={Video Indexer, Access Token}
```

Add a Markdown cell with the following annotation:
```markdown
## Load Environment Variables  
This cell loads the configuration settings from your `.env` file, ensuring that your API credentials and file paths are available for subsequent code cells.
```

Add a Code cell to load these environment variables using the dotenv module:
```python
import os
from dotenv import load_dotenv

env_file = ".env"
load_dotenv(env_file)

COMPUTER_VISION_API_KEY = os.getenv("COMPUTER_VISION_API_KEY")
COMPUTER_VISION_ENDPOINT = os.getenv("COMPUTER_VISION_ENDPOINT")
VIDEO_INDEXER_ACCOUNT_ID = os.getenv("VIDEO_INDEXER_ACCOUNT_ID")
VIDEO_INDEXER_LOCATION = os.getenv("VIDEO_INDEXER_LOCATION")
VIDEO_INDEXER_TOKEN = os.getenv("VIDEO_INDEXER_TOKEN")
```

Execute the cell and restart the kernel to ensure that the new variable is correctly loaded.

------------------------- -------------------------
------------------------- -------------------------

Navigate to [Azure AI | Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

## Exercise 2: Speech Capabilities by Scenario

------------------------- -------------------------

### Captioning with Speech to Text

#### Low Code

#### Pro Code

#### Update Environment Variables

Append the following line to your `.env` file:
```text
LOREM=C:\myProject\ipsum.jpg
```

Append the following code to the "Load Environment Variables" code in the `ai_speech.ipynb` notebook:
```python
LOREM = os.getenv("LOREM")
```

Re-execute "Load Environment Variables" code and restart the kernel.

-------------------------

#### Add Demonstration Code

Click "+ Markdown" and paste the following annotation into the resulting cell:
```markdown
## Lorem
Ipsum
```

Render the Markdown cell by clicking the checkmark in the upper-right controls.

Click ""+ Code"" and paste the following code into the new cell:
```python
...
```

Execute cell and review the returned JSON result.

------------------------- -------------------------

##### Expected Result  
_Note: JSON formatted and abbreviated_

```json
...
```

------------------------- -------------------------

### Post Call Transcription and Analytics
#### Low Code
#### Pro Code

------------------------- -------------------------

### Live Chat Avatar
#### Low Code
#### Pro Code

------------------------- -------------------------

### Language Learning
#### Low Code
#### Pro Code

------------------------- -------------------------

### Video Translation
#### Low Code
#### Pro Code

------------------------- -------------------------
------------------------- -------------------------

## Exercise 3: Speech to Text

### Real-Time Speech to Text
#### Low Code
#### Pro Code

------------------------- -------------------------

### Whisper Model in Azure OpenAI Service
#### Low Code
#### Pro Code

------------------------- -------------------------

### Batch Speech to Text
#### Low Code
#### Pro Code

------------------------- -------------------------

### Custom Speech
#### Low Code
#### Pro Code

------------------------- -------------------------

### Pronunciation Assessment with Speech to Text
#### Low Code
#### Pro Code

------------------------- -------------------------

### Speech Translation
#### Low Code
#### Pro Code

------------------------- -------------------------
------------------------- -------------------------

## Exercise 4: Text to Speech

### Voice Gallery
#### Low Code
#### Pro Code

------------------------- -------------------------

### Custom Voice
#### Low Code
#### Pro Code

------------------------- -------------------------

### Personal Voice
#### Low Code
#### Pro Code

------------------------- -------------------------

### Audio Content Creation
#### Low Code
#### Pro Code

------------------------- -------------------------

### Text to Speech Avatar
#### Low Code
#### Pro Code

------------------------- -------------------------
------------------------- -------------------------

## Exercise 5: Voice Assistant

### Custom Keyword
#### Low Code
#### Pro Code