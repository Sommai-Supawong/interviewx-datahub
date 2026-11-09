
# MASTER PROMPT
# InterviewX — System Architecture Diagram Audit, Gap Analysis & Enhancement

## ROLE

You are a Senior Software Architect, System Analyst, and Technical Documentation Engineer.

Your responsibility is to analyze the existing InterviewX project documentation and architecture, identify missing system diagrams, create only the diagrams that are genuinely needed, and integrate them into the existing documentation structure.

This is an EXISTING PROJECT DOCUMENTATION ENHANCEMENT task.

Do NOT redesign the entire system.
Do NOT invent undocumented features.
Do NOT duplicate existing diagrams.
Do NOT overwrite working architecture documentation unnecessarily.

All generated diagrams must accurately represent the InterviewX project based on verifiable existing information.

Use Thai for explanations and documentation descriptions where appropriate, while preserving English technical terminology, identifiers, API names, entity names, and code symbols.

---

# 1. PROJECT CONTEXT

Project Name: InterviewX

InterviewX is an AI-powered interview practice platform.

Its conceptual purpose includes:
- Simulating interview scenarios
- Allowing users to practice answering interview questions
- Evaluating answers using AI
- Scoring answers according to evaluation rubrics
- Providing actionable feedback for improvement

IMPORTANT:
The above information is only high-level project context.

Actual system behavior, roles, services, features, and implementation must be verified from existing project sources.

Do not assume a conceptual feature is already implemented.

Distinguish clearly between:
1. Implemented functionality
2. Documented planned functionality
3. Proposed or future functionality

---

# 2. DIRECTORY CONFIGURATION

Documentation Root:

C:\Users\Somma\OneDrive\เอกสาร\InterviewX\interviewx-datahub\docs-interviex

Architecture Directory:

C:\Users\Somma\OneDrive\เอกสาร\InterviewX\interviewx-datahub\docs-interviex\04_System_Architecture\InterviewX_Final_System_Diagrams_v1.0

Diagram Directory:

C:\Users\Somma\OneDrive\เอกสาร\InterviewX\interviewx-datahub\docs-interviex\04_System_Architecture\InterviewX_Final_System_Diagrams_v1.0\diagrams

Master Diagram File:

C:\Users\Somma\OneDrive\เอกสาร\InterviewX\interviewx-datahub\docs-interviex\04_System_Architecture\InterviewX_Final_System_Diagrams_v1.0\ALL_DIAGRAMS.md

README File:

C:\Users\Somma\OneDrive\เอกสาร\InterviewX\interviewx-datahub\docs-interviex\04_System_Architecture\InterviewX_Final_System_Diagrams_v1.0\readme.md

README handling:
- Check for README.md, readme.md, or other capitalization.
- Update the existing file.
- Do not create duplicate README files.

Use these paths as authoritative destinations.

Do not rename or move existing project directories.

---

# 3. PRIMARY OBJECTIVE

Audit the existing InterviewX architecture documentation and determine whether the following six diagram types already exist:

1. Use Case Diagram
2. User Flow Diagram
3. Activity Diagram
4. Data Flow Diagram (DFD)
5. Component Diagram
6. Class Diagram

For every requested diagram:

- Check whether the diagram already exists.
- Check whether equivalent information exists under another name.
- Check whether an existing diagram already covers the required purpose.
- Identify missing information or coverage gaps.
- Create a new diagram only when necessary.
- Avoid unnecessary conceptual overlap.

After completing the analysis:

1. Generate missing diagrams as Markdown files containing Mermaid.
2. Add the new diagrams to ALL_DIAGRAMS.md.
3. Update the existing README.
4. Verify consistency with the current InterviewX architecture.
5. Report all findings and modifications.

---

# 4. PHASE 1 — PROJECT DISCOVERY

Before making ANY modification, inspect the existing project.

## 4.1 Inspect documentation

Recursively inspect relevant documentation inside:

docs-interviex/

Look for:
- Software Requirements Specification (SRS)
- Functional Requirements
- Non-Functional Requirements
- System Architecture
- System Design
- API Documentation
- Database Documentation
- Authentication and Authorization
- Role and Permission definitions
- Business Workflows
- AI Evaluation specifications
- Interview workflows
- UI/UX documentation
- Existing diagrams
- Implementation plans
- Deployment documentation

Identify the actual file structure rather than assuming specific filenames.

## 4.2 Inspect existing diagrams

Read and analyze all relevant files in:

InterviewX_Final_System_Diagrams_v1.0/

Especially:

