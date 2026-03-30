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
| `read_timeout` | integer | `3` | Socket read timeout in seconds |
| `heartbeat` | integer | `30` | AMQP heartbeat interval in seconds |
| `automatically_recover` | boolean | `true` | Auto-reconnect on connection loss |
| `continuation_timeout` | integer | `8000` | RPC continuation timeout in milliseconds |
| `network_recovery_interval` | integer | `2` | Seconds between reconnection attempts |
| `connection_timeout` | integer | `10` | TCP connection timeout in seconds |
| `frame_max` | integer | `65536` | Maximum AMQP frame size in bytes |
| `recovery_attempts` | integer | `10` | Maximum reconnection attempts before giving up |
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
| `exclusive` | boolean | `false` | Exclusive to the declaring connection |
| `block` | boolean | `false` | Block on queue declare if it already exists |
| `auto_delete` | boolean | `false` | Delete queue when last consumer disconnects |
| `arguments` | hash | `{"x-queue-type": "quorum"}` | Queue arguments (quorum queues recommended) |

### transport.channel

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `default_worker_pool_size` | integer | `1` | Default channel consumer thread pool size |
| `session_worker_pool_size` | integer | `16` | Session channel thread pool size |

### transport (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `type` | string | `"rabbitmq"` | Transport type (`rabbitmq` or `local` via `LEGION_MODE=lite`) |
| `logger_level` | string | `"info"` | Transport-level log level |
| `prefetch` | integer | `0` | Channel prefetch count (0 = unlimited) |
| `cluster_nodes` | array | `[]` | Additional RabbitMQ cluster nodes (merged with `host` for failover) |
| `connection_pool_size` | integer | `1` | Number of Bunny sessions in the connection pool (1 = single session) |
| `max_payload_bytes` | integer | `1048576` | Maximum message payload size in bytes (1 MB). Raises `PayloadTooLarge` if exceeded |
| `region` | string | `nil` | Region identifier. When set, injects `x-legion-region` header on all published messages |
| `management_port` | integer | `15672` | RabbitMQ Management API port (used by quorum queue policy helper) |

### transport.quorum_queue_policy

Opt-in quorum queue policy applied via the RabbitMQ Management API.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable idempotent quorum queue policy creation |
| `pattern` | string | `"^legion\\."` | Queue name regex pattern for policy scope |
| `delivery_limit` | integer | `5` | Maximum delivery attempts before dead-lettering |

### transport.tenant_topology

Per-tenant exchange/queue isolation. Disabled by default.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable tenant-prefixed topology |
| `prefix_format` | string | `"t.%<tenant_id>s."` | Format string for tenant prefix (Ruby `format` syntax) |
| `shared_exchanges` | array | `["legion.control", "legion.health", "legion.audit"]` | Exchanges that are never tenant-prefixed |
| `auto_provision` | boolean | `true` | Auto-create tenant topology on first use |
| `quotas` | hash | `{}` | Per-tenant rate limits (`messages_per_second`, `bytes_per_second`) |

### transport.tls

TLS settings for RabbitMQ connections. No defaults are defined — TLS is disabled when these keys are absent.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `tls` | boolean | — | Enable TLS for AMQP connection |
| `tls_ca_cert` | string | — | Path to CA certificate file |
| `tls_client_cert` | string | — | Path to client certificate file |
| `tls_client_key` | string | — | Path to client private key file |
| `verify_peer` | boolean | `true` | Verify server certificate (defaults to `true` when not explicitly `false`) |

{: .note }
When `Legion::Crypt::TLS` is available, TLS configuration is resolved through `Crypt::TLS.resolve` instead of these direct settings. Vault PKI mTLS is also supported via `transport.tls.vault_pki: true`.

### transport.kafka

Optional Kafka adapter for event streaming alongside RabbitMQ. Requires `rdkafka` gem (not included by default).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Kafka adapter |
| `brokers` | array | `["127.0.0.1:9092"]` | Kafka broker addresses |
| `consumer_group` | string | `"legion"` | Default consumer group ID |

