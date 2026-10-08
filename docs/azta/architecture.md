---
layout: default
title: AZTA Architecture
permalink: /docs/azta/architecture/
---

# AZTA Architecture

**Agentic Zero Trust Architecture (AZTA)** is a security architecture for AI-agent systems that separates probabilistic model reasoning from deterministic security enforcement.

## Open Standard. Closed Execution.

AZTA is based on a simple architectural premise:

> **The model may reason. The control plane decides.**

Large language models and other probabilistic reasoning systems are capable of planning, interpreting context, selecting tools, and generating actions. They should not, however, be treated as the ultimate authority for security-sensitive decisions.

AZTA therefore places security-critical controls outside the model's reasoning process.

The architecture establishes independent enforcement boundaries for:

- identity
- authorization
- input validation
- tool execution
- runtime isolation
- human authorization
- telemetry
- audit evidence
- incident response

The objective is to ensure that compromising or manipulating the reasoning layer does not automatically compromise the surrounding system.

---

## Architectural Principle

Traditional application security commonly assumes that trusted application code makes security decisions.

Agentic systems complicate this model because an LLM can dynamically:

- interpret untrusted instructions
- select tools
- construct parameters
- retrieve information
- generate multi-step plans
- invoke external systems
- modify application state
- operate across multiple execution cycles

The model's output is therefore treated as **untrusted intent**.

AZTA introduces deterministic enforcement boundaries around that intent.

```text
                    UNTRUSTED / PROBABILISTIC
                    ─────────────────────────

 User / Event
      │
      ▼
┌──────────────────────────────┐
│ 1. Ingress Semantic Firewall │
│                              │
│ Input classification         │
│ Trust-boundary normalization │
│ Semantic threat detection    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 2. Deterministic Policy      │
│    Gateway                   │
│                              │
│ Identity                     │
│ Authorization                │
│ Policy evaluation            │
│ Tool permissions             │
└──────────────┬───────────────┘
               │
               ▼
       ┌─────────────────┐
       │ Agent / LLM     │
       │ Reasoning       │
       │                 │
       │ Planning        │
       │ Interpretation  │
       │ Tool selection  │
       └────────┬────────┘
                │
                │ proposed action
                ▼

                    TRUSTED ENFORCEMENT
                    ───────────────────

┌──────────────────────────────┐
│ 3. Ephemeral MicroVM         │
│    Sandboxing                │
│                              │
│ Isolated execution           │
│ Resource constraints         │
│ Filesystem isolation         │
│ Network controls             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 4. Asynchronous HITL         │
│    Interceptor               │
│                              │
│ Approval gates               │
│ Risk escalation              │
│ Human authorization          │
│ Execution interruption       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ External Tool / System       │
│                              │
│ APIs                         │
│ Databases                    │
│ SaaS systems                 │
│ Infrastructure               │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ 5. Forensic Telemetry        │
│    Engine                    │
│                              │
│ Trace events                 │
│ Policy decisions             │
│ Tool activity                │
│ Identity context             │
│ Runtime evidence             │
└──────────────────────────────┘
```

---

## Trust Boundaries

AZTA divides the agent execution path into explicit trust boundaries.

### Boundary 1 — Ingress

All external instructions enter through a controlled ingress boundary.

Potential sources include:

- users
- applications
- webhooks
- scheduled events
- external agents
- API requests
- retrieved content

The ingress layer establishes normalized security context before downstream execution begins.

---

### Boundary 2 — Policy

The policy gateway determines whether a proposed operation is permitted.

The policy decision should not depend solely on the model's interpretation of its own authority.

Relevant context can include:

- authenticated identity
- agent identity
- tenant
- requested capability
- target resource
- requested operation
- risk classification
- environment
- current policy
- execution state

The policy layer produces a deterministic authorization result.

```text
Request
   │
   ▼
Identity + Context
   │
   ▼
Policy Evaluation
   │
   ├── DENY ───────────────► Stop
   │
   ├── ALLOW ──────────────► Continue
   │
   └── REQUIRE_APPROVAL ──► HITL
```

