---
layout: default
title: AZTA Threat Model
permalink: /docs/azta/threat-model/
---

# AZTA Threat Model

The AZTA threat model defines the security assumptions, trust boundaries, attack surfaces, threat scenarios, and control objectives relevant to production AI-agent systems.

AZTA assumes that an agent may be manipulated, misled, compromised, or simply behave incorrectly.

The architecture therefore does not require the model to be trustworthy in order for the surrounding system to remain governed.

> **Assume the reasoning layer can fail. Design the control plane so that failure remains bounded.**

---

# Threat Model Objectives

The AZTA threat model is designed to protect:

- identities
- credentials
- tools
- data
- tenants
- infrastructure
- application state
- security policy
- human authorization
- audit evidence

The primary objective is to prevent an agent's ability to reason or act from becoming unrestricted authority.

---

# Security Assumptions

AZTA makes several explicit assumptions.

## The model is not a trusted security boundary

An LLM can produce incorrect, manipulated, or adversarial output.

Model output is therefore treated as untrusted intent.

## External content is untrusted

Information retrieved from external systems can contain malicious instructions, misleading content, poisoned data, or adversarial payloads.

Retrieved content should not automatically become trusted instructions.

## Tools can fail or become compromised

A tool integration may contain vulnerabilities, return malicious content, expose excessive permissions, or behave unexpectedly.

## Credentials can be compromised

API keys, tokens, sessions, and delegated credentials may be exposed or misused.

## Users can make malicious requests

Authentication establishes identity.

It does not establish that every requested operation is safe or authorized.

## Agent runtimes can fail

A compromised process or dependency may attempt to escape its intended execution boundary.

## Security controls can fail

Policy engines, telemetry systems, approval systems, and other infrastructure may become unavailable or degraded.

AZTA therefore requires explicit failure behavior rather than assuming perfect availability.

---

# Trust Model

AZTA distinguishes between components based on their role in security decisions.

```text
                    TRUSTED CONTROL PLANE
                    ─────────────────────

       ┌──────────────────────────────────────┐
       │ Identity / Authorization             │
       │ Policy Engine                        │
       │ Execution Controls                   │
       │ Approval Mechanism                   │
       │ Security Administration              │
       └──────────────────┬───────────────────┘
                          │
                          │ governs
                          ▼

                  UNTRUSTED REASONING
                  ───────────────────

       ┌──────────────────────────────────────┐
       │ LLM                                  │
       │ Agent Planner                        │
       │ Generated Instructions               │
       │ Generated Tool Parameters             │
       │ Retrieved Content                    │
       └──────────────────┬───────────────────┘
                          │
                          │ proposes
                          ▼

                  CONTROLLED EXECUTION
                  ─────────────────────

       ┌──────────────────────────────────────┐
       │ Sandbox                              │
       │ Tool Adapters                        │
       │ External Systems                     │
       └──────────────────┬───────────────────┘
                          │
                          ▼

                  FORENSIC EVIDENCE
                  ─────────────────

       ┌──────────────────────────────────────┐
       │ Telemetry                            │
       │ Audit Records                        │
       │ Security Events                      │
       └──────────────────────────────────────┘
```

The exact trust classification of a component depends on its implementation and deployment.

The architectural distinction is that **security authority should remain outside probabilistic reasoning**.

---

# Attack Surface

An agentic system exposes more than a traditional request/response application.

Relevant attack surfaces include:

