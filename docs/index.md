---
layout: default
title: AZTA Documentation
permalink: /docs/
---

# Agentic Zero Trust Architecture

## AZTA

**Open Standard. Closed Execution.**

Agentic Zero Trust Architecture (AZTA) is a security architecture for governing AI agents operating across models, tools, identities, data, and execution environments.

AZTA separates probabilistic reasoning from deterministic security enforcement.

## Core Principle

> The model may reason. The control plane decides.

Security-critical decisions are enforced outside the language model through deterministic policy enforcement, identity controls, execution containment, human intervention, and forensic telemetry.

## Documentation

### Architecture

Understand the AZTA architecture, enforcement layers, execution model, and threat model.

[Explore Architecture →]({{ '/docs/azta/' | relative_url }})

### Specification

Technical specifications for schemas, policies, telemetry, and interoperability.

[Explore Specification →]({{ '/docs/specification/' | relative_url }})

### Security

Threats and controls covering prompt injection, confused-deputy attacks, tenant isolation, and tool execution.

[Explore Security →]({{ '/docs/security/' | relative_url }})

### Compliance

Technical mappings between AZTA controls and relevant AI governance frameworks.

[Explore Compliance →]({{ '/docs/compliance/' | relative_url }})

### Reference

Implementation-oriented references for JSON Schema, OPA/Rego, OpenTelemetry, MCP, and related components.

[Explore Reference →]({{ '/docs/reference/' | relative_url }})

## Public Specification

The public AZTA specification is intended to provide implementable architectural patterns and machine-readable controls for production AI-agent systems.

[Read the AZTA Whitepaper →]({{ '/whitepaper/AZTA_WHITE_PAPER/' | relative_url }})
