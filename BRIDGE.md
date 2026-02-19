# EQC Spec–Implementation Bridge (EQC-SIB) v1.1

**EQC Spec–Implementation Bridge (EQC-SIB)** is the master governance layer for **code and pseudocode artifacts** that are tied to an EQC document portfolio governed by **EQC Ecosystem Specification (EQC-ES)**. It is explicitly **bidirectional** with EQC-ES:

* **Code/Pseudocode → Documents (via EQC-ES):** Any functional change in code or pseudocode produces an impact set of EQC documents that MUST be updated and revalidated under EQC-ES (unless a valid waiver applies).
* **Documents → Code/Pseudocode:** Any change in EQC documents that alters functional meaning produces an impact set of code/pseudocode artifacts that MUST be updated, rebuilt, and revalidated under EQC-SIB (unless a valid waiver applies).

EQC-SIB does **not** replace EQC-ES. EQC-ES remains the governance layer for documents. EQC-SIB governs implementation artifacts and enforces synchronization and determinism across the **document ↔ pseudocode ↔ code** triangle.

---

## 0.0 Identity & Purpose (mandatory)

* **Portfolio Name:** `[e.g., MyMetaheuristicSuite]`
* **Purpose (1 sentence):** Single source of truth that makes every implementation artifact discoverable, deterministically hashed, and provably consistent with the governing EQC documents under bidirectional propagation.
* **Spec Version:** **EQC-SIB-v1.1 | 2026-02-19 | [YourName / Team]**
* **Paired Document Governance:** **EQC-ES-v1.9.1** is mandatory and authoritative for document governance.
* **Governance Mode:** `paired-central` (one EQC-SIB paired to one EQC-ES root) or `paired-distributed` (sub-portfolios; see §9.2).

---

## 0.1 Core Principle: Bidirectional Binding Contract (mandatory)

EQC-SIB defines a binding contract:

1. **Every functional implementation artifact MUST bind to one or more EQC documents** that define its semantics, operators, invariants, or trace schema.
2. **Every EQC document that defines functional semantics MUST declare its implementation bindings** (either pseudocode, code, or both).
3. **Impact analysis is symmetric:**

   * If the implementation changes (functional), the governing documents are impacted.
   * If the governing documents change (functional), the implementation is impacted.

“Functional” is determined by **Digest Classes** (§6) and **Equivalence / Drift Rules** (§8), not by filename, formatting, or intent.

---

## 0.2 Normative Definitions (mandatory)

EQC-SIB uses four change classifications:

* **METADATA:** MD changes only (FD and FSD unchanged).
* **STATE:** FSD changes while FD is unchanged (e.g., dependency pins, toolchain lock, environment profile lock changes) and the change can affect observed behavior; revalidation is required.
* **FUNCTIONAL:** FD changes OR required evidence indicates semantic drift beyond tolerances OR public surface/operator contract changes.
* **VALIDATION:** Changes restricted to validation assets or traces (golden/shadow) without changing governed semantics; may still require reruns and drift reporting.

For **HARD** bindings, **evidence** (traces + contracts + surfaces) participates in classification (§5.2). FD is necessary but not sufficient.

---

# 1. Artifact Registry (mandatory)

Machine-readable inventory stored as `sib-registry.yaml`.

### 1.0 Required Fields (mandatory)

Each registry entry MUST contain:

* **ArtID** (unique short identifier)
* **Title**
* **Type:** `pseudocode | reference-impl | production-impl | operator-impl | test-harness | golden-trace-runner | benchmark | schema | generator | derived-artifact | other`
* **Language:** `eqc-procedure | mas-eqc | python | typescript | c++ | go | rust | other`
* **Layer** (integer ≥ 0; see §1.5)
* **Repo / Workspace Root** (logical root)
* **File Path / Git Ref** (one or both; see §1.1)
* **Current Version** (`vX.Y.Z` semantic versioning)
* **Status:** `active | deprecated | frozen | experimental | migrating`
* **Owner** (optional)
* **Canonicalization Tier** (see §6.1): `T0|T1|T2|T3`
* **Canonicalizer ID** (`<tool>@<version>`)
* **Canonicalizer Digest** (SHA-256 of the canonicalizer artifact, container image digest, or binary digest)
* **FunctionalDigest SHA-256 (FD)** (local semantics; see §6.2)
* **FunctionalStateDigest SHA-256 (FSD)** (FD + pins + surfaces + contracts; see §6.2)
* **MetadataDigest SHA-256 (MD)**
* **PinSet Digest SHA-256 (PSD)** (digest of declared dependency pin set; see §6.2)
* **Public Surface Digest SHA-256 (SD)** (digest of declared PROVIDES surface; see §6.3)
* **Build Profile Binding** (maps to EQC-ES `environment_profiles`)
* **Spec Bindings** (list of DocID + binding type + anchors; see §2)
* **Trace Bindings** (what traces validate it; see §4)
* **Tooling Manifest reference** (see §7)

