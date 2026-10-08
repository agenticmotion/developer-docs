---
layout: default
title: AZTA Execution Model
permalink: /docs/azta/execution-model/
---

# AZTA Execution Model

The AZTA execution model defines how AI-agent activity moves through security boundaries from initial request to completed operation and recorded evidence.

The central principle is:

> **Reasoning produces intent. Policy produces authorization. Execution produces effects. Telemetry produces evidence.**

An LLM may determine what it believes should happen.

It does not determine whether that action is authorized to happen.

---

# Execution Lifecycle

A complete AZTA execution can be represented as:

```text
REQUEST
   │
   ▼
INGRESS
   │
   ▼
NORMALIZE / CLASSIFY
   │
   ▼
IDENTITY + CONTEXT
   │
   ▼
POLICY ADMISSION
   │
   ├──────── DENY ───────────────► TERMINATE
   │
   └──────── ALLOW ─────────────► AGENT EXECUTION
                                      │
                                      ▼
                                  REASONING
                                      │
                                      ▼
                               ACTION PROPOSAL
                                      │
                                      ▼
                              POLICY EVALUATION
                                      │
                       ┌──────────────┼──────────────┐
                       │              │              │
                     DENY           ALLOW       APPROVAL
                       │              │              │
                       ▼              │              ▼
                    STOP             │          HUMAN REVIEW
                                      │              │
                                      │        ┌─────┴─────┐
                                      │      APPROVE      DENY
                                      │        │            │
                                      └────────┤            │
                                               ▼            ▼
                                           EXECUTE         STOP
                                               │
                                               ▼
                                          TARGET SYSTEM
                                               │
                                               ▼
                                        RESULT / EFFECT
                                               │
                                               ▼
                                          TELEMETRY
                                               │
                                               ▼
                                            CLOSE
```

The lifecycle is intentionally different from a conventional application call path.

The model generates a proposed action, but independent controls determine whether the action can produce an external effect.

---

# 1. Request Creation

An execution begins with an external or internal event.

Possible sources include:

- human users
- applications
- API requests
- scheduled jobs
- webhooks
- external agents
- automated workflows
- system events

A request should receive a correlation identifier early in its lifecycle.

Conceptually:

```json
{
  "request_id": "req-example-001",
  "source": "application",
  "principal": "user@example",
  "tenant": "tenant-example",
  "received_at": "2026-01-01T00:00:00Z"
}
```

This example is illustrative and is not the normative AZTA audit schema.

The request identifier allows downstream events to be associated with the originating operation.

---

# 2. Ingress Processing

The request enters the Ingress Semantic Firewall.

The ingress layer establishes the initial security context and evaluates whether the request is suitable for agent processing.

Conceptually:

```text
External Input
      │
      ▼
┌─────────────────────┐
│ Normalize           │
│ Validate            │
│ Classify            │
│ Inspect             │
└──────────┬──────────┘
           │
           ├── REJECT
           │
           └── ACCEPT
                  │
                  ▼
             Agent System
```

The ingress layer should distinguish between:

- instructions
- data
- metadata
- identity information
- retrieved content
- tool-generated content

This distinction reduces the likelihood that untrusted data will automatically become trusted instructions.

---

# 3. Identity Resolution

Before an operation can be authorized, the system must establish the relevant security principal.

Depending on the deployment, this may include:

```text
Human Principal
       │
       ▼
Application Identity
       │
       ▼
Agent Identity
       │
       ▼
Execution Identity
```

These identities should not be assumed to be interchangeable.

For example, the fact that a human user has access to a resource does not automatically mean that every agent acting on that user's behalf should receive unrestricted access to the same resource.

Authorization should account for the actual execution principal and delegated authority.

---

# 4. Context Construction

The policy decision requires more than a raw action string.

A security context may include:

```text
Principal
Agent
Tenant
Capability
Target
Operation
Environment
Risk
Resource sensitivity
Session
Policy version
Execution constraints
Approval state
```

The exact representation is implementation-specific.

The important property is that authorization decisions are based on explicit security context rather than implicit assumptions inside the model.

