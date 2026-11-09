# InterviewX — All Final System Diagrams

**Status: Proposed final design; pending stakeholder sign-off.**

## D01 SYSTEM CONTEXT

```mermaid
flowchart LR
  Guest([Guest]) --> Web[InterviewX Responsive Web / PWA]
  User([Registered User]) --> Web
  Admin([System Admin]) --> Web
  Researcher([Researcher R3]) --> Web
  Finance([Finance Admin R3]) --> Web
  Web --> Platform[InterviewX Platform]
  Platform --> Email[Transactional email provider]
  Platform --> LLM[LLM provider]
  Platform -. "R2 STT" .-> STT[Speech-to-text provider]
  Platform -. "R3 payments" .-> Gateway[Payment gateway]
  Platform --> Telemetry[Monitoring and alerting]
  classDef optional stroke-dasharray:5 5
  class Researcher,Finance,STT,Gateway optional
```

## D02 C4 CONTAINERS

```mermaid
flowchart TB
  Browser[Next.js Web / PWA R1] --> Edge[CDN / WAF / TLS]
  Edge --> API[NestJS Core API / BFF R1]
  subgraph Core[NestJS Modular Monolith R1]
    ID[Identity + RBAC]
    Content[Content + Rubric]
    Interview[Interview]
    Eval[Evaluation Read Model]
    Privacy[Privacy + Governance]
    Platform[Platform / Outbox / Audit]
  end
  API --> ID & Content & Interview & Eval & Privacy & Platform
  ID & Content & Interview & Eval & Privacy & Platform --> PG[(Core PostgreSQL: schema per context)]
  Platform --> Redis[(Redis / BullMQ)]
  Redis --> Worker[Evaluation Worker R1]
  Worker --> AI[FastAPI Inference R1]
  AI --> Provider[Hosted LLM provider]
  Worker --> PG
  API -. R2 .-> Store[(Private Audio Object Storage)]
  API -. R3 authenticated API .-> Billing[Billing Service R3]
  API -. R3 authenticated API .-> Research[Research Release Service R3]
  Billing --> BillingDB[(Billing PostgreSQL)]
  Research --> ResearchDB[(Research PostgreSQL)]
  Billing --> Gateway[Payment provider]
  classDef future stroke-dasharray:5 5
  class Store,Billing,BillingDB,Research,ResearchDB,Gateway future
```

## D03 BOUNDED CONTEXTS

```mermaid
flowchart LR
 subgraph CORE[Core NestJS - single deployable]
  Access[Identity / Access]

  Interview[Interview]
  Content[Content]
  Eval[Evaluation]
  Privacy[Privacy]
  Plat[Platform: idempotency / outbox / audit]
 end
 Client[Public API / BFF] --> Access & Interview & Eval & Privacy & Content
 Interview --> Content
 Interview --> Access
 Eval --> Interview
 Privacy --> Access
 Interview --> Plat
 Eval --> Plat
 Plat --> Worker[Worker deployable]
 Worker --> AI[AI FastAPI - no DB connection]
 Client -. "R3 contract" .-> Billing[Billing - own DB]
 Client -. "R3 contract" .-> Research[Research - own DB]
 Billing -. versioned events .-> Plat
 Privacy -. approved release channel R3 .-> Research
 note["Cross-context access via exported application contracts; no repository imports across owners"]
 note -. rule .-> CORE
```

## D04 CORE ERD

```mermaid
erDiagram
 USERS ||--o{ AUTH_IDENTITIES : owns
 USERS ||--o{ REFRESH_SESSIONS : refreshes
 USERS ||--o{ USER_ROLES : granted
 ROLES ||--o{ USER_ROLES : assigns
 ROLES ||--o{ ROLE_PERMISSIONS : includes
 PERMISSIONS ||--o{ ROLE_PERMISSIONS : defines
 USERS ||--o{ INTERVIEW_SESSIONS : starts
 RUBRIC_VERSIONS ||--o{ INTERVIEW_SESSIONS : pins
 AI_CONFIG_VERSIONS ||--o{ INTERVIEW_SESSIONS : pins
 INTERVIEW_SESSIONS ||--o{ SESSION_QUESTIONS : contains
 QUESTION_BANK o|--o{ SESSION_QUESTIONS : sources
 SESSION_QUESTIONS ||--o{ ANSWERS : receives
 ANSWERS ||--o| EVALUATION_JOBS : triggers
 EVALUATION_JOBS ||--o| FEEDBACK : produces
 ANSWERS ||--o| FEEDBACK : scored
 RUBRIC_VERSIONS ||--o{ FEEDBACK : rubric
 AI_CONFIG_VERSIONS ||--o{ FEEDBACK : model
 FEEDBACK ||--o{ APPEALS : disputed
 USERS ||--o{ CONSENT_EVENTS : grants
 USERS ||--o{ PROCESSING_NOTICES : notified
 USERS ||--o{ DATA_RIGHTS_REQUESTS : requests
 USERS ||--o{ CONTACT_DETAILS : contact
 ANSWERS ||--o{ AUDIO_OBJECTS : audio_R2
 ANSWERS ||--o| SPEECH_METRICS : speech_R2
 USERS ||--o{ IDEMPOTENCY_KEYS : dedupes
 USERS { uuid id PK
   citext email UK
   timestamptz deleted_at }
 INTERVIEW_SESSIONS { uuid id PK
   uuid user_id FK
   uuid rubric_version_id FK
   uuid ai_config_version_id FK }
 SESSION_QUESTIONS { uuid id PK
   uuid session_id FK
   int sequence_no }
 ANSWERS { uuid id PK
   uuid question_id FK
   int attempt_number }
 EVALUATION_JOBS { uuid id PK
   uuid answer_id FK
   text state }
 FEEDBACK { uuid id PK
   uuid job_id FK
   uuid answer_id FK
   numeric score }
 CONSENT_EVENTS { uuid id PK
   uuid user_id FK
   text purpose_key
   text action }
 IDEMPOTENCY_KEYS { text scope PK
   uuid actor_id PK
   text key PK }
```

