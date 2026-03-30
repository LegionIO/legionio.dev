---
title: Extensions
nav_order: 5
description: "Browse all 73 LegionIO extensions — cognitive, operational, AI, and service integrations."
---

# Extension Catalog

LegionIO's capabilities come from extensions — Ruby gems named `lex-*` that are auto-discovered at boot. Drop a gem in your Gemfile and it's live.

```bash
# Scaffold a new extension
legion lex create my_extension

# Browse installed extensions
legion lex list

# Search the marketplace
legion marketplace search redis
```

---

## Core Operations

Framework plumbing — task management, scheduling, health monitoring, internal wiring.

| Extension | Description |
|:----------|:-----------|
| lex-tasker | Task execution engine, chaining, priority management |
| lex-synapse | Confidence-scored cognitive routing with autonomy levels |
| lex-scheduler | Cron and interval-based distributed scheduling |
| lex-node | Node registration, health, cluster membership |
| lex-conditioner | Conditional logic evaluation for task routing |
| lex-transformer | Data transformation with LLM engine (structured output, retries) |
| lex-acp | Agent Control Plane — lifecycle, registry, governance |
| lex-codegen | Code generation and scaffolding |
| lex-exec | Shell command execution with sandboxing |
| lex-lex | Extension manager — install, update, inspect extensions |
| lex-webhook | Inbound webhook receiver |
| lex-detect | File system monitoring (full scan, delta scan, observer) |
| lex-telemetry | Metrics collection and reporting |
| lex-prompt | Prompt template management and versioning |
| lex-dataset | Training data management |
| lex-eval | Model evaluation and benchmarking |
| lex-workflow | Multi-step workflow definitions |
| lex-llm-gateway | LLM request routing and load balancing |
| lex-react | Reactive event processing and signal evaluation |

## Cognitive Architecture

13 domain gems modeling distinct aspects of cognition, plus supporting extensions.

| Extension | Description |
|:----------|:-----------|
| lex-agentic-executive | Planning, control, inhibition, working memory, decision-making |
| lex-agentic-attention | Spotlight, switching, salience, gating |
| lex-agentic-memory | Encoding, storage, retrieval, consolidation, decay |
| lex-agentic-affect | Emotion, mood, empathy, somatic markers, reward |
| lex-agentic-inference | Prediction, causation, belief updating (Bayesian) |
| lex-agentic-social | Theory of mind, cooperation, trust, moral reasoning |
| lex-agentic-self | Metacognition, identity, self-model, narrative, personality |
| lex-agentic-learning | Habit, reinforcement, procedural learning |
| lex-agentic-language | Inner speech, narrative, frame semantics |
| lex-agentic-imagination | Creativity, dreaming, mental simulation |
| lex-agentic-homeostasis | Balance, rhythm, energy, fatigue recovery |
| lex-agentic-defense | Bias detection, error monitoring, cognitive immune system |
| lex-agentic-integration | Cross-modal binding, coherence, Global Workspace Theory |
| lex-apollo | Two-tier shared knowledge store (local SQLite+FTS5 + global PostgreSQL+pgvector) |
| lex-dream | Dream cycle orchestration and journal generation |
| lex-mind-growth | Cognitive development — proposer, analyzer, builder, validator |
| lex-reflect | Self-reflection and introspection engine |
| lex-knowledge | Document knowledge base with corpus monitoring and RAG |
| lex-extinction | Agent lifecycle termination governance |
| lex-cortex | Legacy cognitive coordinator (absorbed into legion-gaia) |

## GAIA-Tier Extensions

Higher-order coordination and cognitive infrastructure.

| Extension | Description |
|:----------|:-----------|
| lex-gaia-tick | Tick cycle phase management |
| lex-gaia-channel | Channel abstraction for perception-to-action routing |
| lex-gaia-router | Message routing within the cognitive mesh |
| lex-gaia-dream | Dream cycle coordination |
| lex-privatecore | Privacy enforcement for the cognitive stack |
| lex-tick | Tick cycle orchestrator (required by GAIA) |

## Mesh Networking

| Extension | Description |
|:----------|:-----------|
| lex-mesh | Inter-agent mesh networking — peer discovery, RPC, broadcast, multicast |

## LLM Provider Integrations

Connect to any major LLM provider. Multi-provider routing with automatic failover.

| Extension | Description |
|:----------|:-----------|
| lex-azure-ai | Azure AI Foundry integration |
| lex-bedrock | AWS Bedrock (Claude, Llama, Mistral) |
| lex-claude | Anthropic Claude API (messages, batches, token counting) |
| lex-foundry | Azure AI Foundry models |
| lex-gemini | Google Gemini (content generation, embeddings, caching) |
| lex-openai | OpenAI (chat, images, audio, embeddings, files) |
| lex-xai | xAI Grok models |

## Common Service Integrations

Production-ready integrations with widely-used services.

| Extension | Description |
|:----------|:-----------|
| lex-consul | HashiCorp Consul service mesh |
| lex-github | GitHub API (repos, issues, PRs, actions) |
| lex-http | General HTTP client (Faraday-based) |
| lex-kerberos | Kerberos authentication |
| lex-vault | HashiCorp Vault secrets and PKI |
| lex-tfe | Terraform Enterprise/Cloud API |
| lex-redis | Redis data store |
| lex-s3 | AWS S3 object storage |
| lex-elasticsearch | Elasticsearch search and analytics |
| lex-memcached | Memcached caching |

## Additional Integrations

35 more integrations for specialized use cases.

| Extension | Description |
|:----------|:-----------|
| lex-chef | Chef infrastructure automation |
| lex-jfrog | JFrog Artifactory |
| lex-ssh | SSH remote execution |
| lex-slack | Slack (12 runners, Block Kit, message polling) |
| lex-smtp | Email sending |
| lex-kafka | Apache Kafka messaging |
| lex-jira | Atlassian Jira issue tracking |
| lex-microsoft_teams | Microsoft Teams (Graph API, Bot Framework) |
| lex-todoist | Todoist task management |
| lex-pushbullet | Pushbullet notifications |
| lex-pushover | Pushover notifications |
| lex-twilio | Twilio SMS and voice |
| lex-pagerduty | PagerDuty incident management |
| lex-influxdb | InfluxDB time-series database |
| lex-elastic_app_search | Elastic App Search |
| lex-sleepiq | SleepIQ smart bed integration |
| lex-sonos | Sonos speaker control |
| *...and more* | |

---

## Building Your Own

```bash
# Scaffold
legion lex create weather_checker

# Generated structure:
# lex-weather_checker/
#   lib/legion/extensions/weather_checker/
#     runners/       # Business logic
#     actors/        # Execution modes
#     helpers/       # Shared utilities
#     transport/     # AMQP wiring
#   spec/            # RSpec tests
#   CHANGELOG.md
#   README.md
#   lex-weather_checker.gemspec

# Add components
legion generate runner forecast
legion generate actor poller
```

See the [Extension Dev Quickstart]({% link getting-started/quickstart-ruby.md %}) for a full walkthrough.

## Absorbers

Absorbers are a new extension component type for pattern-matched content acquisition. An absorber watches for URLs, files, or meeting transcripts that match its patterns, extracts content, and ingests it into the Apollo knowledge store.

```bash
# Scaffold an absorber
legion generate absorber my_absorber

# Absorb a URL
legion absorb url https://example.com/doc

# List registered absorbers
legion absorb list
```

See the [Architecture Overview]({% link architecture.md %}) for how absorbers fit into the extension system.