---

# 5. Admission Policy

The first policy decision determines whether the agent execution itself is permitted.

Possible outcomes include:

```text
ALLOW
DENY
REQUIRE_ADDITIONAL_CONTROL
```

Admission controls may consider:

- principal status
- agent identity
- tenant status
- environment
- requested capabilities
- policy state
- system availability
- risk conditions

An execution that cannot satisfy admission policy should not proceed merely because the model can technically process the request.

---

# 6. Agent Reasoning

Once execution has been admitted, the agent can perform its reasoning process.

The agent may:

- interpret the request
- retrieve information
- generate a plan
- reason over context
- select candidate tools
- construct parameters
- determine an execution sequence

The reasoning process remains probabilistic.

AZTA does not attempt to make the model itself a deterministic security mechanism.

Instead, model output becomes an input to deterministic enforcement.

```text
             AGENT
               │
        ┌──────┴──────┐
        │             │
      Reason        Plan
        │             │
        └──────┬──────┘
               │
               ▼
        Proposed Action
               │
               ▼
       External Policy
```

---

# 7. Action Proposal

Before a consequential operation produces an external effect, the agent's proposed action should be represented in a form that downstream controls can evaluate.

Conceptually:

```text
Action Proposal
├── Principal
├── Agent
├── Capability
├── Target
├── Operation
├── Parameters
├── Context
└── Risk metadata
```

The proposal is not an authorization.

For example:

```text
Agent:
"Update ticket 12345 with the following content."
```

means:

```text
PROPOSED ACTION
```

It does not mean:

```text
AUTHORIZED ACTION
```

The distinction is fundamental to AZTA.

---

# 8. Action-Level Policy Evaluation

The proposed action is evaluated independently of the model.

A conceptual policy function is:

```text
authorize(
    principal,
    agent,
    capability,
    target,
    operation,
    context
)
```

The policy engine returns a deterministic decision.

```text
             ACTION
               │
               ▼
       ┌─────────────────┐
       │ Policy Gateway  │
       └────────┬────────┘
                │
        ┌───────┼────────┐
        │       │        │
        ▼       ▼        ▼
      ALLOW   DENY    APPROVAL
        │       │        │
        │       │        ▼
        │       │     HITL Gate
        │       │        │
        │       │    ┌───┴───┐
        │       │    ▼       ▼
        │       │ APPROVE   DENY
        │       │    │       │
        └───────┴────┘       │
                │             │
                ▼             ▼
             EXECUTE         STOP
```

No model-generated instruction should be able to bypass this decision.

---

# 9. Capability Verification

An action should be evaluated against the capabilities actually granted to the execution.

For example:

```text
Agent capability:
ticket.read
```

does not automatically imply:

```text
ticket.update
```

Likewise:

```text
database.read
```

does not automatically imply:

```text
database.delete
```

Capabilities should be narrowly scoped where practical.

Scope can include:

- operation
- resource
- tenant
- environment
- time
- quantity
- network destination
- data classification

The objective is to reduce unnecessary authority.

---

# 10. Execution Admission

After policy authorization, the operation enters the execution boundary.

The execution environment should enforce the constraints established by the control plane.

Conceptually:

```text
Policy Decision
      │
      ▼
Execution Admission
      │
      ├── Runtime identity
      ├── Capabilities
      ├── Network policy
      ├── Filesystem policy
      ├── Resource limits
      └── Execution lifetime
      │
      ▼
Controlled Runtime
```

The execution environment should not silently expand the authority granted by policy.

---

# 11. Ephemeral Execution

Where isolation requirements justify it, the operation executes inside an ephemeral runtime.

A conceptual lifecycle is:

```text
Provision
   │
   ▼
Configure
   │
   ▼
Verify Controls
   │
   ▼
Execute
   │
   ▼
Capture Result
   │
   ▼
Terminate
   │
   ▼
Destroy
```

Ephemeral execution helps reduce persistence between executions.

Potential controls include:

- restricted filesystem
- isolated network
- limited process privileges
- resource quotas
- execution timeouts
- controlled credentials
- restricted tool access

