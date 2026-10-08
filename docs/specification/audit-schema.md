---
layout: default
title: AZTA Audit Event Schema
permalink: /docs/specification/audit-schema/
---

# AZTA Audit Event Schema

**Status:** Public Draft  
**Version:** 1.0.0-draft  
**Scope:** Agentic Zero Trust Architecture (AZTA)

The AZTA Audit Event Schema defines a normalized machine-readable representation of security-relevant events occurring throughout an agentic system.

The schema is designed to support:

- security monitoring
- policy verification
- authorization decisions
- forensic investigation
- incident reconstruction
- compliance evidence
- cross-system correlation
- runtime anomaly detection
- immutable audit storage

The schema intentionally separates **what an agent proposed** from **what the control plane authorized** and **what the execution environment actually performed**.

---

## Design Principle

> **Reasoning is observable. Authorization is deterministic. Execution is verifiable.**

An LLM-generated action is not itself evidence that an action occurred.

AZTA therefore distinguishes between:

1. **Intent** — what was requested or proposed.
2. **Decision** — what policy determined.
3. **Authorization** — whether execution was permitted.
4. **Execution** — what actually happened.
5. **Evidence** — the resulting security record.

---

## Event Model

An AZTA audit event represents a single security-relevant transition or observation.

Conceptually:

```text
Request
   ↓
Inspection
   ↓
Policy Evaluation
   ↓
Authorization
   ↓
Execution
   ↓
Result
   ↓
Evidence
```

Events should be correlated through stable identifiers rather than relying exclusively on timestamps or textual descriptions.

---

## Required Event Properties

Every audit event should provide enough information to answer:

- What happened?
- When did it happen?
- Where did it happen?
- Who or what initiated it?
- Which request or execution did it belong to?
- What decision was made?
- Which policy produced that decision?
- What capability was requested?
- What was actually executed?
- What evidence was generated?

---

## Canonical Event Structure

The following structure defines the public AZTA v1 draft model.

```json
{
  "schema_version": "azta.audit.v1",
  "event_id": "evt_01...",
  "event_type": "policy.decision",
  "event_time": "2026-01-01T00:00:00Z",
  "trace_id": "trace_01...",
  "request_id": "req_01...",
  "execution_id": "exec_01...",
  "tenant_id": "tenant_01...",
  "actor": {
    "type": "agent",
    "id": "agent_01..."
  },
  "action": {
    "type": "tool.call",
    "name": "example_tool",
    "target": "example-resource"
  },
  "authorization": {
    "decision": "deny",
    "policy_id": "policy.example",
    "policy_version": "1.0.0",
    "reason_code": "CAPABILITY_NOT_PERMITTED"
  },
  "execution": {
    "status": "not_executed"
  },
  "security": {
    "risk_level": "high",
    "control_layer": "deterministic_policy_gateway"
  }
}
```

The exact implementation may add deployment-specific fields, but implementations should preserve the semantic meaning of the core fields.

---

# Core Fields

## `schema_version`

Identifies the AZTA schema version used to encode the event.

Example:

```text
azta.audit.v1
```

Schema versions should be immutable once published.

Breaking changes require a new major schema version.

---

## `event_id`

Globally unique identifier for the individual audit event.

Example:

```text
evt_01JEXAMPLE123
```

The identifier should remain stable throughout the event's lifecycle.

---

## `event_type`

Describes the semantic category of the event.

Examples:

```text
request.created
request.inspected
policy.evaluated
policy.decision
authorization.granted
authorization.denied
execution.started
execution.completed
execution.failed
execution.terminated
approval.requested
approval.granted
approval.denied
credential.issued
credential.revoked
telemetry.recorded
security.alert
```

Implementations may define additional event types provided they do not redefine the meaning of existing types.

---

## `event_time`

Timestamp representing when the event occurred.

Recommended representation:

```text
RFC 3339 / ISO 8601 UTC
```

Example:

```text
2026-01-01T00:00:00Z
```

Systems should synchronize clocks where practical and should preserve sufficient timestamp precision for event ordering.

---

## Correlation Identifiers

AZTA events should support correlation across the complete execution lifecycle.

### `trace_id`

Identifies the broader distributed operation.

### `request_id`

Identifies the originating user or system request.

### `execution_id`

Identifies a specific execution attempt.

These identifiers allow investigators to reconstruct an execution without depending on individual log messages.

Conceptually:

```text
trace_id
└── request_id
    ├── policy evaluation
    ├── agent reasoning
    ├── authorization
    └── execution_id
        ├── tool call
        ├── sandbox activity
        └── result
```

---

# Actor Model

The `actor` object identifies the entity responsible for initiating an event.

Possible actor types include:

```text
user
agent
service
policy_engine
system
administrator
human_approver
external_system
```

Example:

```json
{
  "actor": {
    "type": "agent",
    "id": "agent_01"
  }
}
```

Actor identity should be independently resolvable where possible.

