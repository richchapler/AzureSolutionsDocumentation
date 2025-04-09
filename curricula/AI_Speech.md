# Lab: AI Speech

<img src="https://github.com/user-attachments/assets/c8ac15ea-d2f8-48a2-9810-4342cdef2a93" width="1000" />

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

### Azure
Instantiate Azure AI Services

LOREM IPSUM

------------------------- -------------------------

## Exercise 2: Speech Capabilities by Scenario

### Captioning with Speech to Text

#### Low Code

Navigate to [Azure AI | Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src="https://github.com/user-attachments/assets/caddfe5c-fef1-4fc1-8f52-269be78bd298" width="800" title="Snipped April, 2025" />

Click "Captioning with speech to text".

<img src="https://github.com/user-attachments/assets/c97cc6fc-7228-4230-b6a6-84f558cc0e64" width="800" title="Snipped April, 2025" />

Navigate to the "Try it out" tab >> "Sample videos" tab, click "Real-time captioning", and then scroll down on the page.

<img src="https://github.com/user-attachments/assets/6893638c-0621-4e8a-85df-cc147223f01c" width="800" title="Snipped April, 2025" />

Click the Play button the video and observe captioning.

##### Captioning Settings

The captioning settings shown on the **Speech Studio > Captioning > Real-time captioning** page are detailed on the right side of the interface and include the following parameters:

- **Recognition event**: `--realTime`  
  Enables real-time mode, which returns stable partial results as audio is processed live.

- **Stable partial result threshold**: `--threshold 3`  
  Default value is 3. This determines how many recognized words must be repeated before the result is considered stable enough to show. Higher values increase stability but delay appearance.

- **Maximum caption length (character)**: `--maxLineLength 60`  
  Sets the character limit per line for each caption. Default is 60.

- **Maximum number of caption lines**: `--lines 2`  
  Controls how many lines of text can appear simultaneously in the caption block.

- **Profanity masking**: `--profanity mask`  
  Masks any profane words detected during transcription using asterisks or similar characters.

- **Phrase list**: `--phrases "neural TTS;Cognitive Services"`  
  Custom phrase list to improve recognition of domain-specific terms or phrases. Items are semicolon-separated.

###### Example #1: Live Event Broadcasting 
> For high-energy, fast-paced content where low latency matters {e.g., sports event}.
> - `--realTime`: stream captions as the speaker talks  
> - `--threshold 2`: fast response, lower stability  
> - `--maxLineLength 50`: fits cleanly on screen overlays  
> - `--lines 2`: prevents crowding the display  
> - `--profanity mask`: keeps output appropriate for public viewing  
> - Phrase list: team names, campus locations, event-specific terms

###### Example #2: Corporate Meeting 
> Simulates real-world business environments where clarity and professionalism are key.
> - `--realTime`: supports live note-taking  
> - `--threshold 3`: balances speed and stability  
> - `--maxLineLength 60`: improves readability in transcripts  
> - `--lines 2`: clean format for screen share overlays  
> - `--profanity mask`: appropriate for formal settings  
> - Phrase list: project names, department acronyms, business terms

###### Example #3: Classroom (accessibility focus)
> Helps support learning environments with better readability and accuracy.
> - `--realTime`: enables live following of lectures  
> - `--threshold 4`: prioritizes accuracy, reduces caption flicker  
> - `--maxLineLength 60`: allows complete sentences  
> - `--lines 3`: gives room for full thoughts in captions  
> - `--profanity raw`: keeps educational integrity  
> - Phrase list: lecture terms, instructor names, course-specific vocab

#### Offline Captioning

Scroll up on the page, and then click "Offline captioning"

<img src="https://github.com/user-attachments/assets/0ffd50b4-5164-4d09-aaeb-912a8ed6d035" width="800" title="Snipped April, 2025" />

Click the Play button the video and observe captioning.

##### **Real-Time vs. Offline Captioning**

Offline captioning differs from real-time captioning in **timing, processing, and result stability**.

| Aspect | Real-Time Captioning | Offline Captioning |
|--------|----------------------|---------------------|
| **Timing** | Captions are generated and displayed live as the person speaks | Captions are generated after the full audio/video is available |
| **Result Type** | Partial results shown, may change mid-sentence | Final, stable transcript is used |
| **Use Case** | Live broadcasts, streams, webinars | Pre-recorded videos, films, audio before release |
| **Editing** | Limited (real-time only) | Can adjust caption formatting, timing, and content before publishing |
| **Stability** | May revise words after initial appearance | No revisions — output is final |

------------------------- -------------------------

#### Pro Code


------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------
RESUME HERE
------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------


#### Update Environment Variables

Append the following line to your `.env` file:

```text
OFFLINE_AUDIO=C:\myProject\offline_audio.wav
```

Append the following code to the "Load Environment Variables" code in the `ai_speech.ipynb` notebook:

```python
OFFLINE_AUDIO = os.getenv("OFFLINE_AUDIO")
```

Re-execute "Load Environment Variables" code and restart the kernel.

-------------------------

#### Add Demonstration Code

Click "+ Markdown" and paste the following annotation into the resulting cell:

```markdown
## Offline Captioning  
Generate final captions for pre-recorded audio content using recognized results.
```

Render the Markdown cell by clicking the checkmark in the upper-right controls.

Click "+ Code" and paste the following code into the new cell:

```python
import os
import requests

def generate_offline_captions(audio_path):
    if not audio_path or not os.path.isfile(audio_path):
        print("Audio file not found.")
        return

    with open(audio_path, "rb") as audio_file:
        audio_data = audio_file.read()

    endpoint = f"{COMPUTER_VISION_ENDPOINT.rstrip('/')}/speechtotext/v3.0/transcriptions"
    headers = {
        "Ocp-Apim-Subscription-Key": COMPUTER_VISION_API_KEY,
        "Content-Type": "audio/wav"
    }

    params = {
        "profanity": "mask",
        "display": "true",
        "language": "en-US"
    }

    response = requests.post(endpoint, headers=headers, params=params, data=audio_data)

    if response.status_code == 202:
        print("Transcription request accepted.")
        print(response.headers.get("Location"))
    else:
        print("Transcription failed:", response.status_code, response.text)

if OFFLINE_AUDIO:
    generate_offline_captions(OFFLINE_AUDIO)
else:
    print("OFFLINE_AUDIO not defined.")
```

Execute the cell and monitor the returned URL to track transcription progress and retrieve results when ready.

-------------------------

##### Expected Result  
_Note: JSON formatted and abbreviated_

```json
{
  "status": "Succeeded",
  "combinedRecognizedPhrases": [
    {
      "display": "Welcome to our pre-recorded session on Azure AI Speech.",
      "lexical": "welcome to our pre recorded session on azure ai speech",
      "itn": "welcome to our pre-recorded session on azure ai speech"
    },
    ...
  ],
  "recognizedPhrases": [ ... ]
}
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
