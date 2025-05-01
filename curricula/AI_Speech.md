# Lab: AI Speech

<img src="..\images\AI_Speech\AI_Speech_Header.png" width="1000" />

## Introduction

Azure AI Speech is a cloud-based service from Microsoft that leverages advanced algorithms to process and understand spoken language, extracting valuable insights and enabling dynamic communication solutions. It encompasses a range of capabilities—from real-time speech-to-text transcription and speaker recognition to lifelike text-to-speech synthesis and multilingual translation—all designed to solve real-world problems quickly and effectively.

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 1: Prepare Resources  

### Azure

- **Speech Services**: Pricing Tier `Standard S0`

### On-Prem

> Additional content (e.g., PowerShell, Visual Studio Code, etc.) pending work on **Low Code** sections

<!-- ------------------------- ------------------------- -->

## Exercise 2: Speech Capabilities by Scenario

### Captioning with Speech to Text

#### Low Code

Navigate to [Azure AI Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src=".\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click "Captioning with speech to text".

<img src=".\images\AI_Speech\SpeechStudio_TryItOut_Captioning.png" width="800" title="Snipped April, 2025" />

On the "Try it out" >> "Sample videos" tabs, click "Real-time captioning", and then scroll down on the page.

<img src=".\images\AI_Speech\SpeechStudio_RealTimeCaptioning.png" width="800" title="Snipped April, 2025" />

Click the Play button the video and observe captioning.

##### Captioning Settings

The captioning settings shown on Speech Studio > Captioning > **Real-time captioning with "stable partial results"** are detailed on the right side of the interface and include the following parameters:

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
> For high-energy, fast-paced content where low latency matters (e.g., sports event).
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

<img src=".\images\AI_Speech\SpeechStudio_OfflineCaptioning.png" width="800" title="Snipped April, 2025" />

Click the Play button and observe captioning.

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

LOREM IPSUM

------------------------- -------------------------

### Post Call Transcription and Analytics

#### Low Code

Navigate to [Azure AI Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src=".\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click "Post Call Transcription and Analytics".

<img src=".\images\AI_Speech\SpeechStudio_TryItOut_Transcription.png" width="800" title="Snipped April, 2025" />

On the "Try it out" >> "Try with samples" tabs, click "Apply for a loan", and then scroll down on the page. Default selection is the "Analyze sentences" tab.

<img src=".\images\AI_Speech\SpeechStudio_AnalyzeSentences.png" width="800" title="Snipped April, 2025" />

Review sample content:

- **Audio Player**: A simple media bar with play/pause controls and a visible duration (e.g., 03:02s)
- **Transcript Content**: Simulated dialogue from a customer support call demonstrates live transcription, redaction, and sentiment tagging.
  - **Sentiment**: Each transcript line is tagged on the left with an evaluation of sentiment (e.g., Positive, Neutral, etc.)  
  - **Speakers**: Transcript lines are grouped and labeled by speaker (e.g., Speaker1, Speaker2) with timestamps
  - **PII Masking**: Sensitive information (e.g., names) is redacted and replaced with asterisks (e.g., ****), with labels like "Name" underneath 
- **Hide PII Toggle**: A switch in the upper right of the transcript section allows turning PII masking on or off
- **Call Summary Tab**: A second tab, "Call summary", is visible but not selected, likely for higher-level insights

Try other Scenario Cards (e.g., "Signing up for Insurance").

#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Live Chat Avatar
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Language Learning
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Video Translation
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------
------------------------- -------------------------

## Exercise 3: Speech to Text

### Real-Time Speech to Text
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Whisper Model in Azure OpenAI Service
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Batch Speech to Text
#### Low Code
#### Pro Code

LOREM IPSUM
------------------------- -------------------------

### Custom Speech
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Pronunciation Assessment with Speech to Text
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Speech Translation
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------
------------------------- -------------------------

## Exercise 4: Text to Speech

### Voice Gallery
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Custom Voice
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Personal Voice
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Audio Content Creation
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Text to Speech Avatar
#### Low Code
#### Pro Code

LOREM IPSUM

------------------------- -------------------------
------------------------- -------------------------

## Exercise 5: Voice Assistant

### Custom Keyword
#### Low Code
#### Pro Code

LOREM IPSUM