### 1.1 Strict Path Resolution for Implementation (mandatory)

`eqc-sib validate` MUST resolve each registry entry to a concrete file in a defined workspace:

* If **File Path** is provided: resolve relative to the implementation workspace root; reject symlinks that escape the workspace root.
* If **Git Ref** is provided: materialize `(repo, ref, path)` into a deterministic validation workspace.
* **Non-empty means:** file length > 0 bytes after canonicalization (§6.1).

### 1.2 Version Marker Rule (mandatory)

Each artifact MUST contain a type-appropriate version marker:

* **Code files:** a header line containing
  `EQC-SIB ArtID: <ArtID> | Version: <vX.Y.Z>`
* **Pseudocode files:** a header containing
  `Spec Version:` or `Art Version:` with the exact version string.
* **Generated / derived artifacts:** must contain
  `Derived-From: <SourceArtID>@<SourceVersion>`

Validation FAILS if the marker is missing.

### 1.3 Binary / Non-text Artifacts (mandatory)

Artifacts that cannot be safely canonicalized as UTF-8 text MUST declare:

* `Canonicalization Tier: T0`
* FD computed over raw bytes of the materialized file (post workspace materialization), without whitespace stripping.

If an artifact is produced by a build step (compiled binary), it MUST be marked `derived-artifact` and MUST have `Derived-From` markers in accompanying metadata (or sidecar) binding it to source ArtIDs.

### 1.5 Layered Architecture (anti-cycle rule)

Layers define allowed dependency directions within **implementation artifacts**:

* Layer 0: schemas, core runtime contracts, base utilities
* Layer 1: operator implementations, deterministic helpers
* Layer 2: algorithm implementations, pseudocode procedure bindings
* Layer 3: test-harness, golden-trace-runner, benchmarks
* Layer 4+: derived artifacts, generated code, integrations

**Rule:** An artifact may only depend on artifacts of layer ≤ its own layer.

**Metadata exception:** “RECOGNIZES” metadata edges may point upward (never affects FD or FSD; see §3).

---

# 2. Document ↔ Implementation Binding Map (mandatory)

EQC-SIB introduces the binding sidecar: `sib-bindings.yaml`.

This is the **bidirectional bridge** between EQC-ES and implementation artifacts.

## 2.1 Binding Types

Each binding declares one or more of:

* **GOVERNS:** Doc defines functional semantics; implementation must conform.
* **IMPLEMENTS:** Artifact implements a Doc’s procedure/operators.
* **DERIVES-FROM:** Artifact generated from pseudocode/spec.
* **VALIDATES:** Artifact validates the Doc via traces/tests.
* **REFERENCES:** Non-binding citation.

Only **GOVERNS** and **IMPLEMENTS** are bidirectionally binding by default.

## 2.2 Required Binding Structure

```yaml
bindings:
  - doc: "ALGO-001"
    governs:
      - art: "IMPL-ALGO-001-PY"
        anchors:
          - doc_anchor_id: "DOC-ALGO-001-BVI-MAIN"
            art_anchor_id: "IMPL-ALGO-001-TRAIN_LOOP"
            binding_strength: "HARD"    # HARD|SOFT
            minimum_equivalence: "E1"   # E0–E3
      - art: "PSEUDO-ALGO-001"
        anchors:
          - doc_anchor_id: "DOC-ALGO-001-BVI-MAIN"
            art_anchor_id: "PSEUDO-ALGO-001-PROC-MAIN"
            binding_strength: "HARD"
            minimum_equivalence: "E0"
    validates:
      - art: "TEST-ALGO-001"
        traces:
          - "GOLDEN-TRACE-ALGO-001"
```