```text
┌──────────────────────────────────────────────────────┐
│                    ATTACK SURFACES                    │
├──────────────────────────────────────────────────────┤
│                                                      │
│  User Input                                           │
│       │                                               │
│       ├── Prompt Injection                             │
│       ├── Social Engineering                          │
│       └── Privilege Abuse                             │
│                                                      │
│  Retrieved Content                                    │
│       │                                               │
│       ├── Indirect Prompt Injection                   │
│       ├── Data Poisoning                              │
│       └── Malicious Instructions                      │
│                                                      │
│  Model Output                                         │
│       │                                               │
│       ├── Unsafe Tool Selection                       │
│       ├── Incorrect Parameters                        │
│       └── Policy Evasion                              │
│                                                      │
│  Tool Layer                                           │
│       │                                               │
│       ├── Excessive Privilege                         │
│       ├── Tool Confusion                              │
│       └── Malicious / Compromised Tool               │
│                                                      │
│  Identity                                             │
│       │                                               │
│       ├── Credential Theft                            │
│       ├── Token Abuse                                 │
│       └── Delegation Abuse                            │
│                                                      │
│  Runtime                                              │
│       │                                               │
│       ├── Sandbox Escape                              │
│       ├── Resource Exhaustion                         │
│       └── Persistence                                 │
│                                                      │
│  External Systems                                     │
│       │                                               │
│       ├── Unauthorized State Mutation                 │
│       ├── Data Exfiltration                           │
│       └── Cascading Actions                           │
│                                                      │
│  Telemetry                                            │
│       │                                               │
│       ├── Evidence Manipulation                       │
│       ├── Data Leakage                                │
│       └── Insufficient Visibility                     │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

# Threat Categories

AZTA organizes threats around the point at which unauthorized behavior can enter or propagate through the system.

## T1 — Malicious or Manipulated Input

An attacker supplies instructions designed to cause the agent to violate intended policy.

Examples include:

- direct prompt injection
- malicious user instructions
- instruction hierarchy manipulation
- attempts to override system controls
- requests designed to induce unsafe tool use

### Primary Controls

- Ingress Semantic Firewall
- Deterministic Policy Gateway
- capability restrictions
- action-level authorization
- telemetry

---

# T2 — Indirect Prompt Injection

Untrusted instructions enter through content the agent retrieves or processes rather than through the original user request.

Potential sources include:

- webpages
- documents
- email
- support tickets
- repositories
- databases
- tool responses
- external APIs

Conceptually:

```text
External Content
      │
      ▼
Retrieved by Agent
      │
      ▼
Malicious Instruction
      │
      ▼
Model interprets content
      │
      ▼
Unsafe Action Proposal
      │
      ▼
Policy Gateway
      │
      ├── DENY
      │
      └── ALLOW / APPROVAL
```

The important security property is that successful manipulation of the model does not automatically authorize the resulting action.

### Primary Controls

- semantic input inspection
- trust classification
- tool-output handling
- action-level policy
- least-privilege capabilities
- HITL controls

---

# T3 — Excessive Agent Authority

An agent receives more permissions than are necessary to complete its task.

Examples include:

- unrestricted database access
- broad filesystem access
- production credentials
- unrestricted network access
- permission to modify unrelated tenants
- administrative API access

### Primary Controls

- capability-based authorization
- scoped identities
- policy gateway
- credential broker
- runtime restrictions
- tenant isolation

---

# T4 — Confused Deputy

A less-privileged or compromised agent causes a more privileged component to perform an operation on its behalf.

Conceptually:

```text
Attacker
   │
   ▼
Low-Privilege Agent
   │
   │ malicious request
   ▼
Privileged Tool
   │
   ▼
