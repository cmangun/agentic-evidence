# Quick tour

> If you have 5 minutes, read this. It points you at the three documents that show the architectural thesis.

## What this is

The Agentic Evidence Suite is a reference architecture for building agentic AI systems whose execution is independently verifiable. It targets engineering teams shipping AI agents into regulated, audited, or high-stakes environments where "did this run actually do what it claims" is a hard requirement.

## The architectural thesis

Enterprise AI fails when the system cannot answer five questions with confidence:

1. **What happened?**
2. **Why was it allowed?**
3. **What evidence was produced?**
4. **Which controls were in force?**
5. **Can an independent reviewer verify it?**

The suite addresses each question with a specific architectural component, with deny-by-default policy, hash-chained receipts, and reviewer-safe evidence bundles as the load-bearing primitives.

## Read these three documents in this order

### 1. [REFERENCE-ARCHITECTURE.md §1–§3](REFERENCE-ARCHITECTURE.md) — 8 minutes
The architectural overview. Read the abstract (§1), the design principles (§2), and the three trust boundaries (§3). The Mermaid diagram in §1 is the single picture that shows how the six components compose. After this you understand what the suite is and why the boundaries are drawn where they are.

### 2. [INTEROP.md §3 (OpenTelemetry)](INTEROP.md) — 5 minutes
How the suite relates to a standard the reader almost certainly already uses. This section names a deliberate divergence from the project whitepaper's earlier framing and explains the reasoning. Reading it shows the suite is not trying to replace standards adopters already have, and it's also where the architectural opinions are sharpest.

### 3. [VERSIONING.md §6 (Stability markers)](VERSIONING.md) — 4 minutes
The `_experimental` field convention. Tells you what's stable in v0.1 and what isn't, with the architectural reasoning for the spec-stable / implementation-experimental split.

## After those three documents

If you want to go deeper on a specific concern:

- **Cryptographic and threat model** → [`agentic-receipts/spec/threat-model.md`](https://github.com/cmangun/agentic-receipts/blob/main/spec/threat-model.md)
- **Policy enforcement architecture** → [REFERENCE-ARCHITECTURE.md §5.2](REFERENCE-ARCHITECTURE.md) and the three [policy-engine ADRs](https://github.com/cmangun/agentic-policy-engine/tree/main/adrs)
- **Evaluation methodology** → [`agentic-eval-harness/adrs`](https://github.com/cmangun/agentic-eval-harness/tree/main/adrs)
- **The 26 conformance vectors that prove the architecture** → [`agentic-receipts/vectors/`](https://github.com/cmangun/agentic-receipts/tree/main/vectors)

## What is and isn't shipped at v0.1.0

The spec layer (receipt schemas, canonicalization rules, hashing, signing, bundle layout) is stable. Implementation layers across the six component repos are scaffolded with v0.1.0 tags marking the start of the v0.1 release line — they're explicitly experimental and tracked toward conformance via the vector suite. See [ROADMAP.md](ROADMAP.md) for the live status grid.
