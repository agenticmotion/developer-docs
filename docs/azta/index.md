---
layout: default
title: AZTA Architecture
permalink: /docs/azta/
---

# AZTA Architecture

Agentic Zero Trust Architecture (AZTA) separates non-deterministic agent reasoning from deterministic security enforcement.

## Architectural Model

```text
User / System
      │
      ▼
┌─────────────────────────────┐
│ Layer 1                     │
│ Ingress Semantic Firewall   │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Layer 2                     │
│ Deterministic Policy        │
│ Gateway                     │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Agent / LLM Reasoning       │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Layer 3                     │
│ Ephemeral MicroVM           │
│ Sandboxing                  │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Layer 4                     │
│ Asynchronous HITL           │
│ Interceptor                 │
└─────────────┬───────────────┘
              │
              ▼
┌─────────────────────────────┐
│ Layer 5                     │
│ Forensic Telemetry Engine   │
└─────────────────────────────┘
````
 
## Enforcement Layers
 
1. **Ingress Semantic Firewall** — classify and sanitize inbound requests before they reach agent execution.
2. **Deterministic Policy Gateway** — evaluate authorization and execution policy independently of LLM reasoning.
3. **Ephemeral MicroVM Sandboxing** — contain agent execution and reduce blast radius.
4. **Asynchronous HITL Interceptor** — introduce human authorization where policy requires it.
5. **Forensic Telemetry Engine** — maintain traceable, integrity-protected runtime evidence.

 
## Design Principle
 
AZTA treats the LLM as an untrusted reasoning component rather than a trusted security boundary.
 
Security controls therefore remain enforceable even when model behavior is manipulated, misaligned, or compromised.