Sensitive Resource
```

The privileged tool may incorrectly trust the agent simply because the request originated from an internal component.

### Primary Controls

- explicit principal identity
- capability validation
- delegated authority constraints
- target-specific authorization
- policy evaluation at the point of action

The existence of an internal network relationship should not itself constitute authorization.

---

# T5 — Credential and Token Abuse

An attacker obtains or misuses credentials available to an agent.

Potential consequences include:

- unauthorized API access
- privilege escalation
- cross-tenant access
- persistence
- data exfiltration
- infrastructure compromise

### Primary Controls

- short-lived credentials
- scoped tokens
- credential isolation
- external authorization
- token revocation
- execution-specific identities
- telemetry

Credentials should be treated as security-sensitive assets rather than ordinary model context.

---

# T6 — Tool Abuse

An agent invokes a legitimate tool in an unsafe or unintended way.

Examples include:

```text
read → acceptable
write → potentially sensitive
delete → high impact
execute → potentially privileged
```

The fact that a tool is available does not imply that every operation exposed by the tool is authorized.

### Primary Controls

- tool allowlists
- capability-based access
- parameter validation
- policy evaluation
- risk classification
- human approval
- execution isolation

---

# T7 — Unauthorized State Mutation

An agent modifies an external system without sufficient authorization.

Examples include:

- deleting records
- modifying customer data
- changing access permissions
- deploying infrastructure
- altering configuration
- sending messages
- initiating transactions

### Primary Controls

- deterministic authorization
- target-specific policy
- action-level re-authorization
- HITL approval
- downstream authorization
- telemetry

---

# T8 — Cross-Tenant Data Exposure

An agent accesses or modifies information belonging to another tenant.

This can result from:

- incorrect identity propagation
- insufficient authorization
- retrieval-layer leakage
- shared credentials
- confused-deputy behavior
- improper caching
- broad tool permissions

### Primary Controls

- tenant-bound identity
- tenant-aware policy
- resource-level authorization
- isolated execution
- scoped credentials
- telemetry correlation

Tenant identity should be treated as an authorization attribute rather than merely an application metadata field.

---

# T9 — Runtime Compromise

An attacker or malicious workload compromises the execution environment.

Potential consequences include:

- filesystem access
- credential theft
- lateral movement
- network abuse
- persistence
- host compromise
- resource exhaustion

### Primary Controls

- ephemeral execution
- MicroVM or equivalent isolation
- filesystem restrictions
- network policy
- resource quotas
- execution timeouts
- credential isolation
- runtime telemetry

---

# T10 — Runaway or Recursive Execution

An agent enters an unintended execution loop or generates excessive activity.

Potential causes include:

- recursive tool invocation
- uncontrolled retries
- agent-to-agent loops
- malformed planning
- unexpected external responses
- resource exhaustion

Potential consequences include:

- excessive API usage
- infrastructure exhaustion
- financial cost
- data mutation
- cascading failures

### Primary Controls

- execution budgets
- timeouts
- rate limits
- step limits
- resource quotas
- policy enforcement
- runtime termination
- kill switch

---

# T11 — Human Approval Manipulation

An attacker attempts to cause an approver to authorize an unsafe operation.

Potential techniques include:

- misleading summaries
- omitted context
- urgency manipulation
- incomplete risk information
- deceptive action descriptions

### Primary Controls

Approval interfaces should expose relevant security context, including where appropriate:

- principal
- agent
- target
- operation
- requested capability
- risk classification
- affected resources
- policy requiring approval
- expected consequence

The approval mechanism should not rely solely on model-generated descriptions when independent information is available.

---

# T12 — Policy Bypass

An agent, tool, integration, or administrator attempts to circumvent the deterministic policy boundary.

Examples include:

- direct access to a protected API
- alternate tool paths
- hidden credentials
- unauthorized service accounts
- bypassing the policy gateway
- modifying policy without authorization

### Primary Controls

- centralized authorization
- network segmentation
- deny-by-default pathways
- protected policy configuration
- administrative access controls
- policy change telemetry
- downstream authorization

The policy gateway is only meaningful if protected resources cannot routinely be accessed through an alternate uncontrolled path.

---

# T13 — Telemetry Manipulation

An attacker attempts to prevent or alter evidence of malicious activity.

Potential methods include:

- deleting logs
- modifying records
- disabling telemetry
- altering timestamps
- breaking correlation
- compromising the logging destination

### Primary Controls

- append-only storage
- immutable retention
- integrity mechanisms
- independent storage boundaries
- access controls
- telemetry health monitoring
- administrative audit events

Security evidence should be protected independently from the workload generating it where practical.

---

# T14 — Sensitive Data Leakage

Agent activity exposes sensitive information to unauthorized destinations.

Potential leakage channels include:

- model context
- tool parameters
- tool responses
- logs
- telemetry
- network requests
- generated files
- external APIs

### Primary Controls

- data classification
- authorization
- minimization
- output controls
- network restrictions
- credential isolation
- telemetry access controls
- retention policies

Not every piece of model context should automatically be available to every tool.

---

# T15 — Policy and Configuration Tampering

An attacker changes the controls governing agent behavior.

Potential targets include:

- authorization rules
- tool policies
- runtime restrictions
- approval requirements
- tenant configuration
- telemetry configuration
- credential policies

### Primary Controls

- administrative authorization
- version-controlled policy
- change review
- integrity protection
- policy versioning
- configuration telemetry
- separation of duties where appropriate

A security control must itself have a security boundary.

---

# Threat Propagation

A key AZTA concern is preventing a compromise in one component from propagating into unrestricted authority.

For example:

```text
Malicious Document
       │
       ▼
