# Roadmap

> Stability classification and release-line framing for the Evidence Suite. Companion to `VERSIONING.md`, which carries the SemVer rules in detail. This document tells a reader what to depend on and what to expect to change.

## §1. Suite release lines

The Evidence Suite versions its components independently per `REFERENCE-ARCHITECTURE.md` §10; this document tracks the suite-level release lines that pin compatible component-version ranges. Three lines are conceptually defined; only **v0.1** is in active build.

**v0.1 — current line.** Spec layer stable; implementation layers experimental. Released when each component reaches its conformance-vector target — quality-gated, not date-gated. The v0.1 line is suitable for adopters who want to integrate against the spec and tolerate implementation-surface mutability.

**v0.2 — sketched.** Adds the post-quantum signature path (dual-signature scheme with Dilithium3 or equivalent), cross-organization bundle interop (federated key disclosure / shared-custody model), and the first stable command surfaces for the CLI and viewer. Conceptual framing only — no work scheduled until v0.1 conformance is complete.

**v1.0 — sketched.** Full suite stability across all six components. Standards-body engagement (W3C / IETF working-group submission). External implementations of the receipt spec, with conformance vectors verifying cross-implementation compatibility. Conceptual framing only.

## §2. Component status (v0.1 line)

| Component | Lane | v0.1 status |
|---|---|---|
| `agentic-receipts` | Spec layer | **Stable** — schemas locked; threat model published; conformance vectors define the contract |
| `agentic-policy-engine` | Governance | **Experimental** — decision-receipt body fields stabilizing; rule format may add slots |
| `agentic-eval-harness` | Evaluation | **Experimental** — scenario taxonomy still maturing; gate-threshold rules subject to refinement |
| `agentic-artifacts` | Lineage | **Experimental** — manifest fields likely to add slots in v0.2 (cross-org and post-quantum) |
| `agentic-trace-cli` | Tooling | **Experimental** — CLI command surface not stabilized until v1.0; bundle format stable |
| `agentic-evidence-viewer` | Inspection | **Experimental** — UI surface mutable through v1.0; verification logic stable |

The spec-stable, implementations-experimental pattern is deliberate. Stabilizing implementations before their underlying spec settles is how versioning death spirals start; locking the receipt schema first lets the implementation surfaces evolve against a fixed contract.

## §3. v0.1 compatibility matrix

The matrix below names the component-version ranges that compose into a coherent v0.1 suite release.

```
suite v0.1.0
  agentic-receipts        ≥ 0.1.0, < 0.2.0
  agentic-policy-engine   ≥ 0.1.0, < 0.2.0
  agentic-eval-harness    ≥ 0.1.0, < 0.2.0
  agentic-artifacts       ≥ 0.1.0, < 0.2.0
  agentic-trace-cli       ≥ 0.1.0, < 0.2.0
  agentic-evidence-viewer ≥ 0.1.0, < 0.2.0
```

## §4. What's stable in v0.1

The following are committed to remain stable through the v0.1 line — no breaking changes without a major bump and a stated deprecation window:

- Receipt schemas in `agentic-receipts/spec/`
- Canonicalization rules (whitespace, key ordering, float handling, UTF-8 normalization)
- Hashing scheme (SHA-256)
- Signing scheme (Ed25519, single-signature)
- Bundle file layout (`receipts.jsonl`, `artifacts/`, `bundle.json`)
- Cross-reference field shape between artifacts and receipts

## §5. What's experimental in v0.1

The following carry no stability guarantee in v0.1 and may change between minor releases:

- CLI command surface (`agentic-trace-cli` argument shapes, output formats)
- Viewer UI (`agentic-evidence-viewer` panels, navigation, export options)
- Decision-receipt body fields beyond the required core
- Eval-harness scenario taxonomy edges (the five canonical classes are stable; sub-classifications may change)
- Policy rule-format extensions
- Artifact manifest body fields beyond the required core

## §6. Known limitations

- **Cross-organization bundle interop** is deferred to v0.2. v0.1 expects bundles to be intra-organizational; cross-org composition requires a shared-custody model in active design.
- **Post-quantum signature path** is deferred to v0.2. v0.1 uses Ed25519 alone; the dual-signature migration plan is sketched in `agentic-receipts/spec/threat-model.md`.
- **Redaction-with-integrity edges**: selective-disclosure cryptography (BBS+ signatures, zkSNARKs in limited domains) is promising but not yet practical at execution volumes; v0.1 redaction loses content while preserving hash integrity.
- **External conformance**: v0.1 conformance vectors test the reference implementations only; cross-implementation conformance against external receipt-spec implementations is a v1.0 goal.