The exact runtime mechanism is implementation-specific.

---

# 12. Human Authorization

Some operations require a human decision.

When policy returns:

```text
REQUIRE_APPROVAL
```

the execution should enter a controlled pending state.

```text
Action Proposal
      │
      ▼
Policy Gateway
      │
      ▼
REQUIRE_APPROVAL
      │
      ▼
Execution Paused
      │
      ▼
Approval Request
      │
      ├── APPROVED ──► Resume
      │
      ├── DENIED ────► Terminate
      │
      └── EXPIRED ───► Policy-defined outcome
```

The model cannot approve its own request.

The approval event must originate from the designated authorization mechanism.

---

# 13. External Effect

Only after all applicable controls have succeeded should the operation produce an external effect.

Examples include:

- creating or modifying a record
- sending a message
- executing infrastructure changes
- writing to a database
- invoking an external API
- changing permissions
- initiating a financial operation

The target system should ideally provide its own authentication and authorization controls as an additional security boundary.

AZTA does not replace downstream system security.

It provides an additional control plane around agentic behavior.

---

# 14. Result Handling

The target system returns a result.

That result may contain:

- success
- failure
- partial completion
- validation error
- authorization failure
- timeout
- unexpected response
- external state

Tool output should continue to be treated according to its trust classification.

A successful tool call does not automatically make returned content trustworthy as instructions.

---

# 15. Telemetry and Evidence

Security-relevant events should be recorded throughout the execution lifecycle.

At minimum, implementations should be capable of correlating:

```text
Request
  │
  ├── Identity
  │
  ├── Agent
  │
  ├── Policy Decision
  │
  ├── Action Proposal
  │
  ├── Tool Invocation
  │
  ├── Runtime
  │
  ├── Approval
  │
  ├── External Effect
  │
  └── Result
```

The Forensic Telemetry Engine provides the evidence necessary to reconstruct this sequence.

The detailed event representation belongs to the AZTA audit schema.

---

# 16. Execution Closure

An execution should have an explicit terminal state.

Possible states include:

```text
COMPLETED
DENIED
REJECTED
FAILED
CANCELLED
TIMED_OUT
TERMINATED
EXPIRED
```

Implementations may define additional states.

The final state should be correlated with the originating execution and its security decisions.

---

# State Machine

The complete lifecycle can be represented as:

```text
                 ┌──────────────┐
                 │   RECEIVED   │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │  INSPECTED   │
                 └──────┬───────┘
                        │
                  ┌─────┴─────┐
                  │           │
                  ▼           ▼
               REJECT       ADMIT
                  │           │
                  ▼           ▼
                STOP       REASON
                              │
                              ▼
                         PROPOSE ACTION
                              │
                              ▼
                       POLICY EVALUATION
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
            DENY           ALLOW          APPROVAL
              │               │               │
              ▼               │         ┌─────┴─────┐
            STOP              │         │           │
                              │      APPROVE      DENY
                              │         │           │
                              └────┬────┘           │
                                   │                │
                                   ▼                ▼
                              EXECUTION           STOP
                                   │
                                   ▼
                               EFFECT
                                   │
                                   ▼
                                RESULT
                                   │
                                   ▼
                               TELEMETRY
                                   │
                                   ▼
                                 CLOSE
```

---

# Re-Authorization

Agentic workflows may contain multiple actions.

Authorization should therefore not necessarily be treated as a one-time decision for the entire workflow.

For example:

```text
Request
   │
   ▼
Agent Plan
   │
   ├── Read customer record
   │       │
   │       └── Policy → ALLOW
   │
   ├── Modify customer record
   │       │
   │       └── Policy → ALLOW / APPROVAL
   │
   └── Delete customer record
           │
           └── Policy → DENY
```

Each consequential action can be independently evaluated.

This prevents an initially authorized request from becoming an implicit authorization for every subsequent action generated by the agent.

---

# Delegation

Agent systems frequently operate through delegated identities or subordinate agents.

Delegation should preserve the authority boundaries established by the original principal.

Conceptually:

```text
Principal
    │
    │ delegates bounded authority
    ▼
Agent
    │
    │ proposes bounded action
    ▼
Tool / Sub-Agent
```

A delegated agent should not automatically receive authority greater than the authority available to the delegating principal and execution context.

The exact delegation model is implementation-specific and should be explicitly represented in policy and audit records.

---

# Credential Boundaries

Credentials should be separated from model reasoning wherever practical.

A preferred conceptual flow is:

```text
Agent
  │
  │ requests capability
  ▼
Control Plane
  │
  │ validates authorization
  ▼
Credential / Token Broker
  │
  │ issues scoped access
  ▼
Tool / Target System
```

The model does not need unrestricted possession of the underlying credential simply because the agent has permission to perform an operation.

Credential lifetime and scope should be minimized according to deployment requirements.

---

# Kill Switch and Termination

An AZTA deployment should provide an administrative mechanism capable of interrupting agent execution.

Potential triggers include:

- active security incident
- runaway execution
- policy violation
- compromised credential
- anomalous behavior
- operator intervention
- infrastructure compromise

Conceptually:

```text
             KILL SWITCH
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
    Stop Agent  Revoke    Isolate
               Tokens     Runtime
        │         │         │
        └─────────┼─────────┘
                  ▼
             Record Event
```

Termination should itself generate telemetry.

A kill switch that cannot be observed or audited reduces the quality of incident reconstruction.

---

# Replay and Investigation

Where sufficient evidence exists, telemetry should allow investigators to reconstruct the relevant execution sequence.

A useful investigation path is:

```text
Execution ID
     │
     ▼
Request
     │
     ▼
Identity
     │
     ▼
Policy Decisions
     │
     ▼
Agent Actions
     │
     ▼
Tool Calls
     │
     ▼
Runtime Events
     │
     ▼
Human Decisions
     │
     ▼
External Effects
     │
     ▼
Final State
```

This does not necessarily imply that an execution can be perfectly reproduced.

The objective is to preserve enough evidence to understand what occurred and why the control plane permitted, denied, interrupted, or terminated the relevant activity.

---

# Security Properties

An AZTA execution model should provide the following properties:

## Authorization precedes consequential execution

A proposed operation should not produce an external effect before applicable authorization controls have succeeded.

## Model reasoning does not establish authority

Model output is treated as proposed intent rather than proof of permission.

## Authorization can be action-specific

A workflow can contain multiple policy decisions rather than inheriting unrestricted authority from its initial request.

## Human approval is external

Approval events originate from an authorization mechanism rather than from model output.

## Execution is constrained

The runtime enforces boundaries appropriate to the operation's risk.

## Credentials are controlled

Agent access to credentials should be scoped and governed independently of model reasoning.

## Security events are observable

Relevant decisions and execution events produce correlated evidence.

## Termination is possible

Authorized operators or automated controls can interrupt execution when required.

---

# Implementation Contract

An AZTA implementation should document the following for every execution path:

| Requirement | Description |
| --- | --- |
| Principal | Who or what initiated the operation |
| Agent | Which agent performed the reasoning |
| Capability | What authority was requested |
| Target | Which resource or system was affected |
| Operation | What action was proposed |
| Policy | Which policy governed the decision |
| Decision | Allow, deny, approval, or implementation-defined state |
| Runtime | Where execution occurred |
| Approval | Whether human authorization was required |
| Effect | What external state changed |
| Evidence | Which telemetry records the lifecycle |
| Final State | How execution terminated |

This contract provides the bridge between the conceptual AZTA architecture and the machine-readable specification.

---

# Core Execution Invariant

The AZTA execution model can be reduced to five distinct concepts:

```text
REASON
  │
  ▼
PROPOSE
  │
  ▼
AUTHORIZE
  │
  ▼
EXECUTE
  │
  ▼
PROVE
```

The separation is intentional.

> **The agent determines what it wants to do. The control plane determines what it may do. The runtime determines how it is contained. The evidence layer records what happened.**

This separation is the operational foundation of Agentic Zero Trust Architecture.
