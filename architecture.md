---
title: Architecture
nav_order: 3
description: "Technical architecture overview of the LegionIO cognitive framework."
---

# Architecture Overview

## System Architecture

```
                          +--------------------------------------+
                          |          LegionIO v1.6.37            |
                          |    CLI  /  REST API  /  MCP  / Chat  |
                          +-----------------+--------------------+
                                            |
              +----------+----------+-------+--------+----------+----------+
              |          |          |       |        |          |          |
          transport    crypt      data   cache    settings    llm       gaia
          (RabbitMQ)  (Vault)  (Sequel) (Redis)  (config)  (pipeline) (tick)
              |          |          |       |        |          |          |
              +----------+----------+-------+--------+----------+----------+
                                            |
                      +---------------------+---------------------+
                      |                     |                     |
                   apollo              19-step LLM           extensions
                (knowledge)             pipeline          (LEX ecosystem)
                      |                     |                     |
              +-------+-------+     +-------+-------+     +------+------+
              |               |     |               |     |             |
          local SQLite    global  governance    tool    18 Core    13 Cognitive
          + FTS5        pgvector  (RBAC,PHI,   dispatch  LEXs      Domains
                                  billing)    (MCP+LEX)            (234 modules)
```

## Core Libraries

| Library | Version | Purpose |
|:--------|:--------|:--------|
| legion-transport | 1.4.11 | RabbitMQ AMQP + optional Kafka, connection pools, spool buffer, tenant topology |
| legion-crypt | 1.4.26 | Encryption, multi-cluster Vault, JWT, SPIFFE/SVID, Kerberos auto-auth, mTLS, cert rotation |
| legion-data | 1.6.16 | Database persistence (SQLite/PostgreSQL/MySQL via Sequel), Extract, AuditRecord, archival |
| legion-cache | 1.3.18 | Two-tier caching (Redis/Memcached with PHI-aware TTL caps, Memory adapter for lite mode) |
| legion-settings | 1.3.24 | Config management, schema validation, secret resolver, DNS bootstrap, absorber defaults |
| legion-llm | 0.5.20 | 19-step LLM pipeline with RBAC, classification, RAG, billing, metering, confidence scoring |
| legion-gaia | 0.9.29 | Cognitive coordination (24-phase tick/dream cycle, channels, router, workflow DSL, proactive messaging) |
| legion-mcp | 0.6.6 | MCP server with 60 tools, Tier 0 behavioral intelligence, pattern learning, self-generating functions |
| legion-apollo | 0.3.5 | Shared knowledge store (local SQLite+FTS5, global pgvector, entity-relationship graph) |
| legion-rbac | 0.2.8 | Role-based access control (Vault-style flat policies, Kerberos/Entra claims mapping) |
| legion-tty | 0.4.40 | Terminal UI (115+ slash commands, daemon-routed chat, raw-mode rendering, onboarding wizard) |
| legion-logging | 1.4.2 | Structured JSON logging, async writer, SIEM export, PII/PHI redaction, event categories |
| legion-json | 1.3.1 | JSON serialization (parse, generate, pretty, fast) |

## The Tick Cycle

Every agent runs a 16-phase cognitive loop every tick:

| Phase | Name | What Happens |
|:------|:-----|:-------------|
| 1 | Sensory Processing | Perceive environment, ingest new inputs |
| 2 | Emotional Evaluation | Evaluate inputs against emotional state |
| 3 | Memory Retrieval | Search episodic and semantic memory for relevant context |
| 4 | Knowledge Retrieval | Query Apollo shared knowledge store (local + global) |
| 5 | Identity Entropy Check | Verify agent identity coherence |
| 6 | Working Memory Integration | Combine inputs, memories, knowledge into working state |
| 7 | Procedural Check | Check for matching procedural patterns (habits) |
| 8 | Prediction Engine | Generate predictions, compare with reality |
| 9 | Mesh Interface | Coordinate with other agents in the mesh |
| 10 | Social Cognition | Update social models of other agents and users |
| 11 | Theory of Mind | Model other agents' beliefs and intentions |
| 12 | Gut Instinct | Fast heuristic evaluation (somatic markers) |
| 13 | Action Selection | Choose action based on all preceding phases |
| 14 | Memory Consolidation | Store new experiences in memory |
| 15 | Homeostasis Regulation | Balance energy, rhythm, and fatigue recovery |
| 16 | Post-Tick Reflection | Metacognitive review of the tick |

## The Dream Cycle

When an agent goes idle, it enters an 8-phase dream cycle:

| Phase | Name | What Happens |
|:------|:-----|:-------------|
| 1 | Memory Audit | Review recent memories for importance |
| 2 | Association Walk | Find connections between unrelated memories |
| 3 | Contradiction Resolution | Resolve conflicting beliefs or memories |
| 4 | Agenda Formation | Form new priorities based on consolidated knowledge |
| 5 | Consolidation Commit | Persist reorganized memories |
| 6 | Knowledge Promotion | Promote dream insights to Apollo shared knowledge |
| 7 | Dream Reflection | Evaluate what the dream cycle produced |
| 8 | Dream Narration | Generate natural-language narrative of the dream cycle |

## Cognitive Domains

234 cognitive modules across 13 domain gems:

