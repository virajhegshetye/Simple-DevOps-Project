Let’s modify **Step 2** of the Copilot Studio solution to set up Azure services using the **Azure Portal UI** instead of Azure CLI. The rest of the solution (Steps 1, 3, 4, 5, 6, and 7) remains unchanged as they don’t rely on Azure CLI for setup. This updated step will guide you through creating the necessary Azure services (Resource Group, Speech Service, Key Vault, Azure AD B2C, Application Insights, and ACS for Solution 1) via the Azure Portal UI, ensuring a seamless setup for both the real-time calling (Solution 1) and mocked calling (Solution 2) implementations in Copilot Studio.

Below is the updated Markdown file with **Step 2** modified to use the Azure Portal UI, while keeping the rest of the solution intact.

---

```markdown
# VoiceCard AI: Implementation Using Microsoft Copilot Studio with Azure Portal UI

This document provides a step-by-step solution to implement "VoiceCard AI" using **Microsoft Copilot Studio**, a low-code platform for building conversational bots. The application is designed for visually challenged users, allowing them to generate a credit card quotation ID and handle follow-up questions via voice interaction. We’ll cover two implementations:

- **Solution 1:** Real-time calling using Azure Communication Services (ACS) to handle actual phone calls to a toll-free number.
- **Solution 2:** Mocked calling for a hackathon demo, simulating the call flow without a real ACS subscription.

Copilot Studio will manage the conversational flow, and we’ll set up Azure services using the **Azure Portal UI** instead of Azure CLI. We’ll integrate with Azure services (Speech Services, ACS, etc.) using Power Automate flows and custom connectors.

## Key Features
- Voice-driven interaction for accessibility.
- Generates a quotation ID using a mock Quotation API.
- Handles follow-up questions during the conversation.
- Supports both real-time calling (Solution 1) and mocked calling (Solution 2).

## Prerequisites
- **Microsoft Copilot Studio** subscription (part of Microsoft Power Platform, available with a Microsoft 365 or Power Apps license).
- **Azure Account** with access to:
  - Azure Communication Services (ACS) for Solution 1.
  - Azure Cognitive Services Speech Service for voice interaction.
  - Azure Key Vault, Application Insights, and Azure AD B2C (optional, for best practices).
- **Power Automate** license (to create custom flows for API calls and integrations).
- **Node.js** and **npm** for the mock Quotation API.
- **PostgreSQL** for data storage (optional, if you want to persist data).

## Today's Date and Time
- **Date and Time:** 03:31 PM IST, Monday, June 02, 2025.

---

## Step 1: Set Up Your Development Environment

### 1.1 Install Required Tools
- **Microsoft Copilot Studio:** Access via [make.powerapps.com](https://make.powerapps.com/). Sign in with your Microsoft 365 account.
- **Power Automate:** Access via [flow.microsoft.com](https://flow.microsoft.com/). Ensure you have a license to create flows.
- **Node.js and npm:** Install [Node.js](https://nodejs.org/). Verify: `node -v` and `npm -v`.
- **PostgreSQL (Optional):** Install [PostgreSQL](https://www.postgresql.org/download/). Set a password (e.g., `your-postgres-password`). Create a database:
  ```bash
  psql -U postgres
  CREATE DATABASE creditcard_db;
  \q
  ```

### 1.2 Set Up the Mock Quotation API
Create a mock API for generating quotation IDs:

In `mock-creditcard-api/server.js`:

```javascript
const express = require('express');
const app = express();
app.use(express.json());

app.post('/api/creditcard/apply', (req, res) => {
    const { name, income, phone } = req.body;
    const quotationId = `QUOT-${Math.random().toString(36).substring(2, 8)}`;
    res.json({ name, phone, quotationId });
});

