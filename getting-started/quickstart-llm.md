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

---

## Step 1: Install

Install via RubyGems or Homebrew.

**RubyGems:**

```bash
gem install legionio
```

**Homebrew:**

```bash
brew tap legionio/tap
brew install legionio
```

Verify the install:

```
$ legion version
LegionIO v1.6.37
legion-llm  v0.5.20
legion-mcp  v0.6.6
```

---

## Step 2: Configure a Single Provider

Create the settings directory and drop in a configuration file for your provider.

```bash
mkdir -p ~/.legionio/settings
```

```bash
cat > ~/.legionio/settings/llm.json << 'EOF'
{
  "llm": {
    "default_provider": "anthropic",
    "providers": {
      "anthropic": {
        "enabled": true,
        "api_key": "env://ANTHROPIC_API_KEY"
      }
    }
  }
}
EOF
```

The `env://` prefix is a secret resolver: at boot, LegionIO reads `ENV["ANTHROPIC_API_KEY"]` and substitutes the value. You can also use `vault://secret/path#key` to pull credentials from HashiCorp Vault without ever writing a key to disk.

Export your key and start a chat session:

```
$ export ANTHROPIC_API_KEY=sk-ant-...

$ legion chat
LegionIO v1.6.37  |  provider: anthropic  |  model: claude-3-5-haiku-20241022  |  tier: cloud
Type 'exit' to quit.

you > What are the primary benefits of async job queues?

legion > Async job queues decouple the caller from the work. The three main benefits are:

  1. Responsiveness — the caller returns immediately instead of blocking on slow work.
  2. Resilience — jobs survive process restarts; the queue holds them until a worker picks them up.
  3. Scalability — you can add workers independently of the producers without changing any application code.

  RabbitMQ (which LegionIO uses) adds routing keys and exchange bindings on top of this, so you can
  direct work to specific worker pools based on job type.

you > exit
$
```

The provider and model are shown at startup. With a single provider configured, every request goes there directly.

---

## Step 3: Add a Second Provider

Update `llm.json` to declare multiple providers. LegionIO will route across them based on the strategy you choose.

```bash
cat > ~/.legionio/settings/llm.json << 'EOF'
{
  "llm": {
    "default_provider": "anthropic",
    "providers": {
      "anthropic": {
        "enabled": true,
        "api_key": "env://ANTHROPIC_API_KEY"
      },
      "openai": {
        "enabled": true,
        "api_key": "env://OPENAI_API_KEY"
      },
      "ollama": {
        "enabled": true,
        "base_url": "http://localhost:11434"
      }
    },
    "routing": {
      "enabled": true,
      "tiers": {
        "local": { "provider": "ollama" },
        "cloud": { "providers": ["anthropic", "openai"] }
      }
    }
  }
}
EOF
```

The `routing` field controls the selection strategy:

| Strategy | Behavior |
|:---------|:---------|
| `cost_optimized` | Prefer the cheapest provider capable of handling the request |
| `latency_optimized` | Prefer the fastest responding provider |
| `round_robin` | Distribute requests evenly across all healthy providers |

With `cost_optimized` and a local Ollama instance running, short low-complexity queries go to Ollama first. Longer or more complex requests escalate to a cloud provider automatically.

---

## Step 4: Chat and See Routing in Action

Start chat again. The startup line now reflects the active routing strategy.

```
$ legion chat
LegionIO v1.6.37  |  routing: cost_optimized  |  providers: ollama, anthropic, openai
Type 'exit' to quit.

you > What day comes after Tuesday?

legion [ollama/llama3.2] > Wednesday.

you > Summarize the tradeoffs between event sourcing and traditional CRUD storage for a
      distributed system with 50+ microservices, including consistency guarantees,
      operational complexity, and schema evolution.

legion [anthropic/claude-3-5-sonnet-20241022] > Event sourcing and CRUD represent fundamentally
  different contracts with your data. Here is a structured comparison across your three dimensions:

  **Consistency guarantees**
  CRUD systems typically offer strong consistency within a single service boundary using transactions.
  Across 50+ services, you are already in eventual consistency territory regardless of storage model,
  but CRUD encourages local transactions that create an illusion of consistency you cannot actually
  provide end-to-end.

  Event sourcing makes the eventual consistency model explicit. Each service subscribes to the event
  stream and builds its own read model. There is no illusion — downstream state is always derived,
  always lagging slightly, and that is by design.

  [... continues ...]

you > exit
$
```

Notice that the simple factual question was routed to the local Ollama model (fast, free, no API call). The complex multi-part question was escalated to `claude-3-5-sonnet` automatically because the routing layer estimated it exceeded the local model's reliable capability threshold.

