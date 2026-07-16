# AI-Native Architecture Concept Pack

[中文入口](./README.zh-CN.md)

An open-source research and design package for helping teams decide what should become agentic, design it with explicit evidence and control boundaries, and govern the resulting capabilities without building another generic MCP gateway.

## Project status

This repository contains an **accepted design and an unvalidated Skill draft**. It does not yet contain a production implementation, a behaviorally verified host integration, or a released MCP control-plane product.

| Track | Accepted direction | Current state |
|---|---|---|
| Part I — AI Native Architect Skill | Agent Skills–compliant portable core with separate host adapters; OpenClaw is the first behavioral verification target | Design accepted; implementation and verification not started |
| Part II — Capability Control Plane | Architecture-to-policy compiler and independent assurance layer; read-only and out of the runtime data path for the MVP | Design accepted; implementation and market validation not started |

## Canonical design RFCs

1. [RFC 0001 — Portable AI Native Architect Skill](./docs/rfcs/0001-portable-ai-native-architect-skill.md)
2. [RFC 0002 — Architecture-Driven Capability Control Plane](./docs/rfcs/0002-architecture-driven-capability-control-plane.md)

These RFCs are the normative design source for Part I and Part II. The earlier strategy document remains research background and product context.

## Research and current artifacts

- [Chinese product strategy](./AI_Native_Architecture_and_MCP_Product_Strategy_zh.md) — original concept, market framing, rollout hypothesis, risks, and research background.
- [Current `ai-native-architect` Skill draft](./ai-native-architect/SKILL.md) — pre-RFC draft retained for comparison. It must not be described as verified.

## Recommended review order

1. Read this project status and the Chinese entry if useful.
2. Review RFC 0001 for the user workflow, portability, evidence, output, safety, and host-verification contract.
3. Review RFC 0002 for the product boundary, capability contracts, policy compilation, drift, assurance, and read-only MVP.
4. Compare the RFCs with the original strategy and current Skill draft.
5. Open review issues for assumptions, missing evidence, falsification conditions, security boundaries, or interoperability gaps.

## Claim policy

- “Agent Skills–compliant” describes a file-format target, not behavioral quality.
- “Portable core” describes the accepted architecture. Demonstrated cross-host portability requires contract evidence from at least two independent hosts.
- “Compatible” and “verified” are scoped to an exact host, version, environment, installation method, bundle digest, and test suite.
- Registry, marketplace, or scanner approval does not prove safe behavior.
- The control-plane MVP does not enforce runtime policy or handle credentials.
- OpenClaw and ClawHub trust verification are separate from behavioral verification.

## Current release blockers

- The existing Skill has not passed the accepted contract or host evaluation suite.
- No host may be labeled `Native/Verified` yet.
- The current Skill declares Apache-2.0 while current ClawHub publishing documentation requires MIT-0 for published Skills. ClawHub publication is therefore blocked until the rights holder approves a compatible licensing strategy.
- Implementation planning begins only after the written RFCs complete maintainer review.