- ALL_DIAGRAMS.md
- Existing README
- diagrams/*.mmd
- diagrams/*.md

Inspect the actual Mermaid code, not just filenames.

The existing diagram collection may include:

D01_system_context.mmd
D02_c4_containers.mmd
D03_bounded_contexts.mmd
D04_core_erd.mmd
D05_r3_erd.mmd
D06_api_trust.mmd
D07_text_interview_sequence.mmd
D08_eval_states.mmd
D09_identity_authorization.mmd
D10_privacy_lifecycle.mmd
D11_voice_sequence_r2.mmd
D12_payment_sequence_r3.mmd
D13_research_release_r3.mmd
D14_managed_production.mmd
D15_kubernetes_showcase.mmd
D16_ci_cd.mmd
D17_observability_recovery.mmd
D18_repo_boundaries.mmd
D19_release_roadmap.mmd

This list is only a starting reference.

Discover the actual files and contents before proceeding.

## 4.3 Inspect source code when available

Determine whether the accessible project contains application source code.

If source code exists, inspect relevant:
- Frontend routes and pages
- Backend routes and controllers
- Services and business logic
- Database schemas and ORM models
- Authentication and authorization
- AI integration and evaluation modules
- API contracts
- Shared types and interfaces

If the source code is located in a clearly identifiable related repository, inspect it in read-only mode where accessible.

Do not search unrelated directories or modify other repositories.

If source code cannot be located, do not pretend that implementation was verified.

Use the existing documentation and mark conclusions with the appropriate evidence level.

## 4.4 Source-of-truth priority

Use this evidence hierarchy:

For CURRENTLY IMPLEMENTED behavior:
1. Accessible source code and database schema
2. Verified API and implementation contracts
3. Current architecture documentation
4. Requirements and planning documents

For PLANNED behavior:
1. Approved requirements and specifications
2. Current architecture decisions
3. Roadmap and design proposals

If documents conflict:
- Identify the conflict.
- Prefer the most authoritative and applicable source.
- Do not silently merge contradictory information.
- Document material uncertainty.

---

# 5. PHASE 2 — DIAGRAM GAP ANALYSIS

Perform a semantic audit of the six requested diagram types.

Do not check filenames alone.

A diagram may already exist under:
- A different filename
- A different Mermaid syntax
- An architecture section
- A combined diagram
- An embedded Mermaid code block

For each requested diagram, assign one status:

EXISTS_COMPLETE
The existing documentation already covers the required purpose sufficiently.

EXISTS_PARTIAL
Relevant information exists, but meaningful required coverage is missing.

MISSING
No existing diagram sufficiently covers the requested purpose.

INSUFFICIENT_EVIDENCE
The available project information does not support an accurate diagram.

Produce an internal comparison table containing:

- Diagram Type
- Existing Diagram or Section
- Status
- Evidence
- Missing Coverage
- Recommended Action

Important distinctions:

- System Context Diagram is not automatically equivalent to DFD.
- Sequence Diagram is not automatically equivalent to User Flow.
- State Diagram is not automatically equivalent to Activity Diagram.
- ERD is not automatically equivalent to Class Diagram.
- C4 Container Diagram is not automatically equivalent to an internal Component Diagram.

Evaluate the actual information and purpose before deciding.

Do not create a new diagram when the existing representation is already sufficient.

If coverage is partial, create a complementary diagram only when it adds substantial value.

---

# 6. PHASE 3 — DIAGRAM GENERATION REQUIREMENTS

Create only the missing or meaningfully incomplete diagrams.

All newly generated files must:
- Use the .md extension.
- Contain valid Mermaid code blocks.
- Match the existing project naming conventions.
- Use consistent terminology.
- Be understandable by both developers and project reviewers.
- Reflect actual documented architecture and functionality.

Do NOT generate standalone .mmd files for the new diagrams.

Each Markdown file must include:

1. Diagram Title
2. Purpose
3. Scope
4. Actors / Components / Entities where applicable
5. Mermaid Diagram
6. Explanation of workflow or relationships
7. Key assumptions and limitations
8. Source References
9. Implementation Status, where relevant

Use source references with repository-relative paths whenever possible.

Do not fabricate citations or file references.

---

# 7. REQUIRED DIAGRAM SPECIFICATIONS

## 7.1 USE CASE DIAGRAM

Suggested filename:

D20_use_case.md

Purpose:
Identify system actors, functional capabilities, and actor-to-use-case relationships.

Analyze actual user roles from the project.

Potential actors may include:
- Guest
- Registered User
- Administrator
- External AI Provider
- Other verified external systems

These are examples, NOT automatically approved actors.

Identify actual use cases from requirements and existing implementation.

Examples to verify:
- User registration
- Login / Logout
- Profile management
- Interview configuration
- Interview session execution
- Answer submission
- AI evaluation
- Evaluation result viewing
- Interview history
- Administration

Do not include unverified capabilities.

Requirements:
- Clearly identify actors.
- Show system boundaries.
- Show which actors interact with each use case.
- Do not invent permissions.
- Avoid unnecessary complexity.
- Split into multiple related views only if necessary.

Mermaid does not provide a native UML Use Case syntax.

Use Mermaid flowchart with UML-inspired actor, use-case, and system-boundary conventions.

Clearly state this notation choice.

Do not claim strict UML compliance when using a flowchart approximation.

---

## 7.2 USER FLOW DIAGRAM

Suggested filename:

D21_user_flow.md

Purpose:
Describe how actual users navigate through InterviewX to complete their goals.

Analyze:
- User entry points
- Authentication flow
- Dashboard navigation
- Interview setup
- Interview execution
- Evaluation result viewing
- Relevant alternative paths
- Relevant administrative workflows

Only include verified functionality.

Requirements:
- Show screen-to-screen navigation.
- Include significant user decisions.
- Include success and failure paths where relevant.
- Distinguish navigation from backend processing.
- Avoid copying the Sequence Diagram.

Use Mermaid flowchart.

Group related flows into clearly labeled sections or diagrams inside the Markdown file if necessary.

---

## 7.3 ACTIVITY DIAGRAM

Suggested filename:

D22_activity_workflows.md

Purpose:
Describe business processes, decisions, and process responsibilities.

Analyze the project's most important workflows.

Prioritize the core interview lifecycle and AI evaluation workflow.

Include relevant:
- Start and end events
- User actions
- System processing
- Decision conditions
- Alternative paths
- Error handling
- Completion conditions

Where useful, separate responsibilities into logical lanes such as:
- User
- Frontend
- Backend
- AI Evaluation Service
- Database

Use Mermaid flowchart or other supported Mermaid notation appropriate to the actual process.

Do not invent workflow steps that are not supported by evidence.

Do not merely reproduce an existing Sequence Diagram.

---

## 7.4 DATA FLOW DIAGRAM (DFD)

Suggested filename:

D23_data_flow.md

Purpose:
Explain how data moves between actors, processes, external systems, and data stores.

First inspect existing database and API diagrams.

Create a coherent DFD model with:

- Context / Level 0 view where useful
- Level 1 decomposition of the core InterviewX processes

Identify actual:
- External entities
- Processes
- Data stores
- Data flows
- System boundaries

Relevant data categories to verify:
- Account information
- Authentication information
- Interview configuration
- Questions and answers
- Evaluation results
- Rubric scores
- Feedback
- Interview history

Use the project's actual names.

Requirements:
- Label data flows clearly.
- Ensure data flows have meaningful sources and destinations.
- Keep Level 0 and Level 1 conceptually consistent.
- Do not expose real credentials, tokens, or personal data.
- Do not treat UI navigation as data flow.
- Do not mistake DFD for an ERD.

Use Mermaid flowchart with explicit DFD conventions and a legend if helpful.

---

## 7.5 COMPONENT DIAGRAM

Suggested filename:

D24_component_architecture.md

Purpose:
Describe the internal architectural components and their dependencies.

Inspect:
- Existing C4 diagrams
- Bounded contexts
- API architecture
- Services
- Modules
- Repositories
- Application layers

Identify actual components, for example where verified:
- Frontend application
- Authentication module
- Interview module
- Evaluation module
- AI provider adapter
- Data access layer
- Persistence components

Requirements:
- Use actual module/service names where known.
- Show dependencies and interfaces.
- Identify system boundaries.
- Distinguish internal components from external services.
- Respect documented architecture decisions.
- Avoid introducing fictional microservices.

If the project is a modular monolith, represent it as such.

Do not transform the architecture into microservices without evidence.

Use Mermaid flowchart or another valid Mermaid representation.

Explain how the diagram complements existing C4 Container and Bounded Context diagrams.

---

## 7.6 CLASS DIAGRAM

Suggested filename:

D25_class_model.md

Purpose:
Describe the application's logical domain classes and their relationships.

Inspect:
- Domain models
- Existing ERDs
- Database schemas
- ORM entities
- TypeScript interfaces and types
- Backend models
- Business service classes

Identify core domain objects supported by evidence.

Potential examples to verify:
- User
- InterviewSession
- InterviewQuestion
- InterviewAnswer
- Evaluation
- EvaluationResult
- EvaluationRubric

Do not assume these exact entities exist.

Requirements:
- Use Mermaid classDiagram.
- Include meaningful attributes.
- Include methods only when supported by source code or a documented domain design.
- Show relevant associations.
- Show inheritance only when justified.
- Show multiplicity where verified.
- Keep names consistent with the existing schema.
- Do not reproduce the ERD without adding useful object-model information.

If no object-oriented implementation exists but a documented domain model is available, a logical class model may be created.

Explicitly label it as:
"Logical Domain Class Model — Design-Level"

Do not present conceptual classes as implemented code classes.

If the available evidence is insufficient, do not invent class details merely to produce a diagram.

Report INSUFFICIENT_EVIDENCE instead.

---

# 8. FILE NAMING RULES

First inspect the actual existing diagram filenames and numbering.

Suggested new sequence:

D20_use_case.md
D21_user_flow.md
D22_activity_workflows.md
D23_data_flow.md
D24_component_architecture.md
D25_class_model.md

However:

- These are preferred names, not mandatory numbers.
- Do not overwrite files with existing names.
- Do not reuse an occupied diagram ID.
- If some requested diagrams already exist, skip them.
- Allocate IDs sequentially to newly created files.
- Preserve all existing D01–D19 files.
- Preserve the actual numbering conventions used by the repository.

The final number of diagrams depends on the gap analysis.

---

# 9. PHASE 4 — UPDATE ALL_DIAGRAMS.md

After creating the necessary Markdown diagrams:

Update:

InterviewX_Final_System_Diagrams_v1.0/ALL_DIAGRAMS.md

Requirements:

1. Read the entire existing file first.
2. Preserve all original diagrams.
3. Preserve existing Mermaid blocks.
4. Preserve the current organization and formatting where possible.
5. Integrate only newly created diagrams.
6. Use meaningful section headings.
7. Add relative links to the individual Markdown files.
8. Keep numbering consistent.
9. Avoid duplicate entries.
10. Ensure the master document remains readable.

For diagrams already included in existing documentation:
- Do not duplicate their full content.
- Reference the existing representation where appropriate.

For new diagrams:
- Include the Mermaid diagram in the master document.
- Add a relative source-file link.
- Keep the content synchronized with the corresponding new Markdown file.
- Avoid contradictory alternate versions.

Treat the individual diagram Markdown file as the canonical source for its new diagram.

Do not remove existing architecture documentation to make room for new material.

---

# 10. PHASE 5 — UPDATE README

Update the existing README inside:

InterviewX_Final_System_Diagrams_v1.0/

Read and preserve its existing purpose before editing.

Update relevant sections to describe:

1. Architecture documentation overview
2. Current diagram inventory
3. Newly added diagrams
4. Purpose of each diagram
5. Diagram organization
6. Navigation links
7. How to preview Mermaid diagrams
8. Relationship between diagrams
9. Documentation conventions
10. Known limitations, if relevant

Provide a clear diagram inventory table:

| ID | Diagram | Purpose | File | Status |

Include all current diagrams, not only the new ones.

Use correct relative links.

Status should distinguish documented implementation from conceptual design where necessary.

Avoid changing unrelated README content.

---

# 11. CONSISTENCY REQUIREMENTS

All new diagrams must agree with the existing InterviewX architecture.

Validate:

## Terminology
- Consistent actor names
- Consistent role names
- Consistent module names
- Consistent database entity names
- Consistent AI evaluation terminology

## Functional consistency
- Use Case vs User Flow
- User Flow vs Activity
- Activity vs Sequence
- DFD vs API and data persistence
- Component vs C4 and Bounded Context
- Class Diagram vs ERD and domain model

## Architectural consistency
- No invented services
- No undocumented authentication flows
- No invented databases
- No unsupported role permissions
- No incorrect module boundaries
- No contradictions with current architecture decisions

If contradictions are found in existing documentation:

- Do not silently rewrite the existing architecture.
- Record the conflict.
- Explain which source was prioritized.
- Avoid generating misleading diagrams.

---

# 12. CHANGE SCOPE & SAFETY

Allowed modifications:

1. Create necessary new .md files inside:
   InterviewX_Final_System_Diagrams_v1.0/diagrams/

2. Update:
   InterviewX_Final_System_Diagrams_v1.0/ALL_DIAGRAMS.md

3. Update the existing:
   InterviewX_Final_System_Diagrams_v1.0/readme.md
   or its existing capitalization variant.

Read-only inspection is allowed for other relevant project files.

Forbidden:

- Deleting existing diagram files
- Overwriting existing D01–D19 diagrams
- Modifying application source code
- Modifying database schema
- Modifying API implementations
- Changing project dependencies
- Unnecessary file renaming
- Creating duplicate README files
- Rewriting unrelated documentation
- Inventing future architecture as current implementation

Preserve existing uncommitted user changes.

If Git is available, inspect the initial working-tree state before editing.

Do not run destructive Git commands.

Do not commit or push changes unless explicitly requested.

Do not install dependencies without permission.

---

# 13. PHASE 6 — QUALITY ASSURANCE

After generating and integrating diagrams:

## 13.1 File validation
Check that:
- All new files exist in the correct directory.
- Every new file uses .md.
- Every new file contains its intended Mermaid block.
- No existing file was accidentally overwritten.
- No diagram IDs are duplicated.

## 13.2 Mermaid validation
Check that:
- Mermaid syntax is valid.
- Node identifiers are unique where required.
- Diagram definitions are well formed.
- Labels do not break parsing.
- Relationships are clear.

Use existing Mermaid validation tools if already available.

Do not install new packages merely to validate diagrams.

If automated rendering or validation is unavailable, perform structural checks and explicitly report that rendering was not verified.

## 13.3 Documentation validation
Check that:
- ALL_DIAGRAMS.md includes every newly created diagram.
- README contains accurate inventory and links.
- Relative Markdown links resolve.
- No duplicate diagram entries exist.
- Original diagrams remain intact.
- New diagrams agree with source documents.

## 13.4 Architecture validation
Cross-check all six diagram categories against actual evidence.

Identify:
- Missing source evidence
- Unresolved architecture conflicts
- Potential documentation inconsistencies
- Areas needing project-owner clarification

Do not claim complete validation for checks that could not be performed.

---

# 14. EXECUTION WORKFLOW

Execute the task in this order:

STEP 1:
Discover the actual documentation structure and related source files.

STEP 2:
Read existing architecture diagrams and documentation.

STEP 3:
Perform semantic comparison against all six requested diagram types.

STEP 4:
Classify each diagram as:
EXISTS_COMPLETE / EXISTS_PARTIAL / MISSING / INSUFFICIENT_EVIDENCE.

STEP 5:
Determine the minimum necessary set of new diagrams.

STEP 6:
Create only those diagrams with sufficient supporting evidence.

STEP 7:
Update ALL_DIAGRAMS.md.

STEP 8:
Update the existing README.

STEP 9:
Validate Mermaid syntax, references, names, and architecture consistency.

STEP 10:
Review the final changes and present a completion report.

Do not stop after analysis if the necessary changes are safe and supported.

Complete the audit, generation, integration, and validation in this execution.

If a specific diagram cannot be produced reliably, skip that diagram and clearly explain why.

---

# 15. REQUIRED FINAL REPORT

At the end, provide a report in Thai.

## A. Existing Diagram Audit

Include a table:

| Diagram Type | Existing Coverage | Status | Action |

Explain the decision for each requested diagram.

## B. Created Files

List every newly generated file with:
- Filename
- Diagram type
- Purpose
- Supporting source documents

## C. Updated Files

Describe changes to:
- ALL_DIAGRAMS.md
- README

## D. Skipped Diagrams

For every skipped diagram, explain:
- Already covered by which existing diagram, OR
- Insufficient evidence, OR
- Other documented reason.

## E. Validation Results

Report:
- Mermaid syntax validation
- File/link integrity
- Duplicate checks
- Architecture consistency checks
- Checks that could not be completed

## F. Final Diagram Inventory

Provide:
- Original diagram count
- Number of new diagrams
- Final diagram count
- All new diagram IDs and filenames

## G. Remaining Issues

Identify unresolved contradictions and documentation gaps.

---

# 16. ACCEPTANCE CRITERIA

The task is complete only when:

[ ] Existing InterviewX documentation was analyzed.
[ ] All six requested diagram types were audited.
[ ] No unnecessary duplicate diagram was created.
[ ] Newly created diagrams are supported by project evidence.
[ ] Every new diagram is a Markdown file containing Mermaid.
[ ] New files are stored in the designated diagrams directory.
[ ] ALL_DIAGRAMS.md reflects the final diagram collection.
[ ] Existing README is updated.
[ ] Relative links are correct.
[ ] No existing diagrams were destroyed or overwritten.
[ ] Mermaid validation was attempted and results reported.
[ ] Unverified assumptions are explicitly labeled.
[ ] A final audit and modification report is provided.

---

# FINAL INSTRUCTION

Prioritize architectural correctness over diagram quantity.

The objective is NOT to produce six files regardless of necessity.

The objective is to complete the missing architectural documentation of InterviewX using verified project information.

First understand the existing system.
Then identify genuine gaps.
Then generate only what is missing.
Finally integrate, validate, and summarize all changes.

Proceed with the full task now.
