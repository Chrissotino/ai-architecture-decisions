# ADR-001: Choosing MCP as the Orchestration Layer

## Status

✅ Accepted

## Date

2025-01-15

## Context

Our enterprise AI platform requires a standardised way for AI agents to interact with external tools, APIs, data sources, and services. As we scale agentic systems across multiple business units, we face the following challenges:

- **Fragmentation**: Different teams were building custom tool-calling integrations in incompatible ways, leading to duplicated effort and inconsistent behaviour
- **Security**: Ad-hoc integrations lacked standardised authentication, permission scoping, and audit logging
- **Maintainability**: Every new tool integration required bespoke code for each agent and each deployment target
- **Interoperability**: We needed agents built with different frameworks (LangChain, Semantic Kernel, AutoGen) to share the same tool ecosystem without reimplementing connectors
- **Governance**: Enterprise compliance required a single, auditable channel through which agents access external resources

We evaluated several approaches: custom REST-based tool calling, LangChain tool abstractions, OpenAI function calling, and the **Model Context Protocol (MCP)** — an open protocol introduced by Anthropic for structured, secure agent-to-tool communication.

---

## Decision

We adopt **Model Context Protocol (MCP)** as the standard orchestration layer for all agent-to-tool interactions across our enterprise AI platform.

All tool servers (file systems, databases, APIs, Microsoft 365, Azure services) will expose MCP-compatible interfaces. All agents will communicate with tools exclusively through the MCP protocol rather than direct API calls.

---

## Rationale

MCP was selected for the following reasons:

**Standardisation**: MCP provides a universal interface contract. Any agent that speaks MCP can use any MCP server without custom integration code, dramatically reducing the surface area of integration work.

**Security model**: MCP enforces explicit capability declarations. Each MCP server announces exactly what operations it supports and what data it can access, enabling fine-grained permission scoping at the infrastructure level — not buried in agent code.

**Ecosystem momentum**: With Anthropic's backing and rapid adoption across the AI tooling industry, MCP is emerging as the de-facto standard for agent tool use. Investing in MCP aligns with industry direction and reduces long-term lock-in risk.

**Framework agnosticism**: MCP servers can be consumed by Claude, LangChain agents, Semantic Kernel plugins, and AutoGen agents using the same server implementation, enabling our multi-framework environment to converge on a single tool layer.

**Auditability**: All tool invocations via MCP are structured and loggable at the protocol level, providing a clean audit trail that satisfies enterprise governance and compliance requirements.

**Local and remote execution**: MCP supports both local (stdio) and remote (HTTP/SSE) server modes, giving us flexibility for both on-premise and cloud-hosted tool servers.

---

## Consequences

### Positive

- Unified tool ecosystem usable across all agent frameworks in the organisation
- Dramatically reduced integration effort when onboarding new tools
- Improved security posture through standardised capability scoping
- Cleaner audit trails for compliance reporting
- Easier onboarding for new developers — one protocol to learn

### Negative / Trade-offs

- MCP is a relatively new protocol; tooling and community resources are still maturing
- Teams must invest time in building or adopting MCP server wrappers for existing internal APIs
- Runtime performance adds a protocol translation layer compared to direct API calls (measured at <10ms overhead — acceptable for our use cases)
- Local MCP server management requires operational investment

### Neutral

- Agents must be updated to use MCP clients; this is a one-time migration cost
- We will maintain a private MCP server registry for internal enterprise tools

---

## Alternatives Considered

| Option | Reason Not Chosen |
|--------|-------------------|
| **LangChain tools** | Framework-specific; not usable by Semantic Kernel or AutoGen agents without re-implementation |
| **OpenAI function calling** | Provider-specific and does not provide a transport or server protocol — only a schema format |
| **Custom REST adapters** | High maintenance burden; no standardised security model; not scalable across teams |
| **Direct SDK calls per agent** | No separation of concerns; agents tightly coupled to tool implementations; poor governance |

---

## References

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [MCP GitHub Repository](https://github.com/modelcontextprotocol)
- [ADR-002: Claude AI as Primary LLM](ADR-002-claude-ai-as-primary-llm.md)
- [ADR-003: Human-in-the-Loop Validation Strategy](ADR-003-human-in-the-loop-validation-strategy.md)