An LLM model name alone should not be treated as an authenticated identity.

---

# Action Model

The `action` object describes the operation being evaluated or executed.

Example:

```json
{
  "action": {
    "type": "tool.call",
    "name": "database.query",
    "target": "customer_records"
  }
}
```

Potential action types include:

- `tool.call`
- `http.request`
- `database.query`
- `file.read`
- `file.write`
- `process.execute`
- `credential.access`
- `message.send`
- `workflow.invoke`
- `state.mutate`

Implementations should use the narrowest practical action type.

---

# Authorization Model

Authorization records the control-plane decision concerning the proposed action.

Example:

```json
{
  "authorization": {
    "decision": "allow",
    "policy_id": "agent.database.read",
    "policy_version": "1.4.0",
    "reason_code": "CAPABILITY_PERMITTED"
  }
}
```

Supported decision values:

```text
allow
deny
require_approval
defer
```

Authorization must be evaluated independently of the LLM's generated reasoning.

---

# Policy Identity

Every policy decision should identify the policy that produced the decision.

Minimum recommended fields:

```text
policy_id
policy_version
decision
reason_code
```

This permits historical reconstruction after policies change.

A later policy revision must not silently rewrite the meaning of an earlier event.

---

# Execution Model

Execution records what actually happened after authorization.

Example:

```json
{
  "execution": {
    "status": "completed",
    "runtime_id": "runtime_01",
    "started_at": "2026-01-01T00:00:01Z",
    "completed_at": "2026-01-01T00:00:03Z"
  }
}
```

Recommended status values:

```text
not_executed
pending
started
completed
failed
terminated
cancelled
```

The distinction between `authorized` and `executed` is mandatory.

An authorization event must never be interpreted as proof that execution occurred.

---

# Security Context

The `security` object provides normalized security metadata.

Example:

```json
{
  "security": {
    "risk_level": "high",
    "control_layer": "deterministic_policy_gateway"
  }
}
```

Recommended risk levels:

```text
unknown
low
medium
high
critical
```

Recommended AZTA control-layer identifiers:

```text
ingress_semantic_firewall
deterministic_policy_gateway
ephemeral_microvm_sandbox
hitl_interceptor
forensic_telemetry_engine
```

---

# Evidence Integrity

Audit events are security evidence and should therefore be protected from unauthorized modification.

Implementations SHOULD support:

- append-only storage
- immutable retention
- cryptographic integrity protection
- trusted timestamps
- access-controlled retrieval
- tamper detection
- retention policies
- chain or batch integrity mechanisms

The event schema itself does not mandate a particular storage provider.

---

# Sensitive Data

Audit records may contain sensitive operational information.

Implementations should avoid unnecessarily storing:

- raw credentials
- access tokens
- private keys
- authentication secrets
- complete user prompts where unnecessary
- unrestricted tool output
- unnecessary personal data

Where sensitive content must be referenced, implementations should prefer:

```text
identifier
hash
pointer
classification
redacted representation
```

over storing the underlying secret directly.

---

# Event Immutability

Once an event has entered the authoritative audit store, its semantic contents should be treated as immutable.

Corrections should be represented through additional events rather than destructive modification.

Conceptually:

```text
Original Event
      ↓
Correction Event
      ↓
Audit History
```

This preserves historical truth and supports forensic reconstruction.

---

# Event Lifecycle

A representative lifecycle may appear as:

```text
request.created
       ↓
request.inspected
       ↓
policy.evaluated
       ↓
policy.decision
       ↓
authorization.granted
       ↓
execution.started
       ↓
execution.completed
       ↓
telemetry.recorded
```

A denied operation may instead terminate at:

```text
request.created
       ↓
request.inspected
       ↓
policy.evaluated
       ↓
policy.decision
       ↓
authorization.denied
```

A human-controlled operation may include:

```text
policy.decision
       ↓
approval.requested
       ↓
approval.granted
       ↓
execution.started
```

---

# Failure Events

Failures should be recorded explicitly rather than inferred from missing events.

Examples:

```text
execution.failed
execution.terminated
authorization.denied
credential.revoked
security.alert
```

A missing event is not equivalent to a successful event.

---

# Termination Events

Forced termination should generate an explicit audit record.

Example:

```json
{
  "event_type": "execution.terminated",
  "execution": {
    "status": "terminated"
  },
  "security": {
    "reason_code": "KILL_SWITCH_ACTIVATED"
  }
}
```

Termination records should preserve the relationship between:

- the execution
- the initiating actor
- the reason
- the authorization state
- the affected runtime
- subsequent containment actions

---

# Human Approval

Human approval events should identify:

- approval request
- requested action
- approver identity
- decision
- timestamp
- authorization scope
- policy context

Example:

```json
{
  "event_type": "approval.granted",
  "actor": {
    "type": "human_approver",
    "id": "approver_01"
  },
  "authorization": {
    "decision": "allow"
  }
}
```