#### transport.kafka.producer

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `acks` | string | `"all"` | Acknowledgement level (`all`, `1`, `0`) |
| `retries` | integer | `3` | Number of retries on transient failure |
| `retry_backoff_ms` | integer | `100` | Backoff between retries in milliseconds |
| `message_timeout_ms` | integer | `30000` | Message delivery timeout in milliseconds |
| `compression` | string | `"none"` | Compression codec (`none`, `gzip`, `snappy`, `lz4`, `zstd`) |
| `batch_size` | integer | `100` | Maximum messages per batch |
| `linger_ms` | integer | `5` | Delay before sending an incomplete batch |

#### transport.kafka.consumer

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `poll_timeout_ms` | integer | `1000` | Poll timeout in milliseconds |
| `max_poll_interval_ms` | integer | `300000` | Maximum time between polls before rebalance |
| `session_timeout_ms` | integer | `30000` | Consumer session timeout |
| `auto_offset_reset` | string | `"latest"` | Where to start reading (`latest`, `earliest`) |
| `enable_auto_commit` | boolean | `false` | Auto-commit consumer offsets |
| `commit_interval_messages` | integer | `100` | Commit after N messages when auto-commit is off |

#### transport.kafka.admin

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `operation_timeout_ms` | integer | `10000` | Admin operation timeout |

#### transport.kafka.security

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `protocol` | string | `"plaintext"` | Security protocol (`plaintext`, `ssl`, `sasl_plaintext`, `sasl_ssl`) |
| `sasl_mechanism` | string | `""` | SASL mechanism (`PLAIN`, `SCRAM-SHA-256`, `SCRAM-SHA-512`) |
| `sasl_username` | string | `""` | SASL username |
| `sasl_password` | string | `""` | SASL password |
| `ssl_ca_cert_path` | string | `""` | Path to CA certificate |
| `ssl_client_cert_path` | string | `""` | Path to client certificate |
| `ssl_client_cert_key_path` | string | `""` | Path to client private key |

### transport.spool

Disk-based message buffer for when AMQP is unavailable. Spool is configured via method defaults, not the settings hash.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `directory` | string | `"~/.legionio/spool"` | Spool file directory |
| `max_file_bytes` | integer | `10485760` | Maximum bytes per spool file (10 MB) |
| `max_total_bytes` | integer | `524288000` | Maximum total spool size (500 MB) |
| `max_files` | integer | `100` | Maximum number of spool files |
| `max_age_seconds` | integer | `259200` | Stale file eviction age (3 days) |

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

Encryption, Vault, JWT, TLS, and SPIFFE/SVID identity. Gem: `legion-crypt`.

File: `~/.legionio/settings/crypt.json`

### crypt (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `cluster_secret` | string | `nil` | Pre-shared cluster secret for message encryption |
| `dynamic_keys` | boolean | `true` | Generate per-node RSA keypairs |
| `save_private_key` | boolean | `true` | Persist private key to disk |
| `read_private_key` | boolean | `true` | Read persisted private key on startup |

### crypt.vault

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `nil` | Enable Vault integration (when omitted/`nil`, auto-detected from Vault gem presence) |
| `protocol` | string | `"http"` | Vault protocol (`http` or `https`) |
| `address` | string | `"localhost"` | Vault server address |
| `port` | integer | `8200` | Vault server port |
| `token` | string | `nil` | Vault auth token (falls back to `env://VAULT_DEV_ROOT_TOKEN_ID` or `env://VAULT_TOKEN_ID`) |
| `renewer_time` | integer | `5` | Token renewal interval in seconds |
| `renewer` | boolean | `true` | Enable automatic token renewal |
| `push_cluster_secret` | boolean | `true` | Push cluster secret to Vault |
| `read_cluster_secret` | boolean | `true` | Read cluster secret from Vault |
| `kv_path` | string | `"legion"` | Vault KV mount path (overridable via `env://LEGION_VAULT_KV_PATH`) |
| `vault_namespace` | string | `"legionio"` | Vault namespace |
| `leases` | hash | `{}` | Active Vault lease tracking |
| `clusters` | hash | `{}` | Named Vault cluster configs (same keys as `crypt.vault`) |

