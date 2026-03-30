---
title: Settings Reference
nav_order: 9
description: "Complete reference for every configurable setting in LegionIO and its core gems."
---

# Settings Reference

LegionIO uses JSON configuration files stored in `~/.legionio/settings/`. Each subsystem gets its own file (e.g., `transport.json`, `data.json`). Settings are loaded by `legion-settings` in priority order:

1. `/etc/legionio/settings/` (system-wide)
2. `~/.legionio/settings/` (user)
3. `./settings/` (project-local)
4. Environment variables via `env://VAR_NAME` URI scheme
5. Vault secrets via `vault://path#key` URI scheme

Higher-priority sources override lower ones. Generate starter configs with:

```bash
legion config scaffold            # minimal templates
legion config scaffold --full     # full templates with all options
legion config validate            # check config syntax
```

---

## transport

RabbitMQ AMQP messaging layer. Gem: `legion-transport`.

File: `~/.legionio/settings/transport.json`

### transport.connection

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `host` | string | `"127.0.0.1"` | RabbitMQ server hostname or IP |
| `port` | integer | `5672` | RabbitMQ AMQP port |
| `user` | string | `"guest"` | Authentication username |
| `password` | string | `"guest"` | Authentication password (supports `vault://` and `env://` URIs) |
| `vhost` | string | `"/"` | RabbitMQ virtual host |
| `read_timeout` | integer | `1` | Socket read timeout in seconds |
| `heartbeat` | integer | `30` | AMQP heartbeat interval in seconds |
| `automatically_recover` | boolean | `true` | Auto-reconnect on connection loss |
| `continuation_timeout` | integer | `4000` | RPC continuation timeout in milliseconds |
| `network_recovery_interval` | integer | `1` | Seconds between reconnection attempts |
| `connection_timeout` | integer | `1` | TCP connection timeout in seconds |
| `frame_max` | integer | `65536` | Maximum AMQP frame size in bytes |
| `recovery_attempts` | integer | `100` | Maximum reconnection attempts before giving up |
| `logger_level` | string | `"info"` | Bunny client log level |

### transport.messages

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `encrypt` | boolean | `false` | Encrypt message payloads with cluster secret |
| `ttl` | integer | `nil` | Message time-to-live in milliseconds (nil = no expiry) |
| `priority` | integer | `0` | Default message priority (0-9) |
| `persistent` | boolean | `false` | Persist messages to disk |

### transport.exchanges

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `type` | string | `"topic"` | Exchange type (topic, direct, fanout, headers) |
| `arguments` | hash | `{}` | Additional exchange arguments |
| `auto_delete` | boolean | `false` | Delete exchange when last queue unbinds |
| `durable` | boolean | `true` | Survive broker restarts |
| `internal` | boolean | `false` | Internal-only exchange (no direct publishing) |

### transport.queues

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `manual_ack` | boolean | `true` | Require explicit message acknowledgement |
| `durable` | boolean | `true` | Survive broker restarts |
| `block` | boolean | `false` | Block on queue declare if it already exists |
| `auto_delete` | boolean | `false` | Delete queue when last consumer disconnects |
| `arguments` | hash | `{"x-queue-type": "quorum"}` | Queue arguments (quorum queues recommended) |

### transport.channel

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `default_worker_pool_size` | integer | `1` | Default channel consumer thread pool size |
| `session_worker_pool_size` | integer | `8` | Session channel thread pool size |

### transport (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `type` | string | `"rabbitmq"` | Transport type (only rabbitmq supported) |
| `logger_level` | string | `"info"` | Transport-level log level |
| `prefetch` | integer | `0` | Channel prefetch count (0 = unlimited) |

---

## data

Database persistence via Sequel. Gem: `legion-data`.

File: `~/.legionio/settings/data.json`

### data (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `adapter` | string | `"sqlite"` | Database adapter: `sqlite`, `postgres`, `mysql2` |
| `connect_on_start` | boolean | `true` | Connect to database during boot |

### data.creds

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `database` | string | `"legionio.db"` | Database name or file path |
| `host` | string | — | Database server hostname (PostgreSQL/MySQL) |
| `port` | integer | — | Database server port |
| `user` | string | — | Database username |
| `password` | string | — | Database password (supports `vault://` URIs) |

### data.connection

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `log` | boolean | `false` | Log SQL queries |
| `log_connection_info` | boolean | `false` | Log connection details at startup |
| `log_warn_duration` | integer | `1` | Log queries slower than N seconds as warnings |
| `sql_log_level` | string | `"debug"` | SQL log level |
| `max_connections` | integer | `10` | Connection pool size |
| `preconnect` | boolean | `false` | Pre-establish all connections at startup |

