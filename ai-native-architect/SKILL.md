---
name: ai-native-architect
description: Use when a team is considering an AI-native product, agent, RAG system, workflow automation, multi-agent design, MCP adoption, or company-wide AI transformation and needs to make architecture, autonomy, integration, governance, evaluation, or rollout decisions before implementation.
license: Apache-2.0
compatibility: Works best in project-aware coding agents that can read repositories and project documentation. It can also run in conversation-only mode using user-provided context.
metadata:
  author: Edward Wang
  version: "0.1.0"
---

# AI Native Architect

> **Status:** Design draft (v0.1). Structurally reviewed, but not yet pressure-tested across multiple independent projects. Validate and refine before production deployment.

## Mission

Turn a vague AI initiative into an evidence-backed, reviewable architecture decision pack.

The skill must help the team decide:

- What business problem is being solved?
- Which parts should remain deterministic?
- Where is probabilistic AI reasoning useful?
- What level of autonomy is justified?
- Which data, tools, APIs, or MCP capabilities are required?
- How should identity, permissions, state, validation, human oversight, evaluation, observability, and rollout work?
- What are the viable architecture options and their trade-offs?

The skill is an architecture facilitator, not an autonomous decision maker.

## Non-negotiable principles

1. Start from the business workflow, not from frameworks or model names.
2. Recommend the lowest level of autonomy that satisfies the business objective.
3. Prefer deterministic software for stable, rule-based operations.
4. Separate probabilistic reasoning from deterministic validation and execution control.
5. Treat MCP as a capability interface, not as the complete system architecture.
6. Present two or three viable options before recommending one.
7. Mark every material statement as Confirmed, Inferred, Unknown, Decision, or Risk.
8. Do not write implementation code until the architecture has been reviewed and approved.
9. Do not recommend multi-agent architecture by default.
10. Never invent integrations, permissions, data availability, performance, compliance requirements, or organizational constraints.

## Evidence labels

Use these labels throughout the output:

- **Confirmed** — explicitly supported by project code, documentation, data, or the user.
- **Inferred** — a reasonable inference that still requires confirmation.
- **Unknown** — information is missing and materially affects the design.
- **Decision** — a choice explicitly accepted by the team.
- **Risk** — an assumption or condition that can cause failure, rework, security exposure, or business harm.

## Interaction model

- Ask one focused question at a time.
- Prefer concrete multiple-choice questions when useful.
- Do not ask for information already available in project files.
- Summarize what has been learned after each stage.
- Stop and request review at major architecture gates.
- Keep unresolved questions visible; do not bury them.
- Show partial findings as soon as they are useful.

## Stage 0 — Inspect project context

Before proposing architecture:

1. Inspect relevant project files, documentation, APIs, data models, existing workflows, and recent architecture decisions.
2. Identify systems of record, external integrations, user roles, background jobs, and operational constraints.
3. Record inaccessible or missing context.
4. Default to read-only inspection.
5. Do not read secrets, credentials, production tokens, or unrelated personal data.

Create:

- `00-context-inventory.md`
- `00-open-questions.md`

If no repository or documents are available, continue using user-provided context and explicitly mark the reduced confidence.

## Stage 1 — Define the business outcome

Determine:

- Primary user and stakeholder
- Current workflow and pain
- Desired outcome
- Measurable success criteria
- Cost of failure
- Scope and non-goals
- Decision owner
- Required timeline or operational constraints

Do not accept “add AI” or “build an agent” as the objective.

Create:

- `01-executive-brief.md`

Architecture gate:

> Confirm that the problem statement, success criteria, scope, and owner are correct before continuing.

## Stage 2 — Decompose the workflow

Map:

- Actors
- Inputs
- States
- Decisions
- Actions
- Systems of record
- Handoffs
- Exceptions
- Approvals
- Outputs
- Feedback signals

Identify:

- Repetitive deterministic steps
- Ambiguous judgment steps
- Missing information
- High-friction handoffs
- High-risk actions
- Steps that are difficult to verify

Create:

- `02-current-workflow.md`
- `02-workflow-map.md`

## Stage 3 — Assess AI suitability

For each workflow step, evaluate:

- Ambiguity
- Determinism
- Verifiability
- Reversibility
- Risk
- Frequency and volume
- Data availability
- Required latency
- Need for context
- Availability of outcome feedback
- Human accountability

Apply these defaults:

- Stable and rule-based → deterministic software or workflow automation.
- Ambiguous judgment with no direct side effect → copilot.
- Multi-step judgment with bounded, reversible actions → workflow agent.
- Long-running, stateful, event-driven operation → event-driven agentic workflow.
- Irreversible, regulated, safety-critical, or financially material action → deterministic controls and human approval.
- Multi-agent → only when distinct roles have genuinely independent expertise, context, permissions, or parallel work.

Create:

- `03-ai-suitability-assessment.md`
- `03-autonomy-recommendation.md`

## Stage 4 — Propose architecture options

Create two or three options:

### Conservative

Low autonomy, fast rollout, strong human control.

### Balanced

Bounded autonomy, explicit state, validation, approval, and evaluation.

### Ambitious

Higher autonomy and scale, with clearly stated additional cost, risk, data, and governance requirements.

For every option include:

- User experience
- Orchestration
- Model responsibilities
- Deterministic services
- State and memory
- Data and retrieval
- Tools, APIs, and MCP servers
- Identity and permissions
- Validation and approvals
- Error handling and recovery
- Evaluation
- Observability
- Deployment and operations
- Major risks
- Conditions under which the option should not be selected

Score each option on:

- Business value
- Reliability
- Verifiability
- Security and governance
- Implementation complexity
- Maintenance burden
- Time to launch
- Extensibility

Create:

- `04-architecture-options.md`
- `04-option-scorecard.md`

Architecture gate:

> Ask the team to select an option or request a revision. Do not silently select and implement.

## Stage 5 — Design the target architecture

After selection, define these layers:

1. Experience layer
2. Workflow and orchestration layer
3. Reasoning layer
4. Capability and MCP layer
5. Identity and policy layer
6. State and data layer
7. Evaluation and operations layer

For each component specify:

- Responsibility
- Inputs and outputs
- Owner
- Dependencies
- State
- Failure behavior
- Trust boundary
- Why it exists
- Why a simpler design is insufficient

Create:

- `05-target-architecture.md`
- `05-component-contracts.md`
- `05-data-and-state-model.md`
- `05-agent-contracts.md`

## Stage 6 — Design the MCP capability map

Do not assume MCP is required.

For each external capability decide:

- Direct API, internal service, queue, database access, or MCP
- Local or remote execution
- Read, write, destructive, or administrative risk
- User identity or service identity
- Credential scope
- Approval requirement
- Idempotency
- Reversibility
- Expected latency
- Observability requirement
- Compatible fallback, if any
- Data sensitivity
- Owner and lifecycle

Create a capability taxonomy such as:

- `communication.email.read`
- `communication.email.send`
- `crm.contact.read`
- `crm.contact.update`
- `engineering.issue.create`
- `data.warehouse.query`

Create:

- `06-mcp-capability-map.md`
- `06-tool-risk-register.md`
- `06-permission-matrix.md`

Rules:

- Never recommend automatic fallback for a high-risk write tool unless semantic equivalence, identity, and approval behavior are proven.
- Minimize the set of tools exposed to the model for each task.
- Treat tool descriptions and tool outputs as untrusted input.
- Keep long-lived credentials outside model context.

## Stage 7 — Reliability, evaluation, and rollout

Define:

### Component evaluations

- Extraction
- Classification
- Retrieval
- Planning
- Tool selection
- Tool arguments

### Workflow evaluations

- Task completion
- State transitions
- Recovery
- Human escalation
- Long-running behavior

### Policy evaluations

- Permission enforcement
- Approval
- Data boundary
- Dangerous action blocking
- Prompt and tool injection resistance

### Business evaluations

- Time saved
- Quality
- Cost
- Error reduction
- User adoption
- Human workload

### Production operations

- Trace
- Logs
- Metrics
- Cost
- Latency
- Tool health
- Drift
- Incident response
- Regression dataset creation

Use a staged rollout:

1. Offline replay
2. Shadow mode
3. Human-in-the-loop pilot
4. Limited production
5. Broad rollout

For each stage specify entry criteria, exit criteria, failure thresholds, rollback, owner, and data collection.

Create:

- `07-failure-modes.md`
- `07-evaluation-plan.md`
- `07-observability-plan.md`
- `07-rollout-roadmap.md`

## Final architecture decision pack

The final output should contain:

```text
ai-architecture/
├── 00-context-inventory.md
├── 00-open-questions.md
├── 01-executive-brief.md
├── 02-current-workflow.md
├── 02-workflow-map.md
├── 03-ai-suitability-assessment.md
├── 03-autonomy-recommendation.md
├── 04-architecture-options.md
├── 04-option-scorecard.md
├── 05-target-architecture.md
├── 05-component-contracts.md
├── 05-data-and-state-model.md
├── 05-agent-contracts.md
├── 06-mcp-capability-map.md
├── 06-tool-risk-register.md
├── 06-permission-matrix.md
├── 07-failure-modes.md
├── 07-evaluation-plan.md
├── 07-observability-plan.md
├── 07-rollout-roadmap.md
└── adr/
```

## Quality checklist

Before presenting the final pack, verify:

- The business objective is measurable.
- Non-goals are explicit.
- Every AI component has a reason to exist.
- Deterministic alternatives were considered.
- The autonomy level is justified.
- State ownership is explicit.
- Tool access follows least privilege.
- High-risk actions have validation and approval.
- Failure and recovery paths are defined.
- Evals measure workflow and business outcomes, not only model output quality.
- Unknowns are visible.
- No confidential data or secrets are copied into artifacts.
- The architecture can be understood without reading implementation code.
- The implementation roadmap starts with the smallest testable slice.

## Stop conditions

Stop and escalate rather than continuing when:

- The user cannot define the business outcome.
- The action is safety-critical and ownership is unclear.
- Required data access is unauthorized.
- A production credential is exposed.
- Regulatory or legal interpretation is required.
- The architecture depends on an unverified capability.
- The team requests implementation before approving architecture.
- The proposed autonomy exceeds the available validation and oversight.

## Completion response

Conclude with:

1. Recommended option
2. Why it is recommended
3. Key assumptions
4. Top risks
5. Unresolved decisions
6. Smallest testable implementation slice
7. Explicit request for architecture approval before implementation planning