---

### Boundary 3 — Reasoning

The LLM operates as a reasoning component rather than a security authority.

Its responsibilities may include:

- interpreting the request
- generating plans
- selecting candidate tools
- constructing proposed operations
- determining execution order
- interpreting tool results

Its responsibilities do not include independently granting itself permissions.

A model-generated statement such as:

```text
"I am authorized to access this resource."
```

is not itself an authorization decision.

Authorization must be established by the control plane.

---

### Boundary 4 — Execution

Agent-generated actions execute inside controlled runtime boundaries.

The execution environment should be capable of limiting:

- filesystem access
- network access
- process creation
- credentials
- available tools
- resource consumption
- execution lifetime
- access to host infrastructure

AZTA uses ephemeral isolation as a mechanism for reducing blast radius.

The exact runtime technology is an implementation choice. MicroVMs are one possible implementation of the isolation boundary.

---

### Boundary 5 — Human Authorization

Certain operations require an explicit human decision.

Examples may include:

- destructive actions
- high-impact state changes
- privileged operations
- financial actions
- production infrastructure changes
- access to sensitive resources
- policy-defined high-risk operations

Human approval should occur outside the model's control loop.

The model may request authorization.

It does not grant authorization.

---

### Boundary 6 — Evidence

Runtime activity must produce sufficient evidence to reconstruct what happened.

Relevant events can include:

- request reception
- identity resolution
- policy evaluation
- model-generated intent
- tool selection
- tool invocation
- execution result
- human approval
- denial
- exception
- termination
- credential or token events
- system state changes

Telemetry is therefore treated as a security control rather than merely an observability feature.

---

# Control Plane and Data Plane

AZTA distinguishes between the **control plane** and the **agent execution/data plane**.

## Control Plane

The control plane contains deterministic security mechanisms responsible for deciding and enforcing what the agent is allowed to do.

Examples include:

- identity services
- policy engines
- authorization services
- approval systems
- execution admission controls
- runtime isolation controls
- audit pipelines
- kill switches
- credential controls

## Agent Execution Plane

The execution plane contains the components performing probabilistic reasoning and permitted operations.

Examples include:

- LLMs
- agent runtimes
- planners
- tool adapters
- retrieval systems
- application logic
- isolated execution environments

The fundamental relationship is:

```text
             CONTROL PLANE
        ┌─────────────────────┐
        │ Identity            │
        │ Policy              │
        │ Authorization       │
        │ Approval            │
        │ Isolation           │
        │ Audit               │
        └──────────┬──────────┘
                   │
                   │ governs
                   ▼
        ┌─────────────────────┐
        │ EXECUTION PLANE     │
        │                     │
        │ Agent               │
        │ LLM                 │
        │ Tools               │
        │ Retrieval           │
        │ Runtime             │
        └─────────────────────┘
```

The execution plane can request capabilities from the control plane, but the execution plane does not become the authority that grants those capabilities.

---

# Capability-Based Execution

AZTA favors explicit capabilities over implicit authority.

Instead of allowing an agent to possess unrestricted access to a system, the control plane should establish narrowly scoped permissions.

A capability can conceptually contain:

```text
Principal
    +
Capability
    +
Target
    +
Operation
    +
Context
    +
Policy
    =
Execution Decision
```

For example:

```json
{
  "principal": "agent.example",
  "capability": "ticket.update",
  "target": "ticket:12345",
  "operation": "update",
  "environment": "production"
}
```

The example is illustrative rather than a normative AZTA schema.

The authoritative decision remains the responsibility of the policy layer.

---

# Failure Containment

AZTA assumes that individual components can fail or become compromised.

Security therefore should not depend on every component behaving correctly.

The architecture aims to prevent failures from propagating across trust boundaries.

```text
Compromised Input
       │
       ▼
Ingress Controls
       │
       X  blocked
       │
       ▼
Compromised Model
       │
       ▼
Policy Gateway
       │
       X  unauthorized action denied
       │
       ▼
Compromised Runtime
       │
       ▼
Ephemeral Isolation
       │
       X  blast radius constrained
       │
       ▼
High-Risk Operation
       │
       ▼
HITL Interceptor
       │
       X  approval unavailable
       │
       ▼
Telemetry / Evidence
```

