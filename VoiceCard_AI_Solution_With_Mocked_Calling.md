Since real-time calling with Azure Communication Services (ACS) requires a subscription and may not be feasible during the hackathon, we can mock the calling functionality to simulate the experience for a demo. This approach will allow you to showcase the "VoiceCard AI" application for visually challenged users without needing an active ACS subscription or real phone calls. We’ll simulate the call interaction by logging voice prompts and user inputs in the console, mimicking the behavior of a real call. The rest of the application (voice interaction with Azure Speech Services, quotation ID generation, database storage, etc.) will remain functional, ensuring a realistic demo experience.

Below is the updated step-by-step solution in Markdown format, incorporating the mocked calling functionality while keeping the rest of the application intact. This solution builds on the previous end-to-end setup but replaces the ACS real-time calling with a mocked version.

---

### **VoiceCard_AI_Solution_With_Mocked_Calling.md**

```markdown
# VoiceCard AI: Complete End-to-End Solution with Mocked Calling for Hackathon Demo

This document provides a complete step-by-step solution for "VoiceCard AI," a voice-driven application designed for visually challenged users. The application simulates a user calling a toll-free customer care number, generating a credit card quotation ID using a Quotation API, and handling follow-up questions—all through voice-based interaction. Since real-time calling with **Azure Communication Services (ACS)** requires a subscription and may not be feasible during the hackathon, we’ll mock the calling functionality by logging interactions in the console to simulate a phone call. The solution uses **Azure Cognitive Services Speech SDK** for speech-to-text and text-to-speech, runs on **Java 21**, and is tailored for a hackathon demo.

## Key Features
- Simulates a user calling a toll-free customer care number (mocked for demo purposes).
- Guides users through the credit card quotation process using voice prompts.
- Generates a quotation ID by calling a mock Quotation API.
- Handles follow-up questions via voice during the simulated call.
- Ensures accessibility for visually challenged users with voice-only interaction.

## Prerequisites
- **Java 21** installed.
- **Maven** for building the Java project.
- **VS Code** or **IntelliJ** for coding.
- **Node.js** and **npm** for the mock API.
- **PostgreSQL** for data storage.
- **Azure CLI** for setting up Azure services (for Speech Services and other components).
- **Note:** Azure Communication Services (ACS) subscription is not required since calling is mocked.

## Today's Date and Time
- **Date and Time:** 12:45 PM IST, Monday, June 02, 2025.

---

## Step 1: Set Up Your Development Environment

### 1.1 Verify Java 21
- Confirm Java 21 is installed:
  ```bash
  java -version
  ```
  Expected output:
  ```
  java version "21" 2023-09-19 LTS
  ```
- Set `JAVA_HOME`:
  - On Windows: `setx JAVA_HOME "C:\Program Files\Java\jdk-21" /M`
  - On macOS/Linux: Add to `~/.bashrc` or `~/.zshrc`:
    ```bash
    export JAVA_HOME=/path/to/jdk-21
    source ~/.bashrc
    ```
- Verify: `echo $JAVA_HOME`.

### 1.2 Install Other Tools
- **Maven**: Download from [apache.org](https://maven.apache.org/download.cgi), extract, and add to `PATH`. Verify: `mvn -version`.
- **VS Code/IntelliJ**: Install [VS Code](https://code.visualstudio.com/) or [IntelliJ IDEA](https://www.jetbrains.com/idea/). Add Java extensions if using VS Code.
- **Node.js and npm**: Install [Node.js](https://nodejs.org/). Verify: `node -v` and `npm -v`.
- **PostgreSQL**: Install [PostgreSQL](https://www.postgresql.org/download/). Set a password (e.g., `your-postgres-password`). Create a database:
  ```bash
  psql -U postgres
  CREATE DATABASE creditcard_db;
  \q
  ```
- **Azure CLI**: Install from [learn.microsoft.com/en-us/cli/azure/install-azure-cli](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli). Verify: `az --version`.

---

## Step 2: Set Up Azure Services Using Azure CLI (Limited to Speech Services)

Since real-time calling is mocked, we only need Azure Cognitive Services Speech Service for voice interaction. Other services like Key Vault, Application Insights, and Azure AD B2C are still included for a complete demo setup.

### 2.1 Create an Azure Account
- Sign up for a free Azure account at [azure.com](https://azure.com) ($200 credit for 30 days).
- Log in via the CLI:
  ```bash
  az login
  ```

### 2.2 Set Up a Resource Group
Create a resource group named `voicecard-rg` in the East US region:
```bash
az group create --name voicecard-rg --location eastus
```

### 2.3 Create a Speech Service
1. Create a Speech Service resource:
   ```bash
   az cognitiveservices account create \
       --name voicecard-speech \
       --kind SpeechServices \
       --sku F0 \
       --location eastus \
       --resource-group voicecard-rg
   ```
2. Retrieve the Speech Service key and region:
   ```bash
   az cognitiveservices account keys list --name voicecard-speech --resource-group voicecard-rg --query key1 --output tsv
   ```
   Example: `your-speech-key`
   ```bash
   az cognitiveservices account show --name voicecard-speech --resource-group voicecard-rg --query location --output tsv
   ```
   Example: `eastus`

### 2.4 Create a Key Vault
Create a Key Vault to store secrets (even though we’re mocking ACS, we’ll use Key Vault for best practices):
```bash
az keyvault create --name voicecard-vault --resource-group voicecard-rg --location eastus
```
- Store the Speech Service key:
  ```bash
  az keyvault secret set --vault-name voicecard-vault --name speech-key --value "your-speech-key"
  ```
- Note the Key Vault URI:
  ```bash
  az keyvault show --name voicecard-vault --resource-group voicecard-rg --query properties.vaultUri --output tsv
  ```
  Example: `https://voicecard-vault.vault.azure.net/`