app.listen(3000, () => console.log('Mock Quotation API running on port 3000'));
```

To set up:
1. Create a directory: `mock-creditcard-api`.
2. Initialize a Node.js project:
   ```bash
   cd mock-creditcard-api
   npm init -y
   npm install express
   ```
3. Save the code as `server.js`.
4. Start the API:
   ```bash
   node server.js
   ```

---

## Step 2: Set Up Azure Services Using Azure Portal UI

We’ll set up the necessary Azure services (Resource Group, Speech Service, Key Vault, Azure AD B2C, Application Insights, and ACS for Solution 1) using the Azure Portal UI.

### 2.1 Create an Azure Account
1. Go to [azure.com](https://azure.com) and sign up for a free account ($200 credit for 30 days).
2. Sign in to the Azure Portal at [portal.azure.com](https://portal.azure.com).

### 2.2 Set Up a Resource Group
1. In the Azure Portal, click **Create a resource** (or use the search bar and type "Resource group").
2. Click **Create** under "Resource group".
3. Fill in the details:
   - **Resource group:** `voicecard-rg`.
   - **Region:** East US.
4. Click **Review + create**, then **Create**.

### 2.3 Create a Speech Service (For Both Solutions)
1. In the Azure Portal, click **Create a resource**.
2. Search for **Speech** and select **Speech** under Azure Cognitive Services.
3. Click **Create**.
4. Fill in the details:
   - **Subscription:** Select your subscription.
   - **Resource group:** `voicecard-rg`.
   - **Region:** East US.
   - **Name:** `voicecard-speech`.
   - **Pricing tier:** Free F0 (for demo purposes; upgrade to a paid tier for production).
5. Click **Review + create**, then **Create**.
6. Once deployed, go to the resource:
   - Navigate to **Keys and Endpoint** in the left sidebar.
   - Note **Key 1** (e.g., `your-speech-key`) and **Location/Region** (e.g., `eastus`).

### 2.4 Create a Key Vault
1. In the Azure Portal, click **Create a resource**.
2. Search for **Key Vault** and select **Key Vault**.
3. Click **Create**.
4. Fill in the details:
   - **Subscription:** Select your subscription.
   - **Resource group:** `voicecard-rg`.
   - **Key vault name:** `voicecard-vault`.
   - **Region:** East US.
   - **Pricing tier:** Standard.
5. Click **Review + create**, then **Create**.
6. Once deployed, go to the Key Vault resource:
   - Navigate to **Secrets** in the left sidebar.
   - Click **+ Generate/Import**.
   - Create a secret:
     - **Name:** `speech-key`.
     - **Value:** `your-speech-key` (from Step 2.3).
   - Click **Create**.
7. Note the Key Vault URI:
   - Go to **Overview**.
   - Copy the **Vault URI** (e.g., `https://voicecard-vault.vault.azure.net/`).

### 2.5 Set Up Azure Active Directory B2C (Optional)
1. In the Azure Portal, click **Create a resource**.
2. Search for **Azure Active Directory B2C** and select it.
3. Click **Create**.
4. Choose **Create a new Azure AD B2C Tenant**:
   - **Tenant name:** `VoiceCardB2C`.
   - **Initial domain name:** `voicecardb2c`.
   - **Country/Region:** United States.
   - Click **Review + create**, then **Create**.
5. Once the tenant is created, link it to your subscription:
   - Go to **Azure Active Directory** in the Azure Portal.
   - Switch to the `VoiceCardB2C` directory.
   - Go to **App registrations** > **New registration**.
   - **Name:** `voicecard-app`.
   - **Redirect URI:** `http://localhost:8080/login/oauth2/code/azure`.
   - Click **Register**.
   - Note the **Application (client) ID**.
6. Create a client secret:
   - Go to **Certificates & secrets**.
   - Click **+ New client secret**.
   - **Description:** `voicecard-secret`.
   - Click **Add**.
   - Note the **Client Secret** value.

### 2.6 Create an Application Insights Resource (Optional)
1. In the Azure Portal, click **Create a resource**.
2. Search for **Application Insights** and select it.
3. Click **Create**.
4. Fill in the details:
   - **Subscription:** Select your subscription.
   - **Resource group:** `voicecard-rg`.
   - **Name:** `voicecard-insights`.
   - **Region:** East US.
   - **Resource Mode:** Classic.
5. Click **Review + create**, then **Create**.
6. Once deployed, go to the resource:
   - Navigate to **Overview**.
   - Note the **Instrumentation Key** (e.g., `your-instrumentation-key`).

### 2.7 Set Up Azure Communication Services (For Solution 1 Only)
If you’re implementing Solution 1 (real-time calling), set up ACS:
1. In the Azure Portal, click **Create a resource**.
2. Search for **Communication Services** and select **Azure Communication Services**.
3. Click **Create**.
4. Fill in the details:
   - **Subscription:** Select your subscription.
   - **Resource group:** `voicecard-rg`.
   - **Resource name:** `voicecard-acs`.
   - **Data location:** United States.
