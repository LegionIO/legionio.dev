---
title: "LLM Routing Quickstart"
parent: Getting Started
nav_order: 3
description: "Multi-provider LLM routing with three-tier model escalation and Tier 0 cached bypass. 10 minutes."
---

# LLM Routing Quickstart

**Time:** 10 minutes
**You'll see:** Multi-provider routing, three-tier model escalation, and Tier 0 cached bypass that learns to skip the LLM entirely.
**Prerequisites:** Ruby >= 3.4, at least one LLM provider API key

{: .note }
> This quickstart shows legion-llm's routing intelligence. By the end, you'll see how LegionIO routes across providers, escalates between model tiers, and learns which operations can skip the LLM entirely.

<!-- TODO: Fill in with tested end-to-end walkthrough -->
<!-- This is the #3 priority quickstart — LLM power users frustrated with single-provider lock-in -->

## Step 1: Install

```bash
gem install legionio
```

## Step 2: Configure a Provider

Create a settings file with your API key:

```bash
mkdir -p ~/.legionio/settings
```

```json
// ~/.legionio/settings/llm.json
{
  "llm": {
    "provider": "anthropic",
    "api_key": "env://ANTHROPIC_API_KEY"
  }
}
```

The `env://` prefix resolves environment variables. You can also use `vault://secret/path#key` for Vault-stored keys.

## Step 3: Multi-Provider Chat

```bash
legion chat
```

LegionIO routes across providers automatically. Configure multiple providers and the framework handles failover, load balancing, and cost optimization.

## Step 4: Three-Tier Model Escalation

LegionIO uses three tiers for model selection:

| Tier | Description | Example |
|:-----|:-----------|:--------|
| Local | Ollama or local models | Fast, free, limited capability |
| Fleet | Shared/managed models | Mid-tier, balanced cost/capability |
| Cloud | Full cloud APIs | Most capable, highest cost |

Simple queries route to lower tiers. Complex queries escalate automatically. Failed responses trigger re-routing to a higher tier.

## Step 5: Tier 0 — Cached Bypass

This is where it gets interesting. Tier 0 sits *below* the local tier:

- The MCP observer watches tool usage patterns
- When it sees repeated operations, it caches the response
- On subsequent identical operations, it returns the cached result **without calling any LLM**

Try it: ask the same question twice. The second time, the response is instant — Tier 0 recognized the pattern and bypassed the model entirely.

## What Just Happened?

You used an LLM integration that:
- Routes across multiple providers with automatic failover
- Escalates between model tiers based on query complexity
- **Learns** which operations can skip the LLM entirely

This isn't just a client library. It's intelligent routing that gets smarter over time.

## What's Next

- [Architecture Overview]({% link architecture.md %}) — understand the LLM integration layer
- [Extension Catalog]({% link extensions.md %}) — see all 7 LLM provider extensions
- [Enterprise Overview]({% link enterprise.md %}) — security and credential management
- [Cognitive Agent Quickstart]({% link getting-started/quickstart-agent.md %}) — see the full cognitive architecture
