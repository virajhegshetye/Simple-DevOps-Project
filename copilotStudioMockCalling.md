For your **hackathon demo**, you’re building a **voice-based quotation management bot** using **Microsoft Copilot Studio** to handle the conversational flow (collecting first name, last name, monthly income, confirming input, and providing a mock API response like "accepted"). In the previous steps, you created a voice-enabled agent (`QuotationBot`) and designed the conversational flow in a topic named `QuotationFlow`. Now, in **Step 3: Publish and Embed the Agent in a Web App**, you’ll publish the agent in Copilot Studio and embed it into a web app to enable browser/mobile voice interactions, simulating telephony without an **Azure Communication Services (ACS)** phone number. This approach uses the **Azure Portal** (UI-based, no CLI), avoids certificate management, and leverages **Ngrok** for external access, ensuring a demo-ready setup for your hackathon. Since you’re facing a microphone issue, I’ll also include tips to troubleshoot it during this step. Below is a detailed, beginner-friendly guide in **Markdown format** to walk you through the process.

---

# Step 3: Publish and Embed the Agent in a Web App

## Overview
In this step, you’ll **publish** your Copilot Studio agent (`QuotationBot`) to make it available for use and then **embed** it into a custom web app (`index.html`) for browser/mobile access. The embedded agent will support voice interactions using Copilot Studio’s **Speech & DTMF modality**, allowing users to interact via microphone and hear voice responses (via Azure Speech Services). You’ll serve the web app locally using `http-server` and expose it externally with Ngrok, ensuring HTTPS access without certificate management. This setup enables you to demo the voice-based conversational flow (e.g., "John", "Doe", "5000", "yes") in a browser or mobile device, ideal for your hackathon presentation.

---