## 2.3 Anchor Semantics (mandatory)

Anchors MUST be stable identifiers that survive refactors.

* **Doc Anchor:** MUST be an explicit `AnchorID:` tag in the doc for HARD bindings.
* **Artifact Anchor:** MUST be an explicit `EQC_ANCHOR` tag or stable symbol ID for HARD bindings.

### 2.3.1 HARD Anchor Requirements (mandatory)

For HARD bindings:

* `doc_anchor_id` MUST reference an explicit `AnchorID:` in the EQC document.
* `art_anchor_id` MUST reference an explicit code tag, e.g.
  `# EQC_ANCHOR: IMPL-ALGO-001-TRAIN_LOOP`
  or a tool-defined stable symbol identifier that is guaranteed stable under refactors.

Anchors are part of **MetadataDigest**. However, breaking a required anchor resolution is a **blocking validation error**.

---

# 3. Implementation Dependency Graph (mandatory)

Stored in `sib-graph.yaml` with explicit edge types:

* **IMPORTS** (code-level dependency)
* **USES** (operator-level use)
* **PROVIDES** (exported stable API/operator surface)
* **DERIVES** (generated targets)
* **VALIDATES** (test/trace runner validates target)
* **REFERENCES** (non-binding)
* **RECOGNIZES** (metadata-only; excluded from cycle detection)
* **OBSERVED_IMPORTS** (observed at runtime during validation; see §3.1)
* **OBSERVED_FS_READS** (observed file reads during validation; see §3.1)

Cycle detection excludes RECOGNIZES, but **includes** all other edges.

## 3.1 Declared vs Observed Dependency Enforcement (mandatory)

For each validation run, tooling MUST produce:

* `declared_deps(Art)` from static extraction (imports/manifests/graph)
* `observed_deps(Art)` from runtime observation (imports + file reads)

Rules:

1. `observed_deps ⊆ declared_deps` MUST hold for HARD-governed artifacts (else FAIL).
2. Layer rule (§1.5) MUST be enforced on both `declared_deps` and `observed_deps`.
3. Any delta MUST be reported in JSON output (§7.2) deterministically sorted (§7.4).

---

# 4. Trace & Validation Binding (mandatory)

EQC-SIB requires that implementation artifacts are validated through traces consistent with EQC-ES.

## 4.1 Trace Artifacts

Implementation validation uses one or more of:

* **Golden trace sets** (shared with EQC-ES)
* **Shadow-traces per environment profile** (must match EQC-ES `environment_profiles`)
* **Equivalence diffs** (E0–E3, plus profile drift rules)

## 4.2 Clean-Room + Deterministic Sandbox Contract (mandatory)

Shadow-traces MUST run in a clean-room workspace **and** a deterministic sandbox:

### Clean-room requirements

* fresh workspace directory
* only declared inputs mounted read-only
* outputs written under `./run_artifacts/<run_id>/` only
* cache dirs cleared and cache-disabling env vars applied per profile
* start-state must match the paired portfolio checkpoint (§9)

### Deterministic sandbox requirements

Each environment profile MUST reference a sandbox policy that enforces at minimum:

* **Network:** disabled by default; allowlist only if explicitly declared and content-addressed
* **Time:** fixed or recorded (time source deterministic for replay)
* **Filesystem:** deny-by-default outside declared mounts
* **Environment:** allowlist env vars; all others cleared
* **Locale/TZ:** pinned
* **Threading:** pinned (OMP/MKL/BLAS/etc.)
* **GPU determinism:** required mode or explicit waiver; nondeterministic kernels must be disallowed unless tolerated by equivalence scope

If any sandbox policy requirement is missing, validation FAILS for HARD-governed artifacts.

---

# 5. Bidirectional Change Propagation Protocol (mandatory)

EQC-SIB defines a symmetric protocol. A change event is classified, then impact is computed across **both** registries and both graphs:

* implementation side: `sib-registry.yaml` + `sib-graph.yaml`
* document side: EQC-ES `ecosystem-registry.yaml` + `ecosystem-graph.yaml`
* bridge: `sib-bindings.yaml`

## 5.1 Change Types and Required Actions (mandatory)

| Change Source       | Change Type                                                                     | Classified As | Affected                                | Required Actions                                                                                       | Version Impact (most severe wins)                  |
| ------------------- | ------------------------------------------------------------------------------- | ------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------- |
| Code / Pseudocode   | formatting / comments / non-functional refactor (FD stable; FSD stable)         | METADATA      | bound docs (anchor check) + dependents  | update anchors if needed; rerun validate                                                               | PATCH                                              |
| Code / Pseudocode   | dependency pins / toolchain lock / profile lock change (FD stable; FSD changes) | STATE         | validators + dependents + traces        | rerun validations and traces; doc edits only if drift beyond tolerances                                | PATCH/MINOR (if it changes required compat bounds) |
| Code / Pseudocode   | functional behavior change (FD changes)                                         | FUNCTIONAL    | bound docs + tests + traces + consumers | update docs where semantics changed; update compatibility; rerun golden/shadow traces; EQC-ES validate | MINOR (or MAJOR if breaking surface/contract)      |
| Document            | editorial only (no EQC-ES FD change)                                            | METADATA      | bound artifacts                         | anchor checks only; no functional rebuild required                                                     | PATCH                                              |
| Document            | functional semantic change (EQC-ES FD change)                                   | FUNCTIONAL    | bound pseudocode + code artifacts       | update pseudocode/code; rerun traces; update registry digests                                          | MINOR/MAJOR                                        |
| Operator spec (Doc) | operator contract change                                                        | FUNCTIONAL    | operator impls + consumers + tests      | update operator code; update dependent algorithms; run targeted traces                                 | MAJOR if breaking                                  |
| Test/Trace assets   | golden trace update                                                             | VALIDATION    | validators + governed docs              | rerun validations; update drift report if within tolerance; update waivers if used                     | PATCH unless breaking                              |

## 5.2 HARD Binding Classification Triggers (mandatory)

For artifacts governed via **HARD** bindings, a change MUST be classified as FUNCTIONAL if **any** is true:

1. **FD changes**, or
2. Any bound **trace diff** exceeds effective equivalence tolerances (§8.3), or
3. Any declared **PROVIDES surface** changes (§6.3), or
4. Any declared **operator contract signature** changes (as defined by the governing doc/operator manifest), or
5. Any observed-deps rule is violated (§3.1) (this is a blocking error; functional classification applies to impact computation).

---

# 6. Canonical Hashing & Digest Classes (mandatory)

EQC-SIB mirrors EQC-ES digest separation and adds an explicit **state** layer to prevent false “functional churn” while still capturing dependency/toolchain reality.

## 6.1 Canonicalization Tiers (mandatory)

Each artifact MUST declare one of these tiers:

* **T0 (Byte Canonical):** FD hashes raw bytes of the materialized file (no whitespace stripping).
* **T1 (Token Canonical):** tokenize; normalize whitespace; drop comments/formatting per language adapter rules.
* **T2 (AST Canonical):** parse → canonical AST → stable re-emit used for hashing.
* **T3 (Semantic Canonical):** optional; only permitted if the canonicalizer provides machine-checkable semantic equivalence guarantees.

Tier selection, canonicalizer identity, and canonicalizer digest MUST be recorded in the registry (§1.0) and included in checkpoint hashing (§9).

## 6.2 Digest Definitions (mandatory)

* **FunctionalDigest FD(Art):** MUST change iff the artifact’s **local semantics** change under its declared canonicalization tier. FD MUST NOT include transitive dependency closure.

* **PinSet Digest PSD(Art):** digest of the artifact’s declared dependency pins (ArtID + version range or exact + required FD/FSD references, as defined in Tooling Manifest).

* **Public Surface Digest SD(Art):** digest of the declared **PROVIDES** surface list (§6.3).

* **FunctionalStateDigest FSD(Art):** MUST change iff **the artifact’s effective execution state commitments** change. Defined as:

  `FSD(Art) = H( FD(Art) || PSD(Art) || SD(Art) || ToolchainLockDigest(profile) || ProfileLockDigest(profile) )`

