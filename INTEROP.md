# Standards Interop

> Map from Evidence Suite primitives to adjacent standards. Companion to `REFERENCE-ARCHITECTURE.md`. Canonical interop reference for the v0.1 release line.

## §1. How to use this document

The Evidence Suite is designed to coexist with the standards already deployed in production agent stacks: OpenTelemetry for observability, SLSA for software supply-chain provenance, W3C Verifiable Credentials for portable cryptographic claims, the Model Context Protocol for agent-tool interactions, and CloudEvents for cross-system event transport. Each section below names the integration shape, the concrete field mappings, and the cases where the suite delegates to the existing standard rather than carrying the same information itself.

The adjacency principle is load-bearing. The suite *augments* these standards rather than replacing any of them. Adopters who already operate an OpenTelemetry pipeline, a SLSA-instrumented build system, or a credential-issuance service do not retire that infrastructure to use the Evidence Suite — the receipt and bundle layer cross-references it.

This document is the canonical interop reference. Where a position here tightens or supersedes a statement made in the project whitepaper or a per-component README, that supersession is explicit. The reading order is by need: operators with an OpenTelemetry stack should read §3; build-time provenance teams should read §4; identity-and-credentialing integrators should read §5; MCP-based agent platform builders should read §6; teams transmitting bundles across system boundaries should read §7. The compatibility matrix in §8 names the specific versions of each standard the suite has been tested against in v0.1.

## §2. Quick reference

| Standard | What both do | Where they diverge | Suite's integration shape |
|---|---|---|---|
| **OpenTelemetry** | Model agent execution as structured records. | OTel does not sign, chain, or detect omission. Receipts are signed, hash-chained, integrity-checkable. | Parallel record streams. `correlation_id` (receipt) ↔ `trace_id` (OTel). |
| **SLSA** | Cryptographic tamper-evidence for software-lifecycle artifacts. | SLSA describes how an artifact came to exist; receipts describe what an agent did. | Custom in-toto predicate `evidence-suite/receipt-link/v1` references the receipt that produced the artifact. |
| **W3C VC** | Issuer-signed claims with offline verification. | VCs target discrete human-scale claims; receipts target high-volume execution trails. | Selective receipt-to-VC re-framing for portable claims. VCDM 2.0, JWT serialization. |
| **MCP** | Standard agent-tool interaction shape. | MCP defines the call; receipts define the evidence of the call. | One signed receipt per MCP tool call; policy gate at the tool-call boundary. |
| **CloudEvents** | Common envelope for events across systems. | Envelopes travel; bundles persist as portable evidence inside them. | HTTP binding canonical; bundle becomes the CloudEvents `data` payload. |

## §3. OpenTelemetry

The Evidence Suite and OpenTelemetry are designed to run as **parallel record streams**, with cross-references between them rather than embedding one inside the other. An agent runtime emits both: an OpenTelemetry span for each unit of execution, and a signed receipt for each agent action and each policy decision. The two streams reach different consumers — spans flow to observability backends, receipts flow to evidence storage — and have different lifecycles, retention requirements, and access controls.

**Field mapping.** The cross-reference is by correlation:

| OpenTelemetry | Evidence Suite receipt | Notes |
|---|---|---|
| `trace_id` | `correlation_id` (top-level chain field) | 1:1 per agent session, set by the runtime at session start |
| `span_id` | `event_id` (per-receipt) | 1:1 at the per-action call site |
| Span attributes | Receipt body fields | Independent; do not duplicate receipt content into spans |

A receipt's `correlation_id` carries the same value as the corresponding span's `trace_id`. Every action receipt within a session carries an `event_id` that the OpenTelemetry span emitted at the same call site can reference via `span_id`. The result: an SRE looking at an OpenTelemetry trace can locate the related receipts; an auditor reading a receipt can locate the associated OpenTelemetry span where the span data is still retained — which is often shorter than receipts are.

**Embedding is rejected.** A receipt is not embedded inside an OpenTelemetry span attribute. This is a deliberate divergence from a framing in the project whitepaper Chapter 4, which described embedding as one of two acceptable options. This document supersedes that framing for v0.1. The reasons:

1. **Sampling loss.** OpenTelemetry samples spans for cost. A signed receipt embedded inside a span attribute is silently lost when the span is sampled out, breaking the chain integrity the suite depends on.
2. **Size limits.** OpenTelemetry exporters and backends impose per-attribute size caps that vary by deployment. A receipt carrying signatures, hash references, and structured action data routinely exceeds those caps and is truncated; truncation is detected only at verification time, often after the data is unrecoverable.
3. **Standalone verifiability.** A receipt is intended to be verifiable from a bundle, alone, by a third party. Embedded inside a span attribute, the receipt's chain context is buried inside an observability format that the verifier may not have parsers for.