### data.cache

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `auto_enable` | boolean | `false` | Auto-enable Sequel model caching |
| `ttl` | integer | `60` | Model cache TTL in seconds |

### data.migrations

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `continue_on_fail` | boolean | `false` | Continue loading if a migration fails |
| `auto_migrate` | boolean | `true` | Run pending migrations at startup |

### data.models

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `continue_on_load_fail` | boolean | `false` | Continue if a model fails to load |
| `autoload` | boolean | `true` | Auto-load Sequel models after migration |

---

## cache

In-memory caching via Redis or Memcached. Gem: `legion-cache`.

File: `~/.legionio/settings/cache.json`

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `driver` | string | `"dalli"` | Cache driver: `dalli` (Memcached) or `redis` |
| `servers` | array | `["127.0.0.1:11211"]` | Cache server addresses |
| `enabled` | boolean | `true` | Enable caching subsystem |
| `namespace` | string | `"legion"` | Key prefix namespace |
| `compress` | boolean | `false` | Compress cached values |
| `failover` | boolean | `true` | Failover to next server on error |
| `threadsafe` | boolean | `true` | Thread-safe client operations |
| `expires_in` | integer | `0` | Default TTL in seconds (0 = no expiry) |
| `cache_nils` | boolean | `false` | Cache nil values |
| `pool_size` | integer | `10` | Connection pool size |
| `timeout` | integer | `5` | Operation timeout in seconds |

{: .note }
Memcached `value_max_bytes` is set to 8MB by `legion-cache`. The Memcached server also needs the `-I 8m` flag in its launch configuration.

---

## crypt

Encryption, Vault, and JWT. Gem: `legion-crypt`.

File: `~/.legionio/settings/crypt.json`

### crypt (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `cluster_secret` | string | `nil` | Pre-shared cluster secret for message encryption |
| `cluster_secret_timeout` | integer | `5` | Seconds to wait for cluster secret distribution |
| `dynamic_keys` | boolean | `true` | Generate per-node RSA keypairs |
| `save_private_key` | boolean | `true` | Persist private key to disk |
| `read_private_key` | boolean | `true` | Read persisted private key on startup |

### crypt.vault

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable HashiCorp Vault integration |
| `protocol` | string | `"http"` | Vault protocol (`http` or `https`) |
| `address` | string | `"localhost"` | Vault server address |
| `port` | integer | `8200` | Vault server port |
| `token` | string | `nil` | Vault authentication token (supports `env://` URIs) |
| `renewer_time` | integer | `5` | Token renewal interval in seconds |
| `renewer` | boolean | `true` | Enable automatic token renewal |
| `push_cluster_secret` | boolean | `true` | Push cluster secret to Vault |
| `read_cluster_secret` | boolean | `true` | Read cluster secret from Vault |
| `kv_path` | string | `"legion"` | Vault KV mount path |

### crypt.vault.clusters

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `<name>` | hash | — | Named Vault cluster config (same keys as `crypt.vault`). Enables multi-cluster Vault support. |

### crypt.jwt

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable JWT token support |
| `default_algorithm` | string | `"HS256"` | Default signing algorithm |
| `default_ttl` | integer | `3600` | Default token TTL in seconds |
| `issuer` | string | `"legion"` | JWT issuer claim |
| `verify_expiration` | boolean | `true` | Verify token expiration |
| `verify_issuer` | boolean | `true` | Verify issuer claim |

---

## logging

Structured logging. Gem: `legion-logging`.

File: `~/.legionio/settings/logging.json`

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `level` | string | `"info"` | Log level: `debug`, `info`, `warn`, `error`, `fatal` |
| `location` | string | `"stdout"` | Log destination: `stdout`, `stderr`, or file path |
| `trace` | boolean | `true` | Include trace IDs in log output |
| `backtrace_logging` | boolean | `true` | Include backtraces in error logs |

---

## llm

LLM integration and routing. Gem: `legion-llm`.

File: `~/.legionio/settings/llm.json`

### llm (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable LLM subsystem |
| `default_provider` | string | `nil` | Default provider name (e.g., `"anthropic"`, `"bedrock"`) |
| `default_model` | string | `nil` | Default model ID (e.g., `"claude-sonnet-4-20250514"`) |