## D05 R3 ERD

```mermaid
erDiagram
 PACKAGES ||--o{ PACKAGE_ENTITLEMENTS : defines
 PACKAGES ||--o{ SUBSCRIPTIONS : offers
 SUBSCRIPTIONS ||--o{ TRANSACTIONS : billed
 TRANSACTIONS ||--o{ REFUND_REQUESTS : refund
 DATASETS ||--o{ AGGREGATE_ROWS : includes
 DATASETS ||--o{ ACCESS_LOG : records
 PACKAGES { uuid id PK
   text tier
   int version
   bigint price_minor }
 PACKAGE_ENTITLEMENTS { uuid package_id PK,FK
   text feature_key PK
   bigint quota_limit }
 SUBSCRIPTIONS { uuid id PK
   uuid user_id "external Core ID"
   uuid package_id FK }
 USAGE_LEDGER { uuid id PK
   uuid user_id "external Core ID"
   text request_key UK }
 TRANSACTIONS { uuid id PK
   uuid subscription_id FK
   bigint amount_minor }
 PAYMENT_EVENTS { uuid id PK
   text provider
   text external_event_id UK }
 REFUND_REQUESTS { uuid id PK
   uuid transaction_id FK
   uuid user_id }
 DATASETS { uuid id PK
   text purpose_key
   text state }
 AGGREGATE_ROWS { uuid id PK
   uuid dataset_id FK
   int distinct_users
   boolean suppressed }
 ACCESS_LOG { uuid id PK
   uuid dataset_id FK
   text decision }
```

## D06 API TRUST

```mermaid
flowchart LR
 subgraph Untrusted[Public / Untrusted]
  Web[Next.js browser client]
  Mobile[Mobile future optional]
 end
 subgraph Edge[Public edge]
  WAF[CDN / WAF / TLS]
  API[Core NestJS REST /v1]
 end
 subgraph Internal[Private network]
  Redis[Redis / BullMQ]
  Worker[Eval Worker]
  AI[FastAPI /internal/v1/evaluate]
  PG[(Core Postgres)]
 end
 Web --> WAF
 Mobile -. R2/R3 .-> WAF
 WAF --> API
 API --> PG
 API --> Redis
 Redis --> Worker
 Worker --> PG
 Worker -->|authenticated + timeouts| AI
 AI -->|provider adapter| LLM[External LLM]
 API -. R3 only /v1/billing .-> Billing[Billing service]
 API -. R3 only /v1/research .-> Research[Research service]
 classDef future stroke-dasharray:5 5
 class Mobile,Billing,Research future
```

## D07 TEXT INTERVIEW SEQUENCE

```mermaid
sequenceDiagram
 autonumber
 actor U as User
 participant W as Web
 participant A as NestJS Interview API
 participant D as Core PostgreSQL
 participant O as Outbox dispatcher
 participant Q as Redis/BullMQ
 participant K as Evaluation Worker
 participant F as FastAPI AI
 participant L as LLM Provider
 U->>W: Submit text answer
 W->>A: POST /v1/interviews/{id}/answers
Idempotency-Key
 A->>A: Auth + ownership + state + content checks
 A->>D: BEGIN; insert answer, job, outbox, idempotency
 D-->>A: COMMIT
 A-->>W: 202 Accepted: jobId
 O->>D: Read undispatched outbox
 O->>Q: Enqueue stable event ID
 Q-->>O: Accepted
 O->>D: Mark dispatched
 Q->>K: Deliver at least once
 K->>K: Inbox/job-state deduplication
 K->>F: POST /internal/v1/evaluate with versions
 F->>L: Restricted provider call
 L-->>F: Untrusted candidate
 F->>F: Validate rubric, schema and evidence
 F-->>K: Validated feedback / needs-review
 K->>D: Atomic job transition + feedback result
 W->>A: GET /v1/evaluations/{jobId}
 A-->>W: pending / completed / failed / needs-review
```

## D08 EVAL STATES

```mermaid
stateDiagram-v2
 [*] --> pending : answer committed
 pending --> processing : worker claims
 processing --> completed : valid checked result
 processing --> retry_wait : transient error
 retry_wait --> processing : retry <= max attempts
 processing --> needs_review : invalid / weak evidence
 processing --> failed : permanent error / max attempts
 pending --> canceled : user deletion workflow
 retry_wait --> canceled : user deletion workflow
 completed --> [*]
 needs_review --> [*]
 failed --> [*]
 canceled --> [*]
 note right of completed
  Only validated rubric score
  Persist model / prompt / rubric version
 end note
 note right of needs_review
  Never fabricate numeric score
 end note
```

