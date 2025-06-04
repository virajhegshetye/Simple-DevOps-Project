For your **hackathon demo**, you want to showcase the voice-based functionality of your **Azure Bot Service** quotation management bot without an **Azure Communication Services (ACS)** phone number, as you don’t have one available. The goal is to replicate the conversational flow (collecting first name, last name, monthly income, confirming input, and returning a mock API response like “accept,” “decline,” or “refer”) using **voice input** (Speech-to-Text) and **voice output** (Text-to-Speech) via a **browser** or **mobile device**, mimicking the ACS telephony experience. Since you’re using **Azure Speech Services**, **Azure Bot Service**, a **Node.js** bot, and **Ngrok** (exposed at `https://abc123.ngrok.io/api/messages` and `https://abc123.ngrok.io/api/calls`), and the text-based messaging works, you want to mock the ACS telephony component to avoid needing a phone number. The solution must use the **Azure Portal** (UI-based, no CLI), avoid certificate management, and be demo-ready for a hackathon.

Below is a concise, beginner-friendly guide in **Markdown format** to mock the ACS telephony functionality using a browser-based web app that simulates voice calls, leveraging **Azure Speech Services** for voice input/output and the existing Node.js bot with mock API responses. This approach bypasses the need for an ACS phone number while maintaining the voice-based demo experience.

---

# Mocking ACS Telephony for Voice-Based Quotation Bot Hackathon Demo

## Overview
Without an ACS phone number, you can’t use real telephony, but you can mock the voice call experience by:
- Creating a **web app** that uses **Azure Speech Services** to capture voice input (Speech-to-Text) and generate voice responses (Text-to-Speech) directly in the browser or mobile device.
- Simulating the ACS **Call Automation** events (e.g., `CallConnected`, `RecognizeCompleted`) by sending mock payloads to the bot’s `/api/calls` endpoint.
- Reusing the existing Node.js bot to handle the conversational flow, with the mock API response already implemented.
- Using **Ngrok** to expose the bot’s endpoints (`/api/messages`, `/api/calls`) for the web app to interact with.
- Ensuring a human-like voice interaction for the hackathon demo, accessible via a browser or mobile device.

This solution modifies the web app from the previous response (Step 3) to mock ACS call events, eliminating the dependency on ACS telephony while preserving the voice-based flow.

---

## Prerequisites
- **Existing Node.js Bot**:
  - The `index.js` from the previous solution (Step 2 of the latest response) is set up with the `/api/calls` endpoint for ACS Call Automation events and the mock API response:
    ```javascript
    const mockResponse = {
      data: {
        status: Math.random() < 0.5 ? 'accept' : Math.random() < 0.5 ? 'decline' : 'refer'
      }
    };
    ```
  - Required packages are installed:
    ```bash
    npm install botbuilder microsoft-cognitiveservices-speech-sdk axios dotenv @azure/communication-call-automation @azure/communication-common restify
    ```
  - The `.env` file includes:
    ```env
    MICROSOFT_APP_ID=your-bot-app-id
    MICROSOFT_APP_PASSWORD=your-bot-client-secret
    SPEECH_KEY=your-speech-service-key
    SPEECH_REGION=your-speech-service-region
    ACS_CONNECTION_STRING=endpoint=https://your-acs-resource.communication.azure.com/;accesskey=your-access-key
    QUOTE_API_URL=https://your-api.com/quote
    PORT=3978
    ```
    - Note: The `ACS_CONNECTION_STRING` is not critical for this mock demo, as we won’t use real ACS calls, but keep it for compatibility.
- **Ngrok**: Running to expose the bot (`ngrok http 3978`), providing an HTTPS URL (e.g., `https://abc123.ngrok.io`).
- **Azure Resources**:
  - **Azure Bot Service** (`QuotationBot`) with **Microsoft App ID** and **Client Secret**.
  - **Azure Speech Service** (`QuotationSpeechService`) with **Key 1** and **Region**.
- **Browser/Mobile Device**: Chrome or Edge with microphone access for voice input/output.
- **Web Server**: `http-server` installed (`npm install -g http-server`) to serve the web app.
- **Hackathon Demo**: Focus on a simple, voice-based interaction with mock responses.

---

