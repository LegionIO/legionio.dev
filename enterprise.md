---
title: Enterprise
nav_order: 5
description: "Enterprise capabilities: security, compliance, audit, and operational maturity."
---

# Enterprise Overview

LegionIO is built by someone who does infrastructure for a living. Security, audit, and operational maturity are built in, not bolted on.

## Security Model

### HashiCorp Vault Integration (legion-crypt)

- Dynamic credentials with configurable TTLs (30 min default, 4 hr max)
- JWT support (HS256/RS256) for inter-node and API authentication
- Multi-cluster Vault support — per-cluster clients with automatic failover
- Cluster secrets distributed via encrypted AMQP messages
- Universal secret resolver: `vault://path#key`, `env://VAR_NAME`, or plain strings with fallback chains

### Transport Security (legion-transport)

- TLS for all RabbitMQ connections (Optum-sanctioned CAs or custom)
- Message encryption available via legion-crypt
- Vault-backed credential retrieval for RabbitMQ
- Transport spool: JSONL disk buffer when AMQP is unavailable (72hr retention, encrypted at rest)
- RabbitMQ clustering with pool management and region-aware routing

### API Security (LegionIO)

- JWT + API key authentication middleware
- Config endpoints auto-redact sensitive values (password, token, secret, key)
- Read-only guards on security-critical settings sections
- Host authorization controls

### Extension Security

- Extensions loaded from installed gems only (no arbitrary code execution)
- Runner functions execute in managed thread pools with supervision
- Database operations use parameterized queries via Sequel ORM
- File permission system with deny list (~/.ssh, ~/.gnupg, ~/.aws/credentials)

## Access Control (legion-rbac)

Vault-style flat policies for fine-grained access control:

- Role-based profiles filter extensions at boot
- Policy evaluation on runner execution, API endpoints, and config access
- Audit trail for policy decisions

## Observability

### Structured Logging (legion-logging)

- Console output for development
- Structured JSON for production (machine-parseable)
- SIEM export integration
- Per-component log levels

### Telemetry

- Extension lifecycle tracking via catalog state machine
- Health checks and node status reporting
- Distributed scheduling with leader election

## Deployment

### Requirements

| Component | Required | Versions |
|:----------|:---------|:---------|
| Ruby | Yes | >= 3.4 |
| RabbitMQ | Yes | 3.12+, 4.x |
| PostgreSQL | Optional | 14, 15, 16 |
| MySQL | Optional | 8.x |
| SQLite | Optional | 3.40+ (default for dev) |
| Redis | Optional | 7.x |
| Memcached | Optional | 1.6+ |
| HashiCorp Vault | Optional | 1.15+ |

### Deployment Options

- **On-premise**: Standard Ruby deployment with RabbitMQ
- **Cloud**: Any cloud with RabbitMQ support (AWS, Azure, GCP)
- **Hybrid**: Mix on-prem and cloud nodes — AMQP handles the mesh
- **Container**: Standard Docker/Kubernetes deployment

### Configuration

Everything is a JSON config file. Config resolution order:
1. `/etc/legionio/` (system-wide)
2. `~/.legionio/settings/` (user)
3. `./settings/` (project)
4. Environment variables
5. DNS bootstrap (`legion-bootstrap.<domain>` TXT records)

Secret resolution happens after Vault is available: `vault://secret/path#key` URIs are resolved automatically.

## Scale Characteristics

- RabbitMQ-backed messaging with priority queues and dead-letter exchanges
- Task chaining: `Task A -> [transform] -> Task B -> [condition] -> Task C`
- Distributed scheduling with cron expressions and interval locking
- Extension auto-discovery — add capabilities by adding gems
- Managed thread pools with supervision and back-pressure