### llm.providers.bedrock

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable AWS Bedrock provider |
| `api_key` | string | `nil` | AWS access key ID |
| `secret_key` | string | `nil` | AWS secret access key |
| `session_token` | string | `nil` | AWS session token (STS) |
| `region` | string | `"us-east-2"` | AWS region |
| `vault_path` | string | `nil` | Vault path for credential retrieval |

### llm.providers.anthropic

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Anthropic provider |
| `api_key` | string | `nil` | Anthropic API key (supports `env://ANTHROPIC_API_KEY`) |
| `vault_path` | string | `nil` | Vault path for credential retrieval |

### llm.providers.openai

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable OpenAI provider |
| `api_key` | string | `nil` | OpenAI API key |
| `vault_path` | string | `nil` | Vault path for credential retrieval |

### llm.providers.gemini

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Google Gemini provider |
| `api_key` | string | `nil` | Gemini API key |
| `vault_path` | string | `nil` | Vault path for credential retrieval |

### llm.providers.ollama

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Ollama local provider |
| `base_url` | string | `"http://localhost:11434"` | Ollama server URL |

---

## chat

Interactive AI chat REPL settings.

File: `~/.legionio/settings/chat.json`

### chat (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `permissions` | string | `"interactive"` | Tool permission mode: `interactive`, `auto_approve`, `read_only` |
| `model` | string | `nil` | Override default LLM model for chat |
| `provider` | string | `nil` | Override default LLM provider for chat |
| `personality` | string | `nil` | Chat personality style |
| `markdown` | boolean | `true` | Enable terminal markdown rendering |
| `incognito` | boolean | `false` | Disable session persistence |
| `max_budget_usd` | float | `nil` | Maximum spend per session in USD |

### chat.subagent

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `max_concurrency` | integer | `3` | Maximum concurrent background subagents |
| `timeout` | integer | `300` | Subagent timeout in seconds |

### chat.headless

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `max_turns` | integer | `10` | Maximum turns for headless `legion chat prompt` |

### chat.notifications

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `patterns` | array | `[]` | Event patterns to surface as chat notifications (fnmatch format) |

---

## client

Node identity settings. Set automatically but can be overridden.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `name` | string | hostname | Node name used in heartbeats and consumer tags |
| `hostname` | string | auto | Machine hostname |
| `ready` | boolean | `false` | (internal) Set `true` when all subsystems are ready |
| `shutting_down` | boolean | `false` | (internal) Set `true` during graceful shutdown |

---

## api

REST API configuration.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `port` | integer | `4567` | HTTP server port |
| `bind` | string | `"0.0.0.0"` | Bind address |
| `enabled` | boolean | `true` | Enable REST API |

---

## role

Role-based extension filtering. Controls which extensions load at boot.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `profile` | string | `nil` | Role profile: `nil` (all), `core`, `cognitive`, `service`, `dev`, `custom` |
| `extensions` | array | `[]` | Explicit extension list (used with `custom` profile) |

### Role profiles

| Profile | What loads |
|---------|-----------|
| `nil` | All discovered extensions |
| `core` | 14 core operational extensions only |
| `cognitive` | core + all agentic extensions |
| `service` | core + service + other integrations |
| `dev` | core + AI + essential agentic (~20 extensions) |
| `custom` | Only extensions listed in `role.extensions` |

---

## region

Multi-region support for geographically distributed clusters.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `current` | string | auto-detected | Current region (AWS IMDSv2 / Azure IMDS) |
| `primary` | string | — | Primary region for writes |
| `failover` | string | — | Failover region |
| `peers` | array | `[]` | Peer region identifiers |
| `default_affinity` | string | `"prefer_local"` | Routing affinity: `prefer_local`, `require_local`, `any` |
| `data_residency` | hash | `{}` | Per-tenant data residency constraints |

---

## extensions

Per-extension configuration. Each extension gets a key matching its gem name (without `lex-` prefix).

```json
{
  "extensions": {
    "slack": {
      "enabled": true,
      "workers": 4
    },
    "scheduler": {
      "enabled": true
    }
  }
}
```

### Common per-extension keys

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable or disable this extension |
| `workers` | integer | `1` | Number of subscription actor worker threads |

### extensions (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `auto_install` | boolean | `false` | Auto-install missing extensions via `Gem.install` |
| `parallel_pool_size` | integer | `24` | Thread pool size for parallel extension loading |

---

## alerts

Configurable alerting rules engine.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable alerting subsystem |
| `rules` | array | `[]` | Alert rule definitions (pattern matching, count conditions, cooldown) |

---

## telemetry

OpenTelemetry tracing and session analytics.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable OpenTelemetry tracing |
| `endpoint` | string | — | OTLP exporter endpoint |
| `service_name` | string | `"legion"` | Service name for traces |

