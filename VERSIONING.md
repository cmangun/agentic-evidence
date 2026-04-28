# Versioning

> SemVer rules, breaking-change definitions, and deprecation policy for the Evidence Suite. Companion to `ROADMAP.md` (live release status) and `REFERENCE-ARCHITECTURE.md` §10 (architectural stance on independent component versioning).

## §1. Scope and relationship to ROADMAP

This document is the rulebook for how the Evidence Suite versions. `ROADMAP.md` is the live status — which components are stable today, which are experimental, what versions compose into a coherent suite release. The two compose: ROADMAP names what is true now; VERSIONING explains the rules that determine what is true.

The architectural stance — independent component versions with a meta-repository compatibility matrix — is established in `REFERENCE-ARCHITECTURE.md` §10. This document operationalizes that stance: the SemVer rules each component follows, the breaking-change definitions per layer, the deprecation window, the receipt schema versioning mechanism, and the stability markers for fields under development.

The suite is pre-1.0. Pre-1.0 release lines have no LTS designation; LTS framing applies once v1.0 ships and external implementers depend on the spec at scale.

## §2. Per-component SemVer interpretation

Each component repository in the suite versions independently per [SemVer](https://semver.org/). The interpretation of MAJOR / MINOR / PATCH varies by lane:

**Spec layer (`agentic-receipts`).** Strictest interpretation. MAJOR = breaking change to receipt schema, canonicalization, hashing scheme, signing scheme, or bundle layout. MINOR = additive change (new optional field, new schema version that older verifiers can ignore safely). PATCH = clarifications, typo fixes, threat-model updates that do not change schema shape, and conformance-vector additions that test existing behavior rather than declare new behavior.

**Runtime layers (`agentic-policy-engine`, `agentic-eval-harness`, `agentic-artifacts`).** Standard interpretation. MAJOR = breaking change to the public API or to receipt fields the component emits. MINOR = additive (a new policy-rule slot, a new scenario class, a new manifest field). PATCH = bug fix or non-functional change.

**Tooling (`agentic-trace-cli`, `agentic-evidence-viewer`).** Most permissive interpretation through v0.1.x. MAJOR = bundle-format breaking change (inherited from the spec layer). MINOR = changed CLI argument shape, changed UI surface, removed or renamed command. PATCH = bug fix, performance work, internal refactor.

The asymmetry is deliberate: tooling surfaces evolve faster than the spec layer they implement. A reader pinning the suite to a specific compatibility window pins on `agentic-receipts` first; the tooling components carry wider compatible ranges. After v1.0, all six components converge on the standard interpretation — tooling surfaces stabilize, and minor bumps no longer rev CLI argument shapes or UI structure.

## §3. Suite version and compatibility matrix

The suite version (v0.1, v0.2, v1.0) is a meta-version that names a compatible set of component versions. The live matrix lives in `ROADMAP.md` §3; this section explains how it is constructed and updated.

A new suite minor release (e.g., v0.1 → v0.2) is declared when one or more components ship a breaking change that the suite as a whole adopts, OR when a substantive new capability is added that requires coordinated version bumps across components.

The compatibility matrix in ROADMAP names the version range of each component compatible with each suite line. A parallel matrix in `INTEROP.md` §8 names the standards versions (OpenTelemetry, SLSA, W3C VCDM, MCP, CloudEvents) the suite has been tested against per release line. Together, the two matrices form the full version contract for adopters: `ROADMAP.md` §3 covers components, `INTEROP.md` §8 covers standards.

Adopters integrate against a suite version, not against individual components. The matrix is the contract: any combination of component versions within the named ranges composes into a verifiable bundle.

## §4. Receipt schema versioning

A receipt declares its schema version via an explicit `schema_version` field. The field appears at the chain root (in the bundle's top-level metadata) and on each individual receipt. The value is a SemVer string (e.g., `"0.1.0"`).

**Verifier behavior on version mismatch:**
- **Major-version mismatch** between verifier and bundle fails loudly. The verifier reports the mismatch and refuses to interpret fields whose semantics are not known to its schema.
- **Minor-version mismatch with the bundle ahead of the verifier** parses with known fields; unknown fields are preserved in pass-through but not interpreted. The verifier reports the version difference as a non-fatal notice.
- **Minor-version mismatch with the verifier ahead of the bundle** parses normally; the verifier interprets known fields and applies defaults for fields the bundle predates.
- **Patch-version mismatch** parses normally and is not reported.

The chain-root version applies to the whole bundle. Per-receipt versions allow a long-running chain to evolve over its lifetime — a session that began under one minor version and continues under the next remains valid as long as each receipt declares its own version. Verifiers walking the chain may encounter a version transition; the rules above apply per-receipt at that boundary.

## §5. Breaking changes and deprecation window

**What counts as breaking for the spec layer (`agentic-receipts/spec/`):**

- Renamed field — **breaking**.
- Removed field — **breaking**.
- Changed field type (e.g., string → integer) — **breaking**.
- Changed field semantics with the same name — **breaking** (the most common SemVer footgun: the field name is unchanged, the meaning is not, and adopters do not notice until verification fails on edge cases).
- Added required field — **breaking** (records produced without it cease to be valid).
- Tightened validation that rejects previously-valid input — **breaking** (the second most common footgun, often shipped as a "non-breaking patch" by projects without explicit policy; explicit here).
- Added optional field — non-breaking.
- Loosened validation that accepts previously-rejected input — non-breaking, but reported in release notes.

**What counts as breaking for runtime layers and tooling:** standard SemVer interpretation. Public API changes, removed CLI flags, removed UI affordances, and changes to receipt fields the component emits all count. Pre-1.0 tooling explicitly carries no stability through v0.1.x — see §6.

**Deprecation window.** When a breaking change ships, the previous behavior remains supported alongside the new behavior for **two minor releases OR 90 days, whichever is longer**. The "whichever is longer" framing is deliberate: projects with a rapid minor-release cadence need a time floor; projects with a slow cadence need a version floor.

During the window, the deprecated behavior emits a deprecation notice (in CLI output, in release notes, and as a non-fatal warning event in receipt-emitting code where applicable). After the window, the deprecated behavior is removed in the next minor or major release per the change's nature.

**EOL policy.** The previous minor receives security fixes for one further minor cycle after the next minor releases. v0.1.x continues to receive security fixes while v0.2.x is the current line; v0.1.x becomes EOL when v0.3.0 ships. Pre-1.0 release lines have no LTS designation; LTS framing is reserved for post-1.0 lines where external implementers depend on long-term stability guarantees.

## §6. Stability markers and experimental fields

Within a release line, individual fields or features may carry a stability marker that overrides the line's general stability stance.

**Marking experimental.** A field, message variant, scenario class, or CLI flag is marked experimental in its spec by including the keyword `experimental` in the field's description and, where applicable, by emitting it under a top-level `_experimental` bag distinct from the stable receipt body. The leading underscore signals to parsers that the bag is non-canonical metadata that can be ignored without affecting stable-body verification — a near-universal convention. Experimental markers carry **no stability guarantee** — the field may change in any release, including patch. Verifiers preserve experimental fields in pass-through but do not validate them.

**Removing the marker.** A field is promoted from experimental to stable in a MINOR release. The promotion includes a release-note entry and is reflected in the next `ROADMAP.md` update. Demotion (stable → experimental) is treated as a breaking change and follows the rules in §5.

**v0.1.x default split.** Receipt schemas, canonicalization rules, hashing scheme (SHA-256), signing scheme (Ed25519, single-signature), bundle file layout, and the cross-reference field shape between artifacts and receipts are **stable through v0.1.x** — no breaking changes without a 0.2.0 bump. Tooling surfaces (CLI argument shapes, viewer UI panels and navigation) explicitly carry **no stability through v0.1.x**; patch and minor releases may change them. Decision-receipt body fields beyond the required core, eval-harness scenario taxonomy edges, policy rule-format extensions, and artifact manifest body fields beyond the required core are **experimental in v0.1.x** and may change in any release.

This split mirrors `ROADMAP.md` §4–§5 in versioning-rule form: read ROADMAP for live status, read this section for the rules that govern that status.