## Solution Approach
To mock ACS telephony without a phone number:
1. **Reuse Node.js Bot**: Keep the existing `/api/calls` endpoint to handle mock Call Automation events.
2. **Modify Web App**: Update the browser/mobile web app to:
   - Use Azure Speech Services SDK for voice input (Speech-to-Text) and output (Text-to-Speech).
   - Simulate ACS call events by sending HTTP POST requests to `https://abc123.ngrok.io/api/calls` with mock payloads.
3. **Mock Call Automation Events**:
   - Simulate `CallConnected` to start the conversation.
   - Use Speech-to-Text to capture user voice input and send mock `RecognizeCompleted` events to the bot.
4. **Demo Setup**: Serve the web app locally, expose it via Ngrok, and test the voice-based flow.
5. **No Certificates**: Use Ngrok’s HTTPS URLs for all endpoints.

---

## Step-by-Step Guide

### Step 1: Verify Node.js Bot
1. **Check `index.js`**:
   - Ensure the bot code (from Step 2 of the latest response) includes the `/api/calls` endpoint to handle `CallConnected` and `RecognizeCompleted` events:
     ```javascript
     server.post('/api/calls', async (req, res) => {
       const callData = req.body;
       const callConnectionId = callData.data.callConnectionId;
       const callConnection = acsClient.getCallConnection(callConnectionId);
       const state = await userProfile.get({ conversation: { id: callConnectionId } }, { step: 'start', data: {} });

       if (callData.event === 'CallConnected') {
         await synthesizeSpeech('Hello! Please tell me your first name.', callConnectionId);
         state.step = 'firstName';
       } else if (callData.event === 'RecognizeCompleted') {
         const recognizedText = callData.data.recognizedText;
         // Handle firstName, lastName, income, confirm
       }
       // ...
       res.status(200).send();
     });
     ```
   - The mock API response is already included:
     ```javascript
     const mockResponse = {
       data: {
         status: Math.random() < 0.5 ? 'accept' : Math.random() < 0.5 ? 'decline' : 'refer'
       }
     };
     ```

2. **Run the Bot**:
   - In the `QuotationBot` folder, run:
     ```bash
     node index.js
     ```
   - Confirm:
     ```
     Server running at http://localhost:3978
     ```

3. **Run Ngrok**:
   - In a separate terminal, run:
     ```bash
     ngrok http 3978
     ```
   - Copy the HTTPS URL (e.g., `https://abc123.ngrok.io`).