## D09 IDENTITY AUTHORIZATION

```mermaid
flowchart TD
 Request[Incoming request] --> Auth{Authenticated?}
 Auth -->|No| Public{Public endpoint?}
 Public -->|Yes| Allow[Allow public action]
 Public -->|No| Deny[401 / deny]
 Auth -->|Yes| Active{Account active and token valid?}
 Active -->|No| Deny
 Active -->|Yes| Role{Has permission?}
 Role -->|No| Forbidden[403 / deny]
 Role -->|Yes| Own{Ownership or scoped admin grant?}
 Own -->|No| Forbidden
 Own -->|Yes| Scope{Purpose and lawful basis / consent required?}
 Scope -->|Missing| Forbidden
 Scope -->|Satisfied or not applicable| Quota{Entitlement / quota required?}
 Quota -->|Exceeded| QuotaReject[Reject server side]
 Quota -->|Satisfied or not applicable| DB[Apply DB/storage defense-in-depth policy]
 DB --> Final[Execute + audit critical change]
 note["Free / Basic / Pro = package tiers, not RBAC roles"] -. rule .-> Role
```

## D10 PRIVACY LIFECYCLE

```mermaid
flowchart TB
 Notice[Versioned processing notice + lawful basis] --> Core[service_core processing R1]
 Notice --> Choices[Optional per-purpose consent event]
 Choices --> Analysis[research_analysis R3]
 Choices --> Contact[research_contact R3]
 Choices --> Voice[voice_retention R2]
 Choices --> Marketing[marketing future]
 Analysis --> Release[Eligible data release approval pipeline]
 Contact --> ContactService[Approved contact API]
 Voice --> AudioPolicy[Audio lifecycle + storage expiry]
 User[User] --> Request[Data-rights request R1]
 Request --> Verify[Verify identity / scope / exceptions]
 Verify --> Export[Export eligible personal data]
 Verify --> Erase[Delete active records + scheduled backup expiry]
 Erase --> Audit[Masked immutable audit event]
 Request --> Audit
 Choices --> Audit
 classDef future stroke-dasharray:5 5
 class Analysis,Contact,Voice,Marketing,Release,ContactService,AudioPolicy future
```

## D11 VOICE SEQUENCE R2

```mermaid
sequenceDiagram
 actor U as User
 participant W as Web
 participant A as NestJS Core
 participant S as Private object storage
 participant ST as STT adapter/provider
 participant D as PostgreSQL
 U->>W: Select voice mode (R2)
 W->>U: Microphone permission + separate retention choice
 U-->>W: Permission + opt in/out
 W->>A: Request signed upload session
 A->>D: Record purpose/version/time for retention choice
 A-->>W: Scoped temporary signed upload
 W->>S: Upload encrypted audio (short-lived)
 W->>A: Signal upload completed
 A->>ST: Transcribe with scoped object access
 ST-->>A: Transcript
 A-->>W: Editable transcript
 U->>W: Correct and submit transcript
 W->>A: POST text answer + idempotency key
 alt No retention opt-in
  A->>S: Schedule delete after processing (<=24h draft ceiling)
 else Explicit opt-in
  A->>S: Apply approved retention policy (<=90d draft)
 end
 Note over A,S: Release gate needs legal and privacy approval
```

## D12 PAYMENT SEQUENCE R3

```mermaid
sequenceDiagram
 actor U as User
 participant API as API / BFF
 participant B as Billing Service R3
 participant G as Payment Gateway
 participant DB as Billing PostgreSQL
 participant C as Core Entitlement Integration
 U->>API: Checkout selection
 API->>B: Authenticated checkout request
 B->>DB: Snapshot published package price
 B->>G: Create hosted payment session
 G-->>B: Hosted checkout reference
 B-->>U: Gateway hosted redirect
 G->>B: Signed webhook event
 B->>B: Verify authenticity + provider event ID
 B->>DB: Atomic inbox + state transition + outbox
 alt Confirmed, new valid payment
  B->>C: Versioned entitlement-granted event
  C-->>B: Deduplicated fulfillment ACK
 else Duplicate or pending
  B->>B: No duplicate fulfillment
 end
 U->>API: GET payment status
 API->>B: Fetch authorized status
 B-->>U: pending / confirmed / failed
 Note over B,DB: Refund needs request -> approval -> execution; rules not approved
```

## D13 RESEARCH RELEASE R3

```mermaid
flowchart TD
 Request[Researcher request R3] --> Gate{Purpose approval + role scope + valid basis?}
 Gate -->|No| Deny[Denied + audited]
 Gate -->|Yes| Pipeline[Privileged release pipeline]
 Source[(Core production data restricted)] --> Pipeline
 Pipeline --> Eligibility[Check research_analysis eligibility; no contact flag substitution]
 Eligibility --> Aggregate[Group + count distinct users]
 Aggregate --> Risk{Cell size >=10 and complementary / differencing checks?}
 Risk -->|No| Suppress[Suppress / revise release]
 Risk -->|Yes| Approve[Authorized release approval]
 Suppress --> Review[Human privacy review]
 Review --> Approve
 Approve --> Store[(Research DB approved aggregates)]
 Store --> API[Research scoped API]
 API --> Result[Aggregate only; never raw answers/audio]
 API --> Log[Audit allow/deny, purpose, dataset, query, row count]
 Deny --> Log
 classDef secure fill:#e2e8f0,stroke:#334155
 class Store secure
```