### 2.5 Set Up Azure Active Directory B2C
1. Create an Azure AD B2C tenant:
   ```bash
   az ad app create --display-name VoiceCardB2C
   az ad sp create --id <application-id-from-previous-command>
   az tenant create --display-name VoiceCardB2C --domain-name voicecardb2c --country-code US
   ```
   Note the tenant ID.

2. Switch to the B2C tenant:
   ```bash
   az login --tenant voicecardb2c.onmicrosoft.com
   ```

3. Register an application:
   ```bash
   az ad app create --display-name voicecard-app --reply-urls http://localhost:8080/login/oauth2/code/azure
   ```
   Note the **Application (client) ID**.

4. Create a client secret:
   ```bash
   az ad app credential reset --id <application-client-id> --append
   ```
   Note the **Client Secret**.

### 2.6 Create an Application Insights Resource
```bash
az monitor app-insights component create --app voicecard-insights --location eastus --resource-group voicecard-rg
```
- Note the Instrumentation Key:
  ```bash
  az monitor app-insights component show --app voicecard-insights --resource-group voicecard-rg --query instrumentationKey --output tsv
  ```
  Example: `your-instrumentation-key`

---

## Step 3: Create the Java Application with Mocked Calling

### 3.1 Project Structure
The project structure for the Spring Boot application (`voicecard-ai`) is as follows:

```
voicecard-ai/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── voicecard/
│   │   │           ├── VoiceCardAiApplication.java
│   │   │           ├── config/
│   │   │           │   ├── AppInsightsConfig.java
│   │   │           │   └── SecurityConfig.java
│   │   │           ├── controller/
│   │   │           │   └── DemoController.java
│   │   │           ├── model/
│   │   │           │   └── Application.java
│   │   │           ├── repository/
│   │   │           │   └── ApplicationRepository.java
│   │   │           ├── service/
│   │   │           │   ├── SpeechService.java
│   │   │           │   ├── MockCallingService.java
│   │   │           │   ├── QuotationService.java
│   │   │           │   ├── ApplicationService.java
│   │   │           │   └── KeyVaultService.java
│   │   └── resources/
│   │       └── application.properties
└── mock-creditcard-api/
    └── server.js
```