4. **Update Azure Bot Service**:
   - In the [Azure Portal](https://portal.azure.com), go to **Home > All resources > QuotationBot**.
   - Under **Settings**, set the **Messaging endpoint** to:
     ```
     https://abc123.ngrok.io/api/messages
     ```
   - Click **Apply**.

---

### Step 2: Create a Mock Web App for Voice Interaction
Replace the previous web app (`index.html`) with one that mocks ACS telephony using Azure Speech Services for voice input/output and sends mock Call Automation events to `/api/calls`.

1. **Create `index.html`**:
   - In a new folder (e.g., `webapp`), create `index.html`:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
       <title>Quotation Bot Voice Demo</title>
       <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
       <script src="https://cdn.jsdelivr.net/npm/microsoft-cognitiveservices-speech-sdk@1.30.0/distrib/lib/microsoft.cognitiveservices.speech.sdk.min.js"></script>
       <style>
         body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
         button { padding: 10px 20px; margin: 10px; font-size: 16px; }
         #status { color: #333; font-size: 18px; }
       </style>
     </head>
     <body>
       <h1>Quotation Bot Voice Demo - Hackathon 2025</h1>
       <button id="startCall">Start Voice Interaction</button>
       <button id="endCall" disabled>End Interaction</button>
       <p id="status">Ready to start voice interaction</p>
       <script>
         const speechKey = 'your-speech-service-key';
         const speechRegion = 'your-speech-service-region';
         const botCallEndpoint = 'https://abc123.ngrok.io/api/calls';
         const callConnectionId = 'mock-call-' + Math.random().toString(36).substring(2, 15);

         let recognizer, synthesizer;

         async function startInteraction() {
           document.getElementById('status').innerText = 'Starting voice interaction...';
           document.getElementById('startCall').disabled = true;
           document.getElementById('endCall').disabled = false;

           // Initialize Speech SDK
           const speechConfig = SpeechSDK.SpeechConfig.fromSubscription(speechKey, speechRegion);
           const audioConfig = SpeechSDK.AudioConfig.fromDefaultMicrophoneInput();
           recognizer = new SpeechSDK.SpeechRecognizer(speechConfig, audioConfig);
           synthesizer = new SpeechSDK.SpeechSynthesizer(speechConfig);

           // Simulate CallConnected event
           await axios.post(botCallEndpoint, {
             event: 'CallConnected',
             data: { callConnectionId }
           });

           // Start listening for user speech
           listenForSpeech();
         }

         async function listenForSpeech() {
           document.getElementById('status').innerText = 'Listening for your response...';
           recognizer.recognizeOnceAsync(result => {
             if (result.reason === SpeechSDK.ResultReason.RecognizedSpeech) {
               const recognizedText = result.text;
               // Simulate RecognizeCompleted event
               axios.post(botCallEndpoint, {
                 event: 'RecognizeCompleted',
                 data: { callConnectionId, recognizedText }
               }).then(response => {
                 // Continue listening for next input
                 listenForSpeech();
               });
             } else {
               document.getElementById('status').innerText = 'Speech recognition failed. Try again.';
               listenForSpeech();
             }
           });
         }

         async function endInteraction() {
           document.getElementById('status').innerText = 'Interaction ended.';
           document.getElementById('startCall').disabled = false;
           document.getElementById('endCall').disabled = true;
           if (recognizer) recognizer.close();
           if (synthesizer) synthesizer.close();
         }

         // Mock synthesizeSpeech function for browser
         window.synthesizeSpeech = async (text) => {
           document.getElementById('status').innerText = 'Bot speaking: ' + text;
           await synthesizer.speakTextAsync(text);
         };

         document.getElementById('startCall').addEventListener('click', startInteraction);
         document.getElementById('endCall').addEventListener('click', endInteraction);
       </script>
     </body>
     </html>
     ```

2. **Update Placeholders**:
   - Replace `your-speech-service-key` with the Speech Service Key 1 (Step 1.3 of the original solution).
   - Replace `your-speech-service-region` with the Speech Service region (e.g., `eastus`).
   - Replace `https://abc123.ngrok.io` with your Ngrok URL for the bot.

3. **Key Features**:
   - Uses the **Azure Speech SDK** for browser-based Speech-to-Text (user voice input) and Text-to-Speech (bot responses).
   - Sends mock `CallConnected` and `RecognizeCompleted` events to `https://abc123.ngrok.io/api/calls` using Axios.
   - Simulates a unique `callConnectionId` for each session.
   - Displays status updates for demo clarity (e.g., “Listening for your response...”).

---

### Step 3: Serve and Expose the Web App
1. **Install HTTP Server** (if not already done):
   ```bash
   npm install -g http-server
   ```

2. **Serve the Web App**:
   - Navigate to the `webapp` folder:
     ```bash
     cd webapp
     ```
   - Run:
     ```bash
     http-server -p 8080
     ```
   - Access locally at `http://localhost:8080`.

3. **Expose with Ngrok**:
   - In a new terminal, run a separate Ngrok instance:
     ```bash
     ngrok http 8080
     ```
   - Copy the HTTPS URL (e.g., `https://def456.ngrok.io`).

---

### Step 4: Test the Mock Voice Demo
1. **Access the Web App**:
   - Open `https://def456.ngrok.io` in a browser (Chrome/Edge) or mobile browser.
   - Grant microphone access when prompted.

2. **Start Voice Interaction**:
   - Click **Start Voice Interaction**.
   - The web app sends a mock `CallConnected` event, triggering the bot to respond with “Hello! Please tell me your first name.” (played via Text-to-Speech).
   - Speak responses (e.g., “John”, “Doe”, “5000”, “yes”) into the microphone.
   - The web app:
     - Uses Speech-to-Text to capture your input.
     - Sends mock `RecognizeCompleted` events to `/api/calls`.
     - Plays bot responses via Text-to-Speech.
   - Verify the mock API response (e.g., “Your application has been accepted for a card. Thank you!”).
   - Click **End Interaction** to stop.

3. **Check Ngrok Logs**:
   - Open Ngrok’s web interface (`http://localhost:4040` for `ngrok http 3978`).
   - Confirm POST requests to `/api/calls` with `CallConnected` and `RecognizeCompleted` events.

4. **Test on Mobile**:
   - Access `https://def456.ngrok.io` on a mobile browser.
   - Follow the same steps, ensuring microphone and speaker access.
   - Verify the conversational flow.

---

### Step 5: Polish for Hackathon Demo
1. **Enhance Web App UI**:
   - The `index.html` includes basic CSS for a clean look. Add a hackathon logo or branding:
     ```html
     <img src="https://your-logo-url.com/logo.png" alt="Hackathon Logo" style="width: 100px; margin-bottom: 20px;">
     ```
   - Update the title:
     ```html
     <h1>Voice Quotation Bot - Hackathon 2025</h1>
     ```

2. **Demo Script**:
   - **Setup**: Show the web app on a laptop (Chrome) and a mobile device.
   - **Steps**:
     - Open `https://def456.ngrok.io`.
     - Click **Start Voice Interaction**.
     - Speak: “John”, “Doe”, “5000”, “yes”.
     - Highlight the voice response (e.g., “Your application has been accepted...”).
     - Click **End Interaction**.
   - **Talking Points**:
     - Voice-based interaction using Azure Speech Services.
     - Mocked telephony without an ACS phone number.
     - Browser/mobile accessibility.
   - **Duration**: Keep it 1-2 minutes.

3. **Backup Plan**:
   - Record a video of the demo in case of connectivity issues.
   - Save Ngrok URLs and test before the presentation.

4. **Monitor Azure Usage**:
   - Check **QuotationSpeechService > Monitoring > Metrics** in the Azure Portal to avoid exceeding free-tier limits.

---

### Step 6: Troubleshooting
- **No Voice Input/Output**:
  - Verify `speechKey` and `speechRegion` in `index.html`.
  - Ensure microphone permissions are granted in the browser.
  - Test with Chrome/Edge for best Speech SDK compatibility.
- **No Bot Response**:
  - Check Ngrok logs (`http://localhost:4040`) for POST requests to `/api/calls`.
  - Ensure the bot is running (`node index.js`) and Ngrok is active (`ngrok http 3978`).
- **Speech Recognition Errors**:
  - Confirm the Speech Service is active in the Azure Portal.
  - Test in a quiet environment for better recognition.
- **Ngrok Issues**:
  - Restart Ngrok if the tunnel expires (`ngrok http 3978` for bot, `ngrok http 8080` for web app).
  - Use a free Ngrok account for stable URLs.

---

## Additional Notes
- **No ACS Phone Number**:
  - The mock solution bypasses ACS telephony, using mock Call Automation events (`CallConnected`, `RecognizeCompleted`) sent via HTTP POST.
  - The web app simulates the voice call experience entirely in the browser.
- **No Certificates**:
  - Ngrok’s HTTPS URLs (`https://abc123.ngrok.io`, `https://def456.site`) include wildcard certificates, meeting the bot’s HTTPS requirement.
- **Hackathon Appeal**:
  - Emphasize the innovative voice interaction and accessibility (browser/mobile).
  - The mock API ensures consistent demo outcomes (“accept,” “decline,” “refer”).
- **Limitations**:
  - Without real telephony, you can’t demo actual phone calls. Focus on the web app’s voice experience.
  - The Speech SDK may have slight differences in recognition quality compared to ACS telephony.

---

## Resources
- [Azure Speech SDK](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/speech-sdk)
- [Azure Communication Services Call Automation (Reference)](https://learn.microsoft.com/en-us/azure/communication-services/concepts/call-automation/call-automation)
- [Ngrok Documentation](https://ngrok.com/docs)
- [Axios Documentation](https://axios-http.com/docs/intro)
- [Azure Portal Help](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

---

## Summary
To demo the voice-based quotation bot without an ACS phone number:
1. Verify the Node.js bot (`index.js`) with the `/api/calls` endpoint and mock API response.
2. Create a web app (`index.html`) using the Azure Speech SDK to capture voice input/output and send mock Call Automation events to `https://abc123.ngrok.io/api/calls`.
3. Serve the web app (`http-server -p 8080`) and expose it with Ngrok (`ngrok http 8080`).
4. Test the voice flow in a browser or mobile device, speaking responses (“John”, “Doe”, “5000”, “yes”) and receiving voice output.
5. Polish the UI and script for a 1-2 minute hackathon demo.

This solution delivers a compelling voice-based demo without telephony, using mocks to simulate ACS Call Automation. If you need further tweaks or face issues, let me know, and I’ll help ensure your hackathon demo shines!