| Domain | Modules | What It Models |
|:-------|:--------|:---------------|
| Executive | 23 | Planning, control, inhibition, working memory, decision-making |
| Attention | 24 | Spotlight, switching, salience, gating |
| Memory | 18 | Encoding, storage, retrieval, consolidation, decay |
| Affect | 17 | Emotion, mood, empathy, somatic markers, reward |
| Inference | 27 | Prediction, causation, belief updating |
| Social | 17 | Theory of mind, cooperation, trust, moral reasoning |
| Self | 16 | Metacognition, identity, self-model, narrative, personality |
| Learning | 14 | Habit, reinforcement, procedural learning, adaptation |
| Language | 9 | Inner speech, narrative, frame semantics |
| Imagination | 17 | Creativity, dreaming, mental simulation, prospection |
| Homeostasis | 20 | Balance, rhythm, energy, fatigue recovery |
| Defense | 15 | Bias detection, error monitoring, cognitive immune system |
| Integration | 17 | Cross-modal binding, coherence, Global Workspace Theory |

## Extension System (LEX)

Extensions are Ruby gems named `lex-*`, auto-discovered at boot via `Bundler.load.specs`.

Each extension defines:
- **Runners** — callable functions (business logic)
- **Actors** — execution modes (subscription, polling, interval, one-shot, loop)
- **Absorbers** — pattern-matched content acquisition (URLs, files, meetings)
- **Hooks** — lifecycle interceptors (auth, routing, transformation)
- **Transport** — AMQP exchanges, queues, routing keys
- **Helpers** — shared utilities, client connections

```bash
# Scaffold a new extension
legion lex create my_extension

# Add components
legion generate runner my_runner
legion generate actor my_actor
legion generate tool my_tool
```

## Boot Sequence

Deterministic startup order:

```
Logging -> Settings -> Crypt -> Transport -> Cache -> Data
   -> RBAC -> LLM -> Apollo -> GAIA -> Telemetry -> Extensions -> API
```

Apollo boots between LLM and GAIA so the knowledge store is available for cognitive phase wiring.

Two-phase extension loading: all extensions require + autobuild first, then `hook_all_actors` starts AMQP timers and subscriptions.

Shutdown is the reverse order.

## Four Entry Points

```bash
# CLI — 60+ commands, every one supports --json
legion start
legion task run http.request.get url:https://example.com
legion dashboard

# Interactive AI Chat (routed through daemon 19-step pipeline)
legion chat

# REST API (Sinatra + Puma)
curl http://localhost:4567/api/v1/tasks

# MCP Server (60 tools, Tier 0 behavioral routing)
legion mcp
```

## Synapse: Three-Layer Routing

| Layer | Component | Role |
|:------|:----------|:-----|
| Bones | lex-tasker | Raw task execution and chaining |
| Nerves | lex-synapse | Confidence-scored routing, autonomy levels, auto-revert |
| Mind | GAIA + Apollo | Dream replay, knowledge promotion, shared memory |

Autonomy scales with confidence: **Observe** (0-0.3) -> **Filter** (0.3-0.6) -> **Transform** (0.6-0.8) -> **Autonomous** (0.8-1.0).

Below the Autonomous threshold, Synapse emits proposals and routes them through an adversarial challenge pipeline before execution. Actions that pass challenge are logged with full trace; actions that fail are auto-reverted.

## Apollo: Shared Knowledge

Durable knowledge store for the cognitive mesh. Two tiers: node-local SQLite+FTS5 and global PostgreSQL+pgvector.

- **Local store**: SQLite with FTS5 full-text search, content hash dedup, configurable TTL (5-year default), cosine rerank on embedded entries
- **Global store**: PostgreSQL + pgvector for cross-agent semantic retrieval
- **Entity-relationship graph**: Local SQLite-backed graph with BFS traversal, typed edges, and depth limiting
- **Confidence decay**: Knowledge starts at 0.5, strengthened by corroboration, weakened by time
- **Semantic retrieval**: Cosine similarity over 1024-dimensional embeddings
- **Cross-agent sharing**: Agents interact via RabbitMQ only — no direct DB access
- **Knowledge lifecycle**: candidate -> confirmed -> decayed -> archived
- **Self-knowledge**: Ships with 10 built-in documents about its own architecture, auto-ingested on boot

## LLM Pipeline

Every LLM call routes through a 19-step governance pipeline. Steps are profile-aware — GAIA cognitive calls skip governance, external calls get full enforcement.

| Step | Name | What Happens |
|:-----|:-----|:-------------|
| 1 | Authentication | Verify caller identity |
| 2 | RBAC | Enforce role-based access control |
| 3 | Context Load | Load conversation history |
| 4 | Intent Analysis | Classify request intent and complexity |
| 5 | Router | Select provider/model/tier based on routing rules |
| 6 | Classification | PII/PHI content scanning and classification |
| 7 | GAIA Advisory | Pre-flight enrichment from cognitive layer |
| 8 | RAG Context | Retrieve relevant knowledge from Apollo |
| 9 | MCP Discovery | Discover available tools from MCP servers |
| 10 | Billing | Budget enforcement and cost estimation |
| 11 | Provider Call | Execute LLM inference |
| 12 | Response Normalization | Normalize provider-specific response format |
| 13 | Confidence Scoring | Score response quality (logprobs, heuristics, or caller-provided) |
| 14 | Tool Calls | Dispatch tool calls to MCP or LEX runners |
| 15 | Context Store | Persist conversation state |
| 16 | RAG Guard | Post-generation faithfulness check |
| 17 | Metering | Record token usage and cost |
| 18 | Audit | Publish audit event with full trace |
| 19 | Knowledge Capture | Write research synthesis back to Apollo |

Three caller profiles control which steps run: **external** (all steps), **gaia** (skips governance), **system** (minimal — auth and provider only).

[Full Pipeline Reference]({% link pipeline.md %})
