---
title: Getting Started
nav_order: 8
has_children: true
description: "Quickstart guides for different audiences — AI builders, Ruby developers, LLM power users."
---

# Getting Started

Pick the path that matches you:

| You are... | Guide | Time |
|:-----------|:------|:-----|
| **An AI/agent builder** | [Cognitive Agent Quickstart]({% link getting-started/quickstart-agent.md %}) | 15 min |
| **A Ruby developer** | [Extension Dev Quickstart]({% link getting-started/quickstart-ruby.md %}) | 10 min |
| **An LLM power user** | [Multi-Provider Routing]({% link getting-started/quickstart-llm.md %}) | 10 min |

Each guide gets you to a working "aha" moment with minimal dependencies.

**Prerequisites for all paths:**
- Ruby >= 3.4 (`ruby --version`)
- RabbitMQ running locally (`brew install rabbitmq && brew services start rabbitmq`)

Or install via Homebrew to skip Ruby setup:
```bash
brew tap LegionIO/tap
brew install legionio
```
