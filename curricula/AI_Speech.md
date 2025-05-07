# Lab: AI Speech

<img src="..\images\AI_Speech\AI_Speech_Header.png" width="1000" />

## Introduction

Azure AI Speech is a cloud-based service from Microsoft that leverages advanced algorithms to process and understand spoken language, extracting valuable insights and enabling dynamic communication solutions. It encompasses a range of capabilities—from real-time speech-to-text transcription and speaker recognition to lifelike text-to-speech synthesis and multilingual translation—all designed to solve real-world problems quickly and effectively.

<!-- ------------------------- ------------------------- -->

## Exercise 1: Prepare Resources  

### Azure

- **Speech Services**: 
  - Pricing Tier `Standard S0`, 
  - Supported Region (**West US 2**, West Europe, Southeast Asia, South Central US, Sweden Central, North Europe, East US 2)

### On-Prem

> Additional content (e.g., PowerShell, Visual Studio Code, etc.) pending work on **Low Code** sections

<!-- ------------------------- ------------------------- -->

## Exercise 2: Speech Capabilities by Scenario

### Captioning with Speech to Text

<!-- ------------------------- ------------------------- -->

#### Low Code

Navigate to [Azure AI Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src="..\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click **Captioning with speech to text**.

<img src="..\images\AI_Speech\SpeechStudio_Captioning_TryItOut.png" width="800" title="Snipped April, 2025" />

On the **Try it out** >> **Sample videos** tabs, click **Real-time captioning**, and then scroll down on the page.

<img src="..\images\AI_Speech\SpeechStudio_Captioning_RealTime.png" width="800" title="Snipped April, 2025" />

Click the Play button the video and observe captioning.

<!-- ------------------------- -->

##### Captioning Settings

...detailed on the right side of the interface and include the following parameters:

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

<!-- ------------------------- -->

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

<!-- ------------------------- ------------------------- -->

#### Offline Captioning

Scroll up on the page, and then click **Offline captioning**

<img src="..\images\AI_Speech\SpeechStudio_Captioning_Offline.png" width="800" title="Snipped April, 2025" />

Click the Play button and observe captioning.

##### **Real-Time vs. Offline Captioning**

Offline captioning differs from real-time captioning in **timing, processing, and result stability**.

| Aspect | Real-Time Captioning | Offline Captioning |
| :--- | :--- | :--- |
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

<img src="..\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click **Post Call Transcription and Analytics**.

<img src="..\images\AI_Speech\SpeechStudio_Transcription_TryItOut.png" width="800" title="Snipped April, 2025" />

On the **Try it out** >> **Try with samples** tabs, click **Apply for a loan**, and then scroll down on the page. Default selection is the **Analyze sentences** tab.

<img src="..\images\AI_Speech\SpeechStudio_Transcription_AnalyzeSentences.png" width="800" title="Snipped April, 2025" />

Review sample content:

- **Audio Player**: A simple media bar with play/pause controls and a visible duration (e.g., 03:02s)
- **Transcript Content**: Simulated dialogue from a customer support call demonstrates live transcription, redaction, and sentiment tagging.
  - **Sentiment**: Each transcript line is tagged on the left with an evaluation of sentiment (e.g., Positive, Neutral, etc.)  
  - **Speakers**: Transcript lines are grouped and labeled by speaker (e.g., Speaker1, Speaker2) with timestamps
  - **PII Masking**: Sensitive information (e.g., names) is redacted and replaced with asterisks (e.g., ****), with labels like **Name** underneath 
- **Hide PII Toggle**: A switch in the upper right of the transcript section allows turning PII masking on or off
- **Call Summary Tab**: A second tab, **Call summary**, is visible but not selected, likely for higher-level insights

Try other Scenario Cards (e.g., **Signing up for Insurance**).

#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Live Chat Avatar

<!-- ------------------------- ------------------------- -->

#### Low Code

Navigate to [Azure AI Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src="..\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click **Live Chat Avatar**.

<img src="..\images\AI_Speech\SpeechStudio_Avatar.png" width="800" title="Snipped May, 2025" />

<!-- ------------------------- -->

##### Try with sample

On the **Try with sample** tabs, click the Play button the video and observe the live chat sample.

<img src="..\images\AI_Speech\SpeechStudio_Avatar_TryWithSample.png" width="800" title="Snipped May, 2025" />

###### Settings

...detailed on the right side of the interface and include the following parameters:

- **Language**: `English (United States)`  
  Specifies the language used for speech synthesis and understanding during the interaction.

- **Voice**: `Sara`  
  Selects the synthetic voice used by the avatar for spoken responses. "Sara" is one of the standard voice personas available.

- **Speaking style**: `Friendly`  
  Modifies the tone and inflection of the voice to sound warm and approachable, appropriate for customer-facing scenarios.

- **Conversation style**: `"You are a voice assistant, and when you answer questions, your response should not exceed 25 words."`  
  Provides prompt-level control over the avatar’s response behavior—limiting verbosity and guiding tone to simulate a concise support agent.

- **Avatar**: `Lisa-casual-sitting`  
  Specifies the visual representation of the avatar, in this case a casual seated female persona used for customer support roles.

- **Sample response**:  
  `"Yes, Microsoft offers special discounts and pricing for students and educators on the Surface Go and other devices through their Education Store."`  
  Demonstrates how the avatar responds to a question using the configured voice, style, and character.

<!-- ------------------------- -->

##### Try on your own

On the **Try on your own** tab, try different questions, settings, voices, and avatars.

<img src="..\images\AI_Speech\SpeechStudio_Avatar_TryOnYourOwn.png" width="800" title="Snipped May, 2025" />

###### Settings

...detailed on the right side of the interface and include the following parameters:

- **Language**: `English (United States)`  
  Sets the primary language for both input and synthesized speech. This determines pronunciation, transcription, and voice selection defaults.

- **Multi-language**: Toggle is set to `Off`  
  When enabled, this allows the avatar to automatically detect and respond in multiple languages during conversation.

- **Voice**: `Ava Multilingual`  
  Specifies the synthetic voice used by the avatar. “Ava Multilingual” supports dynamic language shifts when paired with multi-language input.

- **Speaking style**: `Default`  
  Sets the vocal tone and delivery style; can be adjusted to styles like cheerful, angry, or empathetic where supported.

- **Model**: `GPT-4o` (grayed out but labeled)  
  Indicates that the GPT-4o model is selected for language generation, though selection may be locked to this choice.

- **Response style**: `Default`  
  Governs the formatting and verbosity of the generated replies. Useful for tuning tone to formal, concise, or creative.

- **Prompt / System Message**: `You are an intelligent assistant.`  
  Provides the system prompt for controlling assistant behavior, tone, and personality.

- **Avatar**: Three male avatars shown (e.g., business casual, bright red sweater, blue dress shirt)  
  Allows the user to visually customize the avatar’s appearance to match different use cases or brand identities.

- **Mic Interaction**: "Press the mic to start talking"  
  Enables live voice input, letting the user speak naturally with the avatar using their microphone.

<!-- ------------------------- ------------------------- -->

#### Pro Code

LOREM IPSUM

------------------------- -------------------------

### Language Learning

#### Low Code

Navigate to [Azure AI Speech Studio](https://speech.microsoft.com/portal), and log in with your Azure credentials.

<img src="..\images\AI_Speech\SpeechStudio_GetStarted.png" width="800" title="Snipped April, 2025" />

Click **Language Learning**.

<img src="..\images\AI_Speech\SpeechStudio_Language.png" width="800" title="Snipped May, 2025" />

<!-- ------------------------- -->

##### Try it out

<img src="..\images\AI_Speech\SpeechStudio_Language_TryItOut.png" width="800" title="Snipped May, 2025" />

On the **Try it out** tab, chat with your language teacher, then click **Assess my response**, and then review the produced report:

* **Pronunciation Score**: `78`
  <br>Shown in a circular gauge with a legend color-coded for ranges 0 to 59 (red), 60 to 79 (yellow), and 80 to 100 (green).
* **Pronunciation Breakdown**: `Accuracy Score 79 / 100`, `Fluency Score 81 / 100`, `Prosody Score 77 / 100`
  <br>Progress bars detail the specific metrics contributing to the overall pronunciation score.
* **Content Score**: `10`
  <br>Displayed in a circular gauge using the same three-range color legend to indicate content performance.
* **Content Breakdown**: `Grammar Score 10 / 100`, `Vocabulary Score 10 / 100`
  <br>Progress bars highlight low performance in grammar and vocabulary aspects.
* **Feedback Button**: `Get feedback on how to improve`
  <br>Provides targeted suggestions and guidance to improve both pronunciation and content scores.

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