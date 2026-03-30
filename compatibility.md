---
title: Compatibility
nav_order: 6
description: "Tested versions and compatibility matrix for LegionIO."
---

# Compatibility Matrix

## Runtime

| Component | Tested Versions | Notes |
|:----------|:---------------|:------|
| Ruby | 3.4.x, 4.0.x | CI matrix tests both versions |
| Bundler | 2.5+ | Required for gem management |

## Infrastructure

| Component | Tested Versions | Required? | Notes |
|:----------|:---------------|:----------|:------|
| RabbitMQ | 3.12+, 4.x | Yes | AMQP 0.9.1. Transport spool provides offline buffer. |
| PostgreSQL | 14, 15, 16 | Optional | Recommended for production. Required for Apollo (pgvector). |
| MySQL | 8.x | Optional | Supported via Sequel ORM. |
| SQLite | 3.40+ | Optional | Default for development. Ships with Ruby. |
| Redis | 7.x | Optional | For legion-cache. |
| Memcached | 1.6+ | Optional | For legion-cache. 8MB value limit configured. |
| HashiCorp Vault | 1.15+ | Optional | For legion-crypt. Multi-cluster supported. |

## LLM Providers

| Provider | Extension | Models Tested |
|:---------|:----------|:-------------|
| Anthropic | lex-claude | Claude Opus 4.6, Sonnet 4.6, Haiku 4.5 |
| AWS Bedrock | lex-bedrock | Claude, Llama, Mistral via Bedrock |
| OpenAI | lex-openai | GPT-4.1, GPT-4o, o3, o4-mini |
| Google | lex-gemini | Gemini 2.5 Pro, 2.5 Flash |
| Azure AI | lex-azure-ai | Azure-hosted models via Foundry |
| xAI | lex-xai | Grok models |
| Local (Ollama) | legion-llm | Any Ollama-served model |

## Operating Systems

| OS | Status | Notes |
|:---|:-------|:------|
| macOS (Apple Silicon) | Primary development platform | Homebrew formulas available |
| macOS (Intel) | Supported | |
| Linux (x86_64) | Supported | CI runs on Ubuntu |
| Linux (ARM64) | Supported | |
| Windows | Not tested | WSL2 should work but is untested |

## Homebrew

```bash
brew tap LegionIO/tap
brew install legionio-ruby  # Ruby + YJIT + native gems
brew install legionio       # Daemon + pure-Ruby gems
brew install legion-tty     # Terminal UI gems
brew install legion-dev     # Development tools
```

Four-formula split: `legionio-ruby` (Ruby runtime), `legionio` (daemon), `legion-tty` (terminal tools), `legion-dev` (development).