### crypt.vault.kerberos

Kerberos authentication for Vault.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `service_principal` | string | `nil` | Kerberos service principal for Vault auth |
| `auth_path` | string | `"auth/kerberos/login"` | Vault Kerberos auth mount path |

### crypt.tls

TLS configuration for inter-service communication.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable TLS |
| `verify` | string | `"peer"` | Verification mode (`peer`, `none`) |
| `ca` | string | `nil` | Path to CA certificate |
| `cert` | string | `nil` | Path to client certificate |
| `key` | string | `nil` | Path to client private key |

### crypt.spiffe

SPIFFE/SVID workload identity.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable SPIFFE identity |
| `socket_path` | string | `"/tmp/spire-agent/public/api.sock"` | SPIRE agent socket path |
| `trust_domain` | string | `"legion.internal"` | SPIFFE trust domain |
| `workload_id` | string | `nil` | Explicit workload SPIFFE ID |
| `renewal_window` | float | `0.5` | Fraction of SVID lifetime before renewal |

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

Structured logging with async writer, SIEM export, and PHI redaction. Gem: `legion-logging`.

File: `~/.legionio/settings/logging.json`

### logging (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `level` | string | `"info"` | Log level: `debug`, `info`, `warn`, `error`, `fatal` |
| `format` | string | `"text"` | Output format: `text` or `json` (structured) |
| `log_file` | string | `"./legionio/logs/legion.log"` | File log destination path |
| `log_stdout` | boolean | `true` | Also log to stdout |
| `trace` | boolean | `true` | Include trace IDs in log output |
| `async` | boolean | `true` | Enable async log writer |
| `include_pid` | boolean | `false` | Include process ID in log output |

### logging.transport

Forward log events over AMQP for centralized collection.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable AMQP log forwarding |
| `forward_logs` | boolean | `true` | Forward standard log events |
| `forward_exceptions` | boolean | `true` | Forward exception events |

### logging.async

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `buffer_size` | integer | `10000` | Async writer ring buffer size |

### logging.shipper

Log shipping to external SIEM/aggregation systems.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable log shipping |
| `transport` | string | `"file"` | Transport type: `file` or `http` |
| `batch_size` | integer | `100` | Events per batch before flush |
| `flush_interval` | integer | `5` | Seconds between automatic flushes |
| `levels` | array | `["warn"]` | Minimum levels to ship (lowest in array wins) |
| `endpoint` | string | `nil` | HTTP transport endpoint URL (Splunk HEC, etc.) |
| `auth_token` | string | `nil` | HTTP transport auth token (Bearer or Splunk) |
| `file.path` | string | `"/var/log/legion/siem.log"` | File transport output path |

### logging.redactor

Automatic PHI/PII redaction in log output.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `custom_patterns` | hash | `{}` | Custom regex patterns (`name: regex_string`) merged with built-in patterns |

Built-in patterns redact: SSN, email, phone, MRN, DOB, credit card numbers, Vault tokens, JWTs, bearer tokens, and `vault://`/`lease://` URIs. Sensitive field names (`password`, `secret`, `token`, `api_key`, `authorization`) are always redacted.

---

## llm

LLM integration, routing, and governance pipeline. Gem: `legion-llm`.

File: `~/.legionio/settings/llm.json`

### llm (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable LLM subsystem |
| `pipeline_enabled` | boolean | `true` | Enable the 19-step governance pipeline for every LLM call |
| `default_provider` | string | `nil` | Default provider name (e.g., `"anthropic"`, `"bedrock"`) |
| `default_model` | string | `nil` | Default model ID (falls back to `env://ANTHROPIC_MODEL`) |

### llm.providers.bedrock

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable AWS Bedrock provider |
| `default_model` | string | `"us.anthropic.claude-sonnet-4-6-v1"` | Default Bedrock model ID |
| `api_key` | string | `nil` | AWS access key ID |
| `secret_key` | string | `nil` | AWS secret access key |
| `session_token` | string | `nil` | AWS session token (STS) |
| `bearer_token` | string | `"env://AWS_BEARER_TOKEN_BEDROCK"` | AWS bearer token (alternative auth) |
| `region` | string | `"us-east-2"` | AWS region |