The canonical integration pattern is therefore *parallel emission*: the runtime calls both the OpenTelemetry span emitter and the receipt-append operation at the same call site, with `correlation_id` set explicitly to match the active `trace_id`. This document does not specify how the runtime maintains that binding; that is a runtime-level concern. The constraint is that the binding is set at the call site, not derived after the fact.

## §4. SLSA

The Evidence Suite operates in a different layer of the agent system's lifecycle than SLSA. SLSA records build-time provenance — what source produced what binary, with what builders, against what materials. The Evidence Suite records runtime provenance — what an agent did once running. The two compose into a chain from source repository through build artifact through deployed runtime through agent action through output artifact, giving an auditor a single verifiable thread from code to behavior.

**Cross-reference by custom in-toto predicate.** The suite defines a custom in-toto predicate, `evidence-suite/receipt-link/v1`, carried in the SLSA attestation for an artifact produced by an agent. The predicate body names the receipt that produced the artifact and the bundle that contains it. The choice of a custom predicate, rather than overloading an existing SLSA field or carrying the link in adopter-level metadata, is deliberate: the cross-reference is a first-class claim in the suite's trust model, not a footnote, and a custom predicate signals to SLSA-aware verifiers that this attestation makes a structured statement about runtime evidence.

**Predicate URI.** The canonical predicate type identifier in v0.1 is `https://github.com/cmangun/agentic-evidence/predicates/receipt-link/v1`. The identifier moves to a project-controlled domain when one is established; the v0.1 line carries the GitHub URL as the stable identifier.

**Field mapping.** For an output artifact (a file, a structured record, a generated document) produced by an agent action:

