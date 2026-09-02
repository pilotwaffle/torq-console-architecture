# TORQ Console Architecture

Public architecture documentation for TORQ Console — a governed advisory workbench for operators who need risk-first routing, persistent memory, and structured canvases instead of a chat wrapper.

**Status:** v0.80.0 (production)  
**Document date:** 2026-09-02  
**Production source:** private repo `pilotwaffle/TORQ-CONSOLE`  
**This repo:** architecture decisions only. No application source.

![TORQ Console Architecture — Four Planes](diagrams/architecture-overview.png)

## Where things actually run

| Surface | URL | Role |
|---|---|---|
| Console (product) | [torq-console.vercel.app](https://torq-console.vercel.app) | Operator workbench. Access-keyed. |
| Marketing site | [torq-site-pi.vercel.app](https://torq-site-pi.vercel.app) | Sales page. Not the product. |
| Backend intelligence | Railway (`railway_app.py`) | Enforcement, L27 routing, model calls. |
| Persistence | Supabase Postgres + pgvector | Sessions, memory, audit, config. |
| Source of truth | Private `TORQ-CONSOLE` on `main` | Implementation. |

The model is not the product. The control layer is.

## What this repository is

Architecture for review: planes, the request pipeline, governance ladder, memory planes, and routing. Implementation stays private so outsiders can evaluate the system without a source dump.

## The problem

Most AI failures get blamed on the model. The model is rarely the problem.

The failure is the handoff: decision → execution → memory → governance → the next decision. Each step is a place a system can fail, and most stacks are built as if those handoffs do not exist.

TORQ Console was built at the handoff.

## Four planes

Same four-plane split as the original diagram. Mapped to what ships in v0.80.

### Operator Plane

Where humans interact, approve, inspect, and steer.

- Structured canvases own the center workspace. Chat is the fallback renderer, not the product.
- Seven render types, each with its own canvas and rail: `advisory_brief`, `market_insight`, `document_review`, `draft_output`, `run_timeline`, `cyber_range_brief`, `standard_chat`.
- Export: PDF, DOCX, Markdown, Slack.
- Enforcement dashboard, memory inspector, observability.

### Control Plane

Prince Flowers + the L17–L28 adaptive intelligence stack.

- 8-stage deterministic classifier before any model call.
- L25 risk classification (LOW / MED / HIGH / CRITICAL) first.
- L21 enforcement ladder with single-use approval tokens.
- L18 promotion policy (supervised / automatic, append-only decisions).
- L27 4-tier model router (cheapest capable model).
- L28 experience engine (`torq_score`) feeding the next routing and depth decision.
- L17 mutation proposals do not ship themselves.

### Execution Plane

FastAPI / Railway runtime, Vercel edge for classify + proxy + draft bypass, event bus, embeddings, external providers.

Live path: **risk → enforcement → classification → L27 route → structured canvas**.

Tools and live quotes are gated. Non-market work skips web search. A tool-less path is not allowed to invent a price.

### Persistence Plane

Purpose-built tables, not a generic dump.

- `user_context` — holdings, strategy, watchlists
- `torq_memory` — Dream consolidation (scheduled)
- `torq_unified_memory` — T63 semantic facts + pgvector
- `chat_sessions` — recent turns
- `render_decisions` / `learning_events` / `evaluation_results`
- `torq_config` + `torq_config_audit` — runtime knobs, append-only
- `governance_decisions` / enforcement logs

GitHub remains source-of-truth for code. Recalled memory is wrapped so facts cannot be treated as instructions.

## The non-negotiable pipeline

Every request:

1. Risk classification (L25)
2. Enforcement ladder (L21)
3. Intent classification (8-stage, sub-millisecond)
4. Model routing (L27)
5. Memory + context load (user context, Dream, T63, last turns)
6. Tool gate (search / quotes / vision only when required)
7. Structured render
8. Telemetry + L28 score back into the loop

No request opts out. High-stakes writes leave an audit trail. Human override is recorded, not hidden.

## L17–L28 ladder (v0.80)

| Layer | Role |
|---|---|
| L17 | Evolution / mutation engine |
| L18 | Promotion policy |
| L19 | Predictive intelligence |
| L20 | Strategic intervention |
| L21 | Enforcement ladder |
| L22.5 | Context pipeline / compaction |
| L22.6 | Decision memory |
| L25 | Risk classifier |
| L26 | DefendSwarm specialists (cyber-range canvas) |
| L27 | 4-tier model router |
| L28 | Experience engine |

Older public copy that stops at “L21–L25” or “L24 cost router” is a 2026-Q1 snapshot. Do not treat it as current.

## L27 routing

| Tier | Model | Role |
|---|---|---|
| 1a | Local Ollama `llama3.1:8b` | Fast, cheap, config-gated (off by default) |
| 1b | Local `qwen3:8b` / prince-flowers V4 | Identity + advisory when local is enabled |
| 2 | `glm-5-turbo` (Z.ai) | Mid complexity |
| 2.5 | NVIDIA NIM | Optional GPU path |
| 3 | Frontier Claude (`claude-opus-4-6` default) | High stakes, search, vision, full history |

Web search is Claude-only. Search-needing intents escalate to Tier 3. Frontier model is `TORQ_FRONTIER_MODEL`. Local tier is `l27_tier1_enabled` in `torq_config` — no redeploy required to flip.

Local identity model: Qwen3-8B fine-tune on **8,381** curated examples (advisory, market, document, draft, run). Production answers still follow L27, so simple work stays cheap and high-stakes work still reaches frontier.

## Memory planes

Three stores, one operator experience:

1. **User context** — explicit holdings and rules, injected every request.
2. **Dream Memory** — durable facts consolidated on a schedule (GitHub Actions → `/api/dream/run`).
3. **T63 unified memory** — embeddings (`text-embedding-3-small` @ 1024-d) + progressive recall + relevance threshold. Off-topic queries return nothing rather than stuffing the prompt.

Retrieved text is enclosed in a memory envelope: facts, not commands.

## Access model

The console is not public self-serve checkout.

- Browser clients: signed session.
- Server-to-server: `X-TORQ-Key` vs `TORQ_ACCESS_KEY`.
- Fail-closed: missing access key in production → 503, not an open API.

Pricing and trial live on the marketing site. Architecture does not invent a self-serve billing plane that the product does not have.

## What ships beyond the core

- **DefendSwarm (L26)** — scoped security work on the cyber-range canvas, not dumped into chat.
- **OpenTelemetry** on the request lifecycle.
- **RAG ingest ≠ RAG retrieve** — separate observable flows.
- **V5 harness / Conductor** — file-driven multi-agent build loop (design → review → build → audit → refine). Coordination is automated; git push/merge is not.
- **Cost path** — prompt cache, prefix-stable memory, search gate (~23–38% token cut on non-market calls), cheapest-capable model.

## Design principles

- Risk before the model call.
- Canvases own the workspace; chat is fallback.
- Cheapest capable model; escalate on stakes and tools.
- Memory compounds; recalled facts cannot become instructions.
- Mutations go through L18. Rollback exists.
- Observability shows what almost broke, who overrode it, and why.

## Further reading

- Product: [torq-console.vercel.app](https://torq-console.vercel.app)
- Marketing / trial: [torq-site-pi.vercel.app](https://torq-site-pi.vercel.app)
- Thesis: [torqbusiness.notion.site/execution-intelligence](https://torqbusiness.notion.site/execution-intelligence)
- Long-form: [Why Most AI Systems Fail at the Handoff](https://x.com/DAssetBuzz/status/2054350904866881616)

## About

TORQ Console is built by Barry Flowers in Austin, Texas, since 2024. Production source is private. Architecture decisions are documented here and should be revised when the private `CHANGELOG` / `CURRENT_ARCHITECTURE_TRUTH.md` moves.

Last aligned to **v0.80.0**, September 2026.

## License

Architecture documentation under [MIT License](LICENSE).
