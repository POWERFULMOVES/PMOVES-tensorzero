@AGENTS.md

# PMOVES Integration Overlay

This fork integrates TensorZero as the **canonical LLM gateway** for the PMOVES.AI mesh. The upstream `AGENTS.md` covers all Rust / Python / TypeScript dev workflow; this overlay holds only PMOVES-specific facts that don't belong upstream.

## Service identity in PMOVES

| Field | Value |
|---|---|
| Container name | `tensorzero-gateway` |
| Internal port | `3000` |
| External port (host) | `3030` (per parent `.claude/BOOTSTRAP.md`) |
| OpenAI-compatible base | `http://tensorzero-gateway:3000/openai/v1` |
| Embeddings endpoint | `http://tensorzero-gateway:3000/openai/v1/embeddings` |
| Chat completions | `http://tensorzero-gateway:3000/openai/v1/chat/completions` |
| Observability | ClickHouse on `tensorzero-clickhouse:8123` |
| Config file | `pmoves/configs/tensorzero/tensorzero.toml` (parent repo) |
| Compose target (parent) | `make -C pmoves up-tensorzero` |

**Watch out:** the path is `/openai/v1/...`, NOT `/v1/...`. Anything calling `http://tensorzero:3000/v1/embeddings` is wrong and will 404.

## Embedding model conventions

Production embedding is `qwen3_embedding_4b_local` at **2560 dimensions**. The Qdrant primary collection is `pmoves_chunks_qwen3` (2560d). The 384-d MiniLM collection exists only as a legacy fallback — do not target it for new ingest.

When adding a new embedding model to `tensorzero.toml`:
- Keep dim alignment explicit (Qdrant collection dim must match model output dim)
- Default new models to `weight = 0.0` on first rollout so existing routing isn't disturbed; promote to active weight only after `/health:quick` plus an end-to-end embedding round-trip pass

## Safe-rollout rules

- **`weight = 0.0` on first add** — new models go in with zero routing weight; load test, then promote.
- **Never edit `tensorzero.toml` in production without a parent-repo PR** — the file is templated through the parent's secrets funnel; direct edits get clobbered on next regenerate.
- **Provider keys live in CHIT manifest** — see parent `pmoves/configs/secrets_manifest_v2.yaml`. Do not paste API keys into config files or `.env*` in this fork.

## Cross-repo references

- Parent service file: `pmoves/services/extract-worker/CLAUDE.md` — the canonical gotcha doc for the embedding endpoint path
- Parent context: `.claude/context/tensorzero.md` — full operational reference
- Memory: `project_pmoves_core_vision.md` — TensorZero's role in the hardware-adaptive vision

<!-- PMOVES.AI-CONTEXT-TAGS -->
## PMOVES.AI Skill Hints

**Primary Skills:** `/tensorzero:models`, `/model:load`, `/model:unload`, `/deploy:up`, `/health:quick`
**Context Files:** `tensorzero.md`, `services-catalog.md`
**Domain Tags:** `llm`, `infra`
**Context Tier:** 2 (On-Demand (Major Subsystem))
<!-- /PMOVES.AI-CONTEXT-TAGS -->