### llm.providers.anthropic

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Anthropic provider |
| `default_model` | string | `"claude-sonnet-4-6"` | Default Anthropic model ID |
| `api_key` | string | `"env://ANTHROPIC_API_KEY"` | Anthropic API key |

### llm.providers.openai

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable OpenAI provider |
| `default_model` | string | `"gpt-4o"` | Default OpenAI model ID |
| `api_key` | string&nbsp;|&nbsp;array | `["env://OPENAI_API_KEY", "env://CODEX_API_KEY"]` | OpenAI API key (fallback chain) |

### llm.providers.gemini

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Google Gemini provider |
| `default_model` | string | `"gemini-2.0-flash"` | Default Gemini model ID |
| `api_key` | string | `"env://GEMINI_API_KEY"` | Gemini API key |

### llm.providers.azure

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Azure OpenAI provider |
| `default_model` | string | `nil` | Azure deployment model ID |
| `api_base` | string | `nil` | Azure OpenAI endpoint base URL |
| `api_key` | string | `nil` | Azure API key |
| `auth_token` | string | `nil` | Azure AD bearer token (alternative to API key) |

### llm.providers.ollama

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable Ollama local provider |
| `default_model` | string | `"llama3"` | Default Ollama model |
| `base_url` | string | `"http://localhost:11434"` | Ollama server URL |

### llm.routing

Dynamic model routing with tiered provider selection and health tracking.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable dynamic routing |
| `default_intent` | hash | `{privacy: "normal", capability: "moderate", cost: "normal"}` | Default request intent for routing decisions |
| `rules` | array | `[]` | Custom routing rules |

#### llm.routing.tiers

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `local.provider` | string | `"ollama"` | Provider for local-tier requests |
| `fleet.queue` | string | `"llm.inference"` | AMQP queue for fleet-tier inference |
| `fleet.timeout_seconds` | integer | `30` | Fleet request timeout |
| `cloud.providers` | array | `["bedrock", "anthropic"]` | Ordered provider preference for cloud tier |

#### llm.routing.health

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `window_seconds` | integer | `300` | Health metric sliding window |
| `circuit_breaker.failure_threshold` | integer | `3` | Failures before opening circuit |
| `circuit_breaker.cooldown_seconds` | integer | `60` | Seconds before retrying a tripped provider |
| `latency_penalty_threshold_ms` | integer | `5000` | Latency above which provider is penalized |
| `budget.daily_limit_usd` | float | `nil` | Daily spend limit (nil = unlimited) |
| `budget.monthly_limit_usd` | float | `nil` | Monthly spend limit (nil = unlimited) |

#### llm.routing.escalation

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable automatic model escalation on low quality |
| `max_attempts` | integer | `3` | Maximum escalation attempts |
| `quality_threshold` | integer | `50` | Minimum quality score before escalating |

### llm.confidence

Confidence scoring bands for LLM responses.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `bands.low` | float | `0.3` | Threshold for low confidence |
| `bands.medium` | float | `0.5` | Threshold for medium confidence |
| `bands.high` | float | `0.7` | Threshold for high confidence |
| `bands.very_high` | float | `0.9` | Threshold for very high confidence |

### llm.rag

Retrieval-Augmented Generation context injection from Apollo knowledge store.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable RAG context injection in the pipeline |
| `full_limit` | integer | `10` | Maximum knowledge entries at full context |
| `compact_limit` | integer | `5` | Maximum knowledge entries at compact context |
| `min_confidence` | float | `0.5` | Minimum confidence for included entries |
| `utilization_compact_threshold` | float | `0.7` | Context utilization ratio to switch to compact mode |
| `utilization_skip_threshold` | float | `0.9` | Context utilization ratio to skip RAG entirely |
| `trivial_max_chars` | integer | `20` | Messages shorter than this are checked against trivial patterns |
| `trivial_patterns` | array | `["hello", "hi", "hey", "ping"]` | Patterns that skip RAG lookup (example set; extend with other trivial greetings as needed) |

