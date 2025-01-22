# Lab: Translator

<img src="https://github.com/user-attachments/assets/96481bac-c772-4dae-98c3-139bfd04f9db" width="1000" />

## Solution Requirements

This documentation assumes the following resources are ready for use:

* [Translator](https://learn.microsoft.com/en-us/azure/ai-services/translator/create-translator-resource)

## Documentation Note

I used to spend a lot of time explaining code blocks, but now skip that part of my sharing process in favor of a recommendation to copy code blocks to Copilot along with prompts like "what does this do?" or "how does this work" for amazing description and detail!

---

### **Exercise 1: Text Translation**
**Objective**: Use Azure Translator to translate text between languages and explore language detection using the Azure Portal.

#### **Steps**
1. **Access the Translator Resource**
   - Navigate to the Azure Portal and open the Translator resource already assigned to your account.
   - <img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/56315a73-0864-4319-a568-63bc84a01b40" width="800" title="Snipped January 21, 2025" />


2. **Use the Portal Interface for Text Translation**
   - Go to the **"Try It"** section available within the Translator resource.
   - Input the text you want to translate, for example:  
     *"Hello, how are you?"*
   - Select a **target language** (e.g., French: `fr`) from the drop-down menu.
   - Click **Translate** and observe the output, which should display:  
     *"Bonjour, comment ça va ?"*

3. **Test Language Detection**
   - In the same **"Try It"** interface, input text in any language (e.g., Spanish: *"¿Cómo estás?"*).
   - Enable the **Detect Language** option.
   - Observe the detected language and its ISO language code (e.g., `es` for Spanish).

4. **Experiment with Multiple Target Languages**
   - Modify the **target language** to include multiple options, such as French (`fr`), Spanish (`es`), and German (`de`).
   - Observe translations in all selected languages.

#### **Expected Output**
- Input: *"Hello, how are you?"*
- Output:
  - French: *"Bonjour, comment ça va ?"*
  - Spanish: *"Hola, ¿cómo estás?"*
  - German: *"Hallo, wie geht es dir?"*

#### **Bonus Activity**
- Try translating longer or more complex sentences to see how the Translator handles idiomatic expressions.
- Compare translations across different languages for the same text.

---

### **Exercise 2: Document Translation**
**Objective**: Use the Azure Portal and Document Translation API to translate files (e.g., Word, PDF) in batches to specified target languages.

#### **Steps**
1. **Access the Translator Resource**
   - Navigate to the Azure Portal and open the Translator resource already assigned to your account.

2. **Set Up Storage Containers**
   - Go to the Azure Portal and create two blob storage containers:
     - **Source Container**: Upload the documents you want to translate (e.g., `.docx`, `.pdf`).
     - **Target Container**: Ensure this is empty; it will store translated files.
   - Grant appropriate permissions to the Translator resource for both containers using **RBAC**.

3. **Perform Translation Using the Azure Portal**
   - In the Translator resource, go to the **Document Translation** feature.
   - Configure the translation task:
     - **Source Container**: Provide the URL of the source blob container.
     - **Target Container**: Provide the URL of the target blob container.
     - **Target Language(s)**: Select one or more languages (e.g., French `fr`, Spanish `es`).
   - Start the translation process.

4. **Monitor Translation Progress**
   - Check the **status** of the translation task in the Azure Portal.
   - Once completed, navigate to the target blob container to view the translated documents.

#### **Expected Output**
- Source Document: `hello.docx` containing the text *"Hello, how are you?"*
- Translated Document: A file named `hello.fr.docx` in the target container with the content *"Bonjour, comment ça va ?"*

#### **Bonus Activity**
- Experiment with translating documents into multiple languages simultaneously by specifying more than one target language.
- Test with large documents or batches to explore the performance and scalability of the service.

---

Here is the **Exercise 3: Speech Translation** content:

---

### **Exercise 3: Speech Translation**
**Objective**: Configure Azure Translator to perform real-time speech-to-text translation and explore integration workflows.

#### **Steps**
1. **Access the Speech Resource**
   - Navigate to the Azure Portal and open the Speech resource assigned to your account.

2. **Test Speech Translation in the Portal**
   - Use the **Speech Studio** available within the Speech resource.
   - Go to the **Real-Time Speech Translation** feature.
   - Configure the input settings:
     - **Language of Speech**: Select the source language (e.g., English: `en-US`).
     - **Target Language(s)**: Choose one or more target languages (e.g., French: `fr`, Spanish: `es`).
   - Speak into your device's microphone or upload a pre-recorded audio file.
   - View the live transcription and translations in real-time.

3. **Optional: Use Python for Speech Translation**
   - Install the required package:
     ```bash
     pip install azure-cognitiveservices-speech
     ```
   - Use the following Python script to perform speech translation:
     ```python
     import azure.cognitiveservices.speech as speechsdk

     # Configuration
     speech_key = "<your-key>"
     service_region = "<your-region>"
     speech_config = speechsdk.translation.SpeechTranslationConfig(
         subscription=speech_key, region=service_region
     )
     speech_config.speech_recognition_language = "en-US"
     speech_config.add_target_language("fr")

     # Translation recognizer
     recognizer = speechsdk.translation.TranslationRecognizer(speech_config)

     print("Speak into your microphone...")
     result = recognizer.recognize_once()
     print("Recognized Text:", result.text)
     print("Translation (French):", result.translations["fr"])
     ```

4. **Verify and Test**
   - If using the portal, confirm the translations displayed for the spoken input.
   - If using Python, verify that the recognized text and translations match the expected output.

#### **Expected Output**
- Spoken Input: *"Hello, how are you?"*
- Recognized Text: *"Hello, how are you?"*
- Translations:
  - French: *"Bonjour, comment ça va ?"*
  - Spanish: *"Hola, ¿cómo estás?"*

#### **Bonus Activity**
- Test with different source languages (e.g., Spanish: `es-ES`) and target languages.
- Experiment with noisy environments or varying accents to observe system performance.

---

### **Exercise 4: Custom Translator**
**Objective**: Build and integrate a custom translation model by creating a glossary, uploading it to Azure, and testing its integration.

#### **Steps**
1. **Access Custom Translator in Azure**
   - Open the Azure Portal and navigate to your Translator resource.
   - Locate and access the **Custom Translator** feature via the Azure AI services.

2. **Create a Custom Translator Project**
   - In the Custom Translator interface:
     - Click **New Project** and configure the project with:
       - **Name**: Unique name for the project
       - **Source Language**: The language of your source text (e.g., English: `en`)
       - **Target Language**: The target translation language (e.g., French: `fr`)
     - Save the project.

3. **Upload Glossary**
   - Prepare a glossary file in `.tsv` format with your custom terms. Example:
     ```
     hello    bonjour
     world    monde
     ```
   - In the project interface, upload the glossary under the **Glossaries** section.
   - Validate and publish the glossary to integrate it with the translation system.

4. **Test the Custom Translator in Azure Portal**
   - Go to the **Test** section within the Custom Translator interface.
   - Input a sentence containing glossary terms (e.g., "Hello world").
   - Select the custom project and target language (e.g., `fr`).
   - Verify that the translation aligns with the glossary (e.g., *"Bonjour monde"*).

5. **Optional: Use Python for Custom Translator**
   - Install the required package:
     ```bash
     pip install requests
     ```
   - Sample Python script to use the custom glossary:
     ```python
     import requests
     import json

     endpoint = "https://<your-resource-name>.cognitiveservices.azure.com/translator/text/v3.0/translate"
     api_version = "3.0"
     headers = {
         "Content-Type": "application/json",
         "Ocp-Apim-Subscription-Key": "<your-key>",
         "Ocp-Apim-Subscription-Region": "<your-region>"
     }

     data = [{"Text": "Hello world"}]
     params = {
         "api-version": api_version,
         "to": "fr",
         "project-name": "<your-custom-project-name>"
     }

     response = requests.post(f"{endpoint}", headers=headers, params=params, json=data)
     print(response.json())
     ```

6. **Verify Translations**
   - Compare translations using the custom glossary versus the default translation model.

#### **Expected Output**
- Input: *"Hello world"*
- Default Translation: *"Bonjour le monde"*
- Custom Translation: *"Bonjour monde"*

#### **Bonus Activity**
- Expand the glossary to include industry-specific terms or phrases.
- Test the custom model with complex sentences to evaluate its effectiveness.

---

### **Exercise 5: Error Handling and Optimization**
**Objective**: Identify common API issues and implement error-handling strategies while exploring Azure Monitor to track and optimize performance.

#### **Steps**

1. **Test Error Scenarios Using Azure Portal**
   - In the Translator resource, deliberately input invalid or incomplete configurations:
     - Use unsupported language codes (e.g., `xx`).
     - Leave required fields empty, such as source text or target language.
   - Observe the error messages returned by the portal.

2. **Monitor API Usage and Errors in Azure Monitor**
   - Navigate to the **Azure Monitor** in the Azure Portal.
   - Set up **Application Insights** for the Translator resource if not already enabled.
   - In the Monitor:
     - View the **Metrics** to analyze API response times and usage patterns.
     - Check **Logs** to review error codes, such as `400 Bad Request` or `429 Too Many Requests`.
   - Create custom alerts for specific error thresholds (e.g., high failure rates or latency).

3. **Implement Error Handling in Python**
   - Modify your Python code to include robust error handling:
     ```python
     import requests
     import json

     endpoint = "https://<your-resource-name>.cognitiveservices.azure.com/translator/text/v3.0/translate"
     headers = {
         "Content-Type": "application/json",
         "Ocp-Apim-Subscription-Key": "<your-key>",
         "Ocp-Apim-Subscription-Region": "<your-region>"
     }

     data = [{"Text": "Hello"}]
     params = {"api-version": "3.0", "to": "fr"}

     try:
         response = requests.post(endpoint, headers=headers, params=params, json=data)
         response.raise_for_status()  # Raise an exception for HTTP errors
         translation = response.json()
         print("Translation:", translation)
     except requests.exceptions.HTTPError as e:
         print(f"HTTP error: {e.response.status_code} - {e.response.text}")
     except requests.exceptions.RequestException as e:
         print(f"Request error: {e}")
     ```

4. **Optimize Translation Performance**
   - Adjust API requests to improve efficiency:
     - Batch multiple translation texts into a single request.
     - Limit the frequency of requests to avoid throttling (HTTP 429 errors).
   - In the Azure Portal:
     - Review **API Limits** in the Translator resource and adjust workload distribution as needed.
     - Scale the Translator resource (if using a paid tier) to handle higher workloads.

#### **Expected Output**
- For invalid inputs:
  - **Error Example**: `HTTP 400 - Bad Request: Invalid language code`
- For throttling:
  - **Error Example**: `HTTP 429 - Too Many Requests`
- Successful retry logic should handle temporary issues gracefully.

#### **Bonus Activity**
- Use retry logic to automatically resend requests after a delay if they fail with transient errors (e.g., 429 or 503).
- Create dashboards in Azure Monitor to visualize resource usage trends and API performance.