The goal is not to guarantee that an agent can never fail.

The goal is to ensure that failure does not automatically become unrestricted authority.

---

# Defense in Depth

AZTA does not rely on a single security mechanism.

The five enforcement layers provide multiple independent opportunities to detect, prevent, constrain, interrupt, and reconstruct agent activity.

| Layer | Primary Function |
| --- | --- |
| Ingress Semantic Firewall | Inspect and normalize inbound activity |
| Deterministic Policy Gateway | Authorize capabilities and operations |
| Ephemeral MicroVM Sandboxing | Contain execution |
| Asynchronous HITL Interceptor | Require human authorization |
| Forensic Telemetry Engine | Preserve runtime evidence |

Each layer addresses a different failure mode.

A control should therefore not be considered redundant simply because another layer exists.

---

# Security Invariants

An AZTA implementation should preserve several architectural invariants.

## 1. Model output is untrusted

Generated content, plans, tool selections, and proposed actions are treated as untrusted inputs to downstream security controls.

## 2. Authorization is external to model reasoning

The model cannot grant itself additional privileges through generated instructions.

## 3. Tool execution is policy-mediated

Sensitive tool operations pass through deterministic authorization before execution.

## 4. Execution has a containment boundary

Agent workloads should execute within explicitly defined runtime constraints appropriate to their risk.

## 5. High-risk actions can be interrupted

The architecture must provide a mechanism for requiring human authorization when policy demands it.

## 6. Security decisions are observable

Important authorization and execution decisions produce auditable evidence.

## 7. Failure should be bounded

A compromised model, tool, credential, or runtime should not automatically obtain unrestricted authority over the surrounding environment.

---

# Implementation Independence

AZTA defines architectural responsibilities rather than requiring one specific vendor or technology stack.

Possible implementations may use different:

- LLM providers
- agent frameworks
- policy engines
- identity providers
- runtime isolation technologies
- telemetry platforms
- storage systems
- human approval systems

The architectural requirement is that security responsibilities remain enforceable independently of probabilistic model reasoning.

---

# Reference Architecture

A production implementation can therefore be represented as:

```text
                    ┌──────────────────────┐
                    │      PRINCIPAL       │
                    │ User / Service / App │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  INGRESS FIREWALL    │
                    │                      │
                    │ Normalize            │
                    │ Classify             │
                    │ Inspect              │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   POLICY GATEWAY     │
                    │                      │
                    │ Identity             │
                    │ Authorization        │
                    │ Capability           │
                    │ Risk policy          │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    AGENT / LLM       │
                    │                      │
                    │ Reason               │
                    │ Plan                 │
                    │ Propose              │
                    └──────────┬───────────┘
                               │
                         Proposed Action
                               │
                               ▼
                    ┌──────────────────────┐
                    │  EXECUTION CONTROL   │
                    │                      │
                    │ MicroVM              │
                    │ Network              │
                    │ Filesystem           │
                    │ Credentials          │
                    │ Resource limits      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     HITL GATE        │
                    │                      │
                    │ Allow                │
                    │ Deny                 │
                    │ Escalate             │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   TARGET SYSTEM      │
                    │                      │
                    │ API / DB / SaaS /    │
                    │ Infrastructure       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ FORENSIC TELEMETRY  │
                    │                      │
                    │ Trace                │
                    │ Record               │
                    │ Correlate            │
                    │ Preserve             │
                    └──────────────────────┘
```

---

# Architectural Objective

AZTA is designed to make agentic execution **governable**.

The architecture does not attempt to make the language model deterministic.

Instead, it establishes deterministic boundaries around the model.

The resulting security model can be summarized as:

> **Untrusted reasoning. Deterministic policy. Contained execution. Explicit authorization. Verifiable evidence.**

These principles form the architectural foundation for the AZTA enforcement layers, specification, security controls, and compliance mappings.

