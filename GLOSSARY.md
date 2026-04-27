# Glossary

> Precise definitions for terms used across the Evidence Suite. Each entry names what the term *is*, the conflations it disambiguates from, and where in the suite it is used. Terms used inline within a single component repo without conflation risk (`correlation_id`, `chain`, `canonicalization`, `conformance vector`) are defined where they live; this glossary is for the cross-suite vocabulary that gets conflated.

## Receipt

**Definition.** A canonicalized, hash-chained, signed record of a single agent action or policy decision. Receipts are the atomic unit of evidence in the suite. Two subtypes — *action receipt* and *decision receipt* — share the receipt envelope and chain rules but carry different payloads.

**Disambiguates from.** Logs (which lack signing and chaining and are not checkable for omission); audit events (typically descriptive after the fact, not tamper-evident at write time); OpenTelemetry spans (which model execution but neither sign nor chain).

**Used in.** `agentic-receipts` defines the receipt schema; every component in the suite either produces, consumes, or references receipts.

## Action receipt

**Definition.** A receipt whose payload describes a proposed or executed agent action — a tool call, an LLM completion, an external resource access. An agent emits two action receipts per action: one for the proposal (before policy evaluation) and one for the result (after execution).

**Disambiguates from.** Decision receipt (which records *why* an action was permitted or refused, not what was attempted).

**Used in.** Emitted by the agent runtime; recorded in `agentic-receipts`; cross-referenced by artifact manifests in `agentic-artifacts`.

## Decision receipt

**Definition.** A receipt whose payload describes a single policy evaluation outcome — the inputs the policy engine received, the rule (or absence thereof) that matched, and the allow/deny verdict. Decision receipts are emitted symmetrically: every allow and every deny produces one.

**Disambiguates from.** Action receipt (records the action itself, not the authorization); policy log entry (the non-signed equivalent in pre-suite stacks).

**Used in.** Emitted by `agentic-policy-engine`; recorded in `agentic-receipts`; consumed by audit verification flows.

## Trace

**Definition.** The hash-chained sequence of receipts produced during a single agent session. A trace's root hash is the integrity anchor for the entire session; a single receipt's `correlation_id` carries the trace's identifier.

**Disambiguates from.** *OpenTelemetry trace* — a related but distinct concept covering observability spans rather than signed evidence. The two cross-reference (see `INTEROP.md` §3) but are not the same object.

**Used in.** `agentic-receipts` defines the chain rules; `agentic-trace-cli` operates on traces; `agentic-evidence-viewer` displays them.

## Artifact

**Definition.** A signed output object produced by an agent — a file, a structured record, a generated document. An artifact carries its own integrity hash and links back to the receipt that produced it.

**Disambiguates from.** Receipt (records action; artifacts record output objects). *Build artifact* in SLSA terminology — which describes a *build-time* output; here, "artifact" is *runtime* output.

**Used in.** `agentic-artifacts` defines the manifest schema; `agentic-evidence-viewer` previews artifacts safely.

## Manifest

**Definition.** The structured envelope describing an artifact: filename, content hash, the receipt that produced it, the bundle that contains it, and (where applicable) cross-references to SLSA build provenance.

**Disambiguates from.** *Bundle manifest* (`bundle.json`, which describes the bundle as a whole and signs the bundle root hash). Artifact manifest is per-artifact; bundle manifest is per-bundle.

**Used in.** `agentic-artifacts` exclusively.

## Bundle

**Definition.** The portable export unit of the suite. A bundle is a directory or zipfile containing the receipts chain, artifact manifests with their files, and a top-level `bundle.json` identifying the signing identity and the bundle root hash. A bundle is the canonical exchange object between systems and the canonical input to verification.

**Disambiguates from.** Receipt (atomic unit) and trace (chain of receipts in one session). A bundle is a complete, exportable, self-contained verification target.

**Used in.** Produced by `agentic-trace-cli export`; consumed by `agentic-evidence-viewer`; transmitted via CloudEvents envelopes in cross-system flows.

## Provenance

**Definition.** *Action provenance* — the signed record of what an agent did. In the suite, "provenance" is used specifically to mean the action history captured in the receipts chain, not the data lineage of output objects.

**Disambiguates from.** *Data lineage* (object lineage, used for artifacts; see next entry). *SLSA provenance* (build-time provenance of the agent runtime, distinct from runtime action provenance).

**Used in.** `agentic-receipts` — the action provenance is the receipt chain itself.

## Data lineage

**Definition.** *Object lineage* — the signed record of what data flowed where, with integrity hashing. In the suite, "data lineage" is used specifically to mean the artifact-side counterpart to action provenance: the chain that links an output object back through the receipts to the action that produced it and the inputs that fed it.

**Disambiguates from.** *Provenance* (action-side; what an agent did). Conflating the two is one of the most common mistakes in this design space — they share verification primitives (signatures, hashes, references) but solve different audit questions.

**Used in.** `agentic-artifacts` — the manifest's lineage links to the receipt that produced the artifact.