### llm.embedding

Vector embedding settings for knowledge store and RAG.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `dimension` | integer | `1024` | Target embedding dimension |
| `enforce_dimension` | boolean | `true` | Reject embeddings that don't match target dimension |
| `provider_fallback` | array | `["ollama", "bedrock", "openai"]` | Ordered provider preference for embedding |
| `provider_models.ollama` | string | `"mxbai-embed-large"` | Ollama embedding model |
| `provider_models.bedrock` | string | `"amazon.titan-embed-text-v2:0"` | Bedrock embedding model |
| `provider_models.openai` | string | `"text-embedding-3-small"` | OpenAI embedding model |
| `ollama_preferred` | array | `["mxbai-embed-large", "bge-large", "snowflake-arctic-embed"]` | Preferred Ollama models (checked in order) |

### llm.prompt_caching

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable prompt caching |
| `min_tokens` | integer | `1024` | Minimum prompt tokens before caching applies |
| `response_cache.enabled` | boolean | `true` | Cache identical responses |
| `response_cache.ttl_seconds` | integer | `300` | Response cache TTL |

### llm.arbitrage

Cost-optimized model selection across providers.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable cost arbitrage |
| `prefer_cheapest` | boolean | `true` | Prefer cheapest model that meets quality floor |
| `quality_floor` | float | `0.7` | Minimum quality score for cheaper alternatives |
| `cost_table_refresh` | integer | `86400` | Cost table refresh interval in seconds (1 day) |
| `cost_table` | hash | `{}` | Custom cost overrides (merged into built-in table) |

### llm.batch

Batch request aggregation for throughput optimization.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable request batching |
| `window_seconds` | integer | `300` | Time window for collecting batch requests |
| `max_batch_size` | integer | `100` | Maximum requests per batch |
| `eligible_intents` | array | `["batch", "background", "low_priority"]` | Intent types eligible for batching |

### llm.scheduling

Off-peak scheduling to reduce costs during peak hours.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable peak-hour deferral |
| `peak_hours_utc` | string | `"14-22"` | Peak hours range in UTC (14-22 = 9 AM-5 PM CT) |
| `defer_intents` | array | `["batch", "background"]` | Intent types deferred during peak |
| `max_defer_hours` | integer | `8` | Maximum hours to defer a request |

### llm.discovery

Automatic provider and model discovery at startup.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable model discovery |
| `refresh_seconds` | integer | `60` | Discovery refresh interval |
| `memory_floor_mb` | integer | `2048` | Minimum free memory for local model loading |

### llm.gateway

Optional centralized LLM gateway endpoint.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Route requests through a gateway |
| `endpoint` | string | `nil` | Gateway URL |
| `api_key` | string | `nil` | Gateway authentication key |
| `timeout_seconds` | integer | `30` | Gateway request timeout |
| `model_policy` | hash | `{}` | Per-model gateway routing policies |
| `headers` | hash | `{}` | Additional headers for gateway requests |
| `fallback_to_direct` | boolean | `true` | Fall back to direct provider on gateway failure |

### llm.daemon

Remote LLM daemon mode (run inference on a separate machine).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable daemon mode |
| `url` | string | `nil` | Daemon endpoint URL |

### llm (code-referenced keys)

These keys are consumed by various modules but have no entry in the defaults hash. They must be explicitly set to take effect.

