# InterviewX — Final System Diagram Set v1.0

**Design baseline:** 2026-10-08 | **Status:** Proposed final design, NOT approved or deployed.

## Authority and scope

Based on `InterviewX_SRS_Final_For_Approval_v1.0.docx`, `InterviewX_Architecture_Diagrams_v1.0.md`, and `InterviewX_Design_Package_v1.0.zip`. If the SRS is not signed, these are design proposals, not approved requirements. R1: responsive web, text interview, AI evaluation, identity, privacy and admin. R2: voice/STT, speech indicators, dashboard and enhancements. R3: billing and research in independent services/databases. The Kubernetes environment is synthetic-only demonstration, not production mirror with real users' data.

## Diagrams and navigation

| ID | Diagram | Purpose | File | Status |
|---|---|---|---|---|
| D01 | System context | Actors and external dependencies | [D01](diagrams/D01_system_context.mmd) | Proposed R1/R3 |
| D02 | C4 containers | Core, AI and optional services | [D02](diagrams/D02_c4_containers.mmd) | Proposed R1/R3 |
| D03 | Bounded contexts | Ownership and dependency rules | [D03](diagrams/D03_bounded_contexts.mmd) | Proposed R1/R3 |
| D04 | Core physical ERD | R1 and R2-ready tables | [D04](diagrams/D04_core_erd.mmd) | Proposed schema |
| D05 | R3 physical ERD | Independent Billing/Research stores | [D05](diagrams/D05_r3_erd.mmd) | Proposed R3 |
| D06 | API and trust boundaries | External vs internal contracts | [D06](diagrams/D06_api_trust.mmd) | Proposed R1/R3 |
| D07 | Text interview sequence | Atomic persistence and async evaluation | [D07](diagrams/D07_text_interview_sequence.mmd) | Proposed R1 |
| D08 | Evaluation state machine | Retry and review states | [D08](diagrams/D08_eval_states.mmd) | Proposed R1 |
| D09 | Identity and authorization | RBAC, ownership and purpose checks | [D09](diagrams/D09_identity_authorization.mmd) | Proposed R1/R3 |
| D10 | Privacy lifecycle | Notices, consent and data rights | [D10](diagrams/D10_privacy_lifecycle.mmd) | Proposed R1/R3 |
| D11 | Voice sequence | STT and retention choice | [D11](diagrams/D11_voice_sequence_r2.mmd) | Proposed R2 |
| D12 | Payment sequence | Webhook verification and fulfillment | [D12](diagrams/D12_payment_sequence_r3.mmd) | Proposed R3 |
| D13 | Research release | Eligibility, suppression and audit | [D13](diagrams/D13_research_release_r3.mmd) | Proposed R3 |
| D14 | Managed production | Network and service deployment | [D14](diagrams/D14_managed_production.mmd) | Proposed deployment |
| D15 | Kubernetes showcase | Synthetic-only sandbox | [D15](diagrams/D15_kubernetes_showcase.mmd) | Proposed demonstration |
| D16 | CI/CD and GitOps | Immutable image promotion | [D16](diagrams/D16_ci_cd.mmd) | Proposed pipeline |
| D17 | Observability and recovery | Telemetry and restore drill | [D17](diagrams/D17_observability_recovery.mmd) | Proposed operations |
| D18 | Repo boundaries | Package and interface boundaries | [D18](diagrams/D18_repo_boundaries.mmd) | Proposed repo design |
| D19 | Release roadmap | Feature gates and independence | [D19](diagrams/D19_release_roadmap.mmd) | Proposed roadmap |
| D20 | Use case | Actors and R1 capabilities | [D20](diagrams/D20_use_case.mmd) | Design-level R1 |
| D21 | User flow | Web navigation for text practice | [D21](diagrams/D21_user_flow.mmd) | Design-level R1 |
| D22 | Activity workflow | Interview and evaluation decisions | [D22](diagrams/D22_activity_workflows.mmd) | Design-level R1 |
| D23 | Data flow, context | External data exchange (Level 0) | [D23](diagrams/D23_data_flow.mmd) | Design-level R1 |
| D24 | Component architecture | Internal module dependencies | [D24](diagrams/D24_component_architecture.mmd) | Design-level R1 |
| D25 | Logical class model | Domain objects and relationships | [D25](diagrams/D25_class_model.mmd) | Design-level R1 |
| D26 | Data flow, Level 1 | Core processes and data stores | [D26](diagrams/D26_data_flow_level1.mmd) | Design-level R1 |

All D01-D26 files are standalone raw Mermaid sources. [ALL_DIAGRAMS.md](ALL_DIAGRAMS.md) collects every diagram for review and preserves the purpose, scope, explanation, assumptions, source references, and implementation status of D20-D26. Its embedded Mermaid blocks mirror the individual `.mmd` files. Open the master in a Mermaid-enabled Markdown preview. For `.mmd` files, use a Mermaid-capable viewer or copy the source into a Mermaid code block.

D20 identifies actors and capabilities; D21 follows user navigation; D22 follows business decisions; D23 shows the DFD context and D26 its Level 1 decomposition; D24 expands D02/D03 into module dependencies; D25 complements the physical D04 ERD with a logical object view. D07/D08 remain the detailed evaluation sequence and state references. These are design proposals, not a verified InterviewX deployment. This repository's executable application is InterviewX DataHub, a separate product scope. The SRS category choice exceeds the draft API's `interviewType=job` and `locale`; the final UI contract needs a decision.

## Important invariants

1. **Single ownership:** Core data changes originate in the owning NestJS module; worker writes evaluation status/results through its sanctioned persistence adapter. AI FastAPI only receives scoped task payloads and never connects to Core Postgres.
2. **Exactly-once business effect:** submit answer, evaluation job, idempotency record and outbox event atomically; queue delivery is at-least-once, consumers dedupe.
3. **Strong boundaries:** R3 Billing and Research have independent Postgres credentials and databases; no cross-service FK or direct SQL access to Core tables. Read grants are purpose-scoped.
4. **Privacy:** research_contact != research_analysis != voice_retention; service_core processing requires declared lawful basis/notice, not bundled optional consent. Researcher never queries raw Core tables.
5. **Deployment parity:** managed production and showcase can share *built image digest* but never data, domains, credentials or operational secrets.
6. **Scope:** R1 cannot require active R2/R3 services. Voice, payments, research and native mobile require separately approved scope.

## Traceability

- R1 Identity: FR01/02, FR17, FR32/35 → D03/04/06/09/10
- R1 Interview+feedback: FR04/05/08/11/14/33/34/36 → D02/04/07/08
- R2 AI/voice: FR05-V/05.1/09/10/13/16 → D10/11
- R3 billing: FR19–31 → D05/12
- R3 research: FR18/18.1/32.1 → D05/13
- NFR01/04/05/09/17–24 → D06/09/10/13–17

**Outstanding decisions:** sign-off, retention time and lawful basis, R3 prices/gateway/refunds, research release policy, final internal service-auth mode, topology sizing, production thresholds and provider spend. Diagram labels show planned controls, not already validated controls.
