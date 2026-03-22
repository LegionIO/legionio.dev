---
title: "Cognitive Agent Quickstart"
parent: Getting Started
nav_order: 1
description: "See the cognitive tick cycle in action — an agent that perceives, remembers, dreams, and adapts. 15 minutes."
---

# Cognitive Agent Quickstart

**Time:** 15 minutes
**You'll see:** An agent running the 13-phase tick cycle, having a conversation with memory and mood, then dreaming during idle time.
**Prerequisites:** Ruby >= 3.4, RabbitMQ running

{: .note }
> This quickstart is designed to show you what makes LegionIO different from prompt-loop frameworks. By the end, you'll have seen an agent think, remember, and dream.

## Step 1: Install

```bash
gem install legionio
# or: brew tap LegionIO/tap && brew install legionio
```

## Step 2: Start the Engine

{: .warning }
> `legionio start` requires RabbitMQ to be running. Start it first with `brew services start rabbitmq` or your platform equivalent.

```bash
legionio start
```

Watch the boot sequence — you'll see extensions loading, the tick cycle initializing, and GAIA coming online:

```
$ legionio start
  Using ~/.legionio/settings for settings
  Legion::Settings Loaded
  Legion::Rbac loaded
  Legion::LLM loaded (provider: anthropic, model: claude-sonnet-4-6)
  Legion::Gaia loaded (tick cycle: 13 phases, dream cycle: 7 phases)
  Logging hooks registered for RMQ publishing
  73 extensions loaded with subscription:42,every:18,poll:8,once:3,loop:2
  Starting Legion API on 0.0.0.0:4567
  Started Legion v1.4.107
```

The key lines to notice: GAIA comes up with 13 tick phases and 7 dream phases before any extensions load. The cognitive layer is the foundation, not a plugin.

## Step 3: Chat with a Cognitive Agent

In a separate terminal, open the chat shell:

```bash
legion chat
```

You'll see an ASCII banner, then the prompt. The `you >` prompt is purple in a color terminal.

Here is a session that shows the cognitive architecture in action. First, a basic exchange:

```
you > what do you know about RabbitMQ?

legion > RabbitMQ is an AMQP 0.9.1 message broker. It routes messages between
         producers and consumers using exchanges, queues, and bindings. I use it
         as my primary transport layer — every task, event, and inter-extension
         message flows through it.
```

Now reference something from earlier to see memory retrieval at work:

```
you > remind me what transport layer you mentioned

legion > I mentioned RabbitMQ a moment ago — AMQP 0.9.1. That came up when you
         asked about message brokers. It's central to how I route work between
         extensions. Want me to go deeper on the exchange topology?
```

The agent did not re-query the LLM for prior context. It retrieved the earlier exchange from its memory store, the same memory that gets consolidated during the dream cycle.

Now shift the tone to see emotional valence tracking:

```
you > honestly this whole setup is kind of overwhelming

legion > That's fair. There's a lot of moving parts here — broker, extensions,
         tick phases, dream cycles. It can feel like a lot to hold at once.

         Start with this: the daemon runs, you talk to it, it remembers. The rest
         is machinery you don't have to think about until you need it. What's the
         part that feels most unclear right now?
```

The response shifted register — shorter sentences, more direct, less technical density. That's emotional valence affecting output selection, not a prompt instruction.

You can also watch tool calls happen inline. If you ask something that triggers an extension:

```
you > check if the API is up

[tool] http.get(url: "http://0.0.0.0:4567/status")

legion > The API is responding. Status 200, version 1.4.107, uptime 4m 12s.
```

## Step 4: Watch It Dream

Leave the chat idle for 30-60 seconds. In another terminal, watch the dream log as it writes:

```bash
tail -f LegionIO/logs/dreams/dream-$(date +%Y-%m-%d)_*.md
```

{: .tip }
> Dream journals are written to the daemon's working directory under `LegionIO/logs/dreams/`, not to `~/.legionio/`. Run `tail -f LegionIO/logs/dreams/*.md` from the directory where you launched `legionio start`.

You'll see log lines as each phase runs:

```
[dream] cycle starting
[dream] starting phase: memory_audit
[dream] memory_audit: decayed=12 pruned=3 migrated=1 candidates=4 unresolved=7
[dream] starting phase: association_walk
[dream] association_walk: links_found=9 clusters_formed=2 orphans=1
[dream] starting phase: contradiction_resolution
[dream] contradiction_resolution: conflicts=2 resolved=1 deferred=1
[dream] starting phase: agenda_formation
[dream] agenda_formation: priorities_set=3 dropped=1
[dream] cycle complete: memory_audit, association_walk, contradiction_resolution, identity_entropy, agenda_formation, consolidation_commit, dream_reflection
[dream] journal written to logs/dreams/dream-2026-03-21_143022.md
```

Once the cycle finishes, the journal is written as a markdown file. Here is an excerpt from the first few phases:

```markdown
# Dream Journal — 2026-03-21 14:30:22

## Memory Audit
Reviewed 847 traces. Decayed 12 low-confidence memories. Pruned 3 contradicted
facts. Migrated 1 short-term trace to long-term store. Flagged 4 candidates for
association walk. Left 7 unresolved for deferred review.

## Association Walk
Traversed flagged candidates. Found 9 associative links across 2 semantic
clusters. One orphan trace — "RabbitMQ exchange topology" — added to agenda for
elaboration. Cluster A: transport layer architecture. Cluster B: user confusion
signals around onboarding complexity.

## Contradiction Resolution
Detected 2 conflicts. Resolved 1: earlier confidence in default queue durability
setting corrected against observed behavior. Deferred 1: conflicting latency
expectations under high fanout — insufficient data to resolve.
```

## Step 5: Come Back

Start chatting again. The agent has consolidated what it learned during the session:

```
you > where were we?

legion > We were talking about the setup feeling overwhelming. I flagged that
         during the dream cycle — there's a recurring signal around onboarding
         complexity that I want to address more directly. What would make the
         initial setup feel less like a lot?
```

It went idle with a loose end and woke up with it prioritized. That's not retrieval-augmented generation. That's a sleep cycle.

## What Just Happened?

Every tick, the agent ran 13 cognitive phases — not just "call LLM, return response." It perceived your input, evaluated it emotionally, retrieved relevant memories, checked predictions, selected an action, and reflected on the interaction.

When it went idle, it dreamed — consolidating memories, resolving contradictions, forming new priorities. Not because you told it to. Because that's what brains do.

## What's Next

- [Architecture Overview]({% link architecture.md %}) — understand the tick cycle in depth
- [Philosophy]({% link philosophy.md %}) — why LegionIO is built this way
- [Extension Dev Quickstart]({% link getting-started/quickstart-ruby.md %}) — build your own extension
- [Extension Catalog]({% link extensions.md %}) — browse all 73 extensions
