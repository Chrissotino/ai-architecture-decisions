# ADR-003: Human-in-the-Loop Validation Strategy

## Status

✅ Accepted

## Date

2025-02-01

## Context

Our enterprise AI platform deploys agentic systems that autonomously execute multi-step workflows involving real-world consequences: sending communications, modifying business data, triggering financial processes, and interacting with enterprise systems (Microsoft 365, ERP, CRM).

Fully autonomous operation introduces unacceptable risk in several scenarios:

**Irreversibility**: Actions such as sending emails, committing database changes, or initiating approval workflows cannot easily be undone. Errors in AI reasoning can cascade through downstream systems before detection.

**Compliance and liability**: Regulated industries (finance, healthcare, legal) require human accountability for decisions — an AI system acting without human review may violate compliance obligations.

**Trust and adoption**: Enterprise users are more comfortable delegating to AI agents when they have clear visibility and control over consequential decisions. Removing human oversight entirely reduces trust and adoption.

**Hallucination risk**: Even high-quality LLMs (including Claude; see ADR-002) can generate plausible but factually incorrect outputs. For high-stakes actions, the cost of an undetected hallucination may be severe.

**Edge case handling**: Production environments surface unexpected inputs and scenarios. Agents that can escalate to human reviewers gracefully are more robust than those that must either fail or proceed with low confidence.

At the same time, requiring human approval for every agent action defeats the efficiency purpose of AI automation. We need a principled strategy that applies human oversight proportionate to action risk.

---

## Decision

We adopt a **tiered, risk-proportionate Human-in-the-Loop (HITL) validation strategy** for all agentic workflows. Every agent action is classified into one of three risk tiers, each with a defined review requirement:

### Tier 1 — Autonomous (No Review Required)

**Criteria**: Read-only operations, reversible low-impact writes, internal data transformations.

**Examples**: Querying databases, retrieving documents, generating draft content (not sent), summarising data, internal logging, updating personal workspace settings.

**Behaviour**: Agent executes immediately with no human intervention. All actions are logged for audit purposes.

### Tier 2 — Notify-and-Proceed (Passive Review)

**Criteria**: Moderate-impact actions that are reversible within a defined window (e.g., 15 minutes) and affect internal stakeholders only.

**Examples**: Saving records to shared systems, creating calendar invites, posting to internal collaboration channels, updating project status fields.

**Behaviour**: Agent executes the action and immediately notifies the responsible human reviewer via Microsoft Teams or email. A cancellation link is provided. If no cancellation is received within the review window, the action is confirmed.

### Tier 3 — Approve-Before-Execute (Active Review Required)

**Criteria**: High-impact, irreversible, or externally visible actions; actions above defined thresholds (financial, data volume, audience size); any action in a regulated workflow.

**Examples**: Sending external emails, initiating financial transactions, publishing content to external channels, modifying access control settings, triggering downstream system integrations (ERP, CRM updates above threshold).

**Behaviour**: Agent prepares the action but does not execute. A structured review request is sent to the designated approver via Microsoft Teams Adaptive Card or email with full action context. The agent waits for explicit approval or rejection. Time-limited approvals expire automatically and escalate if no response is received.

---

## Rationale

**Risk-proportionate design**: Applying HITL universally would negate AI automation benefits; applying it nowhere would introduce unacceptable operational risk. A tiered system optimises for efficiency while maintaining safety at points of real consequence.

**Reversibility as primary classifier**: The most reliable indicator of review need is whether an action can be undone. Irreversible and externally visible actions are treated more conservatively regardless of apparent confidence.

**Microsoft Teams integration**: Our organisation's primary communication and workflow platform is Microsoft Teams. Embedding HITL review flows into Teams (via Adaptive Cards and bot actions) minimises friction — reviewers act from their existing workspace without context switching.

**Explicit approval tokens**: Tier 3 approvals use cryptographically signed tokens tied to the specific action payload. This prevents approval spoofing and ensures the action approved matches the action executed.

