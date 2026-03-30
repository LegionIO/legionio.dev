---
title: Philosophy
nav_order: 2
description: "Design principles and architecture philosophy for the LegionIO cognitive framework."
---

# Philosophy

## What LegionIO Is

LegionIO is a cognitive architecture for AI agents built in Ruby. Agents run a 16-phase tick cycle modeled on biological neural processing — perceiving, remembering, predicting, deciding, acting, and reflecting every tick. When idle, they enter an 8-phase dream cycle that consolidates memory, resolves contradictions, promotes knowledge, and forms new agendas.

234 cognitive modules organized into 13 domain gems model distinct aspects of cognition: memory that fades, emotions that shift decisions, trust that's earned through interaction, predictions that fail and adapt. Every module is optional. They compose, interact, and adapt when others are absent.

Underneath the cognition is a production-grade async job engine: RabbitMQ messaging, task chaining, distributed scheduling, HashiCorp Vault integration, multi-database support, and a plugin system where extensions are auto-discovered Ruby gems.

Everything is a configuration. Everything is swappable. Everything composes.

## What LegionIO Is Not

LegionIO is not a prompt orchestrator. It is not a wrapper around an LLM API. It is not "call the model in a loop and call it an agent."

Most agent frameworks give you a loop: prompt, tool call, check result, repeat. That's a chatbot with a toolbox. It doesn't think — it waits for you to think for it, one prompt at a time.

LegionIO exists because prompt engineering is a phase, not a destination. If you've built with LangChain, CrewAI, or AutoGen, you know what they're good at: coordinating LLM calls. LegionIO does something fundamentally different. It gives agents an internal cognitive process that runs whether or not an LLM is involved.

## Design Principles

### Agents are partners, not servants

An agent should have its own internal state, its own memory, its own judgment. It should be able to say "I'm not confident about this" or "something feels wrong here" — not because you wrote a safety prompt, but because its trust model detected an anomaly. The partner-not-tool principle shapes every API decision in the framework.

### The tick cycle is the execution model

Every agent runs the same 16-phase tick cycle: sensory processing, emotional evaluation, memory retrieval, knowledge retrieval, identity entropy check, working memory integration, procedural check, prediction engine, mesh interface, social cognition, theory of mind, gut instinct, action selection, memory consolidation, homeostasis regulation, post-tick reflection.

This isn't decorative. Each phase does real computational work. Skipping phases changes behavior. The tick cycle is what makes a LegionIO agent an agent rather than a function that calls an LLM.

### Extensions are the composition model

The framework does not grow by adding features to the core. It grows through extensions — Ruby gems named `lex-*` that are auto-discovered at boot. Each extension defines runners (functions) and actors (execution modes). Drop a gem in your Gemfile and it's live.

This means LegionIO never becomes a monolith. The core stays small. Capabilities are additive and removable.

### RabbitMQ is the communication model

Components talk to each other over AMQP, not HTTP. This is a deliberate choice: message queues provide durability, fan-out, priority, dead-letter handling, and back-pressure that HTTP doesn't. It also means extensions can be written in any language that speaks AMQP — the framework is polyglot at the extension layer.

### Cognition is optional and composable

Every cognitive module can be enabled, disabled, or replaced independently. An agent without the emotion module still functions — it just doesn't have emotional evaluation in its tick cycle. An agent without the dream cycle still works — it just doesn't consolidate memory during idle periods.

This composability is non-negotiable. No module should assume another module exists. No module should break when a peer is absent.

## What We'll Reject

Pull requests that flatten the cognitive model into a generic agent loop. "What if we just called the LLM directly instead of running the tick cycle?" defeats the purpose of the architecture.

Changes that bypass the extension system. If a capability should live in a LEX gem, it doesn't go in the core.

Changes that couple components that should be independent. If module A starts requiring module B to function, the design is wrong.

Changes that remove configurability in favor of hardcoded behavior. If something is a config today, it stays a config.

## What We'll Love

New cognitive modules that model aspects of cognition we haven't covered yet.

New extensions that integrate with services or add capabilities.

Performance improvements that make the existing architecture faster without changing what it is.

Better documentation. Better error messages. Better developer experience.

Bug fixes, always.

Things that make LegionIO more powerful while keeping it composable, configurable, and true to the cognitive architecture model.

## The Ruby Way

LegionIO is built in Ruby on purpose. Matz designed Ruby for developer happiness. DHH built Rails because he was tired of Java ceremony. LegionIO exists because its author got tired of prompt ceremony.

Ruby's expressiveness makes the DSL-style configuration natural. The gem ecosystem makes the extension system elegant. The language's focus on developer experience aligns with LegionIO's focus on agent experience. Ruby isn't just the implementation language — it's part of the philosophy.

## Before You Contribute

Read [CONTRIBUTING.md](https://github.com/LegionIO/.github/blob/main/CONTRIBUTING.md) for workflow and mechanics. Read this page for the why.

If your contribution aligns with these principles, it will get reviewed quickly and merged gratefully. If it doesn't, we'll explain why and point you in a direction that works. We'd rather have 10 contributors who understand the vision than 100 who don't.
