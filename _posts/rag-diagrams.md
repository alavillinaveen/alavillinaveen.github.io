
## Diagram-Heavy Reference Architecture

---

## 1) System Context Diagram

```mermaid
flowchart LR
  U[User] -->|fills form| A[Angular Web App]
  A -->|REST/JSON| API[ASP.NET Core API]
  A -->|Assistive guidance request| RAG[RAG Gateway API]
  RAG --> VDB[(Vector DB / Search Index)]
  RAG --> LLM[(LLM Provider)]
  API --> DB[(Transactional DB)]
  API --> Cache[(Cache)]
```

**Intent:** RAG is **parallel** to your transactional API path.
Your form still works without RAG.

---

## 2) Request Flow: Deterministic Data vs RAG Guidance

```mermaid
sequenceDiagram
  autonumber
  participant UI as Angular UI (Reactive Forms)
  participant API as ASP.NET Core API (Domain)
  participant RAG as RAG Gateway (.NET Minimal API)
  participant V as Vector Search
  participant L as LLM
  participant DB as DB

  UI->>API: GET /api/locations/states?countryId=US
  API->>DB: Query states by countryId
  DB-->>API: States
  API-->>UI: 200 States JSON
  Note over UI: Deterministic, cacheable, testable

  UI->>RAG: POST /rag/guidance (masked form context)
  RAG->>V: Similarity search: "US address format + state selection issues"
  V-->>RAG: Top K docs/snippets
  RAG->>L: Prompt + retrieved context
  L-->>RAG: Guidance text + citations
  RAG-->>UI: 200 Guidance payload
  Note over UI: Non-blocking assistive panel
```

---

## 3) Component Architecture: Clean Boundaries

```mermaid
flowchart TB
  subgraph Frontend[Angular Frontend]
    RF[Reactive Forms]
    VLD[Client Validation]
    UX[Assistive Panel / Tooltip UI]
    S1[LookupDataService]
    S2[RagGuidanceService]
    RF --> VLD
    RF --> S1
    RF --> S2
    S2 --> UX
  end

  subgraph Backend[.NET Backend]
    API[ASP.NET Core API]
    DOM[Domain Services]
    VAL[Business Validation]
    AUTH[AuthZ/AuthN]
    API --> AUTH --> DOM --> VAL
  end

  subgraph RagSystem[RAG System]
    RG[RAG Gateway API]
    RET[Retriever]
    GEN[Generator]
    OBS[Telemetry + Guardrails]
    RG --> RET --> VDB[(Vector DB)]
    RG --> GEN --> LLM[(LLM)]
    RG --> OBS
  end

  S1 --> API
  S2 --> RG
  API --> DB[(Transactional DB)]
```

**Key rule:** RAG stays out of domain validation and persistence.

---

## 4) Data Classification & PII Guardrail Flow

```mermaid
flowchart LR
  UI[Angular Form Context] --> MASK[Mask/Redact PII]
  MASK --> RG[RAG Gateway]
  RG --> POL[Policy Filter: allowlisted fields only]
  POL --> RET[Retrieve Docs]
  RET --> GEN[Generate Guidance]
  GEN --> OUT[Guidance Response]

  RG --> LOG[Audit Log: requestId, docIds, latency]
  GEN --> SAFE[Safety Filter: no PII echoing, no secrets]
  SAFE --> OUT
```

**Implementation note (architectural):**

* Do not send raw first/last name, full address, phone, SSN, DOB to RAG unless you have a compelling reason and appropriate governance.
* Prefer **field-level summarization** (“Country=US, State empty, City filled, Zip invalid”) vs raw values.

---

## 5) Deployment View (Typical Enterprise)

```mermaid
flowchart TB
  subgraph Client
    B[Browser]
  end

  subgraph Cloud[Cloud / Data Center]
    CDN[CDN / Front Door] --> SPA[Angular App Host]
    CDN --> APIGW[API Gateway]
    APIGW --> API[ASP.NET Core API]
    APIGW --> RG[RAG Gateway API]

    API --> DB[(SQL DB)]
    API --> REDIS[(Redis Cache)]

    RG --> VDB[(Vector Search)]
    RG --> LLM[(LLM Provider)]
    RG --> TEL[(App Insights / OpenTelemetry)]
  end

  B --> CDN
```

---

## 6) “Golden Path” Use Case: Inline Explanation for Rejections

Use case: Backend rejects submit; RAG explains **why** in human terms.

```mermaid
sequenceDiagram
  autonumber
  participant UI as Angular UI
  participant API as ASP.NET Core API
  participant RAG as RAG Gateway
  participant V as Vector DB
  participant L as LLM

  UI->>API: POST /api/profile (form payload)
  API-->>UI: 409 VALIDATION_ERROR { code: "ADDR_POLICY_12" }

  UI->>RAG: POST /rag/explain-error { code, maskedContext }
  RAG->>V: Retrieve doc snippets for code ADDR_POLICY_12
  V-->>RAG: Policy excerpt + examples
  RAG->>L: Generate explanation + actionable steps
  L-->>RAG: "Address line 2 required for X; examples..."
  RAG-->>UI: Guidance response with doc references

  Note over UI: UI shows helpful guidance instead of cryptic error codes.
```

---

## 7) RAG Decision Matrix as a Flowchart

```mermaid
flowchart TD
  Q1{Is output deterministic\nand schema-driven?} -->|Yes| A1[Use API + Domain Rules]
  Q1 -->|No| Q2{Does user input\ninclude free-text or ambiguity?}
  Q2 -->|No| A2[Use config/rules engine]
  Q2 -->|Yes| Q3{Do you have a\ntrusted knowledge base?}
  Q3 -->|No| A3[Build KB first\nor avoid RAG]
  Q3 -->|Yes| Q4{Can RAG be\nassistive only?}
  Q4 -->|No| A4[Do not use RAG\nin decision path]
  Q4 -->|Yes| A5[Use RAG for\nexplain/guide/suggest]
```

---

## 8) Observability & Governance Diagram (Non-Negotiable)

```mermaid
flowchart LR
  RG[RAG Gateway] --> MET[Metrics: latency, tokens, error rate]
  RG --> TRC[Tracing: correlationId across UI/API/RAG]
  RG --> AUD[Audit: docIds used, prompt template version]
  RG --> COST[Cost: token usage, request volume]
  RG --> QA[Quality: thumbs up/down, deflection rate]
```

**This is how you keep RAG from becoming a black box.**

---

# Recommended “Architect’s Rules” (Short and Sharp)

1. **RAG is not authoritative.** Domain logic stays in ASP.NET Core services.
2. **RAG is optional.** Form UX must function if RAG is down.
3. **RAG is read-only.** No writes, no decisions, no validation outcomes.
4. **PII is minimized.** Redact/mask before retrieval/generation.
5. **Everything is observable.** Correlation IDs + doc IDs + prompt version.