* **MetadataDigest MD(Art):** may change for non-functional edits (comments, formatting not captured by tier rules, anchors, owners, references). MD MUST include anchors and REFERENCES/RECOGNIZES edges.

## 6.3 PROVIDES Surface (mandatory)

Any artifact that exposes a stable interface MUST declare a **PROVIDES surface list** that is:

* enumerated (symbols/operators/APIs), not prose
* deterministically ordered
* referenced by `sib-graph.yaml` as PROVIDES edges
* hashed as `SD(Art)`

A breaking change to PROVIDES surface is **MAJOR** unless governed tolerances explicitly permit it.

## 6.4 Toolchain Lock (mandatory)

Each environment profile used for validation MUST have a **Toolchain Lock Digest** that covers, at minimum:

* compiler/interpreter versions
* build tool versions
* canonicalizer version + digest
* dependency lockfiles (or SBOM digest)
* container image digests (if used)

Toolchain lock inputs MUST be stored in a dedicated sidecar (§7).

---

# 7. Tooling (mandatory)

EQC-SIB requires tooling that can compute impact and enforce symmetry.

## 7.0 Required Sidecars at Implementation Root (mandatory)

* `sib-registry.yaml`
* `sib-graph.yaml`
* `sib-bindings.yaml`
* `sib-validation-log.md`
* `sib-compatibility-aggregate.yaml`
* `sib-release-notes-template.md`
* `sib-waivers.yaml` (see Appendix A)
* `sib-toolchain-lock.yaml` (see Appendix B)
* `sib-error-codes.yaml` (error code enum; see Appendix C)

## 7.1 Mandatory Commands (formal schema)

* `eqc-sib validate`
* `eqc-sib impact --change ARTID@vX.Y.Z`
* `eqc-sib sync --from code|pseudocode|docs`
* `eqc-sib regenerate-sidecars`
* `eqc-sib generate-migration-plan`

## 7.2 JSON Output Schemas (mandatory)

### 7.2.1 `eqc-sib validate` output (mandatory)

```json
{
  "eqc_sib_version": "v1.1",
  "workspace_ref": "<string>",
  "status": "PASS|FAIL",
  "errors": [{"code":"...", "art":"ARTID", "message":"..."}],
  "warnings": [{"code":"...", "art":"ARTID", "message":"..."}],
  "resolved_sib_graph_hash": "<sha256>",
  "toolchain_lock_hash": "<sha256>",
  "profile_lock_hashes": [{"profile":"...", "hash":"<sha256>"}],
  "broken_bindings": [{"doc":"DOCID","art":"ARTID","doc_anchor_id":"...","art_anchor_id":"..."}],
  "waivers_used": [{"waiver_id":"...", "scope":"...", "expires":"..."}],
  "observed_deps_delta": [{"art":"ARTID","missing_declared":["..."],"unexpected_observed":["..."]}],
  "affected_docs_via_bindings": ["DOCID","..."]
}
```

### 7.2.2 `eqc-sib impact` output (mandatory)

```json
{
  "change": "ARTID@vX.Y.Z",
  "classification": "METADATA|STATE|FUNCTIONAL|VALIDATION",
  "impacts": [
    {
      "art": "ARTID",
      "required_actions": ["..."],
      "version_impact": "PATCH|MINOR|MAJOR"
    }
  ],
  "affected_docs": [
    {
      "doc": "DOCID",
      "required_actions": ["Update semantics / anchors / compatibility / traces"],
      "eqc_es_version_impact": "PATCH|MINOR|MAJOR"
    }
  ]
}
```

## 7.3 Error Code Taxonomy (mandatory)

All `errors[].code` and `warnings[].code` MUST come from `sib-error-codes.yaml`. Unknown codes are invalid output.

## 7.4 Deterministic Ordering Rules (mandatory)

All tooling output MUST be deterministic:

* lists sorted lexicographically by stable keys:

  * errors/warnings: `(severity, code, art, doc_anchor_id, art_anchor_id)`
  * impacts: `(art, version_impact)`
  * affected_docs: `(doc)`
  * observed deltas: `(art)`