| SLSA attestation field | Evidence Suite linkage |
|---|---|
| `subject[].name` | Artifact filename (matches the `agentic-artifacts` manifest's filename) |
| `subject[].digest.sha256` | Artifact content hash (matches the manifest's `digest`) |
| `predicateType` | `https://github.com/cmangun/agentic-evidence/predicates/receipt-link/v1` |
| `predicate.receipt_id` | Receipt ID that produced the artifact |
| `predicate.bundle_uri` | Bundle URI containing the receipt |

**Worked example.** An agent produces a CSV report as a tool-call output. The runtime writes:

- An action receipt to the receipts chain, capturing the proposed action, the policy decision, and the result.
- An artifact manifest in `agentic-artifacts`, carrying the SHA-256 of the CSV bytes.
- A SLSA attestation with the custom predicate, naming the CSV's digest, the receipt ID, and the bundle URI.

A verifier with both the bundle and the SLSA attestation can now answer four chained questions independently signed: which source repository was the agent runtime built from (SLSA), which builder produced that runtime (SLSA), which agent action produced this artifact (Evidence Suite receipt), and what policy authorized the action (decision receipt).

**Composition with existing SLSA pipelines.** Adopters who already publish SLSA attestations for their agent runtime do not change those pipelines. The suite's predicate is added to existing attestation flows; it does not require a separate signing identity, a separate distribution channel, or modifications to existing SLSA tooling.

## §5. W3C Verifiable Credentials

Receipts and Verifiable Credentials are complementary primitives, not interchangeable. The suite uses hash-chained receipts as its day-to-day record because VCs carry per-claim overhead — issuer DID resolution, credential subject construction, proof generation — that does not amortize well across the thousands of receipts a single agent session can emit. Receipts target volume; VCs target portability of discrete claims across organizational and credentialing-system boundaries.

**When to re-frame a receipt as a VC.** The suite recommends VC re-framing for specific cases where a single, discrete authority claim needs to flow through a VC-aware ecosystem. Representative cases: an *"agent X was authorized to perform action Y at time Z under policy P"* credential issued from one organization to another for downstream verification; a portable consent record for a specific patient interaction; a regulatory-export bundle that the receiving agency processes as a set of VCs.

**Profile and serialization.** The suite targets **VCDM 2.0** with **JWT serialization** for the worked transformation. VCDM 1.1 is in maintenance mode; new integrations target 2.0. JWT is selected for the worked example because it has the lowest reader-friction for engineering audiences integrating with the suite. **JSON-LD serialization works equivalently** and is appropriate where the consuming ecosystem is JSON-LD-native; the field mapping below does not change.

**Transformation pattern.** A re-framed receipt becomes a VC with:

- `issuer` ← the receipt's signing identity, expressed as a DID or HTTPS URL (whichever the receipt's signing scheme uses)
- `credentialSubject.id` ← the receipt's `event_id`
- `credentialSubject` body ← the receipt's structured claim fields (action, policy decision, outcome)
- `proof` ← the receipt's signature, re-encoded per VCDM 2.0 proof rules

The transformation is lossless in the receipt → VC direction. The reverse direction (VC → receipt) requires the original chain context that VCs do not carry; adopters that need bidirectional flow keep both formats and use the receipt as the chain-of-record, the VC as the credential-of-export.

## §6. Model Context Protocol

The Evidence Suite is a natural evidence layer for MCP-based agent platforms. MCP defines how an agent runtime discovers, calls, and receives results from external tools. The Evidence Suite defines what evidence of those calls looks like. The two integrate at the runtime call site without requiring extensions to the MCP specification.

**Version targeting.** MCP is moving fast; this document is **version-agnostic** for the protocol itself. The integration pattern below applies to any MCP version that exposes a tool-call shape with structured arguments and structured results — a property MCP has carried since its initial public release. The compatibility matrix in §8 names the specific MCP schema version the suite has been tested against; that reference moves forward as the protocol evolves.

**The integration pattern.** For each MCP tool call:

1. The agent runtime issues a `tools/call` request to an MCP server.
2. Before the call dispatches, the runtime emits an action receipt (proposal) to the receipts chain.
3. The runtime invokes the policy engine at the tool-call boundary; the engine emits a decision receipt.
4. If allowed, the call dispatches to the MCP server, which returns a result.
5. The runtime emits an action receipt (result) to the receipts chain, referencing the proposal and decision receipts.
6. If the tool produced an output artifact, the runtime writes an artifact manifest cross-referencing the result receipt.

**What MCP servers expose.** MCP servers do not need to know about the Evidence Suite. Receipts are emitted on the *runtime* side, not the server side, so existing MCP servers integrate without modification. The suite reads the MCP tool schema (`tools/list` response) when computing receipt event-taxonomy classifications, but does not require the MCP server to opt in or expose additional fields.

**Policy boundary.** The policy gate sits between step 1 and step 4 — at the tool-call boundary, per `REFERENCE-ARCHITECTURE.md` §4. The suite does not gate inside the MCP server; the server is downstream of the gate and is treated as an external resource for trust-model purposes.

## §7. CloudEvents

Bundles can travel across system boundaries inside CloudEvents envelopes when an adopter's transport infrastructure is CloudEvents-native. The bundle's signatures and chain integrity survive transit; the envelope provides routing, type identification, and delivery semantics that the bundle itself does not carry.

**Binding scope.** This document specifies the **HTTP binding** as the canonical example. Other bindings (Kafka, AMQP, MQTT, NATS) work identically at the integration layer — the bundle becomes the CloudEvents `data` payload regardless of binding — but the field-level details below are spec'd against HTTP for clarity. Adopters using non-HTTP bindings consult the CloudEvents binding spec for that transport; the bundle-side semantics are unchanged.

**Field mapping (CloudEvents 1.0.x).**

| CloudEvents field | Evidence Suite mapping |
|---|---|
| `id` | Bundle root hash, hex-encoded |
| `source` | Signing identity URL (the same URL the bundle's `bundle.json` names) |
| `type` | `agentic-evidence.bundle.v1` |
| `datacontenttype` | `application/zip` for compressed bundles, or `application/x-evidence-suite-bundle` for tar-serialized directory bundles |
| `data` | The bundle bytes |
| `time` | Bundle export time |

A CloudEvents receiver that does not understand the suite's `type` value can still route the envelope by binding semantics; the bundle parses correctly using only the suite's verification tools (`agentic-trace-cli verify` or `agentic-evidence-viewer`).

## §8. Compatibility matrix

The matrix below names the standard versions the suite has been tested against in the v0.1 release line. *Tested* means conformance vectors exist for the pairing and pass. *Compatible by design* means the integration pattern documented above applies to the named version range, but conformance vectors do not yet exist for that pairing — adopters integrating against those standards should expect to surface gaps as the v0.1 line matures.

| Standard | Version | Status in suite v0.1 |
|---|---|---|
| OpenTelemetry | 1.x | Tested — cross-reference pattern verified end-to-end |
| SLSA | v1.0 | Tested — custom predicate emitted and verified |
| W3C VCDM | 2.0 | Compatible by design; conformance not yet tested |
| MCP | 2024-11-05 schema | Compatible by design; conformance not yet tested |
| CloudEvents | 1.0.x | Compatible by design; conformance not yet tested |

This matrix updates with each suite release. Where a status changes between releases, that change appears in `ROADMAP.md` for the affected release.
