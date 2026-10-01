```mermaid
flowchart TB
    %% --- GLOBAL STYLES ---
    classDef default fill:#ffffff,stroke:#64748b,stroke-width:1px,color:#1e293b;
    classDef hardware fill:#f8fafc,stroke:#94a3b8,stroke-width:1.5px,color:#0f172a;
    classDef edgeApp fill:#f0f9ff,stroke:#0284c7,stroke-width:2px,color:#0c4a6e;
    classDef vision fill:#f5f3ff,stroke:#7c3aed,stroke-width:1.5px,color:#4c1d95;
    classDef mathCore fill:#ecfdf5,stroke:#059669,stroke-width:2px,color:#064e3b;
    classDef cloud fill:#f8fafc,stroke:#475569,stroke-width:2px,color:#0f172a;
    classDef db fill:#e0f2fe,stroke:#0369a1,stroke-width:1.5px,color:#0c4a6e;
    classDef container fill:none,stroke:#cbd5e1,stroke-width:2px,stroke-dasharray: 5 5;

    %% ==========================================
    %% 1. PHYSICAL LABORATORY LAYER
    %% ==========================================
    subgraph Physical ["1. Physical Testing Environment"]
        direction LR
        Scale["NAWI Prototype<br/>(Class I-IIII)"]
        Weights["Reference Standard Weights<br/>(e.g., M1, F1 Class)"]
        Sensors["Ambient Lab Conditions<br/>(Temp, Humidity, Pressure)"]
        
        Weights -.->|Applied to| Scale
    end

    %% ==========================================
    %% 2. OFFLINE EDGE PLATFORM (PWA)
    %% ==========================================
    subgraph EdgePlatform ["2. Vidhik Testing Workstation (Offline-First PWA)"]
        direction TB

        %% Module A: Setup & Test Generation
        subgraph ModSetup ["A. Configuration & Test Engine"]
            direction TB
            Profile["Instrument Profiling<br/>(Max, Min, e, d)"]
            Ladder["Dynamic Test Ladder<br/>(Generates Linearity, Eccentricity,<br/>Repeatability Load Points)"]
            Profile --> Ladder
        end

        %% Module B: Hardware-Aware Vision
        subgraph ModVision ["B. Bench Vision Assistant"]
            direction TB
            CamAPI["W3C Media Streams API"]
            FlickerSync["50Hz/60Hz Mains Exposure Lock<br/>(Eliminates LED Banding)"]
            Burst["3-Frame Micro-Burst Selector"]
            EdgeOCR["On-Device Inference (<6MB)<br/>(YOLOv8-Nano / CRNN)"]
            
            CamAPI --> FlickerSync --> Burst --> EdgeOCR
        end

        %% Module C: Validation Gate
        subgraph ModGate ["C. Data Ingestion & Validation"]
            direction LR
            Manual["Manual Worksheet Entry"]
            Gate{"Transcription<br/>Reconciliation Gate"}
            Sanity["Data Monotonicity &<br/>Completeness Checks"]
            
            Manual --> Gate
            Gate --> Sanity
        end

        %% Module D: Deterministic Math Core
        subgraph ModMath ["D. Deterministic OIML Core"]
            direction TB
            Formula["True Error Engine:<br/>E = I + 0.5e - ΔL - L"]
            Drift["Zero-Load Drift Correction:<br/>Ec = E - E0"]
            MPE["Statutory MPE Evaluation<br/>(±0.5e, ±1.0e, ±1.5e)"]
            Verdict{"Compliance Verdict<br/>(Test-Wise & Overall)"}
            
            Formula --> Drift --> MPE --> Verdict
        end

        %% Module E: Artifact & Storage
        subgraph ModOutput ["E. Output & Local State"]
            direction TB
            PDF["OIML R 76-2:2007 Generator<br/>(PDF & Editable Layout)"]
            Crypto["BSA 2023 Digital Seal<br/>(SHA-256 Cryptographic Hash)"]
            IndexedDB[("IndexedDB Queue<br/>(Local Store-and-Forward)")]
            
            PDF --> Crypto --> IndexedDB
        end

        %% Internal Edge Wiring
        Ladder -.->|Prompts loads| ModVision
        Ladder -.->|Prompts loads| Manual
        EdgeOCR --> Gate
        Sanity -- "Validated Loads" --> Formula
        Verdict --> PDF
    end

    %% ==========================================
    %% 3. CENTRAL GOV CLOUD LAYER
    %% ==========================================
    subgraph GovCloud ["3. Central Regulatory Portal (GovCloud)"]
        direction TB
        
        SyncAPI["Background Sync Microservice<br/>(Handles offline queue flushes)"]
        
        subgraph DataLayer ["National Data Repositories"]
            direction LR
            RuleDB[("Versioned Schema Store<br/>(Decoupled OIML JSON Rules)")]
            LedgerDB[("National Test Ledger<br/>(PostgreSQL)")]
        end

        subgraph RBAC ["4-Tier Role-Based Access Control (RBAC)"]
            direction LR
            RoleReg["Central Regulators<br/>(DoCA Admins)"]
            RoleLab["Lab Metrologists<br/>(Maker/Checker)"]
            RoleOEM["Commercial Applicants<br/>(Manufacturers)"]
            RolePub["Public Portals<br/>(Verification Search)"]
        end

        SyncAPI --> LedgerDB
        RuleDB -.->|Pushes schema updates| SyncAPI
        LedgerDB --> RBAC
    end

    %% ==========================================
    %% CROSS-LAYER CONNECTIONS
    %% ==========================================
    Scale -.->|Optical feed| CamAPI
    Sensors -.->|Logged in| Profile
    IndexedDB == "Opportunistic Auto-Sync<br/>(Upon Network Reconnection)" ==> SyncAPI

    %% ==========================================
    %% CLASS ASSIGNMENTS
    %% ==========================================
    class Physical,EdgePlatform,GovCloud container;
    class Scale,Weights,Sensors hardware;
    class Profile,Ladder,CamAPI,Manual,Sanity,PDF,Crypto,SyncAPI edgeApp;
    class FlickerSync,Burst,EdgeOCR vision;
    class Formula,Drift,MPE mathCore;
    class Gate,Verdict gate;
    class IndexedDB,RuleDB,LedgerDB db;
    class DataLayer,RBAC,RoleReg,RoleLab,RoleOEM,RolePub cloud;