5. Click **Review + create**, then **Create**.
6. Once deployed, go to the resource:
   - Navigate to **Keys** in the left sidebar.
   - Note the **Connection String** (e.g., `endpoint=https://voicecard-acs.communication.azure.com/;accesskey=your-acs-key`).
7. Acquire a toll-free number:
   - Go to **Phone Numbers** in the left sidebar.
   - Click **Get**.
   - **Type:** Toll-Free.
   - **Country/Region:** United States.
   - **Capabilities:** Calling + SMS (Inbound + Outbound).
   - Search and select a number (e.g., `+18001234567`).
   - Click **Get selected number**.
8. Store the ACS connection string in Key Vault:
   - Go back to your Key Vault (`voicecard-vault`).
   - Navigate to **Secrets**.
   - Click **+ Generate/Import**.
   - Create a secret:
     - **Name:** `acs-connection-string`.
     - **Value:** `endpoint=https://voicecard-acs.communication.azure.com/;accesskey=your-acs-key`.
   - Click **Create**.

---

## Step 3: Create a Copilot in Copilot Studio

### 3.1 Access Copilot Studio
1. Go to [make.powerapps.com](https://make.powerapps.com/) and sign in.
2. On the left sidebar, select **Copilots**.
3. Click **+ New copilot** to create a new copilot named `VoiceCardAI`.

### 3.2 Configure Copilot Settings
1. **Name:** `VoiceCardAI`.
2. **Language:** English (or your preferred language).
3. **Voice Channel:** Enable the voice channel for voice interactions:
   - Go to **Settings > Channels**.
   - Enable **Voice** and configure it to use Azure Speech Services:
     - Speech Service Key: `your-speech-key`.
     - Region: `eastus`.
4. **Authentication (Optional):** If using Azure AD B2C, configure authentication:
   - Go to **Settings > Authentication**.
   - Select **Manual (for custom provider)**.
   - Configure with your Azure AD B2C tenant details (client ID, client secret, etc.).

---

## Step 4: Implement Solution 1 - Real-Time Calling with ACS

### 4.1 Configure ACS Integration in Copilot Studio
Copilot Studio supports voice calling via ACS, but you’ll need to set up a webhook to handle incoming calls and integrate ACS with Copilot Studio.

1. **Set Up ACS Webhook:**
   - In Copilot Studio, go to **Settings > Channels > Voice**.
   - Note the webhook URL provided for incoming calls (e.g., `https://your-copilot-webhook-url`).
   - In the Azure Portal, go to your ACS resource (`voicecard-acs`):
     - Navigate to **Events** in the left sidebar.
     - Click **+ Event Subscription**.
     - **Name:** `CopilotWebhook`.
     - **Webhook URL:** `https://your-copilot-webhook-url`.
     - **Events to Subscribe:** Select **Incoming Call**.
     - Click **Create**.

2. **Create a Power Automate Flow for ACS Integration:**
   - Go to [flow.microsoft.com](https://flow.microsoft.com/) and create a new flow named `HandleACSCall`.
   - **Trigger:** HTTP Request (to receive ACS events).
     - Use the schema of an ACS incoming call event (refer to ACS documentation).
   - **Action:** Call Copilot Studio to trigger a conversation:
     - Add an action: **Microsoft Copilot Studio > Trigger a conversation**.
     - Select your `VoiceCardAI` copilot.
     - Pass the caller’s phone number (from the ACS event) as a variable.
   - Save the flow and note the HTTP endpoint URL (e.g., `https://your-power-automate-flow-url`).
   - Update the ACS event subscription in the Azure Portal to use this flow URL instead of the Copilot Studio webhook.

### 4.2 Create the Conversational Flow in Copilot Studio
1. **Create a Topic: `HandleIncomingCall`**
   - Go to **Topics** in Copilot Studio.
   - Create a new topic named `HandleIncomingCall`.
   - **Trigger Phrases:** Add a trigger like "Incoming call" (this will be triggered programmatically via Power Automate).
   - **Variables:**
     - `callerPhoneNumber` (Text): To store the caller’s phone number.
     - `userName` (Text): To store the user’s name.
     - `phoneNumber` (Text): To store the user’s provided phone number.
     - `income` (Number): To store the user’s income.
     - `quotationId` (Text): To store the generated quotation ID.
     - `followUpQuestion` (Text): To store the follow-up question (if any).

2. **Add Nodes to the Topic:**
   - **Message Node:** "Welcome to VoiceCard AI. This is an automated system to help you generate a credit card quotation ID. Please say your name."
   - **Question Node:** Ask for the user’s name (voice input).
     - Save the response to `userName`.
   - **Message Node:** "Please say your phone number, one digit at a time, for example, 1 2 3 4 5 6 7 8 9 0."
   - **Question Node:** Ask for the phone number (voice input).
     - Save the response to `phoneNumber`.
     - Add a condition to clean the input (e.g., remove non-digits using a Power Fx expression like `Substitute(phoneNumber, "[^0-9]", "")`).
   - **Message Node:** "Please say your annual income in dollars, for example, 50 thousand."
   - **Question Node:** Ask for the income (voice input).
     - Save the response to `income`.
     - Add a condition to parse the income (e.g., handle "50 thousand" using a Power Fx expression like `If(Contains(income, "thousand"), Value(Substitute(income, "thousand", "")) * 1000, Value(income))`).

3. **Call the Quotation API Using Power Automate:**
   - Create a Power Automate flow named `CallQuotationAPI`:
     - **Trigger:** PowerApps (to be called from Copilot Studio).
     - **Action:** HTTP - Make a POST request to `http://localhost:3000/api/creditcard/apply`.
       - Body: `{ "name": "@{triggerBody()['name']}", "phone": "@{triggerBody()['phone']}", "income": "@{triggerBody()['income']}" }`.
     - **Action:** Parse JSON to extract the `quotationId`.
     - **Action:** Respond to PowerApps with the `quotationId`.
   - In Copilot Studio, add an **Action** node to call this flow:
     - Pass `userName`, `phoneNumber`, and `income` as inputs.
     - Save the output (`quotationId`) to the `quotationId` variable.

4. **Save to Database (Optional):**
   - Create a Power Automate flow named `SaveToDatabase`:
     - **Trigger:** PowerApps.
     - **Action:** SQL Server - Insert row (if using PostgreSQL, use a custom connector or API to insert into the database).
       - Table: `application`.
       - Columns: `name`, `phone`, `income`, `decision` (map to `userName`, `phoneNumber`, `income`, `quotationId`).
   - Add an **Action** node in Copilot Studio to call this flow.

5. **Inform the User of the Quotation ID:**
   - **Message Node:** "Thank you, [userName]. Your credit card quotation ID is [quotationId]. Please note this down."

6. **Handle Follow-Up Questions:**
   - **Question Node:** "Do you have any follow-up questions about your quotation? Please say yes or no."
     - Save the response to a temporary variable (e.g., `hasFollowUp`).
   - **Condition Node:** If `hasFollowUp` contains "yes":
     - **Question Node:** "Please state your question."
       - Save the response to `followUpQuestion`.
     - **Message Node:** "Thank you for your question: [followUpQuestion]. A customer care representative will follow up with you soon."
   - **Message Node:** "Thank you for using VoiceCard AI. Goodbye!"

### 4.3 Test Solution 1
1. **Publish the Copilot:**
   - Click **Publish** in Copilot Studio to deploy the copilot.
2. **Call the Toll-Free Number:**
   - Dial the toll-free number (`+18001234567`) acquired in Step 2.7.
   - The ACS webhook will trigger the Power Automate flow (`HandleACSCall`), which will start the `HandleIncomingCall` topic in Copilot Studio.
   - Follow the voice prompts to provide your name, phone number, income, and follow-up questions.
   - The copilot will generate a quotation ID and end the call.

---

## Step 5: Implement Solution 2 - Mocked Calling for Hackathon Demo

### 5.1 Remove ACS Dependency
Since Solution 2 mocks the calling functionality, we don’t need ACS. We’ll simulate the call flow within Copilot Studio using a web-based trigger.

### 5.2 Create a Demo Topic in Copilot Studio
1. **Create a Topic: `SimulateIncomingCall`**
   - Go to **Topics** in Copilot Studio.
   - Create a new topic named `SimulateIncomingCall`.
   - **Trigger Phrases:** Add a trigger like "Start demo" (or trigger programmatically via a web endpoint).
   - **Variables:** Reuse the same variables as in Solution 1 (`callerPhoneNumber`, `userName`, `phoneNumber`, `income`, `quotationId`, `followUpQuestion`).

2. **Add Nodes to Simulate the Call:**
   - **Message Node:** "[MockCallingService] Simulating incoming call from +18001234567..."
   - **Message Node:** "[MockCallingService] Call accepted. Call is now active."
   - Reuse the same flow as in Solution 1 (`HandleIncomingCall` topic):
     - **Message Node:** "Welcome to VoiceCard AI. This is an automated system to help you generate a credit card quotation ID. Please say your name."
     - **Question Node:** Ask for the user’s name (voice input).
       - Save the response to `userName`.
     - **Message Node:** "Please say your phone number, one digit at a time, for example, 1 2 3 4 5 6 7 8 9 0."
     - **Question Node:** Ask for the phone number (voice input).
       - Save the response to `phoneNumber`.
       - Clean the input using Power Fx.
     - **Message Node:** "Please say your annual income in dollars, for example, 50 thousand."
     - **Question Node:** Ask for the income (voice input).
       - Save the response to `income`.
       - Parse the income using Power Fx.
     - Call the `CallQuotationAPI` flow to generate the quotation ID.
     - Save to the database (optional) using the `SaveToDatabase` flow.
     - **Message Node:** "Thank you, [userName]. Your credit card quotation ID is [quotationId]. Please note this down."
     - **Question Node:** "Do you have any follow-up questions about your quotation? Please say yes or no."
       - Save the response to `hasFollowUp`.
     - **Condition Node:** If `hasFollowUp` contains "yes":
       - **Question Node:** "Please state your question."
         - Save the response to `followUpQuestion`.
       - **Message Node:** "Thank you for your question: [followUpQuestion]. A customer care representative will follow up with you soon."
     - **Message Node:** "Thank you for using VoiceCard AI. Goodbye!"
     - **Message Node:** "[MockCallingService] Simulating call hang-up..."

### 5.3 Create a Web Endpoint to Trigger the Demo
1. **Create a Power Automate Flow: `TriggerDemo`**
   - **Trigger:** HTTP Request (to trigger the demo via a web endpoint).
   - **Action:** Microsoft Copilot Studio - Trigger a conversation.
     - Select the `VoiceCardAI` copilot.
     - Trigger the `SimulateIncomingCall` topic.
   - Save the flow and note the HTTP endpoint URL (e.g., `https://your-demo-flow-url`).

2. **Create a Simple Web Page to Start the Demo:**
   - In Copilot Studio, go to **Settings > Channels > Custom Website**.
   - Add a custom website channel and note the embed code.
   - Alternatively, create a simple HTML page to trigger the demo:
     In `demo.html`:
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>VoiceCard AI Demo</title>
     </head>
     <body>
         <h1>VoiceCard AI Hackathon Demo</h1>
         <button onclick="startDemo()">Start Demo</button>
         <script>
             async function startDemo() {
                 await fetch('https://your-demo-flow-url', {
                     method: 'POST',
                     headers: { 'Content-Type': 'application/json' },
                     body: JSON.stringify({})
                 });
                 alert('Demo started. Follow the voice prompts.');
             }
         </script>
     </body>
     </html>
     ```
   - Host this page locally using a simple server (e.g., `npx http-server`) or deploy it to a static hosting service.

### 5.4 Test Solution 2
1. **Publish the Copilot:**
   - Click **Publish** in Copilot Studio.
2. **Run the Demo:**
   - Open `demo.html` in a browser.
   - Click the "Start Demo" button.
   - The `SimulateIncomingCall` topic will start, and you’ll hear voice prompts (e.g., "Welcome to VoiceCard AI... Please say your name.").
   - Respond via voice to provide your name, phone number, income, and follow-up questions.
   - Console logs in Copilot Studio will show the simulated call flow (e.g., "[MockCallingService] Simulating incoming call from +18001234567...").
   - The demo will end with a goodbye message and a simulated hang-up.

---

## Step 6: Changes Required to Adapt Java Solution to Copilot Studio

### 6.1 Architectural Changes
- **Java Application Replaced:** The Spring Boot application (Java 21) is replaced by Copilot Studio’s conversational flow.
- **Voice Interaction:** Instead of using the Azure Speech SDK directly in Java, Copilot Studio’s built-in voice channel (backed by Azure Speech Services) handles speech-to-text and text-to-speech.
- **Calling Service:**
  - **Solution 1:** ACS integration is managed via Copilot Studio’s voice channel and Power Automate flows.
  - **Solution 2:** The `MockCallingService` is replaced by simulated messages within the `SimulateIncomingCall` topic.
- **Quotation API and Database:** These are still called via Power Automate flows, similar to the HTTP and SQL actions in the Java solution.
- **Security:** Azure AD B2C authentication is configured directly in Copilot Studio’s settings, replacing Spring Security.

### 6.2 Code Changes
- **Removed Java Code:** All Java files (`VoiceCardAiApplication.java`, `ApplicationService.java`, etc.) are no longer needed.
- **Power Automate Flows:** Replace Java service logic with flows:
  - `CallQuotationAPI` flow replaces the Java `QuotationService`.
  - `SaveToDatabase` flow replaces the Java database integration.
  - `HandleACSCall` (Solution 1) and `TriggerDemo` (Solution 2) flows replace the Java controllers.
- **Conversational Flow:** The logic in `ApplicationService.java` (e.g., prompting for name, phone, income) is now implemented as a topic in Copilot Studio.

### 6.3 Deployment Changes
- **No Spring Boot Deployment:** Instead of running a Spring Boot app (`mvn spring-boot:run`), you publish the copilot in Copilot Studio.
- **Web Endpoint for Demo:** Solution 2 uses a web-based trigger (`demo.html`) instead of a Java controller (`DemoController.java`).

---

## Step 7: Prepare for Hackathon Presentation (Solution 2)

### 7.1 Demonstrate the Flow
- Open `demo.html` in a browser.
- Click "Start Demo" to trigger the `SimulateIncomingCall` topic.
- Use a microphone to respond to voice prompts (name, phone number, income, follow-up questions).
- Show the Copilot Studio logs to highlight the simulated call flow (e.g., "[MockCallingService] Simulating incoming call...").
- Display the database entries (if implemented) to confirm the quotation ID was saved.

### 7.2 Explain the Mocked Calling
- Mention that real-time calling with ACS was mocked for the demo due to subscription constraints.
- Highlight that the voice interaction (via Copilot Studio’s voice channel) and backend logic (quotation generation, database storage) are fully functional.
- If judges ask about real calling, explain that Solution 1 integrates ACS, as shown in Step 4.

---

## Accessibility Considerations

- **Voice-Only Interaction:** Copilot Studio’s voice channel ensures a seamless voice-driven experience, with clear prompts for visually challenged users.
- **Error Handling:** Input parsing (e.g., for phone numbers and income) is handled using Power Fx expressions in Copilot Studio.
- **Follow-Up Support:** Follow-up questions are logged for later action by a representative.

---

## Summary

- Implemented VoiceCard AI using Microsoft Copilot Studio, replacing the Java-based solution with a low-code conversational flow.
- **Solution 1:** Integrated ACS for real-time calling to a toll-free number, using Power Automate to handle incoming call events.
- **Solution 2:** Mocked the calling functionality for a hackathon demo, simulating the call flow within Copilot Studio.
- Set up Azure services using the Azure Portal UI instead of Azure CLI, making it more accessible for users unfamiliar with command-line tools.
- Used Azure Speech Services (via Copilot Studio’s voice channel) for voice interaction.
- Integrated with a mock Quotation API and database using Power Automate flows.
- Ensured accessibility with voice-only prompts and error handling.

This solution is now fully tailored for both real-time calling and a hackathon demo, leveraging Copilot Studio’s low-code capabilities and Azure Portal UI for service setup.
```
---

### **Key Changes in Step 2**

- **Azure Portal UI Instead of CLI:** Step 2 now uses the Azure Portal’s graphical interface to create and configure services, making it more accessible for users who prefer a UI over command-line tools.
- **Same Functionality:** The setup achieves the same outcome as the CLI-based approach (e.g., creating a resource group, Speech Service, Key Vault, etc.), but through a browser-based interface.
- **Screenshots and Clicks:** Instructions are detailed with steps to navigate the Azure Portal, click buttons, and fill in forms, ensuring clarity for users unfamiliar with Azure CLI.

The rest of the solution (Copilot Studio setup, conversational flows, Power Automate integrations, and demo preparation) remains unchanged, as those steps were not dependent on Azure CLI.