## D14 MANAGED PRODUCTION

```mermaid
flowchart TB
 Internet[Users] --> WAF[CDN + WAF + TLS]
 WAF --> WEB[Managed Next.js hosting]
 WEB --> API[Managed NestJS Core API]
 subgraph Private[Private production network]
  PG[(Managed Core PostgreSQL)]
  Redis[(Managed Redis)]
  Worker[Managed evaluation worker]
  AI[Private FastAPI inference]
  Buck[(Private object storage R2)]
 end
 API --> PG & Redis
 API -. R2 .-> Buck
 Redis --> Worker
 Worker --> PG & AI
 AI --> Provider[LLM provider]
 API --> Otel[Logs / metrics / traces]
 Worker --> Otel
 AI --> Otel
 Secrets[Production secret manager] -. restricted credentials .-> API & Worker & AI
 Backup[Backup + restore policy] --> PG
 API -. R3 authenticated call .-> Billing[Billing deployable + separate DB]
 API -. R3 authenticated call .-> Research[Research deployable + separate DB]
 classDef future stroke-dasharray:5 5
 class Buck,Billing,Research future
```

## D15 KUBERNETES SHOWCASE

```mermaid
flowchart TB
 Demo[Synthetic demo users / load] --> Ingress[Ingress + TLS]
 subgraph Cluster[Kubernetes SHOWCASE sandbox - NO production data]
  Ingress --> Web[Web Deployment]
  Ingress --> API[Core API Deployment]
  API --> PG[(Synthetic PostgreSQL)]
  API --> Redis[(Sandbox Redis)]
  Redis --> Worker[Worker Deployment]
  Worker --> PG
  Worker --> AI[FastAPI Deployment]
  AI --> Stub[LLM stub / sandbox provider]
  HPA[HPA CPU / queue metric when installed] -. scale .-> API & Worker
  OTel[OpenTelemetry Collector] --> Metrics[Grafana + metrics/traces]
  API --> OTel
  Worker --> OTel
  AI --> OTel
 end
 GitOps[Argo CD - app resources] -. reconciles .-> Cluster
 IaC[Terraform - cluster/network/infra] -. provisions .-> Cluster
 Source[(Production accounts / secrets / backups)] -. X NEVER CONNECT .-> Cluster
```

## D16 CI CD

```mermaid
flowchart TB
 Commit[Git SHA] --> CI[GitHub Actions: lint / unit / contract / security]
 CI --> Test[Integration / migrations / AI eval / image scanning]
 Test --> Build[Build images once + SBOM + sign]
 Build --> Registry[(OCI registry: immutable digest)]
 Registry --> Staging[Managed staging]
 Staging --> Gate{QA / approval gate}
 Gate -->|Approved| Prod[Managed production uses same digest]
 Gate -->|Rejected| Stop[Fix + new commit]
 Registry --> GitOpsPR[Showcase GitOps PR pinned to same digest]
 GitOpsPR --> Argo[Argo CD sync]
 Argo --> Demo[Kubernetes synthetic showcase]
 Prod --> Real[(Real personal production data)]
 Demo --> Fake[(Synthetic data only)]
 Real -. never copied .-> Fake
```

## D17 OBSERVABILITY RECOVERY

```mermaid
flowchart LR
 Web[Web] --> Trace[Correlation ID]
 API[NestJS API] --> OTel[OpenTelemetry]
 Worker[Evaluation worker] --> OTel
 AI[FastAPI] --> OTel
 Trace --> OTel
 OTel --> Logs[Sanitized logs]
 OTel --> Metrics[Latency / errors / queue depth / cost]
 OTel --> Traces[Distributed traces]
 Metrics --> Alerts[Alerts + incident runbooks]
 Logs --> Alerts
 PG[(Managed PG)] --> Backups[Encrypted backups]
 Backups --> Restore[Quarterly restore drill - proposed]
 Alerts --> Oncall[Team response]
 Cost[LLM token usage + budgets] --> Metrics
 Note["SRS proposed: non-AI P95 <=2s; text feedback P95 <=15s; RPO <=24h; RTO <=8h; availability >=99.5% - NOT measured"] -. target .-> Metrics
```

## D18 REPO BOUNDARIES

```mermaid
flowchart TB
 subgraph Apps[apps]
  Web[web Next.js]
  API[api NestJS modular monolith]
  Worker[evaluation-worker]
  AI[ai-service FastAPI]
  Billing[billing-service R3]
  Research[research-service R3]
 end
 subgraph Packages[packages]
  Contracts[contracts OpenAPI / events / generated]
  CoreDB[db-core Prisma + migrations]
  BillingDB[db-billing R3]
  ResearchDB[db-research R3]
  Obs[observability]
  UI[ui]
 end
 Web --> Contracts & UI
 API --> Contracts & CoreDB & Obs
 Worker --> Contracts & CoreDB & Obs
 AI --> Contracts
 Billing --> Contracts & BillingDB
 Research --> Contracts & ResearchDB
 Infra[infra Terraform / managed / k8s / Argo] --> Apps
 Tests[tests architecture / contract / integration / security / AI eval] --> Contracts
 Rule["NO: cross-owner Prisma imports or AI direct DB access"] -. enforce .-> Tests
```

