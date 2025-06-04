For your **hackathon demo**, you want to extend the functionality of the **Azure Bot Service** quotation management bot to allow users to interact via voice calls (using **Azure Communication Services (ACS)** for telephony) and perform the same conversational flow as the existing text-based messaging system (via `/api/messages`). The bot should collect the user’s **first name**, **last name**, **monthly income**, confirm the input, call the quotation management API (or use a mock response), and respond with **voice output** instead of text messages. You also want to access this functionality from a **browser** or **mobile device** using a microphone for input and receiving voice responses, ensuring a human-like interaction with **Azure Speech Services**. The solution must avoid certificate management (as per your previous requirement) and use the **Azure Portal** (UI-based, no CLI). Since you’re preparing for a hackathon, I’ll provide a concise, beginner-friendly guide in **Markdown format**, focusing on enabling voice calls and browser/mobile access, leveraging the existing Node.js bot and Ngrok setup.

---

# Voice-Based Quotation Management Bot Demo for Hackathon

## Overview
Your existing bot, built with **Azure Bot Service**, **Azure Speech Services**, **ACS**, and the **Alexa channel**, handles text-based interactions via the `/api/messages` endpoint, exposed locally with Ngrok (e.g., `https://abc123.ngrok.io/api/messages`). The bot collects user details (first name, last name, monthly income), confirms input, and returns a mock API response (“accept,” “decline,” or “refer”). For the hackathon demo, you want to:
- Enable **voice call interactions** using ACS telephony, replicating the text-based flow.
- Allow users to access the bot from a **browser** or **mobile device** using a microphone for **voice input** (Speech-to-Text) and receive **voice output** (Text-to-Speech) instead of text messages.
- Use the existing Node.js code and Ngrok setup, ensuring no certificate management.
- Keep the solution simple and demo-ready, focusing on voice-based interaction.

This guide modifies the existing solution to support voice calls via ACS and provides a browser/mobile interface using the **Azure Communication Services JavaScript SDK** for voice input/output, integrated with **Azure Speech Services** for a seamless experience.

---

## Prerequisites
- **Existing Setup**: The Node.js bot from the previous solution (Step 4.2) is running locally with Ngrok, exposing `https://abc123.ngrok.io/api/messages` and `https://abc123.ngrok.io/api/calls`.
- **Azure Resources**:
  - **Azure Bot Service** (`QuotationBot`) with **Microsoft App ID** and **Client Secret** (Step 2).
  - **Azure Speech Service** (`QuotationSpeechService`) with **Key 1** and **Endpoint** (Step 1).
  - **Azure Communication Services** (`QuotationACS`) with a **phone number** and **Connection String** (Step 3).
- **Ngrok**: Installed and running (`ngrok http 3978`) to expose the local bot.
- **Node.js and npm**: Installed with required packages:
  ```bash
  npm install botbuilder microsoft-cognitiveservices-speech-sdk axios dotenv @azure/communication-call-automation @azure/communication-common restify
  ```
- **Web Browser/Mobile Device**: A modern browser (e.g., Chrome, Edge) or mobile browser with microphone access for voice input/output.
- **Azure Communication Services JavaScript SDK**: For browser/mobile voice calls.
- **Hackathon Demo Focus**: A simple, functional demo with voice interaction, using the mock API response.

---

## Solution Approach
To enable voice-based interactions for the hackathon demo:
1. **Reuse Existing Node.js Bot**: Modify the bot code to handle voice input/output for ACS telephony calls, ensuring the same conversational flow as text-based messaging.
2. **Enhance ACS Integration**: Configure ACS to process voice input (Speech-to-Text) and output (Text-to-Speech) via the `/api/calls` endpoint.
3. **Browser/Mobile Interface**: Create a simple web app using the **Azure Communication Services JavaScript SDK** to initiate voice calls from a browser or mobile device, connecting to the ACS phone number.
4. **Leverage Speech Services**: Use Azure Speech Services for Speech-to-Text (to process user voice input) and Text-to-Speech (to generate voice responses).
5. **Mock API**: Use the existing mock response for demo purposes.
6. **No Certificates**: Rely on Ngrok’s HTTPS URL or Azure App Service’s built-in SSL for all endpoints.

---

## Step-by-Step Guide