## Prerequisites
- **Copilot Studio Agent**: You’ve created the `QuotationBot` agent (Step 1) and designed the `QuotationFlow` topic (Step 2) in Copilot Studio ([copilotstudio.microsoft.com](https://copilotstudio.microsoft.com)).
- **Conversational Flow Tested**: The flow works in Copilot Studio’s **Test your agent** panel, responding to voice inputs (e.g., "start quotation", "John", "Doe", "5000", "yes") with voice outputs.
- **Tools Installed**:
  - **http-server**: Installed globally (`npm install -g http-server`) to serve the web app locally.
  - **Ngrok**: Installed and ready to expose the web app externally.
- **Browser/Mobile Device**: Chrome or Edge with microphone access for testing.
- **Web App Folder**: A `webapp` folder exists from previous steps, where you’ll create `index.html`.

---

## Step 3: Publish and Embed the Agent in a Web App - Detailed Steps

### 1. Publish the Agent in Copilot Studio
Publishing makes your agent live and accessible for embedding in channels like a web app.

1. **Open Copilot Studio**:
   - Log in to [copilotstudio.microsoft.com](https://copilotstudio.microsoft.com) with your Microsoft 365 account.
   - On the **Home** page, under **Recent agents**, click on your agent (`QuotationBot`).

2. **Navigate to the Publish Section**:
   - Look at the left-hand sidebar for the main navigation menu.
   - Find the **Publish** option (typically near the bottom, represented by an upward arrow or "Publish" label).
   - Click **Publish** to enter the publishing area.
   - Alternatively, if you’re in the topic editor (e.g., `QuotationFlow`), look at the top-right corner of the interface for a **Publish** button (an upward arrow or "Publish" label).

3. **Publish the Agent**:
   - On the **Publish** page, you’ll see a message like "Publish your agent to make the latest changes available to your users."
   - Click the **Publish** button (a blue button labeled "Publish").
   - A confirmation dialog may appear (e.g., "Are you sure you want to publish?").
   - Confirm by clicking **Publish** again.
   - Wait for the publishing process to complete (takes 1-2 minutes).
   - Once done, you’ll see a success message like "Successfully published."

4. **Verify Publish Status**:
   - The **Publish** page may show the last published date/time (e.g., "Last published: June 05, 2025, 01:40 AM IST").
   - Ensure this timestamp matches your latest changes to confirm the agent is up-to-date.

---

### 2. Configure the Web App Channel
To embed the agent in a web app, you need to enable the **Web app** channel in Copilot Studio and obtain the embed code (HTML snippet).

1. **Navigate to Channels**:
   - In the left-hand sidebar, find the **Channels** option (represented by a plug icon or "Channels" label).
   - Click **Channels** to enter the channels management area.
   - The **Channels** page displays a list of available channels (e.g., Microsoft Teams, Web app, Custom).

2. **Select Web App Channel**:
   - In the list of channels, find **Web app** (may be labeled as "Web app (Custom)" or "Custom Website").
   - Click **Web app** to open its configuration settings.
   - If the Web app channel isn’t enabled, click **Add channel** or **Enable** to activate it.

3. **Configure Web App Settings**:
   - In the Web app channel settings, you’ll see options to customize the channel experience.
   - **Channel URL**: Copilot Studio generates a unique URL for your agent (e.g., `https://powerva.microsoft.com/webchat?...`).
   - **Embed Code**: Below the URL, find the **Embed code** section.
     - This section provides an `<iframe>` HTML snippet to embed the agent in a web page.
     - Example embed code:
       ```html
       <iframe src="https://powerva.microsoft.com/webchat?botId=your-bot-id&..." width="100%" height="600px"></iframe>
       ```
   - **Copy the Embed Code**:
     - Click the **Copy** button next to the embed code (a clipboard icon or "Copy" label).
     - Save the copied code in a text editor for the next step.
   - **Optional Settings**:
     - **Speech & DTMF**: Ensure this modality is enabled (default for a voice-enabled agent).
       - Look for a toggle or checkbox labeled "Enable Speech & DTMF" or "Voice input".
       - If not visible, it’s enabled by default since you used the Voice template.
     - **Text Input Fallback**: For the hackathon, enable text input as a backup:
       - Find the option "Allow text input" (or similar) and toggle it on.
       - This ensures users can type if voice fails (e.g., due to the microphone issue).

4. **Save Channel Settings**:
   - If you made changes (e.g., enabled text input), click **Save** (top-right of the channel settings page).

---

### 3. Create the Web App with the Embed Code
You’ll create a simple `index.html` file to embed the Copilot Studio agent, allowing browser/mobile access with voice interaction support.

1. **Navigate to the `webapp` Folder**:
   - Open your `webapp` folder (created in previous steps) in Visual Studio Code or your preferred editor.
   - If the folder doesn’t exist, create it:
     ```bash
     mkdir webapp
     cd webapp
     ```

2. **Create `index.html`**:
   - Create a new file named `index.html` in the `webapp` folder.
   - Add the following code, pasting the embed code from Copilot Studio:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
       <title>Quotation Bot Voice Demo</title>
       <style>
         body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
         iframe { width: 100%; height: 600px; border: none; }
       </style>
     </head>
     <body>
       <h1>Quotation Bot Voice Demo - Hackathon 2025</h1>
       <img src="https://your-logo-url.com/logo.png" alt="Hackathon Logo" style="max-width: 100px;" />
       <!-- Paste the embed code from Copilot Studio here -->
       <!-- Example: <iframe src="https://powerva.microsoft.com/webchat?botId=your-bot-id&..." width="100%" height="600px"></iframe> -->
     </body>
     </html>
     ```
   - Replace the `<iframe>` comment with the actual embed code copied from Copilot Studio.
   - Replace `https://your-logo-url.com/logo.png` with a URL to your hackathon logo (or remove the `<img>` tag if not needed).

3. **Save the File**:
   - Save `index.html` in the `webapp` folder.

---

### 4. Serve the Web App Locally and Expose with Ngrok
You’ll use `http-server` to host the web app locally and Ngrok to make it accessible externally via HTTPS.

1. **Serve the Web App**:
   - Open a terminal in the `webapp` folder:
     ```bash
     cd webapp
     ```
   - Start `http-server` on port 8080:
     ```bash
     http-server -p 8080
     ```
   - Confirm the console shows:
     ```
     Starting up http-server, serving ./
     Available on:
       http://127.0.0.1:8080
     ```
   - Access `http://localhost:8080` in a browser to verify the web app loads (you’ll see the Copilot Studio chat widget).

2. **Expose with Ngrok**:
   - Open a new terminal (leave `http-server` running).
   - Start Ngrok to expose port 8080:
     ```bash
     ngrok http 8080
     ```
   - Ngrok will display a public HTTPS URL:
     ```
     Forwarding https://def456.ngrok.io -> http://localhost:8080
     ```
   - Copy the HTTPS URL (e.g., `https://def456.ngrok.io`).

---

### 5. Test the Embedded Agent with Voice Interaction
Test the web app to ensure the agent supports voice input/output and completes the conversational flow.

1. **Access the Web App**:
   - Open `https://def456.ngrok.io` in Chrome or Edge.
   - The page should display the title, logo (if added), and the Copilot Studio chat widget (a small chat window or iframe).

2. **Start the Conversation**:
   - In the chat widget, look for a microphone icon (typically at the bottom of the chat window, next to the text input field).
   - Click the microphone icon to enable voice input.
     - The browser should prompt for microphone access (e.g., "Allow this site to use your microphone?").
     - Click **Allow**.
   - Speak a trigger phrase (e.g., "start quotation").
   - The agent should respond with: "Hello! Please tell me your first name." (spoken aloud via Text-to-Speech).

3. **Complete the Conversational Flow**:
   - Speak responses (e.g., "John", "Doe", "5000", "yes").
   - Verify the agent:
     - Recognizes voice input (Speech-to-Text).
     - Responds with voice output (e.g., "Got it, John. Now, please tell me your last name.").
     - Confirms details: "Please confirm: First Name: John, Last Name: Doe, Monthly Income: $5000..."
     - Provides the mock response: "Your application has been accepted for a card. Thank you!"
     - Ends the conversation.

4. **Test on Mobile**:
   - Access `https://def456.ngrok.io` on a mobile browser (Chrome).
   - Grant microphone access (check device settings if no prompt: iOS: **Settings** > **Privacy** > **Microphone**; Android: **Settings** > **Apps** > **Chrome** > **Permissions**).
   - Test the same flow.

---

### 6. Troubleshoot the Microphone Issue
Since you’ve faced microphone issues in the previous solution, let’s ensure it works in the embedded Copilot Studio agent.

1. **Verify Microphone Prompt**:
   - When you click the microphone icon in the chat widget, ensure the browser prompts for access.
   - If no prompt appears:
     - Check browser permissions:
       - Chrome: Padlock icon > **Site settings** > **Microphone** > **Allow**.
       - Edge: Three dots > **Settings** > **Cookies and site permissions** > **Microphone** > Allow `https://def456.ngrok.io`.
     - Clear blocked permissions:
       - Chrome: `chrome://settings/content/microphone`, remove `https://def456.ngrok.io` from the "Block" list.
       - Edge: `edge://settings/content/microphone`, clear blocks.

2. **Test Microphone in Copilot Studio**:
   - Go back to Copilot Studio and open **Test your agent**.
   - Enable **Speech & DTMF** mode and test voice input.
   - If the microphone works here but not in the web app, the issue may be with the iframe or browser security.

3. **Force Microphone Access in Web App**:
   - Add a script to `index.html` to pre-request microphone access:
     ```html
     <script>
       navigator.mediaDevices.getUserMedia({ audio: true })
         .then(() => console.log('Microphone access granted'))
         .catch(err => console.error('Microphone access denied:', err));
     </script>
     ```
   - Place this script just before the closing `</body>` tag.
   - Reload the page to trigger the prompt.

4. **Check Browser Console**:
   - Open the browser console (F12 > Console) while testing.
   - Look for errors related to microphone access (e.g., "NotAllowedError", "SecurityError").
   - If errors appear, share them for further troubleshooting.

5. **Fallback to Text**:
   - If the microphone still fails, use text input as a backup:
     - In the chat widget, type "start quotation" and proceed with the flow.
     - This ensures the demo can proceed even if voice fails.

---

### 7. Polish for Hackathon Demo
1. **Enhance UI**:
   - The `index.html` includes basic CSS for a clean look.
   - Adjust the iframe height if needed (e.g., `height: 800px` for mobile).
   - Add a background color or style:
     ```html
     <style>
       body { background-color: #f0f0f0; }
     </style>
     ```

2. **Demo Script**:
   - Open `https://def456.ngrok.io` in Chrome (laptop or mobile).
   - Start the conversation with voice (or text if voice fails).
   - Speak/type: "start quotation", "John", "Doe", "5000", "yes".
   - Highlight the conversational flow and mock response.
   - Duration: 1-2 minutes.

3. **Backup Plan**:
   - Record a video of the demo.
   - Test on multiple devices to ensure reliability.

---

## Additional Notes
- **No ACS Phone Number**: The web app embedding mocks telephony, relying on Copilot Studio’s voice capabilities.
- **No Certificates**: Ngrok’s HTTPS URL (`https://def456.ngrok.io`) satisfies security requirements.
- **Microphone Issue**: Copilot Studio’s managed Speech SDK should resolve the issue, but browser settings may still interfere.
- **Hackathon Appeal**: Highlight the no-code setup with Copilot Studio and voice interaction (or text fallback).

---

## Summary
To publish and embed the agent in a web app:
1. Publish the `QuotationBot` agent in Copilot Studio via the **Publish** menu.
2. Configure the **Web app** channel and copy the embed code.
3. Create `index.html` in the `webapp` folder, embedding the agent with the copied iframe code.
4. Serve the web app locally (`http-server -p 8080`) and expose it with Ngrok (`ngrok http 8080`).
5. Test the voice interaction flow in browser/mobile, troubleshooting the microphone issue.
6. Polish the demo for your hackathon presentation.

If the microphone issue persists, share browser console errors, and I’ll provide further assistance to ensure your hackathon demo succeeds!