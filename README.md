# TORQ Console Architecture

Architecture documentation for TORQ Console, an execution intelligence platform for AI agents in production.

![TORQ Console Architecture — Four Planes](diagrams/architecture-overview.png)

## What this repository is

This is the public architecture documentation for TORQ Console. The production source code is maintained in a private repository. This repository exists to make the architecture decisions, governance model, and execution pipeline available for review without exposing implementation detail.

The system itself runs in production at [torq-site-pi.vercel.app](https://torq-site-pi.vercel.app). The thesis behind the architecture is documented at [torqbusiness.notion.site/execution-intelligence](https://torqbusiness.notion.site/execution-intelligence).

## The problem TORQ Console addresses

Most AI failures get blamed on the model. The model is rarely the problem.

The problem is the handoff. Decision to execution. Execution to memory. Memory to governance. Governance back to decision. Each step is a place where a system can fail, and most AI systems are built as if those handoffs don't exist.

The handoff failure is the one that does not show on a dashboard. It compounds in the gaps between components, where no single team owns the responsibility, and it kills production deployments months after the demo worked.

TORQ Console was built at the handoff.

## The architecture

Four planes, not one stack.

### Operator Plane

Where humans interact, approve, inspect, and steer. Not as an afterthought. Not as a dashboard bolted onto the side. A first-class surface where intervention is part of the loop.

### Control Plane

The adaptive intelligence layer. Generates routing decisions, predicts outcomes, mutates policies, evaluates results, governs the next iteration. This is where the system learns.

### Execution Plane

Every request is classified for risk, routed to the right model tier, executed, and observed. No request bypasses the pipeline. Route-shadowing is the principle: policy that the request path does not see is not policy, it is paperwork.

### Persistence Plane

Purpose-built tables for decision memory, governance audit, embeddings, and projections. Not a generic database. Each surface holds a specific kind of state, structured for the workload that consumes it.

## The non-negotiable pipeline

Every request flows through the same path:

1. **Risk classification**
2. **Enforcement ladder**
3. **Model routing** across worker, mid, and frontier tiers
4. **Memory and context pipeline**
5. **Telemetry capture**
6. **Outcome feedback** into the learning loop

No request opts out. No human review skips the audit trail. The pipeline is the product.

## Live visualization, not the happy path

The runtime is observable end-to-end. Timeouts, fallbacks, rollback chains, degraded modes, governance approval queues, full decision lineage. Most observability tools show what worked. TORQ shows what almost broke, what was caught, and what was overridden by whom.

## The closed learning loop

Outcomes do not disappear into logs. They feed back into the adaptive layer, which drives predictive routing, strategic interventions, policy mutations, and governance updates. The system gets smarter from every decision. Humans stay on the policy changes with the audit trail backing the approvals.

## What ships beyond the core

- **DefendSwarm** handles high and critical risk paths as a separate integrated component, with its own coordinator workflow and specialist agents.
- **OpenTelemetry** tracing covers the request lifecycle.
- **RAG ingestion and retrieval** are separated as observable flows so failures in one do not poison the other.
- **V5 harness** ties product requirements to runtime measurement, closing the loop between what was promised and what shipped.

## Execution intelligence

This is the category. Not chatbot-with-a-dashboard. Not prompt-engineering-at-scale. Not a model wrapper. A governed intelligence system where the model is one component, the routing is another, the memory is a third, and the governance binds them.

The model is not the product. The control layer is.

## Further reading

- [Why Most AI Systems Fail at the Handoff](https://x.com/DAssetBuzz/status/2054350904866881616) — Long-form architecture explanation with diagram
- [Your Company Doesn't Have a Strategy Problem](https://torqbusiness.notion.site/execution-intelligence) — Thesis on execution intelligence as the constraint on AI deployment
- [TORQ Console — Live System](https://torq-site-pi.vercel.app) — Production demo, pricing, and free trial

## About

TORQ Console is built by Barry Flowers in Austin, Texas, since 2024. Production source code is private. Architecture decisions are documented here.

## License

Architecture documentation released under [MIT License](LICENSE).
