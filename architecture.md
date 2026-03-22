---
title: Architecture
nav_order: 3
description: "Technical architecture overview of the LegionIO cognitive framework."
---

# Architecture Overview

## System Architecture

```
                          +--------------------------------------+
                          |          LegionIO v1.4.108           |
                          |    CLI  /  REST API  /  MCP  / Chat  |
                          +-----------------+--------------------+
                                            |
              +----------+----------+-------+--------+----------+----------+
              |          |          |       |        |          |          |
          transport    crypt      data   cache    settings    llm       gaia
          (RabbitMQ)  (Vault)  (Sequel) (Redis)  (config)  (ruby_llm) (tick)
              |          |          |       |        |          |          |
              +----------+----------+-------+--------+----------+----------+
                                            |
                  +-------------------------+-------------------------+
                  |                         |                         |
           18 Core LEXs             13 Cognitive Domains       29 Service LEXs
         (tasker, synapse,        (234 sub-modules:           (slack, redis, http,
          scheduler, node,         memory, emotion,            ssh, s3, vault,
          conditioner...)          trust, prediction...)       github, chef...)
```

## Core Libraries

| Library | Version | Purpose |
|:--------|:--------|:--------|
| legion-transport | 1.2.6 | RabbitMQ AMQP messaging with clustering, TLS, spool buffer |
| legion-crypt | 1.4.4 | Encryption, HashiCorp Vault, JWT, multi-cluster Vault |
| legion-data | 1.4.8 | Database persistence (SQLite/PostgreSQL/MySQL via Sequel) |
| legion-cache | 1.3.3 | Two-tier caching (Redis/Memcached with local fallback) |
| legion-settings | 1.3.8 | Config management, schema validation, secret resolver, DNS bootstrap |
| legion-llm | 0.3.11 | LLM integration with three-tier model escalation |
| legion-gaia | 0.9.8 | Cognitive coordination (tick cycle, dream cycle, channels) |
| legion-mcp | 0.4.0 | MCP server with Tier 0 behavioral intelligence routing |
| legion-rbac | 0.2.3 | Role-based access control (Vault-style flat policies) |
| legion-tty | 0.4.29 | Terminal UI (prompts, tables, spinners, AI chat shell) |
| legion-logging | 1.2.5 | Console + structured JSON logging + SIEM export |
| legion-json | 1.2.0 | JSON serialization (multi_json) |

## The Tick Cycle

Every agent runs a 13-phase cognitive loop every tick:

| Phase | Name | What Happens |
|:------|:-----|:-------------|
| 1 | Sensory Processing | Perceive environment, ingest new inputs |
| 2 | Emotional Evaluation | Evaluate inputs against emotional state |
| 3 | Memory Retrieval | Search episodic and semantic memory for relevant context |
| 4 | Knowledge Retrieval | Query Apollo shared knowledge store |
| 5 | Identity Entropy Check | Verify agent identity coherence |
| 6 | Working Memory Integration | Combine inputs, memories, knowledge into working state |
| 7 | Procedural Check | Check for matching procedural patterns (habits) |
| 8 | Prediction Engine | Generate predictions, compare with reality |
| 9 | Mesh Interface | Coordinate with other agents in the mesh |
| 10 | Gut Instinct | Fast heuristic evaluation (somatic markers) |
| 11 | Action Selection | Choose action based on all preceding phases |
| 12 | Memory Consolidation | Store new experiences in memory |
| 13 | Post-Tick Reflection | Metacognitive review of the tick |

## The Dream Cycle

When an agent goes idle, it enters a 7-phase dream cycle:

| Phase | Name | What Happens |
|:------|:-----|:-------------|
| 1 | Memory Audit | Review recent memories for importance |
| 2 | Association Walk | Find connections between unrelated memories |
| 3 | Contradiction Resolution | Resolve conflicting beliefs or memories |
| 4 | Agenda Formation | Form new priorities based on consolidated knowledge |
| 5 | Consolidation Commit | Persist reorganized memories |
| 6 | Dream Reflection | Evaluate what the dream cycle produced |
| 7 | Dream Narration | Generate a natural-language dream journal |

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
   -> RBAC -> LLM -> GAIA -> Telemetry -> Extensions -> API
```

GAIA boots before extensions so the tick cycle and channel abstraction are ready when cognitive extensions wire their phase handlers.

Two-phase extension loading: all extensions require + autobuild first, then `hook_all_actors` starts AMQP timers and subscriptions.

Shutdown is the reverse order.

## Four Entry Points

```bash
# CLI — 60+ commands, every one supports --json
legion start
legion task run http.request.get url:https://example.com
legion dashboard

# Interactive AI Chat
legion chat

# REST API (Sinatra + Puma)
curl http://localhost:4567/api/v1/tasks

# MCP Server (Model Context Protocol)
legion mcp
```

## Synapse: Three-Layer Routing

| Layer | Component | Role |
|:------|:----------|:-----|
| Bones | lex-tasker | Raw task execution and chaining |
| Nerves | lex-synapse | Confidence-scored routing, autonomy levels, auto-revert |
| Mind | GAIA + Apollo | Dream replay, knowledge promotion, shared memory |

Autonomy scales with confidence: **Observe** (0-0.3) -> **Filter** (0.3-0.6) -> **Transform** (0.6-0.8) -> **Autonomous** (0.8-1.0).

## Apollo: Shared Knowledge

Durable knowledge store for the cognitive mesh. PostgreSQL + pgvector.

- **Confidence decay**: Knowledge starts at 0.5, strengthened by corroboration, weakened by time
- **Semantic retrieval**: Cosine similarity over 1536-dimensional embeddings
- **Cross-agent sharing**: Agents interact via RabbitMQ only — no direct DB access
- **Knowledge lifecycle**: candidate -> confirmed -> decayed -> archived