## D19 RELEASE ROADMAP

```mermaid
flowchart LR
 R1["R1 CORE
Web/PWA, identity, interview text, AI rubric, feedback, history, privacy, audit"] --> R2["R2 ENHANCED
Voice/STT, speech metrics, grammar, progress, adaptive features"]
 R2 --> R3["R3 COMMERCIAL + RESEARCH
Plans/quotas, gateway/refunds, governed research releases"]
 R1 --> Production[Managed Production]
 R1 --> Showcase[Kubernetes synthetic showcase]
 R2 -. approval gate .-> Production
 R3 -. approval gate .-> Production
 Note["R1 does not depend on R2/R3 runtime. Native mobile remains optional extension."] -. rule .-> R1
```

## D20 — Use Case Diagram (R1 design)

[Mermaid source](diagrams/D20_use_case.mmd)

```mermaid
flowchart LR
  Guest([Guest])
  User([Registered User])
  Admin([System Admin])
  LLM[External LLM provider]
  subgraph IX[InterviewX R1 — proposed system boundary]
    Register([Register])
    Login([Log in])
    Profile([Manage profile])
    Practice([Start text interview])
    Answer([Submit text answer])
    Result([View evaluation and feedback])
    History([View or delete interview history])
    Appeal([Appeal feedback])
    Privacy([Manage consent and data requests])
    Content([Manage questions and publish rubrics])
    Evaluate([Evaluate answer against rubric])
  end
  Guest --> Register & Login
  User --> Profile & Practice & Answer & Result & History & Appeal & Privacy
  Admin --> Content
  Answer -. triggers .-> Evaluate
  Evaluate --> LLM
```

**Purpose:** แสดง actors และความสามารถที่ระบุไว้สำหรับ InterviewX R1.

**Scope:** Proposed R1 text interview; ไม่รวม R2 voice และ R3 billing/research.

**Actors:** Guest, Registered User, System Admin. LLM provider เป็น external dependency ของการประเมิน ไม่ใช่ผู้ใช้ระบบ.

**Explanation:** Guest เข้าถึง registration/login; Registered User ทำกิจกรรมฝึกและจัดการข้อมูลของตน; System Admin ดูแล content ตาม RBAC ที่เสนอ. เส้นประเป็นการกระตุ้น process ไม่ใช่ actor permission. ใช้ Mermaid flowchart เลียนแบบ UML use case; ไม่ใช่ strict UML.

**Assumptions and limitations:** การกำหนด permission รายละเอียดขึ้นกับ RBAC approval. ไม่แสดงผู้วิจัยหรือ Finance Admin เพราะเป็น R3. การเชื่อม LLM เป็น design dependency ไม่ใช่หลักฐานว่า deploy แล้ว.

