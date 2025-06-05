graph TD
    subgraph Users
        A[Customer: Phone] -->|Inbound Call| B[Azure Communication Services]
        C[Customer: Alexa Device] -->|Voice Command| D[Alexa Skill]
    end

    subgraph Azure Infrastructure
        B -->|Call Automation| E[Azure Bot Service]
        D -->|Skill Invocation| E
        E -->|Bot Logic| F[Node.js App Service]
        F -->|Speech-to-Text/Text-to-Speech| G[Azure Speech Services]
        F -->|API Call| H[Quotation Management API<br>or Mock Response]
    end

    subgraph Data Flow
        G -->|Processed Input| F
        H -->|Status: Accept/Decline/Refer| F
        F -->|Response| E
        E -->|Text-to-Speech via ACS| B
        E -->|Text-to-Speech via Alexa| D
        B -->|Voice Response| A
        D -->|Voice Response| C
    end

    classDef azure fill:#0078D4,stroke:#005BA1,color:#fff;
    class B,E,G azure;
    classDef external fill:#D3D3D3,stroke:#333,color:#000;
    class H external;
    classDef user fill:#90EE90,stroke:#006400,color:#000;
    class A,C user;
    classDef alexa fill:#00C4FF,stroke:#0078D4,color:#000;
    class D alexa;
    classDef app fill:#FFD700,stroke:#B8860B,color:#000;
    class F app;