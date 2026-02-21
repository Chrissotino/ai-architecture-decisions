# ADR-002: Claude AI as Primary LLM

## Status

✅ Accepted

## Date

2025-01-22

## Context

Our enterprise AI platform requires a reliable, high-capability large language model (LLM) to serve as the reasoning core for our agentic workflows. The model must meet stringent enterprise requirements across several dimensions:

**Business requirements:**
- Capable of complex multi-step reasoning required for enterprise workflows (document analysis, code generation, data synthesis, planning)
- Reliable performance consistency — predictable output quality across diverse tasks
- Long context window to support processing of large enterprise documents, codebases, and conversation histories
- Commercially available with enterprise SLA guarantees and data privacy protections

**Technical requirements:**
- Strong instruction-following for agentic tool use and structured output generation
- Excellent performance on coding, analysis, and summarisation benchmarks relevant to our use cases
- Compatible with our MCP orchestration layer (see ADR-001)
- Supports streaming for real-time user interfaces
- Available via API with enterprise tier (Azure or direct Anthropic)

**Compliance and safety requirements:**
- Constitutional AI alignment reduces risk of harmful outputs in enterprise contexts
- Enterprise data privacy: inputs must not be used to train models
- Support for content filtering and output guardrails
- Clear data residency options for regulated industries

We evaluated: **Claude (Anthropic)**, **GPT-4o (OpenAI)**, **Gemini 1.5 Pro (Google)**, **Llama 3.1 (Meta, self-hosted)**, and **Mistral Large**.

---

## Decision

We adopt **Claude (Anthropic)** — specifically **Claude Sonnet** as the primary workhorse and **Claude Opus** for high-complexity tasks — as the primary LLM for all agentic workflows, document processing, and reasoning tasks on our enterprise AI platform.

GPT-4o will be retained as a secondary fallback model for specific use cases where OpenAI function calling compatibility is required by existing integrations.

---

## Rationale

**Instruction following and agentic capability**: Claude demonstrates superior instruction adherence in multi-step agentic workflows, particularly when used with MCP tool servers. Its outputs are consistently structured and reliably follow complex system prompt instructions — critical for automated pipelines where human review occurs only at defined checkpoints.

**Extended context window**: Claude's 200K token context window (Sonnet/Opus) enables processing of full enterprise documents — contracts, technical specifications, codebases — without chunking strategies that introduce reasoning errors.

**Constitutional AI and safety**: Anthropic's Constitutional AI training methodology results in a model that is significantly less prone to generating harmful, misleading, or policy-violating content. For enterprise deployments handling sensitive business data, this reduces moderation overhead and risk exposure.

**Enterprise data privacy**: Anthropic's enterprise agreement guarantees that inputs are not used for model training and offers data residency options. This satisfies GDPR, HIPAA, and internal data governance requirements.

**Benchmark performance on enterprise tasks**: Internal evaluations across our primary task categories (document Q&A, code review, structured data extraction, reasoning over long context) showed Claude Sonnet outperforming GPT-4o on 4 of 6 categories, with comparable performance on the remaining two.

**Microsoft Azure availability**: Claude models are available via Azure AI Services, enabling deployment within our existing Microsoft Azure infrastructure, unified billing, and compliance with our cloud procurement policies.

**MCP-native compatibility**: Anthropic's Claude is the reference implementation for MCP tool use. This ensures first-class compatibility with our orchestration layer (ADR-001) and access to the latest MCP feature capabilities as the protocol evolves.

---

## Consequences

### Positive

- High-quality, consistent outputs for complex enterprise agentic tasks
- Strong safety profile reduces risk of harmful outputs reaching end users
- Extended context supports whole-document workflows without workarounds
- Data privacy compliance built in at the model provider level
- Deployed through Azure AI — no new cloud vendor or procurement process required
- First-class MCP support ensures smooth integration with our orchestration layer

### Negative / Trade-offs

- **Vendor concentration risk**: Primary dependency on Anthropic introduces concentration risk; mitigated by GPT-4o fallback capability and abstraction via MCP
- **Cost**: Claude Opus is priced at a premium tier; usage must be gated appropriately — Sonnet for standard tasks, Opus for complex reasoning only
- **API rate limits**: Enterprise tier rate limits require careful capacity planning for high-throughput use cases
- **No self-hosting option**: Unlike Llama 3.1, Claude cannot be self-hosted, which limits options for air-gapped or fully on-premise deployments

### Neutral

- All agent prompts must be validated against Claude's system prompt best practices — existing GPT-4 prompts will require review and adaptation
- Model selection (Sonnet vs Opus) will be codified in agent configuration, not hardcoded

---

## Model Tier Assignment

| Task Type | Model | Rationale |
|-----------|-------|-----------|
| General reasoning, Q&A, summarisation | Claude Sonnet | Cost-efficient, high quality |
| Complex multi-step planning, long-doc analysis | Claude Opus | Maximum capability for critical tasks |
| High-volume, low-complexity extraction | Claude Haiku | Latency and cost optimisation |
| Legacy function-calling integrations | GPT-4o (fallback) | Compatibility |

---

## Alternatives Considered

| Model | Reason Not Chosen |
|-------|-------------------|
| **GPT-4o (OpenAI)** | Slightly lower benchmark scores on long-context tasks; OpenAI enterprise data privacy terms less favourable; retained as fallback only |
| **Gemini 1.5 Pro (Google)** | Promising context window but inconsistent instruction-following in agentic evals; limited Azure integration |
| **Llama 3.1 (self-hosted)** | Significant operational overhead for self-hosting; lower capability ceiling; considered for future on-premise requirements |
| **Mistral Large** | Insufficient benchmark performance on enterprise task categories; smaller ecosystem |

---

## References

- [Anthropic Claude API Documentation](https://docs.anthropic.com/)
- [Claude on Azure AI](https://azure.microsoft.com/en-us/products/ai-services)
- [ADR-001: Choosing MCP as Orchestration Layer](ADR-001-mcp-as-orchestration-layer.md)
- [ADR-003: Human-in-the-Loop Validation Strategy](ADR-003-human-in-the-loop-validation-strategy.md)
