# Workflow Diagram

Before / after process flow for the consolidated Enterprise IT Helpdesk Automation Workbench.

---

## Before — request handling by memory

```mermaid
flowchart LR
    A[Inbound request<br>Outlook · EIP2 · Excel · HR] --> B[Operator mental model]
    B --> C1[Manual Outlook draft]
    B --> C2[Manual EIP2 form sign-off]
    B --> C3[Manual Ivanti admin click-through]
    B --> C4[Manual Cerberus FTP edit]
    C1 --> D[Inconsistent result]
    C2 --> D
    C3 --> D
    C4 --> D
    D --> E[No audit trail · no readback · no dedup]

    classDef bad fill:#f4e8ed,stroke:#8a4d63,color:#351620;
    class B,D,E bad;
```

---

## After — batch-first workbench

```mermaid
flowchart TB
    In[Inbound sources<br>Outlook · EIP2 · Excel · Ivanti · Cerberus]

    subgraph N[Normalize]
      N1[Uniform row shape]
    end

    subgraph M[Merge]
      M1[Dedupe by account identity]
      M2[Merge-reason string]
    end

    subgraph P[Plan]
      P1[CREATE]
      P2[REVIEW]
      P3[SKIP]
    end

    subgraph X[Execute]
      X1[Outlook COM]
      X2[EIP2 Playwright]
      X3[Ivanti API]
      X4[Cerberus CSV]
      X5[SSL-VPN admin]
    end

    subgraph V[Verify]
      V1[Readback]
      V2[JSONL telemetry]
      V3[SQLite idempotency]
    end

    Sum[Execution summary<br>create=N review=M skip=K]

    In --> N1 --> M1 --> M2 --> P
    P1 --> X1
    P1 --> X2
    P1 --> X3
    P1 --> X4
    P1 --> X5
    X1 --> V1
    X2 --> V1
    X3 --> V1
    X4 --> V1
    X5 --> V1
    V1 --> V2
    V1 --> V3
    V2 --> Sum
    V3 --> Sum
    P2 --> Sum
    P3 --> Sum

    classDef good fill:#e9f1ea,stroke:#4b6a50,color:#1d2b1f;
    class N1,M1,M2,P1,P2,P3,V1,V2,V3,Sum good;
```

---

## AI assist layer (sits beside intake, never decides external writes)

```mermaid
flowchart LR
    Bounce[SSL-VPN bounce email] --> FI[Firewall-immunity regex]
    FI -->|pass| KEEP[KEEP]
    FI -->|no-match| FS[OpenAI few-shot<br>vs golden samples]
    FS --> DEL[DELETE]
    FS --> REV[REVIEW]
    FS --> KP2[KEEP]

    Cert[Training-certificate image] --> OCR[Tesseract + OpenCV]
    OCR --> FZ[Fuzzy match vs course catalog<br>thresholds 0.78 / 0.85 / 0.95]
    FZ --> Roster[Excel roster row]

    UserQ[User helpdesk question] --> RAG[Claude + chromadb<br>over internal KB]
    RAG --> Ans[Answer or escalate to ticket]

    classDef assist fill:#faf2dc,stroke:#a07f27,color:#3a2d0a;
    class FI,FS,OCR,FZ,RAG assist;
```

**Boundary rule.** The AI layer outputs a *label* (`DELETE / REVIEW / KEEP`), a *roster row*, or an *answer string*. It never calls a deterministic adapter. External writes only happen via the `Execute` block in the main workflow.

---

## Handoff path

```mermaid
flowchart LR
    Dev[Developer operator] -->|writes| Doc[README + brainstorm.md + plan.md]
    Dev -->|writes| Test[pytest suite]
    Dev -->|writes| Env[.env / config_local.py]
    Doc --> New[New operator]
    Test --> New
    Env --> New
    New -->|runs| Run[python -m demo_app --show-stages]
    Run -->|reads| Sum2[Execution summary]
    Sum2 --> New2[New operator is productive<br>no shadowing required]

    classDef doc fill:#e8eef7,stroke:#40618a,color:#13263f;
    class Doc,Test,Env doc;
```