**Source references:** [InterviewX_SRS_Final_For_Approval_v1.0.docx](../../03_SRS_Requirements/InterviewX_SRS_Final_For_Approval_v1.0.docx) (FR01/02/04/05/08/11/14/17/32–36); [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [D01_system_context.mmd](diagrams/D01_system_context.mmd); [D09_identity_authorization.mmd](diagrams/D09_identity_authorization.mmd).

**Implementation status:** Proposed design, pending stakeholder sign-off; ไม่พบ InterviewX practice app implementation ใน repository นี้.

## D21 — User Flow Diagram (R1 design)

[Mermaid source](diagrams/D21_user_flow.mmd)

```mermaid
flowchart TD
  Entry[Web entry] --> Auth{Signed in?}
  Auth -->|No| Access[Register or log in screen]
  Access -->|Invalid input or credentials| Access
  Access -->|Success| Home[Practice home]
  Auth -->|Yes| Home
  Home --> Setup[Interview setup: job type and locale]
  Home --> History[Interview history]
  Home --> Profile[Profile and privacy]
  Setup -->|Start| Session[Text interview: question and answer]
  Setup -->|Cancel| Home
  Session -->|Invalid or empty answer| Session
  Session -->|Submit accepted| Pending[Evaluation pending]
  Pending -->|Still processing| Pending
  Pending -->|Completed| Feedback[Rubric score and feedback]
  Pending -->|Failed or needs review| Outcome[Status and next action]
  Feedback -->|Next question| Session
  Feedback -->|Appeal| Appeal[Feedback appeal]
  Feedback -->|Finish| History
  Outcome --> History
  History -->|Open own session| Feedback
  History --> Home
  Profile --> Home
```

**Purpose:** แสดงการนำทางของผู้ใช้เพื่อเริ่มฝึก ตอบคำถาม และดู feedback.

**Scope:** Proposed responsive web R1; screen names are logical UX labels, not verified Next.js routes.

**Actors:** Guest and Registered User.

**Explanation:** กล่องคือหน้าหรือสถานะ UI เชิงตรรกะ. การประเมินที่ backend ไม่ถูกจำลองเป็นหน้าจอ; D07 อธิบาย service exchange. History เปิด feedback ได้เฉพาะ session ของเจ้าของข้อมูล.

**Assumptions and limitations:** ชื่อและลำดับหน้าจอเป็น design-level inference จาก API/SRS; ยังไม่มี verified route map ของ InterviewX. การนำทางจาก history ไป feedback ของ session ที่ยังไม่มีผลต้องแสดง status ก่อนใน implementation. ไม่แสดง R2 dashboard หรือ R3 payment.

**Source references:** [InterviewX_SRS_Final_For_Approval_v1.0.docx](../../03_SRS_Requirements/InterviewX_SRS_Final_For_Approval_v1.0.docx) (FR01/04/05/08/11/14/17/36); [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [D07_text_interview_sequence.mmd](diagrams/D07_text_interview_sequence.mmd); [D08_eval_states.mmd](diagrams/D08_eval_states.mmd).

Draft API accepts only interviewType=job and locale; additional category selection in the SRS remains unresolved.

**Implementation status:** Proposed UX flow, pending sign-off; screen routes unverified.

## D22 — Interview and Evaluation Activity (R1 design)

[Mermaid source](diagrams/D22_activity_workflows.mmd)

```mermaid
flowchart TD
  Start([Start])
  subgraph UserLane[Registered User]
    Choose[Choose job interview and locale]
    Write[Write and submit text answer]
    See[View status or feedback]
  end
  subgraph WebLane[Web]
    Show[Show question]
    Pending[Show pending state and poll]
  end
  subgraph CoreLane[NestJS Core and Core PostgreSQL]
    Create[Authorize and create session with pinned versions]
    Check{Valid answer, ownership and active session?}
    Reject[Return validation or authorization error]
    Commit[Atomic answer, job, idempotency and outbox commit]
    Read[Read owned evaluation status]
    Store[Persist checked result and job transition]
  end
  subgraph WorkerLane[Evaluation Worker and AI]
    Dispatch[Dispatch outbox via Redis/BullMQ]
    Assess[Worker calls FastAPI; validate provider result]
    Valid{Validated result?}
    Retry{Transient error and attempts remain?}
    Review[Mark needs_review or failed without invented score]
  end
  Start --> Choose --> Create --> Show --> Write --> Check
  Check -->|No| Reject --> Write
  Check -->|Yes| Commit --> Dispatch --> Assess --> Valid
  Valid -->|Yes| Store
  Valid -->|Invalid candidate| Review
  Valid -->|Provider error| Retry
  Retry -->|Yes| Assess
  Retry -->|No| Review --> Store
  Commit --> Pending --> Read
  Read -->|Pending| Pending
  Store --> Read
  Read -->|Terminal state| See --> End([End or continue session])
```

**Purpose:** แสดงการตัดสินใจและความรับผิดชอบใน text interview/evaluation lifecycle.

**Scope:** Proposed R1 core workflow; logical lanes describe responsibility, not independent deployables.

**Actors/components:** Registered User, Web, NestJS Core, Evaluation Worker, FastAPI AI, Core PostgreSQL, Redis/BullMQ.

**Explanation:** Core transaction ป้องกันคำตอบซ้ำและบันทึก job/outbox พร้อมกัน; worker รับงานแบบ at-least-once และตรวจผลก่อนบันทึก. สาขา failure อาจเป็น `failed` หรือ `needs_review` ตามชนิดข้อผิดพลาด. Web อ่านสถานะแบบ polling.

**Assumptions and limitations:** ไม่กำหนด retry count หรือ UI สำหรับ manual review เพราะยังไม่ตัดสินใจ. Existing D07 ให้ลำดับ message และ D08 ให้ state model; ภาพนี้เน้น decision/responsibility.

**Source references:** [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [internal-evaluation.openapi.yaml](../interviewx-design/contracts/internal-evaluation.openapi.yaml); [001_core.sql](../interviewx-design/database/001_core.sql); [D07_text_interview_sequence.mmd](diagrams/D07_text_interview_sequence.mmd); [D08_eval_states.mmd](diagrams/D08_eval_states.mmd).

**Implementation status:** Proposed architecture, not verified as deployed.

## D23 — Data Flow Diagram: Context / Level 0 (R1 design)

[Mermaid source](diagrams/D23_data_flow.mmd)

```mermaid
flowchart LR
  User[Registered User] -->|Account, interview choices, text answers, privacy requests| Platform(InterviewX R1 platform)
  Platform -->|Questions, evaluation status, feedback, history, request status| User
  Platform -->|Scoped question, answer, rubric and model request| LLM[Hosted LLM provider]
  LLM -->|Untrusted evaluation candidate| Platform
```

**Purpose:** แสดงข้อมูลที่เข้า ออก และไหลระหว่าง core processes และ data stores.

**Scope:** Proposed R1 text interview/evaluation. Mermaid flowchart ใช้รูปแบบ DFD: rectangle = external entity, rounded node = process, cylinder = data store. ลูกศรแสดงข้อมูล ไม่ใช่ UI navigation หรือ control flow.

**External entities/components:** Registered User and hosted LLM provider; Web is a client channel of the user, not a separate data owner.

**Explanation:** Level 0 รวม InterviewX เป็น process เดียว; Level 1 ขยาย account, interview, evaluation, feedback และ privacy. Redis/BullMQ ขนส่ง outbox job ตาม D07 แต่ไม่ใช่ durable source of truth; `evaluation`/`platform` stores เก็บ job และ event. FastAPI ตรวจ LLM candidate; ไม่เข้าถึง Core PostgreSQL โดยตรง.

**Assumptions and limitations:** DFD นี้เป็น logical data flow. การใช้ `IdentityStore` รวม account/session records ไม่แสดง raw credentials. Privacy request processing หลังบันทึกยังขึ้นกับ policy. ไม่รวม R2 audio หรือ R3 data stores.

**Source references:** [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [internal-evaluation.openapi.yaml](../interviewx-design/contracts/internal-evaluation.openapi.yaml); [001_core.sql](../interviewx-design/database/001_core.sql); [D02_c4_containers.mmd](diagrams/D02_c4_containers.mmd); [D07_text_interview_sequence.mmd](diagrams/D07_text_interview_sequence.mmd).

**Implementation status:** Proposed design, pending approval.

## D24 — Core Component Architecture (R1 design)

[Mermaid source](diagrams/D24_component_architecture.mmd)

```mermaid
flowchart LR
  Web[Next.js Web] -->|Public REST /v1| API[NestJS Core API / BFF]
  subgraph Core[NestJS Core — one modular monolith]
    Identity[Identity / Access]
    Content[Content]
    Interview[Interview]
    Evaluation[Evaluation read model and appeals]
    Privacy[Privacy]
    Platform[Platform: idempotency, outbox, audit]
  end
  API --> Identity & Content & Interview & Evaluation & Privacy
  Interview -->|Exported application contract| Identity
  Interview -->|Published question and version contract| Content
  Interview -->|Atomic submission contract| Platform
  Evaluation -->|Owned answer reference| Interview
  Evaluation -->|Job and audit contract| Platform
  Privacy -->|User identity contract| Identity
  Identity & Content & Interview & Evaluation & Privacy & Platform -->|Owning repository only| PG[(Core PostgreSQL, schema per context)]
  Platform -->|Outbox dispatch| Queue[(Redis / BullMQ)]
  Queue --> Worker[Evaluation Worker]
  Worker -->|Sanctioned job/result persistence adapter| PG
  Worker -->|Authenticated /internal/v1/evaluate| AI[FastAPI Inference]
  AI -->|Provider adapter| LLM[Hosted LLM provider]
```

**Purpose:** แสดง module dependencies ภายใน NestJS Core และ interfaces ที่เชื่อม worker/AI.

**Scope:** Proposed R1 modular monolith; R2/R3 omitted.

**Components:** Next.js Web, NestJS Core modules, Evaluation Worker, FastAPI Inference, Core PostgreSQL, Redis/BullMQ, hosted LLM.

**Explanation:** D02 ระบุ containers; D03 ระบุ bounded contexts. D24 เพิ่ม dependency ระหว่าง module และ interface ที่ข้าม process. Core modules อยู่ใน deployable เดียว; worker เป็น deployable แยกที่ใช้ persistence adapter ที่อนุญาต; FastAPI รับ scoped payload ผ่าน HTTP เท่านั้น.

**Assumptions and limitations:** ชื่อ module มาจาก proposed repository structure ไม่ใช่ source code ที่มีอยู่จริง. Connector ไป Core PostgreSQL แสดง schema ownership; ไม่อนุญาต cross-owner repository import. API/BFF ใช้ public contract ซึ่งยังเป็น draft.

**Source references:** [REPOSITORY_STRUCTURE.md](../interviewx-design/repo/REPOSITORY_STRUCTURE.md); [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [internal-evaluation.openapi.yaml](../interviewx-design/contracts/internal-evaluation.openapi.yaml); [D02_c4_containers.mmd](diagrams/D02_c4_containers.mmd); [D03_bounded_contexts.mmd](diagrams/D03_bounded_contexts.mmd); [D18_repo_boundaries.mmd](diagrams/D18_repo_boundaries.mmd).

**Implementation status:** Proposed design, pending stakeholder sign-off; no corresponding InterviewX runtime modules found in this repository.

## D25 — Logical Domain Class Model — Design-Level

[Mermaid source](diagrams/D25_class_model.mmd)

```mermaid
classDiagram
  class User {
    +UUID id
    +string email
    +string accountStatus
  }
  class InterviewSession {
    +UUID id
    +string interviewType
    +string locale
    +string status
    +datetime startedAt
  }
  class SessionQuestion {
    +UUID id
    +int sequenceNo
    +string questionText
  }
  class Answer {
    +UUID id
    +int attemptNumber
    +string answerText
    +datetime answeredAt
  }
  class EvaluationJob {
    +UUID id
    +string state
    +int attempts
  }
  class Feedback {
    +UUID id
    +number score
    +map dimensionScores
    +list evidence
    +string status
  }
  class RubricVersion {
    +UUID id
    +string version
    +map criteria
  }
  class AiConfigVersion {
    +UUID id
    +string version
    +string modelIdentifier
  }
  User "1" --> "0..*" InterviewSession : starts
  InterviewSession "1" *-- "0..*" SessionQuestion : contains
  SessionQuestion "1" *-- "0..*" Answer : receives
  Answer "1" --> "0..1" EvaluationJob : queues
  EvaluationJob "1" --> "0..1" Feedback : produces
  InterviewSession "0..*" --> "1" RubricVersion : pins
  InterviewSession "0..*" --> "1" AiConfigVersion : pins
  Feedback "0..*" --> "1" RubricVersion : scoredWith
  Feedback "0..*" --> "1" AiConfigVersion : evaluatedWith
```

**Purpose:** แสดง logical objects ของ interview/evaluation R1 และความสัมพันธ์ที่ code จะใช้ได้ โดยไม่อ้างว่าเป็น implemented TypeScript classes.

**Scope:** Proposed R1 domain model. Attributes are selected from proposed physical schema and API contract; methods intentionally omitted because no InterviewX class implementation is available.

**Entities:** User, InterviewSession, SessionQuestion, Answer, EvaluationJob, Feedback, RubricVersion, AiConfigVersion.

**Explanation:** Session pins rubric/AI configuration at creation. Answer queues at most one job under the proposed unique constraint; completed job can produce one Feedback. `Feedback.score` may be absent for `needs_review`, although Mermaid does not express nullability. This is an object view of key domain data and version dependencies; D04 remains the physical ERD.

**Assumptions and limitations:** Class identifiers and camelCase attributes are logical projections of SQL/API names. Composition conveys lifecycle ownership at design level, not SQL cascade deletion. No methods or inheritance are asserted. The proposed SQL permits zero questions/answers and nullable score; these are reflected in multiplicities and prose.

**Source references:** [001_core.sql](../interviewx-design/database/001_core.sql); [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [D04_core_erd.mmd](diagrams/D04_core_erd.mmd).

**Implementation status:** Logical Domain Class Model — Design-Level; not verified as implemented code.

## D26 — Data Flow Diagram: Level 1 (R1 design)

[Mermaid source](diagrams/D26_data_flow_level1.mmd)

```mermaid
flowchart LR
  User[Registered User] -->|Registration, login, profile input| Identity(1. Identity and access)
  Identity -->|Account and session records| IdentityStore[(identity schema)]
  IdentityStore -->|Identity and authorization data| Identity
  Identity -->|Owned user context| Interview(2. Interview practice)
  User -->|Category, locale, text answer| Interview
  ContentStore[(content schema)] -->|Published questions, rubric and AI config versions| Interview
  Interview -->|Session, question and answer| InterviewStore[(interview schema)]
  InterviewStore -->|Own history and answer context| Interview
  Interview -->|Question and history| User
  Interview -->|Evaluation job and outbox event| EvalStore[(evaluation and platform schemas)]
  EvalStore -->|Queued job and answer reference| Evaluate(3. Evaluate answer)
  InterviewStore -->|Question and answer text| Evaluate
  ContentStore -->|Pinned rubric and AI config| Evaluate
  Evaluate -->|Scoped evaluation payload| LLM[Hosted LLM provider]
  LLM -->|Untrusted candidate| Evaluate
  Evaluate -->|Validated result or review/failure state| EvalStore
  User -->|Status and feedback request| Feedback(4. Read evaluation)
  EvalStore -->|Job state and approved feedback| Feedback
  Feedback -->|Status, rubric scores and evidence-based feedback| User
  User -->|Consent or data-rights request| Privacy(5. Privacy requests)
  Privacy -->|Notice, consent event, request state| PrivacyStore[(privacy schema)]
  PrivacyStore -->|Recorded request and consent state| Privacy
  Privacy -->|Request status| User
```

**Purpose:** แสดงข้อมูลที่เข้า ออก และไหลระหว่าง core processes และ data stores.

**Scope:** Proposed R1 text interview/evaluation. Mermaid flowchart ใช้รูปแบบ DFD: rectangle = external entity, rounded node = process, cylinder = data store. ลูกศรแสดงข้อมูล ไม่ใช่ UI navigation หรือ control flow.

**External entities/components:** Registered User and hosted LLM provider; Web is a client channel of the user, not a separate data owner.

**Explanation:** Level 0 รวม InterviewX เป็น process เดียว; Level 1 ขยาย account, interview, evaluation, feedback และ privacy. Redis/BullMQ ขนส่ง outbox job ตาม D07 แต่ไม่ใช่ durable source of truth; `evaluation`/`platform` stores เก็บ job และ event. FastAPI ตรวจ LLM candidate; ไม่เข้าถึง Core PostgreSQL โดยตรง.

**Assumptions and limitations:** DFD นี้เป็น logical data flow. การใช้ `IdentityStore` รวม account/session records ไม่แสดง raw credentials. Privacy request processing หลังบันทึกยังขึ้นกับ policy. ไม่รวม R2 audio หรือ R3 data stores.

**Source references:** [public-v1.openapi.yaml](../interviewx-design/contracts/public-v1.openapi.yaml); [internal-evaluation.openapi.yaml](../interviewx-design/contracts/internal-evaluation.openapi.yaml); [001_core.sql](../interviewx-design/database/001_core.sql); [D02_c4_containers.mmd](diagrams/D02_c4_containers.mmd); [D07_text_interview_sequence.mmd](diagrams/D07_text_interview_sequence.mmd).

**Implementation status:** Proposed design, pending approval.
