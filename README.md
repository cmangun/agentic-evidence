# Evidence Suite

> Six interoperating components for verifiable agentic AI. This repo holds the suite-level reference architecture and specifications.

## Overview

The Evidence Suite is a set of six open-source components for building agentic AI systems whose execution can be independently verified. It targets architects and engineering teams shipping AI agents into regulated, audited, or high-stakes environments — where after-the-fact reasoning ("did this run actually do what it claims?") is a hard requirement, not a nice-to-have.

Implementation lives in the six component repos linked below. This repo (`agentic-evidence`) holds the cross-cutting documents: reference architecture, specifications index, standards interop, glossary, roadmap, and versioning policy.

## The Evidence Suite

| Component | Lane | Repo |
|---|---|---|
| **Receipts** | Cryptographic provenance evidence — signed, hash-chained records of what an agent did. | [agentic-receipts](https://github.com/cmangun/agentic-receipts) |
| **Policy engine** | Deny-by-default enforcement at agent-action boundaries; emits decision receipts. | [agentic-policy-engine](https://github.com/cmangun/agentic-policy-engine) |
| **Eval harness** | Bypass / injection / determinism scenarios with regression gates. | [agentic-eval-harness](https://github.com/cmangun/agentic-eval-harness) |
| **Artifacts** | Manifest schemas and provenance rules for integrity-preserved agent outputs. | [agentic-artifacts](https://github.com/cmangun/agentic-artifacts) |
| **Trace CLI** | Create, sign, redact, verify, and export verifiable agent traces. | [agentic-trace-cli](https://github.com/cmangun/agentic-trace-cli) |
| **Evidence viewer** | Drag-drop UI for inspecting traces, receipts, policy decisions, and artifacts. | [agentic-evidence-viewer](https://github.com/cmangun/agentic-evidence-viewer) |

## Reference architecture

The full reference architecture is at [REFERENCE-ARCHITECTURE.md](REFERENCE-ARCHITECTURE.md). It defines the three trust boundaries, the six components, the data flow through a single agent run, and the architectural positions taken for the v0.1 release line.

## Specifications

Specifications live in their respective component repos. As they stabilize toward v0.1, this section will index them directly: the receipt schema and canonicalization rules, the policy decision-receipt format, the artifact manifest format, and the evaluation-scenario contract.

## Standards interop

The Evidence Suite is designed to interoperate with existing standards rather than replace them. Mappings to OpenTelemetry, SLSA, W3C Verifiable Credentials, the Model Context Protocol, and CloudEvents publish in this repo as `INTEROP.md`.

## Whitepaper

A long-form treatment of the design rationale — why receipts, why deny-by-default, why a separable evaluation harness — is available as [PDF](docs/whitepaper/evidence-suite-whitepaper.pdf) and [HTML source](docs/whitepaper/evidence-suite-whitepaper.html). v1.0, April 2026.

## Glossary

Precise definitions for the core suite vocabulary — receipt, trace, artifact, bundle, decision-receipt, provenance — publish as `GLOSSARY.md`. The term boundaries matter: conflating receipts with traces, or artifacts with bundles, is a common source of design confusion in this problem space.

## Roadmap and versioning

Per-component stability status (stable vs experimental) and the suite's SemVer rules publish as `ROADMAP.md` and `VERSIONING.md`. Until each component carries a tagged v0.1.0 release with conformance vectors, treat all interfaces as unstable.

## Contribute

The suite opens to external contribution at v0.1.0, once each component repo carries a tagged release, conformance vectors, and stable specifications. In the interim, issues are tracked publicly in each component repo's `v0.1` milestone for visibility and design discussion.

## License

MIT. See [LICENSE](LICENSE).