Prompt Injection
       │
       ▼
Compromised Model Reasoning
       │
       ▼
Unsafe Tool Proposal
       │
       ▼
Policy Gateway
       │
       ├──────── DENY
       │
       ▼
Contained Runtime
       │
       ▼
HITL Requirement
       │
       ├──────── DENY
       │
       ▼
Controlled Execution
       │
       ▼
Forensic Evidence
```

The architecture creates multiple opportunities to stop propagation.

---

# Threat-to-Control Matrix

|Threat|Ingress|Policy|Sandbox|HITL|Telemetry|
|---|---|---|---|---|---|
|Direct prompt injection|Primary|Supporting|Supporting|Conditional|Evidence|
|Indirect prompt injection|Primary|Primary|Supporting|Conditional|Evidence|
|Excessive authority|Supporting|Primary|Primary|Conditional|Evidence|
|Confused deputy|Supporting|Primary|Supporting|Conditional|Evidence|
|Credential abuse|Supporting|Primary|Primary|Conditional|Primary|
|Tool abuse|Supporting|Primary|Primary|Conditional|Primary|
|Unauthorized mutation|Supporting|Primary|Supporting|Primary|Primary|
|Cross-tenant exposure|Supporting|Primary|Primary|Conditional|Primary|
|Runtime compromise|—|Supporting|Primary|Conditional|Primary|
|Runaway execution|—|Primary|Primary|Conditional|Primary|
|Approval manipulation|—|Supporting|—|Primary|Primary|
|Policy bypass|—|Primary|Primary|Supporting|Primary|
|Telemetry manipulation|—|Supporting|Supporting|—|Primary|
|Data leakage|Primary|Primary|Primary|Conditional|Supporting|
|Configuration tampering|—|Primary|Supporting|Supporting|Primary|

The matrix identifies architectural responsibilities.

It does not imply that a single layer completely mitigates a threat.

---

# Threat Severity

AZTA deployments should classify threats according to their potential impact and likelihood.

A generic model can use:

```text
Severity = Impact × Likelihood
```

Implementations may use a more sophisticated risk methodology.

Potential impact dimensions include:

- confidentiality
- integrity
- availability
- financial impact
- operational impact
- tenant impact
- regulatory exposure
- reputational impact
- irreversibility

Risk classification should influence control strength.

For example:

```text
Low Risk
   │
   └── Automated execution

Moderate Risk
   │
   └── Additional policy controls

High Risk
   │
   └── Human authorization

Critical / Prohibited
   │
   └── Deny / terminate
```

The actual thresholds should be deployment-specific and policy-defined.

---

# Attack Path Analysis

A useful AZTA assessment should analyze complete attack paths rather than isolated vulnerabilities.

Example:

```text
1. Attacker controls external document
          │
          ▼
2. Agent retrieves document
          │
          ▼
3. Document contains malicious instruction
          │
          ▼
4. Model follows instruction
          │
          ▼
5. Agent proposes privileged tool action
          │
          ▼
6. Policy evaluates principal + capability
          │
          ├── DENY
          │
          └── REQUIRE_APPROVAL
                         │
                         ▼
                  Human Review
                         │
                         └── DENY
```

The security objective is not necessarily to prevent every step from occurring.

The objective is to prevent the attack path from reaching an unauthorized external effect.

---

# Blast Radius

AZTA evaluates compromises partly by the amount of authority available after compromise.

A useful conceptual model is:

```text
Blast Radius =
    Identity Scope
  × Capability Scope
  × Resource Scope
  × Runtime Scope
  × Time
