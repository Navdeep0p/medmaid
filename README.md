```mermaid
flowchart TB
    %% --- MINIMAL PROFESSIONAL STYLES ---
    classDef default fill:#ffffff,stroke:#64748b,stroke-width:1px,color:#1e293b;
    classDef hardware fill:#f8fafc,stroke:#94a3b8,stroke-width:1px,color:#1e293b;
    classDef process fill:#ffffff,stroke:#475569,stroke-width:1px,color:#1e293b;
    classDef gate fill:#fffbeb,stroke:#d97706,stroke-width:1px,color:#1e293b;
    classDef pass fill:#f0fdf4,stroke:#16a34a,stroke-width:1px,color:#1e293b;
    classDef fail fill:#fef2f2,stroke:#dc2626,stroke-width:1px,color:#1e293b;
    classDef db fill:#f0f9ff,stroke:#0284c7,stroke-width:1px,color:#1e293b;
    classDef container fill:none,stroke:#cbd5e1,stroke-width:2px,stroke-dasharray: 5 5;

    %% --- 1. PHYSICAL WORLD ---
    subgraph PhysicalWorld ["Physical Enforcement Site"]
        direction LR
        Scale["Non-Automatic Weighing Instrument<br/>(7-Segment LED Display)"]
        Weights["Reference Weights<br/>(M1 Class)"]
        Weights -.->|Applied to| Scale
    end

    %% --- 2. OFFLINE EDGE APPLICATION ---
    subgraph MobileEdge ["Vidhik Field Terminal (Offline PWA)"]
        direction TB

        CameraHardware["Mobile Camera API<br/>(W3C Media Streams)"]
        
        subgraph S1 ["1. Smart Setup"]
            direction TB
            Config["Instrument Specs<br/>(Class, Max/Min, e)"] --> TestPlan["Dynamic Test Ladder"]
        end

        subgraph S2 ["2 & 3. Edge Perception Engine (<6 MB)"]
            direction TB
            AntiFlicker["Anti-Flicker Hardware Driver<br/>(50Hz/60Hz Lock)"] --> Burst["3-Frame Micro-Burst Selector"]
            Burst --> TFLite["On-Device OCR<br/>(Extracts 7-Segment Digits)"]
        end

        subgraph S3 ["4. Dual-Reconciliation Gate"]
            direction TB
            OCRVal["AI Detected Value"]
            ManVal["Inspector Manual Input"]
            Compare{"Exact Match?"}
            Lock["Cryptographic Lock<br/>(GPS + ISO Timestamp)"]
            Reject["Alert: Resync & Retake"]
            
            OCRVal --> Compare
            ManVal --> Compare
            Compare -- "Mismatch" --> Reject
            Compare -- "Match" --> Lock
        end

        subgraph S4 ["5. Legal Metrology Core"]
            direction TB
            Formula["Deterministic R-76 Equation:<br/>E = I + 0.5e - ΔL - L"]
            MPE["MPE Bracket Verification"]
            Verdict{"Verdict Check"}
            PassResult["PASS<br/>(Within Limits)"]
            FailResult["FAIL<br/>(Exceeds Limits)"]

            Formula --> MPE --> Verdict
            Verdict --> PassResult
            Verdict --> FailResult
        end

        subgraph S5 ["6. Statutory Artifact"]
            direction TB
            PDF["Compile Official Certificate"]
            Hash["Generate SHA-256 Digest<br/>(Section 65B Standard)"]
            QR["Embed Cryptographic QR Stamp"]
            
            PDF --> Hash --> QR
        end

        IndexedDB[("IndexedDB<br/>(Offline Sync Queue)")]

        %% Edge Logic Wiring (Strict Top-to-Bottom to prevent arrow overlaps)
        Scale -->|Optical Read| CameraHardware
        CameraHardware --> AntiFlicker
        TestPlan -.-> CameraHardware
        
        TFLite --> OCRVal
        Lock --> Formula
        PassResult --> PDF
        FailResult --> PDF
        QR --> IndexedDB
    end

    %% --- 3. CENTRAL GOVERNMENT CLOUD ---
    subgraph GovCloud ["DoCA Central Metrology Portal"]
        direction TB
        SyncAPI["Sync Microservice"]
        CentralDB[("National Ledger<br/>(PostgreSQL)")]
        StateDash["State Directorate Dashboard"]
        FraudAlerts["Enforcement Trigger<br/>(GPS/Time Anomalies)"]
        
        SyncAPI --> CentralDB
        CentralDB --> StateDash
        CentralDB --> FraudAlerts
    end

    %% Connecting Edge to Cloud
    IndexedDB == "Auto-Sync (When Online)" ==> SyncAPI

    %% Applying Classes
    class PhysicalWorld,MobileEdge,GovCloud container;
    class Scale,Weights hardware;
    class Config,TestPlan,AntiFlicker,Burst,TFLite,OCRVal,ManVal,Formula,MPE,PDF,Hash,QR,CameraHardware process;
    class Compare,Verdict gate;
    class PassResult,Lock pass;
    class Reject,FailResult fail;
    class IndexedDB,CentralDB,SyncAPI,StateDash,FraudAlerts db;