* YAML/JSON emission MUST be canonical (sorted keys, stable float formatting if present)

---

# 8. Equivalence & Round-Trip Consistency Rules (mandatory)

EQC-SIB uses EQC E0–E3 equivalence semantics as the shared language of drift control, and adds a formal waiver system.

## 8.1 Round-Trip Rule (mandatory)

If a binding is **HARD**, then:

* **Doc functional change ⇒ implementation MUST update** such that either:

  * FD changes appropriately, or
  * evidence proves conformance within declared tolerances and a valid waiver exists.
* **Implementation FUNCTIONAL change ⇒ doc MUST update** such that either:

  * EQC-ES FD changes appropriately, or
  * evidence proves equivalence within declared tolerances and a valid waiver exists.

### 8.1.1 Waivers (mandatory)

Waivers are permitted only when:

* the change is proven within declared equivalence tolerances, and
* a matching `waiver_id` exists in both:

  * EQC-ES waiver store / validation log (per EQC-ES rules), and
  * `sib-waivers.yaml` (this spec; Appendix A),
* signatures validate against portfolio trust roots (defined by EQC-ES + SIB trust config),
* waiver is not expired.

Logs may reference waivers, but waivers are the enforceable governance object.

## 8.2 Binding Strength

* **HARD:** required bidirectional updates; missing update is a blocking error.
* **SOFT:** impact is reported as warning; used only for prototypes/experimental artifacts.

## 8.3 Strictest Tolerance Inheritance (paired rule)

For a governed artifact, effective tolerance is derived from paired documents:

* If the governing doc declares strict tolerance scope for that artifact/provider, that strict value applies.
* Otherwise local scope applies.

---

# 9. Paired Portfolio Checkpoint (mandatory)

EQC-SIB extends the EQC-ES portfolio checkpoint with implementation state.

A paired checkpoint snapshot includes:

* EQC-ES: registry, graph, environment profiles, data registry, document FD/MD, resolved graph hash
* EQC-SIB: registry, graph, bindings map, artifact FD/FSD/MD, resolved SIB graph hash
* tool manifests and validation logs
* waivers + toolchain lock
* a combined **Atomic Paired Checkpoint Hash**

## 9.1 Atomic Paired Checkpoint Hash (mandatory)

Atomic Paired Checkpoint Hash MUST include:

* hash of EQC-ES files
* hash of EQC-SIB files
* resolved graph hashes for both sides
* bindings map hash
* **toolchain_lock_hash**
* **dependency lock hashes / SBOM digests**
* canonicalizer digests / container image digests (if applicable)
* waiver store hash (both sides)

Detached checkpoints are invalid.

## 9.2 Paired-Distributed Mode: Portfolio Federation Contract (mandatory)

In `paired-distributed` mode:

* Each sub-portfolio MUST have its own paired checkpoint (EQC-ES + EQC-SIB) with an atomic paired hash.
* A parent portfolio MUST reference sub-portfolios via **imported checkpoint references**, not raw paths:

```yaml
imports:
  - portfolio_id: "SUBPORT-001"
    paired_checkpoint_hash: "<sha256>"
    compatibility_range: ">=1.2.0 <2.0.0"
```

Rules:

* Cross-portfolio bindings are valid only if the referenced sub-portfolio checkpoint is imported.
* Impact and validation MUST treat imported checkpoints as immutable inputs unless the import reference itself changes.

---

# 10. Governance & Lifecycle (mandatory)

Change process (paired):

1. branch → edit (docs or code/pseudocode)
2. run `eqc-sib impact` and `eqc-es impact` using the bindings map
3. apply required updates on both sides
4. run `eqc-sib validate` + `eqc-es validate`
5. run required golden/shadow traces
6. write release notes + migration notes if needed
7. atomic merge with paired checkpoint recorded

Deprecation rules align with EQC-ES:

* Deprecated artifacts remain readable for at least one MAJOR portfolio version + documented grace period.
* Frozen artifacts generate warnings only but MUST NOT be depended on by new ACTIVE artifacts via functional edges (IMPORTS/USES/DERIVES/VALIDATES/OBSERVED_*).

---

# 11. Glossary