```

Reducing any of these dimensions can reduce potential impact.

Controls include:

- least privilege
- tenant isolation
- scoped capabilities
- short-lived credentials
- ephemeral runtimes
- network restrictions
- execution budgets
- approval gates

This is a conceptual risk model rather than a normative mathematical formula.

---

# Security Invariants

The following invariants should remain true even when the reasoning layer is compromised.

## Invariant 1 — Model output cannot grant authority

Generated text cannot independently establish authorization.

## Invariant 2 — External content cannot redefine policy

Retrieved instructions cannot override the control plane.

## Invariant 3 — Tool availability does not equal tool authorization

A callable tool is not automatically authorized for every operation.

## Invariant 4 — Identity follows the actual principal

A downstream component should not infer authority merely from upstream trust.

## Invariant 5 — Tenant boundaries remain enforceable

Agent reasoning cannot override tenant-specific authorization.

## Invariant 6 — Runtime access remains constrained

A compromised workload cannot automatically obtain unrestricted host access.

## Invariant 7 — High-impact actions can be interrupted

Policy can require external human authorization.

## Invariant 8 — Security decisions produce evidence

Important decisions should be observable and correlatable.

## Invariant 9 — Controls cannot silently fail open

Failure behavior must be explicitly defined.

## Invariant 10 — Administrative controls are themselves protected

An attacker who can modify the policy system can potentially defeat the entire architecture.

---

# Threat Modeling Process

An AZTA deployment should perform threat modeling throughout the system lifecycle.

A practical process is:

```text
Identify Assets
      │
      ▼
Identify Principals
      │
      ▼
Map Trust Boundaries
      │
      ▼
Identify Attack Surfaces
      │
      ▼
Enumerate Threats
      │
      ▼
Map Threats to Controls
      │
      ▼
Assess Residual Risk
      │
      ▼
Define Tests
      │
      ▼
Monitor Runtime
      │
      ▼
Update Model
```

Threat modeling should be revisited when:

- a new tool is introduced
- capabilities change
- models change
- agents gain new authority
- new tenants are added
- infrastructure changes
- policies change
- external integrations change
- incidents occur

---

# Adversarial Testing

Threat models should be validated through adversarial testing.

Testing may include:

- direct prompt-injection attempts
- indirect prompt-injection attempts
- unauthorized tool calls
- privilege-escalation attempts
- cross-tenant access attempts
- credential misuse
- policy-bypass attempts
- malformed tool parameters
- sandbox escape attempts
- resource-exhaustion tests
- approval manipulation tests
- telemetry integrity tests
- kill-switch tests

The objective is to test the enforcement boundaries rather than merely evaluating whether the model produces a desirable response.

---

# Residual Risk

No architecture eliminates all risk.

Residual risk can remain in:

- compromised identity infrastructure
- vulnerable external systems
- incorrect policy
- compromised administrators
- runtime vulnerabilities
- supply-chain compromise
- unknown model behaviors
- telemetry failures
- human decision errors

AZTA therefore focuses on making authority explicit, limiting blast radius, and creating observable control boundaries.

---

# Threat Model Summary

The central AZTA threat assumption is:

> **The agent may be compromised without the entire system becoming compromised.**

The architecture achieves this through layered separation:

```text
UNTRUSTED INPUT
      │
      ▼
   INSPECT
      │
      ▼
  AUTHORIZE
      │
      ▼
   REASON
      │
      ▼
   PROPOSE
      │
      ▼
  RE-AUTHORIZE
      │
      ▼
   CONTAIN
      │
      ▼
  HUMAN-CHECK
      │
      ▼
   EXECUTE
      │
      ▼
    RECORD
      │
      ▼
   INVESTIGATE
```

The model remains a powerful reasoning component.

It is not the final security authority.

AZTA's security objective is to ensure that manipulation of probabilistic reasoning does not automatically become unrestricted access to protected resources.

> **Compromise the agent. Contain the compromise. Preserve the evidence.**