| Key | Type | Code fallback | Description |
|-----|------|---------------|-------------|
| `shadow.enabled` | boolean | `false` | Enable shadow evaluation (dual-model comparison) |
| `shadow.sample_rate` | float | `0.1` | Fraction of requests to shadow-evaluate |
| `shadow.model` | string | `"gpt-4o-mini"` | Shadow evaluation model |
| `budget.session_usd` | float | `0.0` | Per-session spend limit (0 = no limit) |
| `cost_tracking.auto` | boolean | `true` | Auto-install cost tracking hooks |
| `metering.auto` | boolean | `true` | Auto-install usage metering hooks |
| `response_guards.enabled` | boolean | `false` | Enable response quality guards |
| `rag_guard.threshold` | float | `0.7` | Minimum faithfulness score for RAG responses |
| `rag_guard.evaluators` | array | `[:faithfulness, :rag_relevancy]` | Active RAG evaluators |
| `structured_output.retry_on_parse_failure` | boolean | `true` | Retry on JSON parse failure |
| `structured_output.max_retries` | integer | `2` | Maximum parse-failure retries |
| `compressor.model` | string | `"gpt-4o-mini"` | Model for context compression |

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

Role-based access control with Vault-style flat policies and Entra ID integration. Gem: `legion-rbac`.

File: `~/.legionio/settings/rbac.json`

### rbac (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable RBAC subsystem |
| `enforce` | boolean | `true` | Enforce policy checks (false = log-only mode) |
| `default_local_role` | string | `"admin"` | Role assigned to local/unauthenticated callers |
| `static_assignments` | array | `[]` | Static identity-to-role mappings |
| `route_permissions` | hash | `{}` | Per-route permission overrides |

### rbac.roles

Four built-in roles with resource/action permissions and deny rules:

| Role | Description | Cross-team |
|------|-------------|------------|
| `worker` | Execute assigned runners within team scope | No |
| `supervisor` | Manage workers and schedules within team scope | No |
| `admin` | Full access, cross-team capability | Yes |
| `governance-observer` | Read-only visibility across all teams for audit/compliance | Yes |

### rbac.entra

Microsoft Entra ID (Azure AD) integration for SSO role mapping.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `tenant_id` | string | `nil` | Entra tenant ID |
| `role_map` | hash | see below | App role to RBAC role mapping |
| `group_map` | hash | `{}` | Group to role mapping |
| `default_role` | string | `"worker"` | Default role for authenticated users without a mapping |

Default `role_map`:

```json
{
  "Legion.Admin": "admin",
  "Legion.Supervisor": "supervisor",
  "Legion.Worker": "worker",
  "Legion.Observer": "governance-observer"
}
```

---

## gaia

Cognitive coordination layer with tick/dream cycle, channels, and workflow DSL. Gem: `legion-gaia`.

File: `~/.legionio/settings/gaia.json`

### gaia (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable GAIA tick cycle |
| `heartbeat_interval` | integer | `1` | Seconds between cognitive ticks |

### gaia.channels

Channel adapters for human-agent communication.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `cli.enabled` | boolean | `true` | Enable CLI channel adapter |
| `teams.enabled` | boolean | `false` | Enable Microsoft Teams channel adapter |
| `slack.enabled` | boolean | `false` | Enable Slack channel adapter |

### gaia.router

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable cross-node message routing |
| `allowed_worker_ids` | array | `[]` | Worker IDs permitted to route through GAIA |

### gaia.session

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `persistence` | string | `"auto"` | Session persistence mode: `auto`, `always`, `never` |
| `ttl` | integer | `86400` | Session TTL in seconds (24 hours) |

### gaia.output

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `mobile_max_length` | integer | `500` | Maximum output length for mobile channels |
| `suggest_channel_switch` | boolean | `true` | Suggest switching to a richer channel when output is truncated |

### gaia.notifications

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable proactive notifications |
| `quiet_hours.enabled` | boolean | `false` | Enable quiet hours suppression |
| `quiet_hours.schedule` | array | `[]` | Quiet hours schedule entries |
| `priority_override` | string | `"urgent"` | Priority level that bypasses quiet hours |
| `delay_queue_max` | integer | `100` | Maximum queued notifications during quiet hours |
| `max_delay` | integer | `14400` | Maximum notification delay in seconds (4 hours) |

### gaia.knowledge

Knowledge retrieval settings for GAIA cognitive phases.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `retrieval_limit` | integer | `5` | Maximum knowledge entries per tick |
| `retrieval_min_confidence` | float | `0.3` | Minimum confidence for retrieved entries |
| `memory_retrieval_limit` | integer | `10` | Maximum memory entries per retrieval |
| `memory_audit_limit` | integer | `20` | Maximum entries for memory audit phase |
| `memory_skip_threshold` | float | `0.8` | Context utilization above which memory retrieval is skipped |

