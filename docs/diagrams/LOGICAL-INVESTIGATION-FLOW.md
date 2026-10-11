# Logical incident investigation flow — proposed MVP

> **On-demand incident analysis**, not continuous live ingestion or automatic remediation. All incidents in v1 are synthetic examples.

```mermaid
flowchart TD
    START(["Analyst opens web dashboard"])
    LIST["Browse / filter sample incidents"]
    SELECT["Select an incident and request investigation"]
    AUTH{"Request authenticated<br/>and authorized?"}
    REJECT["Return 401/403<br/>No model invocation"]
    LOOKUP{"Incident ID exists<br/>in synthetic dataset?"}
    NOTFOUND["Return 404 / safe error"]
    VALIDATE["Validate schema and size<br/>normalize data; minimize/redact"]
    LIMIT{"Within request rate,<br/>token and usage limits?"}
    BLOCK["Reject with limit message<br/>No paid model request"]
    PROMPT["Compose controlled investigation prompt<br/>separate untrusted incident text"]
    CALL["Send HTTPS request to approved<br/>Microsoft Foundry model deployment"]
    RESULT{"Model response successful<br/>and schema-valid?"}
    ERROR["Handle timeout, quota, 429,<br/>invalid output; no fabricated finding"]
    CHECK["Map claims to available fields<br/>tag hypotheses and uncertainty"]
    VIEW["Render evidence, observations,<br/>hypotheses and recommendations"]
    REVIEW["Human analyst validates next steps<br/>No automatic containment"]
    END(["Investigation complete"])

    START --> LIST --> SELECT --> AUTH
    AUTH -->|"No"| REJECT
    AUTH -->|"Yes"| LOOKUP
    LOOKUP -->|"No"| NOTFOUND
    LOOKUP -->|"Yes"| VALIDATE --> LIMIT
    LIMIT -->|"No"| BLOCK
    LIMIT -->|"Yes"| PROMPT --> CALL --> RESULT
    RESULT -->|"No"| ERROR
    RESULT -->|"Yes"| CHECK --> VIEW --> REVIEW --> END

    classDef control fill:#edf5ff,stroke:#3376bb,color:#123a65;
    classDef error fill:#fff1ee,stroke:#c25f4e,color:#773326;
    class AUTH,LOOKUP,LIMIT,RESULT control;
    class REJECT,NOTFOUND,BLOCK,ERROR error;
```

## Logic and evidence handling

| Stage | Intended behavior |
|---|---|
| View | Show sample data clearly marked synthetic, not real SOC detections |
| Authentication | Deny unauthenticated/unauthorized access to incident endpoints |
| Input | Use canonical server-held incident data, not blindly trusted browser-supplied incident text |
| AI prompt | Treat incident details and observables as untrusted input, never executable instructions |
| Model request | Bound timeouts, response tokens, request frequency and daily call allowance |
| Model response | Validate a strict output schema; never present hallucinated evidence as observed telemetry |
| Evidence | Distinguish input-supported observations from generated hypotheses and suggestions |
| Response | Return a consistent API shape and safe error messages |
| Analyst | Human makes investigation/containment decisions; platform does not act autonomously |

The project title uses **Real-Time** to indicate eventual on-demand model responses when a user submits an investigation. The first release does not subscribe to live Sentinel signals and does not guarantee instantaneous results.

## Future integration (not MVP)

A read-only Microsoft Sentinel connector can be added later, guarded by separate authorization, query constraints, paging, data protection and audit controls. It is not represented as an existing data source in this diagram.