Approval should authorize a defined action or capability rather than providing unrestricted authority to the agent.

---

# Credential Events

Credential lifecycle events should be independently auditable.

Examples:

```text
credential.issued
credential.accessed
credential.rotated
credential.revoked
```

Audit records should reference credentials by stable identifiers rather than recording secret material.

---

# Multi-Tenant Isolation

Where an implementation supports multiple tenants, tenant context should be preserved in every event where applicable.

Example:

```json
{
  "tenant_id": "tenant_01"
}
```

Cross-tenant correlation must not expose tenant-sensitive information.

Audit infrastructure itself must enforce tenant isolation.

---

# Minimal Event Requirements

A minimal valid AZTA event should contain:

```text
schema_version
event_id
event_type
event_time
```

Security-sensitive events should additionally provide, where applicable:

```text
trace_id
request_id
execution_id
actor
action
authorization
execution
security
```

Implementations may require additional fields according to their deployment threat model.

---

# Deterministic Decision Evidence

For policy decisions, the audit record should allow an investigator to determine:

```text
INPUT
  ↓
IDENTITY
  ↓
REQUESTED CAPABILITY
  ↓
POLICY VERSION
  ↓
POLICY DECISION
  ↓
AUTHORIZATION
  ↓
EXECUTION
```

This prevents a common forensic failure:

> knowing that an action occurred without knowing why the system permitted it.

---

# Event Correlation

Events should be correlated using identifiers rather than natural-language descriptions.

Recommended hierarchy:

```text
Trace
└── Request
    ├── Inspection
    ├── Policy Decisions
    ├── Approvals
    └── Execution
        ├── Tool Operations
        ├── Runtime Events
        ├── Results
        └── Termination
```

This enables investigators to reconstruct both successful and failed execution paths.

---

# Storage Requirements

The schema is storage-independent.

A compliant implementation may use:

- object storage
- relational databases
- event stores
- security information and event management systems
- distributed telemetry platforms
- immutable archival systems

The storage implementation must preserve event integrity and retention requirements.

For high-assurance environments, immutable or WORM-capable storage should be considered for authoritative evidence.

---

# OpenTelemetry Alignment

AZTA audit events can be correlated with distributed tracing and telemetry systems.

An implementation may map:

```text
trace_id → distributed trace
request_id → application request
execution_id → execution span
event_id → individual audit record
```

OpenTelemetry should provide observability infrastructure.

AZTA remains responsible for defining the security semantics of authorization, execution, and forensic evidence.

---

# Schema Evolution

Schema changes should follow explicit versioning.

### Patch

Non-semantic corrections that do not alter interpretation.

```text
1.0.0 → 1.0.1
```

### Minor

Backward-compatible additions.

```text
1.0.0 → 1.1.0
```

### Major

Breaking semantic or structural changes.

```text
1.x → 2.0.0
```

Historical events must remain interpretable under the schema version with which they were originally recorded.

---

# Implementation Requirements

An AZTA implementation claiming compatibility with this draft should:

1. Generate unique event identifiers.
2. Record event timestamps.
3. Preserve execution correlation identifiers.
4. Distinguish proposed actions from executed actions.
5. Record authorization decisions.
6. Identify policy versions.
7. Record execution outcomes.
8. Preserve security-relevant failures.
9. Protect authoritative audit records from unauthorized modification.
10. Avoid storing secrets directly in audit events.
11. Support forensic correlation across the execution lifecycle.
12. Preserve historical schema versions.

---

# Security Invariants

The audit system should preserve the following invariants:

### Invariant 1 — Authorization Is Observable

A security-relevant execution must have an attributable authorization decision.

### Invariant 2 — Execution Is Distinct From Intent

A model proposal must not be interpreted as proof of execution.

### Invariant 3 — Policy Is Identifiable

Security decisions must be attributable to a specific policy version.

### Invariant 4 — Failures Are Explicit

Security-relevant failures and terminations should produce explicit events.

### Invariant 5 — Evidence Is Tamper Resistant

Authoritative audit records should not be silently rewritten.

### Invariant 6 — Secrets Are Not Evidence

Credentials and secret material must not be used as ordinary audit payloads.

### Invariant 7 — Historical Truth Is Preserved

Corrections should append new evidence rather than destroy previous evidence.

---

# Public Specification Status

This document defines the **public AZTA audit-event model draft**.

It is intentionally implementation-neutral.

The machine-readable JSON Schema is maintained separately at:

```text
/schema/azta_audit_record.v1.json
```

The machine-readable schema and this semantic specification should evolve together.

---

# Summary

The AZTA audit model establishes a common evidence language for agentic systems.

The fundamental distinction is:

```text
What the agent wanted to do
        ≠
What policy allowed
        ≠
What the runtime executed
        ≠
What the system recorded
```

A trustworthy agentic architecture must be able to distinguish all four.

> **Reasoning may be non-deterministic. Evidence must not be.**