* **Artifact (ArtID):** any code/pseudocode/test/schema item governed by EQC-SIB.
* **Bindings Map:** `sib-bindings.yaml`, the bidirectional bridge to EQC-ES.
* **HARD Binding:** bidirectional functional coupling with blocking enforcement.
* **FunctionalDigest (FD):** hash of local semantics under declared canonicalization tier.
* **FunctionalStateDigest (FSD):** hash of FD plus pins, surfaces, toolchain/profile locks.
* **MetadataDigest (MD):** hash for non-functional and reference/anchor metadata.
* **PinSet:** declared dependency pins (ArtIDs + version constraints + required digests).
* **PROVIDES Surface:** enumerated stable API/operator surface.
* **Paired Checkpoint:** atomic snapshot of both document and implementation portfolios.
* **Round-Trip Rule:** functional changes must propagate across doc ↔ implementation unless waived with valid evidence.

---

## Appendix A — Waiver Schema (mandatory)

Stored in `sib-waivers.yaml`.

```yaml
waivers:
  - waiver_id: "WVR-<content-addressed-id>"
    scope:
      doc: "DOCID"              # optional
      art: "ARTID"              # optional
      doc_anchor_id: "..."      # optional
      art_anchor_id: "..."      # optional
    reason_code: "EQUIV_WITHIN_TOLERANCE"  # ENUM controlled by policy
    equivalence_claim:
      minimum_equivalence: "E2"
      tolerances:
        eps: 1e-8
        drift_budget: "..."
    evidence:
      run_ids: ["RUN-..."]
      diff_hashes: ["<sha256>"]
      profile: "ENV-PROFILE-ID"
      resolved_graph_hash: "<sha256>"
      toolchain_lock_hash: "<sha256>"
    approvals:
      - signer_key_id: "KEY-..."
        signature: "<sig>"
        signed_payload_hash: "<sha256>"
        signed_at: "2026-02-19T00:00:00Z"
    expires: "2026-06-19T00:00:00Z"
```

Rules:

* Waivers MUST be content-addressed (waiver_id derived from canonical waiver payload).
* Waivers MUST expire.
* Validation MUST fail if a required waiver is missing or expired.

---

## Appendix B — Toolchain Lock Schema (mandatory)

Stored in `sib-toolchain-lock.yaml`.

```yaml
toolchain_lock:
  profiles:
    - profile: "ENV-PROFILE-ID"
      interpreter:
        id: "python"
        version: "3.11.8"
        digest: "<sha256 or container digest>"
      build_tools:
        - id: "uv"
          version: "..."
          digest: "..."
      canonicalizer:
        id: "<tool>@<version>"
        digest: "<sha256 or container digest>"
      dependency_locks:
        - kind: "pip-lock"
          path: "locks/requirements.lock"
          digest: "<sha256>"
      containers:
        - image: "ghcr.io/org/image:tag"
          digest: "sha256:..."
      sbom:
        path: "sbom/sbom.spdx.json"
        digest: "<sha256>"
```

---

## Appendix C — Error Code Enum Template (mandatory)

Stored in `sib-error-codes.yaml`.

```yaml
error_codes:
  - code: "SIB_PATH_ESCAPE"
    severity: "ERROR"
    description: "Artifact path resolves outside workspace root."
  - code: "SIB_MISSING_VERSION_MARKER"
    severity: "ERROR"
    description: "Required ArtID/version marker not found in artifact."
  - code: "SIB_BROKEN_HARD_BINDING_ANCHOR"
    severity: "ERROR"
    description: "HARD binding anchor could not be resolved."
  - code: "SIB_OBSERVED_DEP_UNDECLARED"
    severity: "ERROR"
    description: "Observed dependency not present in declared dependency set."
  - code: "SIB_LAYER_VIOLATION"
    severity: "ERROR"
    description: "Layer rule violated by declared or observed dependency."
  - code: "SIB_SANDBOX_POLICY_MISSING"
    severity: "ERROR"
    description: "Required deterministic sandbox policy controls not present."
  - code: "SIB_WAIVER_MISSING_OR_EXPIRED"
    severity: "ERROR"
    description: "Required waiver not found or expired."
```

---