### 3.2 Create a Spring Boot Project
1. Go to [start.spring.io](https://start.spring.io/).
2. Fill in:
   - Project: Maven
   - Language: Java
   - Spring Boot: 3.2.5
   - Group: `com.voicecard`
   - Artifact: `voicecard-ai`
   - Java: 21
   - Dependencies: Add "Spring Web", "Spring Data JPA", "PostgreSQL Driver", "Spring Boot Starter Thymeleaf".
3. Generate, download, and open in your IDE.

### 3.3 `pom.xml`
The `pom.xml` file includes dependencies for Speech SDK, Key Vault, Application Insights, and Spring Security. Note that ACS dependencies are removed since calling is mocked:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.voicecard</groupId>
    <artifactId>voicecard-ai</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>voicecard-ai</name>
    <description>VoiceCard AI project for credit card applications</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
        <relativePath/>
    </parent>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Azure Speech SDK -->
        <dependency>
            <groupId>com.microsoft.azure.cognitiveservices</groupId>
            <artifactId>azure-cognitiveservices-speech</artifactId>
            <version>1.38.0</version>
        </dependency>

        <!-- Azure Key Vault -->
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-identity</artifactId>
            <version>1.12.0</version>
        </dependency>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-security-keyvault-secrets</artifactId>
            <version>4.8.0</version>
        </dependency>

        <!-- Application Insights -->
        <dependency>
            <groupId>com.microsoft.applicationinsights</groupId>
            <artifactId>applicationinsights-core</artifactId>
            <version>3.5.1</version>
        </dependency>

        <!-- HTTP Client for Quotation API -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.14</version>
        </dependency>

        <!-- Spring Security for Azure AD B2C -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-oauth2-client</artifactId>
        </dependency>

        <!-- JSON Processing -->
        <dependency>
            <groupId>org.json</groupId>
            <artifactId>json</artifactId>
            <version>20231013</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3.4 `application.properties`
The `application.properties` file contains all configurations. ACS-related properties are removed since calling is mocked:

```properties
# PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/creditcard_db
spring.datasource.username=postgres
spring.datasource.password=your-postgres-password
spring.jpa.hibernate.ddl-auto=update

# Azure Speech Service
azure.speech.key=your-speech-key
azure.speech.region=eastus

# Azure Key Vault
azure.keyvault.uri=https://voicecard-vault.vault.azure.net/
azure.keyvault.client-id=your-adb2c-client-id
azure.keyvault.client-secret=your-adb2c-client-secret
azure.keyvault.tenant-id=your-adb2c-tenant-id

# Application Insights
applicationinsights.instrumentation-key=your-instrumentation-key

# Mock Quotation API
creditcard.api.url=http://localhost:3000/api/creditcard/apply

# Spring Security OAuth2 for Azure AD B2C
spring.security.oauth2.client.registration.azure.client-id=your-adb2c-client-id
spring.security.oauth2.client.registration.azure.client-secret=your-adb2c-client-secret
spring.security.oauth2.client.registration.azure.scope=openid,profile
spring.security.oauth2.client.registration.azure.authorization-grant-type=authorization_code
spring.security.oauth2.client.registration.azure.redirect-uri=http://localhost:8080/login/oauth2/code/azure
spring.security.oauth2.client.provider.azure.authorization-uri=https://voicecardb2c.b2clogin.com/voicecardb2c.onmicrosoft.com/B2C_1_signup_signin/oauth2/v2.0/authorize
spring.security.oauth2.client.provider.azure.token-uri=https://voicecardb2c.b2clogin.com/voicecardb2c.onmicrosoft.com/B2C_1_signup_signin/oauth2/v2.0/token
spring.security.oauth2.client.provider.azure.user-info-uri=https://graph.microsoft.com/oidc/userinfo
spring.security.oauth2.client.provider.azure.user-name-attribute=preferred_username
```

Replace placeholders like `your-postgres-password`, `your-speech-key`, `your-instrumentation-key`, `your-adb2c-client-id`, `your-adb2c-client-secret`, and `your-adb2c-tenant-id` with the actual values obtained from Azure CLI commands in Step 2.

### 3.5 `VoiceCardAiApplication.java`
The main application class to start the Spring Boot application:

In `src/main/java/com/voicecard/VoiceCardAiApplication.java`:

```java
package com.voicecard;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class VoiceCardAiApplication {
    public static void main(String[] args) {
        SpringApplication.run(VoiceCardAiApplication.class, args);
    }
}
```

### 3.6 `Application.java` (Model)
The data model for storing application details:

In `src/main/java/com/voicecard/model/Application.java`:

```java
package com.voicecard.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Application {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String phone;
    private double income;
    private String decision;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getPhone() { return phone; }
    public void setPhone(String phone) { this.phone = phone; }
    public double getIncome() { return income; }
    public void setIncome(double income) { this.income = income; }
    public String getDecision() { return decision; }
    public void setDecision(String decision) { this.decision = decision; }
}
```

### 3.7 `ApplicationRepository.java`
The JPA repository for database operations:

In `src/main/java/com/voicecard/repository/ApplicationRepository.java`:

```java
package com.voicecard.repository;

import com.voicecard.model.Application;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ApplicationRepository extends JpaRepository<Application, Long> {
}
```

### 3.8 `SpeechService.java`
The service to handle speech-to-text and text-to-speech:

In `src/main/java/com/voicecard/service/SpeechService.java`:

```java
package com.voicecard.service;

import com.microsoft.cognitiveservices.speech.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

@Service
public class SpeechService {
    @Value("${azure.speech.key}")
    private String speechKey;

    @Value("${azure.speech.region}")
    private String speechRegion;

    public String recognizeSpeech() throws Exception {
        SpeechConfig config = SpeechConfig.fromSubscription(speechKey, speechRegion);
        try (SpeechRecognizer recognizer = new SpeechRecognizer(config)) {
            SpeechRecognitionResult result = recognizer.recognizeOnceAsync().get();
            return result.getText().trim();
        }
    }

    public void synthesizeSpeech(String text) throws Exception {
        SpeechConfig config = SpeechConfig.fromSubscription(speechKey, speechRegion);
        try (SpeechSynthesizer synthesizer = new SpeechSynthesizer(config)) {
            synthesizer.SpeakTextAsync(text).get();
        }
    }
}
```

### 3.9 `MockCallingService.java`
The mocked calling service to simulate a phone call by logging interactions in the console:

In `src/main/java/com/voicecard/service/MockCallingService.java`:

```java
package com.voicecard.service;

import org.springframework.stereotype.Service;

import java.util.concurrent.atomic.AtomicBoolean;

@Service
public class MockCallingService {
    private final AtomicBoolean isCallActive = new AtomicBoolean(false);

    public void initializeCallAgent() {
        System.out.println("[MockCallingService] Simulating initialization of call agent...");
    }

    public void simulateIncomingCall() {
        if (isCallActive.get()) {
            System.out.println("[MockCallingService] Another call is already active. Rejecting new call.");
            return;
        }

        System.out.println("[MockCallingService] Simulating incoming call from +18001234567...");
        isCallActive.set(true);
        System.out.println("[MockCallingService] Call accepted. Call is now active.");
    }

    public boolean isCallActive() {
        return isCallActive.get();
    }

    public void hangUp() {
        if (isCallActive.get()) {
            System.out.println("[MockCallingService] Simulating call hang-up...");
            isCallActive.set(false);
        } else {
            System.out.println("[MockCallingService] No active call to hang up.");
        }
    }
}
```

**Notes:**
- `MockCallingService` replaces the real `CallingService` that used ACS.
- `simulateIncomingCall()` mimics an incoming call by logging messages to the console.
- `isCallActive()` and `hangUp()` simulate the state management of a call.

### 3.10 `QuotationService.java`
The service to generate a quotation ID by calling the mock Quotation API:

In `src/main/java/com/voicecard/service/QuotationService.java`:

```java
package com.voicecard.service;

import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;
import org.json.JSONObject;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.UUID;

@Service
public class QuotationService {
    @Value("${creditcard.api.url}")
    private String quotationApiUrl;

    public String generateQuotationId(String name, String phone, double income) throws Exception {
        try (var client = HttpClients.createDefault()) {
            var post = new HttpPost(quotationApiUrl);
            var json = new JSONObject();
            json.put("name", name);
            json.put("phone", phone);
            json.put("income", income);
            post.setEntity(new StringEntity(json.toString()));
            post.setHeader("Content-Type", "application/json");
            var response = EntityUtils.toString(client.execute(post).getEntity());
            var responseJson = new JSONObject(response);
            // Mock a quotation ID
            String quotationId = "QUOT-" + UUID.randomUUID().toString().substring(0, 8);
            return quotationId;
        }
    }
}
```

### 3.11 `ApplicationService.java`
The service to manage the application process, using the mocked calling service:

In `src/main/java/com/voicecard/service/ApplicationService.java`:

```java
package com.voicecard.service;

import com.voicecard.model.Application;
import com.voicecard.repository.ApplicationRepository;
import com.microsoft.applicationinsights.TelemetryClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ApplicationService {
    private final SpeechService speechService;
    private final MockCallingService callingService;
    private final QuotationService quotationService;
    private final ApplicationRepository repository;
    @Autowired
    private TelemetryClient telemetryClient;

    public ApplicationService(SpeechService speechService, MockCallingService callingService,
                              QuotationService quotationService, ApplicationRepository repository) {
        this.speechService = speechService;
        this.callingService = callingService;
        this.quotationService = quotationService;
        this.repository = repository;
    }

    public void processApplication() throws Exception {
        // Ensure a call is active
        if (!callingService.isCallActive()) {
            throw new IllegalStateException("No active call to process.");
        }

        var app = new Application();

        // Step 1: Welcome the user and ask for their name
        speechService.synthesizeSpeech("Welcome to VoiceCard AI. This is an automated system to help you generate a credit card quotation ID. Please say your name.");
        var name = speechService.recognizeSpeech();
        app.setName(name);

        // Step 2: Ask for phone number
        speechService.synthesizeSpeech("Please say your phone number, one digit at a time, for example, 1 2 3 4 5 6 7 8 9 0.");
        var phone = speechService.recognizeSpeech();
        // Basic cleaning of the phone number (speech recognition may vary)
        phone = phone.replaceAll("[^0-9]", "");
        app.setPhone(phone);

        // Step 3: Ask for income
        speechService.synthesizeSpeech("Please say your annual income in dollars, for example, 50 thousand.");
        var incomeStr = speechService.recognizeSpeech();
        var income = parseIncome(incomeStr);
        app.setIncome(income);

        // Step 4: Generate quotation ID
        telemetryClient.trackEvent("Generating quotation ID for user: " + name);
        var quotationId = quotationService.generateQuotationId(name, phone, income);
        app.setDecision(quotationId);

        // Step 5: Save to database
        repository.save(app);

        // Step 6: Inform the user of the quotation ID
        speechService.synthesizeSpeech("Thank you, " + name + ". Your credit card quotation ID is " + quotationId + ". Please note this down.");

        // Step 7: Handle follow-up questions
        handleFollowUp();
    }

    private double parseIncome(String incomeStr) {
        // Handle common speech patterns, e.g., "50 thousand" or "50000"
        incomeStr = incomeStr.toLowerCase().replaceAll("[^0-9\\s]", "");
        if (incomeStr.contains("thousand")) {
            var number = Double.parseDouble(incomeStr.replace("thousand", "").trim());
            return number * 1000;
        }
        return Double.parseDouble(incomeStr.trim());
    }

    private void handleFollowUp() throws Exception {
        speechService.synthesizeSpeech("Do you have any follow-up questions about your quotation? Please say yes or no.");
        var response = speechService.recognizeSpeech().toLowerCase();
        if (response.contains("yes")) {
            speechService.synthesizeSpeech("Please state your question.");
            var question = speechService.recognizeSpeech();
            telemetryClient.trackEvent("Follow-up question from user: " + question);
            speechService.synthesizeSpeech("Thank you for your question: " + question + ". A customer care representative will follow up with you soon.");
        }
        speechService.synthesizeSpeech("Thank you for using VoiceCard AI. Goodbye!");
        callingService.hangUp();
    }
}
```

### 3.12 `KeyVaultService.java`
The service to retrieve secrets from Azure Key Vault:

In `src/main/java/com/voicecard/service/KeyVaultService.java`:

```java
package com.voicecard.service;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

@Service
public class KeyVaultService {
    private final SecretClient secretClient;

    public KeyVaultService(@Value("${azure.keyvault.uri}") String keyVaultUri) {
        this.secretClient = new SecretClientBuilder()
                .vaultUrl(keyVaultUri)
                .credential(new DefaultAzureCredentialBuilder().build())
                .buildClient();
    }

    public String getSecret(String secretName) {
        return secretClient.getSecret(secretName).getValue();
    }
}
```

### 3.13 `DemoController.java`
A controller to trigger the demo by simulating an incoming call via a web endpoint:

In `src/main/java/com/voicecard/controller/DemoController.java`:

```java
package com.voicecard.controller;

import com.voicecard.service.ApplicationService;
import com.voicecard.service.MockCallingService;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class DemoController {
    private final MockCallingService callingService;
    private final ApplicationService applicationService;

    public DemoController(MockCallingService callingService, ApplicationService applicationService) {
        this.callingService = callingService;
        this.applicationService = applicationService;
    }

    @GetMapping("/demo")
    public String startDemo(Model model) throws Exception {
        // Initialize the mock call agent
        callingService.initializeCallAgent();

        // Simulate an incoming call
        callingService.simulateIncomingCall();

        // Process the application (voice interaction will happen here)
        applicationService.processApplication();

        model.addAttribute("message", "Demo completed. Check the console for simulated call logs.");
        return "demo-result";
    }
}
```

### 3.14 `AppInsightsConfig.java`
The configuration for Application Insights:

In `src/main/java/com/voicecard/config/AppInsightsConfig.java`:

```java
package com.voicecard.config;

import com.microsoft.applicationinsights.TelemetryClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppInsightsConfig {
    @Value("${applicationinsights.instrumentation-key}")
    private String instrumentationKey;

    @Bean
    public TelemetryClient telemetryClient() {
        TelemetryClient client = new TelemetryClient();
        client.getContext().setInstrumentationKey(instrumentationKey);
        return client;
    }
}
```

### 3.15 `SecurityConfig.java`
The security configuration for Azure AD B2C:

In `src/main/java/com/voicecard/config/SecurityConfig.java`:

```java
package com.voicecard.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/demo").authenticated()
                .anyRequest().permitAll()
            )
            .oauth2Login();
        return http.build();
    }
}
```

### 3.16 Create a Demo Result Page
Create a simple Thymeleaf template to display the demo result:

In `src/main/resources/templates/demo-result.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>VoiceCard AI - Demo Result</title>
</head>
<body>
    <h1>VoiceCard AI Demo Result</h1>
    <p th:text="${message}"></p>
    <a href="/demo">Run Another Demo</a>
</body>
</html>
```

### 3.17 Create the Mock API
The mock Quotation API using Node.js:

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

To set up the mock API:
1. Create a directory: `mock-creditcard-api`.
2. Initialize a Node.js project:
   ```bash
   cd mock-creditcard-api
   npm init -y
   npm install express
   ```
3. Save the above code as `server.js`.

---

## Step 4: Test the Application for the Hackathon Demo

### 4.1 Start the Mock API
```bash
cd mock-creditcard-api
node server.js
```

### 4.2 Start the Spring Boot Application
```bash
cd voicecard-ai
mvn spring-boot:run
```

### 4.3 Run the Demo
1. Open a browser and go to `http://localhost:8080/demo`.
2. Log in via Azure AD B2C (if prompted).
3. The demo will start:
   - The `MockCallingService` will log messages to the console, simulating an incoming call:
     ```
     [MockCallingService] Simulating initialization of call agent...
     [MockCallingService] Simulating incoming call from +18001234567...
     [MockCallingService] Call accepted. Call is now active.
     ```
   - The application will use the Speech Service to speak prompts (e.g., "Welcome to VoiceCard AI... Please say your name.") and listen for your responses.
   - Respond via voice to provide your name, phone number, and income.
   - The application will generate a quotation ID, save it to the database, and ask for follow-up questions.
   - The simulated call will end with a console message:
     ```
     [MockCallingService] Simulating call hang-up...
     ```
4. The browser will display a result page: "Demo completed. Check the console for simulated call logs."

### 4.4 Verify Database and Monitoring
- Check the database:
  ```bash
  psql -U postgres -d creditcard_db
  SELECT * FROM application;
  ```
- Check Application Insights in the Azure Portal for logs (e.g., telemetry events for quotation generation and follow-up questions).

---

## Step 5: Prepare for Hackathon Presentation

### 5.1 Demonstrate the Flow
- Show the browser at `http://localhost:8080/demo` to start the demo.
- Use a microphone to speak your responses (name, phone number, income, follow-up questions).
- Point out the console logs to show the simulated call flow (e.g., "Simulating incoming call from +18001234567...").
- Highlight the voice prompts and responses, showing how the system is accessible for visually challenged users.
- Display the database entries to confirm the quotation ID was saved.

### 5.2 Explain the Mocked Calling
- Mention that real-time calling with ACS was mocked due to subscription constraints.
- Explain that `MockCallingService` simulates the call by logging messages, but the voice interaction (via Speech Service) and backend logic (quotation generation, database storage) are fully functional.
- If the hackathon judges ask about real calling, clarify that ACS can be integrated with a subscription, as shown in previous solutions, but was mocked for the demo.

---

## Accessibility Considerations

- **Voice-Only Interaction:** The entire process is voice-driven, with clear, slow prompts to assist visually challenged users.
- **Error Handling:** The system includes basic input parsing (e.g., for phone numbers and income) to handle speech recognition errors.
- **Follow-Up Support:** Users can ask follow-up questions, which are logged for later follow-up by a representative.

---

## Summary

- Built a voice-driven application for visually challenged users to generate credit card quotation IDs, tailored for a hackathon demo.
- Mocked Azure Communication Services calling functionality using `MockCallingService` to simulate phone calls via console logs.
- Integrated Azure Speech Services for real-time voice interaction.
- Generated quotation IDs via a mock API.
- Supported follow-up questions via voice, with responses logged for later action.
- Ensured accessibility with clear, voice-only prompts and error handling.

This solution is now fully tailored for a hackathon demo, running on Java 21, with mocked calling functionality to simulate real-time interaction.
```

---

### **Key Changes for Mocked Calling**

- **Removed ACS Dependency:** The `pom.xml` no longer includes ACS SDK dependencies, and `application.properties` omits ACS-related configurations.
- **MockCallingService:** A new service (`MockCallingService.java`) simulates the call by logging messages to the console, mimicking the behavior of a real call.
- **DemoController:** A web endpoint (`/demo`) triggers the demo, simulating an incoming call and guiding the user through the process.
- **Console Logs:** The simulated call flow (e.g., "Simulating incoming call from +18001234567...") is logged to the console for the hackathon judges to see.

This solution ensures you can present a fully functional demo without needing an ACS subscription, while still showcasing the core functionality and accessibility features of VoiceCard AI.