---

## apollo

Two-tier shared knowledge store client. Gem: `legion-apollo`.

File: `~/.legionio/settings/apollo.json`

### apollo (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable Apollo knowledge store |
| `transport_mode` | string | `"auto"` | Transport mode: `auto` (detect), `amqp`, `direct` |
| `query_timeout` | integer | `5` | Query timeout in seconds |
| `ingest_timeout` | integer | `10` | Ingest/write timeout in seconds |
| `max_tags` | integer | `20` | Maximum tags per knowledge entry |
| `default_limit` | integer | `5` | Default result limit for queries |
| `min_confidence` | float | `0.3` | Minimum confidence for returned entries |

### apollo.local

Local SQLite + FTS5 knowledge store (always available, no external dependencies).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable local store |
| `retention_years` | integer | `5` | Data retention period |
| `default_query_scope` | string | `"all"` | Default query scope: `all`, `local`, `global` |
| `fts_candidate_multiplier` | integer | `3` | FTS candidate over-fetch multiplier for ranking |
| `min_confidence` | float | `0.3` | Minimum confidence for local queries |
| `default_limit` | integer | `5` | Default result limit for local queries |

---

## mcp

Model Context Protocol server and code generation. Gem: `legion-mcp`.

File: `~/.legionio/settings/mcp.json`

### mcp (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `servers` | hash | `{}` | External MCP server connections |
| `overrides` | hash | `{}` | Tool override mappings |
| `tool_cache_ttl` | integer | `300` | Tool discovery cache TTL in seconds |
| `connect_timeout` | integer | `10` | MCP server connection timeout |
| `call_timeout` | integer | `30` | MCP tool call timeout |
| `mcp.auto_expose_runners` | boolean | `false` | Auto-expose all runners as MCP tools |

### mcp.codegen.self_generate

Automatic code generation for missing capabilities (gap detection).

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable self-generation |
| `cooldown_seconds` | integer | `300` | Minimum seconds between generation cycles |
| `max_gaps_per_cycle` | integer | `5` | Maximum gaps to address per cycle |

#### mcp.codegen.self_generate.tier

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `simple_max_occurrences` | integer | `10` | Occurrences threshold for simple (method-only) generation |
| `complex_min_occurrences` | integer | `11` | Occurrences threshold for full extension generation |
| `recurrence_window_seconds` | integer | `86400` | Window for counting gap occurrences (1 day) |

#### mcp.codegen.self_generate.runner_method

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `output_dir` | string | `"~/.legionio/generated/runners"` | Output directory for generated runner methods |
| `namespace` | string | `"Legion::Generated"` | Ruby namespace for generated code |

#### mcp.codegen.self_generate.full_extension

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `output_dir` | string | `"~/.legionio/generated/extensions"` | Output directory for generated extensions |
| `auto_bundle` | boolean | `false` | Auto-add generated gems to Bundler |

#### mcp.codegen.self_generate.validation

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `syntax_check` | boolean | `true` | Validate generated code syntax |
| `run_specs` | boolean | `true` | Run generated specs |
| `llm_review` | boolean | `true` | LLM code review pass |
| `max_retries` | integer | `2` | Maximum generation retries on failure |
| `quality_gate.enabled` | boolean | `false` | Enable quality score gate |
| `quality_gate.threshold` | float | `0.8` | Minimum quality score to accept |

#### mcp.codegen.self_generate.approval

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `required` | boolean | `false` | Require human approval before activation |
| `auto_approve_confidence` | float | `0.9` | Confidence threshold for auto-approval |
| `auto_approve_gap_types` | array | `[]` | Gap types that auto-approve regardless of confidence |

