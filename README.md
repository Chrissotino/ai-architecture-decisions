# 📋 AI Architecture Decision Records

> A curated collection of Architecture Decision Records (ADRs) documenting key design choices for enterprise AI projects — covering orchestration layers, LLM selection, agentic workflows, and human-in-the-loop strategies.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ADR](https://img.shields.io/badge/Architecture-Decision%20Records-blue)](https://adr.github.io/)
[![AI Architecture](https://img.shields.io/badge/Focus-Enterprise%20AI-purple)](https://github.com/Chrissotino/ai-architecture-decisions)

---

## 🧭 What Are ADRs?

An **Architecture Decision Record (ADR)** is a document that captures an important architectural decision made along with its context and consequences. ADRs help teams:

- **Preserve institutional knowledge** — decisions are documented with full reasoning, not just the outcome
- **Onboard new team members** — newcomers can understand *why* the system is built the way it is
- **Enable better future decisions** — past tradeoffs are visible when revisiting choices
- **Drive accountability** — decisions are tied to context, date, and stakeholders

---

## 🤖 Why ADRs for AI Projects?

AI projects introduce unique architectural challenges: selecting foundation models, choosing orchestration frameworks, managing non-determinism, ensuring compliance, and integrating with enterprise systems. These decisions have outsized impact on cost, latency, maintainability, and safety.

This repository applies the ADR practice specifically to **AI and agentic system architectures**, covering topics such as:

- Model selection and evaluation criteria
- Orchestration and MCP-based agent frameworks
- Human-in-the-loop (HITL) validation patterns
- Enterprise integration strategies (Microsoft ecosystem, Azure AI)
- Governance, safety, and compliance controls

---

## 📁 Repository Structure

```
ai-architecture-decisions/
├── README.md                          # This file
├── LICENSE                            # MIT License
└── decisions/
    ├── ADR-001-mcp-as-orchestration-layer.md
    ├── ADR-002-claude-ai-as-primary-llm.md
    └── ADR-003-human-in-the-loop-validation-strategy.md
```

---

## 📄 ADR Index

| ID | Title | Status | Date |
|----|-------|--------|------|
| [ADR-001](decisions/ADR-001-mcp-as-orchestration-layer.md) | Choosing MCP as Orchestration Layer | ✅ Accepted | 2025-01-15 |
| [ADR-002](decisions/ADR-002-claude-ai-as-primary-llm.md) | Claude AI as Primary LLM | ✅ Accepted | 2025-01-22 |
| [ADR-003](decisions/ADR-003-human-in-the-loop-validation-strategy.md) | Human-in-the-Loop Validation Strategy | ✅ Accepted | 2025-02-01 |

---

## 📝 ADR Template

Each ADR in this repository follows this structure:

```markdown
# ADR-XXX: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Date
YYYY-MM-DD

## Context
[Describe the situation and forces at play]

## Decision
[State the decision clearly]

## Rationale
[Explain why this decision was made]

## Consequences
[Describe the resulting context after the decision]

## Alternatives Considered
[List options that were evaluated but not chosen]
```

---

## 🏗️ How to Contribute

1. Copy the ADR template above
2. Name your file `ADR-XXX-short-descriptive-title.md`
3. Place it in the `decisions/` folder
4. Add it to the ADR Index table in this README
5. Submit a pull request with a brief description

---

## 📚 Further Reading

- [ADR GitHub Organisation](https://adr.github.io/)
- [Michael Nygard's original ADR article](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [Azure AI Services](https://azure.microsoft.com/en-us/products/ai-services)

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

*Maintained by [Chrissotino](https://github.com/Chrissotino) | AI Architect | Agentic Workflows & MCP | Enterprise AI Solutions*