**Configurable thresholds**: Financial and audience-size thresholds for Tier 3 elevation are configurable per business unit and workflow, allowing fine-tuning as trust in specific agent types is established over time.

**Audit-first architecture**: All actions across all tiers are logged to a tamper-evident audit log regardless of tier. This supports compliance reporting, incident investigation, and ongoing calibration of risk tier assignments.

**Graceful confidence-based escalation**: Agents are designed to self-assess confidence levels. If an agent's internal reasoning indicates uncertainty (below a defined threshold), it escalates to a higher tier automatically, even if the action type would normally be Tier 1 or Tier 2.

---

## Consequences

### Positive

- Catastrophic autonomous AI errors are prevented at Tier 3 checkpoints
- Compliance requirements for human accountability are satisfied in regulated workflows
- Trust and adoption are accelerated — users feel appropriately in control
- Audit trail supports compliance, debugging, and model improvement
- Reversibility window in Tier 2 provides safety net without friction of full approval
- Confidence-based escalation creates a self-improving feedback loop

### Negative / Trade-offs

- **Latency**: Tier 3 workflows introduce approval latency (minutes to hours depending on reviewer availability). Time-sensitive processes must plan for this
- **Reviewer fatigue**: If tier classification is too conservative, approvers receive too many review requests, leading to rubber-stamping. Tier assignments must be carefully calibrated
- **Teams dependency**: HITL review UI depends on Microsoft Teams availability. Outage scenarios require fallback approval mechanisms (email failover implemented)
- **Implementation complexity**: Building the tiered approval system, Teams integration, and audit infrastructure requires significant initial engineering investment

### Neutral

- Tier assignments will be reviewed and adjusted quarterly based on operational data and incident reports
- Agent developers must classify all new action types and register them in the action registry before deployment

---

## Implementation Notes

### Action Registry

All agent actions must be registered in a central Action Registry with:
- Action ID and description
- Default tier assignment
- Configurable threshold parameters (where applicable)
- Escalation rules
- Approver group mapping

### Approval Flow (Tier 3)

```
Agent → Generate Action Payload
     → Check Action Registry → Assign Tier
     → [Tier 3] Create Signed Approval Token
     → Send Teams Adaptive Card to Approver
     → Await Explicit Approval (timeout: configurable)
     → [Approved] Execute Action → Log Result
     → [Rejected/Timeout] Abort → Notify Agent → Log
```

### Audit Log Schema

Every action event records:
- Timestamp, Agent ID, Action ID, Tier
- Full input/output payload (encrypted for sensitive fields)
- Approver identity and approval timestamp (Tier 2/3)
- Execution result and downstream system confirmation

---

## Alternatives Considered

| Approach | Reason Not Chosen |
|----------|-------------------|
| **Fully autonomous (no HITL)** | Unacceptable risk for irreversible/high-impact actions; compliance violations in regulated workflows |
| **Universal approve-before-execute** | Negates efficiency benefits of AI automation; reviewer fatigue leads to rubber-stamping, which is worse than no review |
| **Confidence-only gating (no tiers)** | Confidence scores are not reliable enough as the sole safety gate; action type and reversibility must be primary classifiers |
| **Separate review application** | Context switching to a dedicated approval app increases friction and reduces review quality; Teams integration is superior for adoption |

---

## References

- [ADR-001: Choosing MCP as Orchestration Layer](ADR-001-mcp-as-orchestration-layer.md)
- [ADR-002: Claude AI as Primary LLM](ADR-002-claude-ai-as-primary-llm.md)
- [Microsoft Teams Adaptive Cards](https://adaptivecards.io/)
- [NIST AI Risk Management Framework](https://www.nist.gov/system/files/documents/2023/01/26/AI%20RMF%201.0.pdf)
- [EU AI Act — High-Risk AI Systems](https://artificialintelligenceact.eu/the-act/)