#### mcp.codegen.self_generate.corroboration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Check Apollo for existing solutions before generating |
| `min_agents` | integer | `2` | Minimum agent corroborations to boost confidence |
| `apollo_query_before_generate` | boolean | `true` | Query Apollo knowledge store before code generation |
| `priority_boost_per_agent` | float | `0.15` | Confidence boost per corroborating agent |

#### mcp.codegen.self_generate.github

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable GitHub PR workflow for generated code |
| `auto_branch` | boolean | `true` | Auto-create feature branches |
| `auto_pr` | boolean | `true` | Auto-create pull requests |
| `auto_merge` | boolean | `false` | Auto-merge approved PRs |
| `target_repo` | string | `nil` | Target GitHub repository |
| `target_branch` | string | `"main"` | Target branch for PRs |
| `pr_labels` | array | `["auto-generated", "needs-review"]` | Labels applied to generated PRs |
| `adversarial_reviewers` | integer | `3` | Number of adversarial LLM reviewers |

---

## compliance

Compliance framework for regulated environments. Defined in `LegionIO` main gem.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable compliance framework |
| `classification_level` | string | `"confidential"` | Data classification level |
| `phi_enabled` | boolean | `true` | Enable PHI (Protected Health Information) controls |
| `pci_enabled` | boolean | `true` | Enable PCI-DSS controls |
| `pii_enabled` | boolean | `true` | Enable PII (Personally Identifiable Information) controls |
| `fedramp_enabled` | boolean | `true` | Enable FedRAMP controls |
| `log_redaction` | boolean | `true` | Enable automatic log redaction |
| `cache_phi_max_ttl` | integer | `3600` | Maximum cache TTL for PHI data in seconds (1 hour) |

---

## absorbers

Pattern-matched content acquisition from external sources. Defined in `legion-settings`.

File: `~/.legionio/settings/absorbers.json`

### absorbers (top-level)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Enable absorber subsystem |
| `max_depth` | integer | `5` | Maximum recursion depth for content traversal |

### absorbers.sources.meetings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Absorb meeting transcripts and recordings |
| `include_chat` | boolean | `true` | Include meeting chat messages |
| `include_files` | boolean | `true` | Include shared files from meetings |
| `retention_days` | integer | `90` | Days to retain absorbed meeting content |
| `min_duration_min` | integer | `5` | Minimum meeting duration to absorb (minutes) |

### absorbers.sources.email_inbox

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Absorb email inbox content |
| `folder` | string | `"inbox"` | Email folder to monitor |
| `max_age_days` | integer | `30` | Maximum email age to absorb |

### absorbers.sources.github

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Absorb GitHub events |
| `events` | array | `["pull_request", "issues"]` | GitHub event types to absorb |

### absorbers.sources.files

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `true` | Absorb local files |
| `watch_dirs` | array | `[]` | Directories to watch for changes |
| `extensions` | array | `["pdf", "docx", "txt", "md", "pptx", "rtf"]` | File extensions to absorb |

---

## network

Network monitoring and health checks.

### network.watchdog

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable network watchdog |
| `failure_threshold` | integer | `5` | Consecutive failures before pausing actors |
| `check_interval` | integer | `15` | Seconds between health checks |

When the failure threshold is reached, all actors are paused. On recovery, `Legion.reload` is triggered to restore connections.

---

## security

Security settings for mTLS and identity.

### security.mtls

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `enabled` | boolean | `false` | Enable mTLS certificate rotation |
| `vault_pki_path` | string | `"pki/issue/legion-internal"` | Vault PKI issue path |
| `cert_ttl` | string | `"24h"` | Certificate TTL |

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
| `lease://` | `"lease://rabbitmq#username"` | Dynamic Vault lease credential (auto-renewed) |
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

### lex-apollo (extension overrides)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `apollo.confidence_threshold` | float | `0.1` | Minimum confidence before archival |
| `apollo.corroboration_boost` | float | `0.3` | Confidence boost on corroboration |
| `apollo.decay_rate` | float | `0.998` | Hourly confidence decay multiplier |
| `apollo.novelty_threshold` | float | `0.3` | Minimum novelty for write path |

{: .note }
See also the [apollo](#apollo) top-level section for client settings.

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