### Step 1: Verify Existing Bot Setup
1. **Run the Node.js Bot**:
   - In your `QuotationBot` folder, ensure the `index.js` file (from Step 4.2 of the previous solution) is set up with the mock API response:
     ```javascript
     // In the confirm step
     const mockResponse = {
       data: {
         status: Math.random() < 0.5 ? 'accept' : Math.random() < 0.5 ? 'decline' : 'refer'
       }
     };
     const status = mockResponse.data.status;
     await sendAndSpeak(`Your application has been ${status}ed for a card. Thank you!`);
     ```
   - Run the bot:
     ```bash
     node index.js
     ```
   - Confirm the console shows:
     ```
     Server running at http://localhost:3978
     ```

2. **Run Ngrok**:
   - In a separate terminal, run:
     ```bash
     ngrok http 3978
     ```
   - Copy the HTTPS URL (e.g., `https://abc123.ngrok.io`).

3. **Update Azure Bot Service**:
   - In the [Azure Portal](https://portal.azure.com), go to **Home > All resources > QuotationBot**.
   - Under **Settings**, set the **Messaging endpoint** to:
     ```
     https://abc123.ngrok.io/api/messages
     ```
   - Click **Apply**.

4. **Verify ACS Webhook**:
   - Go to **Home > All resources > QuotationACS**.
   - Under **Call Automation**, ensure the webhook URL is:
     ```
     https://abc123.ngrok.io/api/calls
     ```
   - Confirm events: **CallConnected**, **CallDisconnected**, **RecognizeCompleted**.

---

### Step 2: Modify Node.js Bot for Voice Calls
The existing `index.js` handles ACS call events via `/api/calls` but needs enhancements to support the full conversational flow with voice input/output. Update the code to integrate Speech Services for voice interactions during ACS calls.

1. **Update `index.js`**:
   - Replace the existing code with the following, which enhances the `/api/calls` endpoint to handle the conversational flow using Azure Speech Services:
     ```javascript
     const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
     const { SpeechConfig, AudioConfig, SpeechRecognizer, SpeechSynthesizer } = require('microsoft-cognitiveservices-speech-sdk');
     const axios = require('axios');
     const { CallAutomationClient } = require('@azure/communication-call-automation');
     require('dotenv').config();

     // Azure Bot Service configuration
     const adapter = new BotFrameworkAdapter({
       appId: process.env.MICROSOFT_APP_ID,
       appPassword: process.env.MICROSOFT_APP_PASSWORD
     });

     // Azure Communication Services configuration
     const acsClient = new CallAutomationClient(process.env.ACS_CONNECTION_STRING);

     // Conversation state
     const memoryStorage = new MemoryStorage();
     const conversationState = new ConversationState(memoryStorage);
     const userProfile = conversationState.createProperty('userProfile');

     // Azure Speech Service configuration
     const speechConfig = SpeechConfig.fromSubscription(process.env.SPEECH_KEY, process.env.SPEECH_REGION);

     // Helper function to recognize speech
     async function recognizeSpeech(audioStream) {
       const audioConfig = AudioConfig.fromStreamInput(audioStream);
       const recognizer = new SpeechRecognizer(speechConfig, audioConfig);
       return new Promise((resolve, reject) => {
         recognizer.recognizeOnceAsync(result => {
           if (result.reason === 'RecognizedSpeech') {
             resolve(result.text);
           } else {
             reject(new Error('Speech recognition failed'));
           }
           recognizer.close();
         });
       });
     }

     // Helper function to synthesize speech
     async function synthesizeSpeech(text, callConnectionId) {
       const synthesizer = new SpeechSynthesizer(speechConfig);
       await synthesizer.speakTextAsync(text);
       await acsClient.getCallConnection(callConnectionId).playText({ text });
     }

     // Bot logic for messaging (unchanged)
     adapter.onTurn(async (context, next) => {
       const state = await userProfile.get(context, { step: 'start', data: {} });

       async function sendAndSpeak(message) {
         await context.sendActivity(message);
         const synthesizer = new SpeechSynthesizer(speechConfig);
         await synthesizer.speakTextAsync(message);
       }

       if (context.activity.type === 'message') {
         if (state.step === 'start') {
           await sendAndSpeak('Hello! Please tell me your first name.');
           state.step = 'firstName';
         } else if (state.step === 'firstName') {
           state.data.firstName = context.activity.text;
           await sendAndSpeak(`Got it, ${state.data.firstName}. Now, please tell me your last name.`);
           state.step = 'lastName';
         } else if (state.step === 'lastName') {
           state.data.lastName = context.activity.text;
           await sendAndSpeak(`Thanks, ${state.data.lastName}. What is your monthly income?`);
           state.step = 'income';
         } else if (state.step === 'income') {
           state.data.monthlyIncome = parseFloat(context.activity.text);
           const confirmationMessage = `Please confirm: First Name: ${state.data.firstName}, Last Name: ${state.data.lastName}, Monthly Income: $${state.data.monthlyIncome}. Say "yes" to confirm or "no" to restart.`;
           await sendAndSpeak(confirmationMessage);
           state.step = 'confirm';
         } else if (state.step === 'confirm') {
           if (context.activity.text.toLowerCase() === 'yes') {
             try {
               const mockResponse = {
                 data: {
                   status: Math.random() < 0.5 ? 'accept' : Math.random() < 0.5 ? 'decline' : 'refer'
                 }
               };
               const status = mockResponse.data.status;
               await sendAndSpeak(`Your application has been ${status}ed for a card. Thank you!`);
               state.step = 'start';
               state.data = {};
             } catch (error) {
               await sendAndSpeak('Sorry, there was an error processing your request. Please try again.');
             }
           } else {
             await sendAndSpeak('Let’s start over. Please tell me your first name.');
             state.step = 'start';
             state.data = {};
           }
         }
       }

       await conversationState.saveChanges(context);
       await next();
     });

     // Start the server
     const server = require('restify').createServer();
     server.listen(process.env.PORT || 3978, () => {
       console.log(`Server running at ${server.url}`);
     });

     server.post('/api/messages', (req, res) => {
       adapter.processActivity(req, res, async (context) => {
         await context.sendActivity('Bot is processing your request...');
       });
     });

     // Handle ACS call events
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
         if (state.step === 'firstName') {
           state.data.firstName = recognizedText;
           await synthesizeSpeech(`Got it, ${state.data.firstName}. Now, please tell me your last name.`, callConnectionId);
           state.step = 'lastName';
         } else if (state.step === 'lastName') {
           state.data.lastName = recognizedText;
           await synthesizeSpeech(`Thanks, ${state.data.lastName}. What is your monthly income?`, callConnectionId);
           state.step = 'income';
         } else if (state.step === 'income') {
           state.data.monthlyIncome = parseFloat(recognizedText);
           const confirmationMessage = `Please confirm: First Name: ${state.data.firstName}, Last Name: ${state.data.lastName}, Monthly Income: $${state.data.monthlyIncome}. Say "yes" to confirm or "no" to restart.`;
           await synthesizeSpeech(confirmationMessage, callConnectionId);
           state.step = 'confirm';
         } else if (state.step === 'confirm') {
           if (recognizedText.toLowerCase() === 'yes') {
             try {
               const mockResponse = {
                 data: {
                   status: Math.random() < 0.5 ? 'accept' : Math.random() < 0.5 ? 'decline' : 'refer'
                 }
               };
               const status = mockResponse.data.status;
               await synthesizeSpeech(`Your application has been ${status}ed for a card. Thank you!`, callConnectionId);
               state.step = 'start';
               state.data = {};
             } catch (error) {
               await synthesizeSpeech('Sorry, there was an error processing your request. Please try again.', callConnectionId);
             }
           } else {
             await synthesizeSpeech('Let’s start over. Please tell me your first name.', callConnectionId);
             state.step = 'start';
             state.data = {};
           }
         }
       }

       await conversationState.saveChanges({ conversation: { id: callConnectionId } });
       res.status(200).send();
     });
     ```

2. **Key Changes**:
   - Added `recognizeSpeech` and `synthesizeSpeech` functions to handle voice input/output using Azure Speech Services.
   - Updated the `/api/calls` endpoint to process ACS call events (`CallConnected`, `RecognizeCompleted`) and manage the conversational flow with voice.
   - Used the `callConnectionId` as the conversation ID to maintain state during calls.
   - Kept the mock API response for demo purposes.

3. **Ensure `.env` File**:
   - Verify the `.env` file contains:
     ```env
     MICROSOFT_APP_ID=your-bot-app-id
     MICROSOFT_APP_PASSWORD=your-bot-client-secret
     SPEECH_KEY=your-speech-service-key
     SPEECH_REGION=your-speech-service-region
     ACS_CONNECTION_STRING=your-acs-connection-string
     QUOTE_API_URL=https://your-api.com/quote
     PORT=3978
     ```

---

### Step 3: Create a Browser/Mobile Web App for Voice Calls
To allow users to initiate voice calls from a browser or mobile device, create a simple web app using the **Azure Communication Services JavaScript SDK** to connect to the ACS phone number and interact with the bot via voice.

1. **Install ACS JavaScript SDK**:
   - In your `QuotationBot` folder, install the SDK:
     ```bash
     npm install @azure/communication-calling
     ```

2. **Create a Web App**:
   - Create a file named `index.html` in a new folder (e.g., `webapp`):
     ```html
     <!DOCTYPE html>
     <html>
     <head>
       <title>Quotation Bot Voice Demo</title>
       <script src="https://cdn.jsdelivr.net/npm/@azure/communication-calling@1.4.4/dist/communication-calling.min.js"></script>
       <script src="https://cdn.jsdelivr.net/npm/@azure/communication-common@1.1.0/dist/communication-common.min.js"></script>
     </head>
     <body>
       <h1>Quotation Bot Voice Demo</h1>
       <button id="startCall">Start Voice Call</button>
       <button id="endCall" disabled>End Call</button>
       <p id="status">Ready to start call</p>
       <script>
         const acsConnectionString = 'your-acs-connection-string';
         const acsPhoneNumber = 'your-acs-phone-number';
         const callClient = new Azure.Communication.Calling.CallClient();
         let callAgent, call;

         async function initCallAgent() {
           const tokenCredential = new Azure.Communication.Common.CommunicationTokenCredential(acsConnectionString);
           callAgent = await callClient.createCallAgent(tokenCredential);
           document.getElementById('startCall').disabled = false;
         }

         document.getElementById('startCall').addEventListener('click', async () => {
           document.getElementById('status').innerText = 'Starting call...';
           call = callAgent.startCall([{ phoneNumber: acsPhoneNumber }], { audioOptions: { muted: false } });
           document.getElementById('startCall').disabled = true;
           document.getElementById('endCall').disabled = false;
           document.getElementById('status').innerText = 'Call connected. Speak to the bot.';
         });

         document.getElementById('endCall').addEventListener('click', () => {
           call.hangUp({ forEveryone: true });
           document.getElementById('startCall').disabled = false;
           document.getElementById('endCall').disabled = true;
           document.getElementById('status').innerText = 'Call ended. Ready to start new call.';
         });

         initCallAgent();
       </script>
     </body>
     </html>
     ```

3. **Update Placeholders**:
   - Replace `your-acs-connection-string` with the ACS Connection String (Step 3.3 of the previous solution).
   - Replace `your-acs-phone-number` with the ACS phone number (e.g., `+1-XXX-XXX-XXXX`).

4. **Serve the Web App Locally**:
   - Install a simple HTTP server:
     ```bash
     npm install -g http-server
     ```
   - Navigate to the `webapp` folder and run:
     ```bash
     http-server -p 8080
     ```
   - Access the web app at `http://localhost:8080`.

5. **Expose with Ngrok**:
   - Run a separate Ngrok instance for the web app:
     ```bash
     ngrok http 8080
     ```
   - Copy the new Ngrok URL (e.g., `https://def456.ngrok.io`).

---

### Step 4: Test the Voice-Based Demo
1. **Test from Browser**:
   - Open the Ngrok web app URL (e.g., `https://def456.ngrok.io`) in a browser (Chrome or Edge recommended).
   - Grant microphone access when prompted.
   - Click **Start Voice Call**.
   - Speak to the bot (e.g., “John”, “Doe”, “5000”, “yes”).
   - The bot should respond with voice output (e.g., “Hello! Please tell me your first name.”, followed by the conversational flow).
   - Verify the mock response (e.g., “Your application has been accepted for a card. Thank you!”).
   - Click **End Call** to hang up.

2. **Test from Mobile**:
   - Access the Ngrok web app URL on a mobile browser.
   - Follow the same steps as above, ensuring the mobile device has a working microphone and speaker.
   - Test the full conversational flow.

3. **Test with ACS Phone Number**:
   - Call the ACS phone number directly from a phone.
   - Verify the bot responds with voice prompts and processes voice input, completing the flow.

4. **Check Ngrok Logs**:
   - Open Ngrok’s web interface (`http://localhost:4040`) for the bot’s tunnel (`ngrok http 3978`).
   - Confirm POST requests to `/api/calls` for ACS call events and `/api/messages` for any fallback messaging.

---

### Step 5: Polish for Hackathon Demo
1. **Enhance Web App UI**:
   - Add CSS to `index.html` for a professional look:
     ```html
     <style>
       body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
       button { padding: 10px 20px; margin: 10px; font-size: 16px; }
       #status { color: #333; font-size: 18px; }
     </style>
     ```
   - Add a logo or hackathon branding for visual appeal.

2. **Demo Script**:
   - Prepare a script to showcase the flow:
     - Open the web app in a browser or mobile device.
     - Click **Start Voice Call**.
     - Speak: “John”, “Doe”, “5000”, “yes”.
     - Highlight the voice response (e.g., “Your application has been accepted...”).
     - End the call.
   - Optionally, demo a direct call to the ACS phone number.

3. **Monitor Usage**:
   - Check Azure Portal (**QuotationACS > Monitoring > Metrics**, **QuotationSpeechService > Metrics**) to avoid exceeding free-tier limits during the demo.

---

### Step 6: Troubleshooting
- **No Voice Response**:
  - Verify `SPEECH_KEY` and `SPEECH_REGION` in `.env`.
  - Ensure ACS is linked to the Speech Service (Step 5.1 of the previous solution).
- **Call Not Connecting**:
  - Confirm the ACS Connection String and phone number in `index.html`.
  - Check Ngrok logs for POST requests to `/api/calls`.
- **Browser/Mobile Issues**:
  - Ensure microphone permissions are granted.
  - Use Chrome or Edge for best compatibility.
- **Ngrok Errors**:
  - Restart Ngrok if the tunnel expires (`ngrok http 3978` for bot, `ngrok http 8080` for web app).
  - Use a free Ngrok account for stable URLs.

---

## Additional Notes
- **No Certificates Needed**:
  - Ngrok’s HTTPS URLs (`https://abc123.ngrok.io` for bot, `https://def456.ngrok.io` for web app) include wildcard certificates, satisfying ACS and Azure Bot Service requirements.
- **Voice Quality**:
  - Azure Speech Services provides human-like Text-to-Speech. For better demo impact, select a neural voice in the Azure Portal (**Speech Service > Speech Studio > Custom Voice**).
- **Hackathon Tips**:
  - Keep the web app simple and responsive for mobile devices.
  - Record a short video of the demo as a backup in case of connectivity issues.
  - Highlight the voice interaction and mock API response as key features.

---

## Additional Resources
- [Azure Communication Services Calling SDK](https://learn.microsoft.com/en-us/azure/communication-services/quickstarts/voice-video-calling/get-started-with-voice-calling)
- [Azure Speech Services](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/)
- [Bot Framework Activity Schema](https://learn.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-api-reference)
- [Ngrok Web Interface](https://ngrok.com/docs/web-interface)
- [Azure Portal Help](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

---

## Summary
To enable a voice-based demo for your hackathon:
1. Update the Node.js bot (`index.js`) to handle ACS call events with voice input/output using Azure Speech Services.
2. Create a simple web app (`index.html`) with the ACS JavaScript SDK to initiate voice calls from a browser or mobile device.
3. Serve the web app locally (`http-server -p 8080`) and expose it with Ngrok (`ngrok http 8080`).
4. Test the conversational flow via browser, mobile, or direct ACS phone call, ensuring voice responses and mock API output.
5. Polish the UI and prepare a demo script for impact.

This solution reuses your existing setup, adds voice call functionality, and ensures a certificate-free experience with Ngrok. For any issues or further customization, let me know, and I can assist with specific tweaks for your hackathon demo!