---

## rbac

Role-based access control. Gem: `legion-rbac`.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable RBAC subsystem |
| `default_policy` | string | `"deny"` | Default policy when no rules match |

---

## gaia

Cognitive coordination layer. Gem: `legion-gaia`.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable GAIA tick cycle |
| `tick_interval` | float | `1.0` | Seconds between cognitive ticks |
| `mode` | string | `"sentinel"` | Default tick mode: `dormant`, `dormant_active`, `sentinel`, `full_active` |

---

## process

Process management settings.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `role` | string | `nil` | Process role for multi-process deployments |

---

## Other flags

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `local_mode` | boolean | `false` | Run without transport (local-only mode) |
| `dev` | boolean | `false` | Enable development mode (extra logging, relaxed timeouts) |

---

## Secret resolution

Any string value in settings can use URI schemes for dynamic resolution:

| Scheme | Example | Description |
|--------|---------|-------------|
| `vault://` | `"vault://secret/data/myapp#api_key"` | Read from HashiCorp Vault KV |
| `env://` | `"env://RABBITMQ_PASSWORD"` | Read from environment variable |
| plain string | `"my-value"` | Used as-is |

Secrets are resolved after `legion-crypt` starts. Fallback chains (arrays) are supported:

```json
{
  "password": ["vault://secret/data/app#password", "env://APP_PASSWORD", "default-value"]
}
```

---

## Extension-specific settings

Many extensions define their own settings sections. These are configured under the extension's key in the `extensions` hash or as top-level keys.

### lex-synapse

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `lex-synapse.proposals.enabled` | boolean | `true` | Enable proposal generation |
| `lex-synapse.proposals.reactive` | boolean | `true` | Generate proposals on signal evaluation |
| `lex-synapse.proposals.proactive` | boolean | `true` | Generate proposals on periodic analysis |
| `lex-synapse.proposals.max_per_run` | integer | `5` | Maximum proposals per proactive run |
| `lex-synapse.challenge.enabled` | boolean | `true` | Enable adversarial challenge pipeline |
| `lex-synapse.challenge.impact_threshold` | float | `0.3` | Minimum impact score for LLM challenge |
| `lex-synapse.challenge.auto_accept_threshold` | float | `0.85` | Score threshold for auto-accept |
| `lex-synapse.challenge.auto_reject_threshold` | float | `0.15` | Score threshold for auto-reject |
| `lex-synapse.challenge.outcome_observation_window` | integer | `50` | Signals to observe after application |
| `lex-synapse.challenge.max_per_cycle` | integer | `10` | Maximum challenges per cycle |

### lex-extinction

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `lex-extinction.stale_timeout` | integer | `3600` | Seconds before a protocol is considered stale |
| `lex-extinction.auto_deescalate` | boolean | `true` | Auto-deescalate stale protocols |
| `lex-extinction.governance_required` | boolean | `true` | Require governance gate for full termination |
| `lex-extinction.archive_before_terminate` | boolean | `true` | Archive agent state before termination |

### lex-scheduler

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `scheduler.lock_ttl` | integer | `2` | Distributed lock TTL in seconds |

### lex-mesh

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mesh.silence_timeout` | integer | `30` | Seconds before marking an agent offline |
| `mesh.max_hops` | integer | `3` | Maximum message routing hops |

### lex-apollo

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `apollo.confidence_threshold` | float | `0.1` | Minimum confidence before archival |
| `apollo.corroboration_boost` | float | `0.3` | Confidence boost on corroboration |
| `apollo.decay_rate` | float | `0.998` | Hourly confidence decay multiplier |
| `apollo.novelty_threshold` | float | `0.3` | Minimum novelty for write path |

---

## DNS Bootstrap

LegionIO can bootstrap settings from DNS TXT records:

```
legion-bootstrap.<domain> TXT "transport.host=rabbitmq.example.com"
```

Results are cached to `~/.legionio/settings/_dns_bootstrap.json`.

---

## Environment variable bootstrap

Set `LEGIONIO_BOOTSTRAP_CONFIG` to a file path or URL. The bootstrap config is split into per-subsystem files under `~/.legionio/settings/`.

---

## Config file locations

| Priority | Path | Scope |
|----------|------|-------|
| 1 (lowest) | `/etc/legionio/settings/*.json` | System-wide defaults |
| 2 | `~/.legionio/settings/*.json` | User settings |
| 3 (highest) | `./settings/*.json` | Project-local overrides |