**Failover scenario:** If Ollama is not running, the routing layer detects the connection failure and promotes the request to the next available provider without surfacing an error to the caller.

```
$ legion chat
LegionIO v1.6.37  |  routing: cost_optimized  |  providers: ollama (unavailable), anthropic, openai
WARNING: ollama unreachable at http://localhost:11434 — removed from pool
Type 'exit' to quit.

you > What day comes after Tuesday?

legion [anthropic/claude-3-haiku-20240307] > Wednesday.

you > exit
$
```

The cheapest available cloud option fills in for the unavailable local provider. No configuration change required.

---

## Step 5: Tier 0 — Cached Bypass

Tier 0 sits below the local tier. It is not a model — it is the absence of a model call. The MCP observer watches every tool invocation and response. When it sees the same operation repeated with identical inputs, it stores the result. On subsequent identical calls, it returns the cached response directly without touching any LLM provider.

Ask the same question twice to see it in action:

```
$ legion chat
LegionIO v1.6.37  |  routing: cost_optimized  |  providers: ollama, anthropic, openai
Type 'exit' to quit.

you > List the three OSI layers most relevant to load balancer configuration.

legion [ollama/llama3.2] > The three most relevant OSI layers for load balancer configuration are:

  Layer 4 (Transport) — TCP/UDP load balancing based on IP address and port. Fast, no content
  inspection. Used for raw throughput.

  Layer 7 (Application) — HTTP/HTTPS-aware routing. Can inspect headers, paths, and cookies to
  make routing decisions. Supports sticky sessions and health checks based on HTTP status codes.

  Layer 3 (Network) — IP-level routing. Relevant when load balancers participate in BGP or ECMP
  for anycast configurations.

  [tier: local  |  latency: 312ms]

you > List the three OSI layers most relevant to load balancer configuration.

legion [tier-0/cache] > The three most relevant OSI layers for load balancer configuration are:

  Layer 4 (Transport) — TCP/UDP load balancing based on IP address and port. Fast, no content
  inspection. Used for raw throughput.

  Layer 7 (Application) — HTTP/HTTPS-aware routing. Can inspect headers, paths, and cookies to
  make routing decisions. Supports sticky sessions and health checks based on HTTP status codes.

  Layer 3 (Network) — IP-level routing. Relevant when load balancers participate in BGP or ECMP
  for anycast configurations.

  [tier: 0  |  latency: 2ms]

you > exit
$
```

The second response is identical and arrived in 2ms. No provider was contacted.

The MCP observer accumulates these patterns across sessions. Over time it builds a picture of which operations — status checks, factual lookups, deterministic formatting tasks — are stable enough to serve entirely from cache. This is Tier 0: the system learns to do less work, not more.

---

## Three-Tier Model Reference

| Tier | Label | Description | Examples | When Used |
|:-----|:------|:------------|:---------|:----------|
| 0 | Cache | No model call — cached response returned directly | MCP observer pattern store | Repeated identical operations |
| 1 | Local | On-device or local network model | Ollama (llama3.2, mistral, phi3) | Simple queries, low latency required, air-gapped |
| 2 | Fleet | Shared or managed mid-tier model | Internal inference server, OpenAI gpt-4o-mini | Balanced cost and capability |
| 3 | Cloud | Full commercial API | Anthropic claude-3-5-sonnet, OpenAI gpt-4o, Google Gemini 1.5 Pro | Complex reasoning, long context |

Escalation is automatic. A query starts at the lowest capable tier and moves up only when needed.

---

## What Just Happened?

You configured a single provider, then expanded to multi-provider routing, then watched the system:

1. **Route by complexity** — the simple question went to the local model; the complex one escalated to cloud.
2. **Fail over transparently** — when Ollama was unreachable, the next provider in the pool absorbed the load without any configuration change.
3. **Bypass the model entirely** — after seeing the same question once, the MCP observer cached the result and served it from Tier 0 on the second call.

None of this required code changes. The routing intelligence is in the framework. Your application just calls the LLM; LegionIO decides how to fulfill the call.

---

## What's Next

- [Architecture Overview]({% link architecture.md %}) — understand the LLM integration layer in depth
- [Extension Catalog]({% link extensions.md %}) — all 7 LLM provider extensions (Anthropic, OpenAI, Bedrock, Gemini, Azure AI, xAI, Ollama)
- [Enterprise Overview]({% link enterprise.md %}) — `vault://` secret resolvers and credential management
- [Cognitive Agent Quickstart]({% link getting-started/quickstart-agent.md %}) — see the full cognitive architecture built on top of these routing primitives
