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

<!-- TODO: Fill in with tested end-to-end walkthrough -->
<!-- This is the #1 priority quickstart — the AI/agent builder audience is the most likely viral vector -->

## Step 1: Install

```bash
gem install legionio
# or: brew tap LegionIO/tap && brew install legionio
```

## Step 2: Start the Engine

```bash
legion start
```

Watch the boot sequence — you'll see extensions loading, the tick cycle initializing, and GAIA coming online.

## Step 3: Chat with a Cognitive Agent

```bash
legion chat
```

Try these to see the cognitive architecture in action:
- Have a conversation and reference something from earlier — the agent remembers
- Ask about something uncertain — watch the confidence level in responses
- Say something emotionally charged — notice how the response tone shifts

## Step 4: Watch It Dream

Let the agent go idle for 30-60 seconds. Watch the logs:

```bash
# In another terminal
tail -f ~/.legionio/logs/dreams/*.md
```

You'll see the dream cycle: memory audit, association walk, contradiction resolution, agenda formation.

## Step 5: Come Back

Start chatting again. The agent may reference something it consolidated during the dream. It went to sleep confused and woke up clearer.

## What Just Happened?

Every tick, the agent ran 13 cognitive phases — not just "call LLM, return response." It perceived your input, evaluated it emotionally, retrieved relevant memories, checked predictions, selected an action, and reflected on the interaction.

When it went idle, it dreamed — consolidating memories, resolving contradictions, forming new priorities. Not because you told it to. Because that's what brains do.

## What's Next

- [Architecture Overview]({% link architecture.md %}) — understand the tick cycle in depth
- [Philosophy]({% link philosophy.md %}) — why LegionIO is built this way
- [Extension Dev Quickstart]({% link getting-started/quickstart-ruby.md %}) — build your own extension
- [Extension Catalog]({% link extensions.md %}) — browse all 73 extensions
