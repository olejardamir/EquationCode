**# EQC Spec–Implementation Bridge (EQC-SIB) v1.0**

**EQC Spec–Implementation Bridge (EQC-SIB)** is the master governance layer for **code and pseudocode artifacts** that are tied to an EQC document portfolio governed by **EQC Ecosystem Specification (EQC-ES)**. It is explicitly **bidirectional** with EQC-ES:

* **Code/Pseudocode → Documents (via EQC-ES):** Any functional change in code or pseudocode automatically produces an impact set of EQC documents that must be updated and revalidated under EQC-ES.
* **Documents → Code/Pseudocode:** Any change in EQC documents that alters functional meaning automatically produces an impact set of code/pseudocode artifacts that must be updated, rebuilt, and revalidated under EQC-SIB.

EQC-SIB does **not** replace EQC-ES. EQC-ES remains the governance layer for documents. EQC-SIB governs implementation artifacts and enforces synchronization and determinism across the **document ↔ pseudocode ↔ code** triangle.

---

## 0.0 Identity & Purpose (mandatory)

* **Portfolio Name:** [e.g., MyMetaheuristicSuite]
* **Purpose (1 sentence):** Single source of truth that makes every implementation artifact discoverable, deterministically hashed, and provably consistent with the governing EQC documents under bidirectional propagation.
* **Spec Version:** **EQC-SIB-v1.0 | 2026-02-19 | [YourName / Team]**
* **Paired Document Governance:** **EQC-ES-v1.9.1** is mandatory and authoritative for document governance.
* **Governance Mode:** paired-central (one EQC-SIB paired to one EQC-ES root) or paired-distributed (sub-portfolios; see §2.2).

---

## 0.1 Core Principle: Bidirectional Binding Contract (mandatory)

EQC-SIB defines a binding contract:

1. **Every functional implementation artifact MUST bind to one or more EQC documents** that define its semantics, operators, invariants, or trace schema.
2. **Every EQC document that defines functional semantics MUST declare its implementation bindings** (either pseudocode, code, or both).
3. **Impact analysis is symmetric:**

   * If the implementation changes (functional), the governing documents are impacted.
   * If the governing documents change (functional), the implementation is impacted.

“Functional” is defined by digest class (§6) and equivalence rules (§8).

---

## 1. Artifact Registry (mandatory)

Machine-readable inventory stored as `sib-registry.yaml`.

Required fields for every entry:

* **ArtID** (unique short identifier)
* **Title**
* **Type:** pseudocode | reference-impl | production-impl | operator-impl | test-harness | golden-trace-runner | benchmark | schema | generator | derived-artifact | other
* **Language:** eqc-procedure | mas-eqc | python | typescript | c++ | go | rust | other
* **Layer** (integer ≥ 0; see §1.4)
* **Repo / Workspace Root** (logical root)
* **File Path / Git Ref**
* **Current Version** (vX.Y.Z semantic versioning)
* **Status:** active | deprecated | frozen | experimental | migrating
* **Owner** (optional)
* **FunctionalDigest SHA-256 (FD)**
* **MetadataDigest SHA-256 (MD)**
* **Build Profile Binding** (maps to EQC-ES environment_profiles)
* **Spec Bindings** (list of DocID + binding type + anchors; see §2)
* **Trace Bindings** (what traces validate it; see §4)
* **Tooling Manifest reference** (see §7)

### 1.1 Strict Path Resolution for Implementation (mandatory)

`eqc-sib validate` MUST resolve each registry entry to a concrete file in a defined workspace:

* If File Path is provided: resolve relative to the implementation workspace root; reject symlinks that escape the workspace root.
* If Git Ref is provided: materialize (repo, ref, path) into a deterministic validation workspace.

Non-empty means: file length > 0 bytes after canonicalization (§6.1).

### 1.2 Version Marker Rule (mandatory)

Each artifact MUST contain a type-appropriate version marker:

* **Code files:** a header line containing `EQC-SIB ArtID: <ArtID> | Version: <vX.Y.Z>`
* **Pseudocode files:** a header containing `Spec Version:` or `Art Version:` with the exact version string.
* **Generated / derived artifacts:** must contain `Derived-From: <SourceArtID>@<SourceVersion>`.

Validation fails if marker is missing.

---

## 1.4 Layered Architecture (anti-cycle rule)

Layers define allowed dependency directions within **implementation artifacts**:

* Layer 0: schemas, core runtime contracts, base utilities
* Layer 1: operator implementations, deterministic helpers
* Layer 2: algorithm implementations, pseudocode procedure bindings
* Layer 3: test-harness, golden-trace-runner, benchmarks
* Layer 4+: derived artifacts, generated code, integrations

**Rule:** An artifact may only depend on artifacts of layer ≤ its own layer.
**Metadata exception:** “RECOGNIZES” style metadata edges may point upward (never affects FunctionalDigest).

---

## 2. Document ↔ Implementation Binding Map (mandatory)

EQC-SIB introduces the binding sidecar: `sib-bindings.yaml`.

This is the **bidirectional bridge** between EQC-ES and implementation artifacts.

### 2.1 Binding Types

Each binding declares:

* **GOVERNS:** Doc defines functional semantics; implementation must conform.
* **IMPLEMENTS:** Artifact implements a Doc’s procedure/operators.
* **DERIVES-FROM:** Artifact generated from pseudocode/spec.
* **VALIDATES:** Artifact validates the Doc via traces/tests.
* **REFERENCES:** Non-binding citation.

Only **GOVERNS** and **IMPLEMENTS** are bidirectionally binding by default.

### 2.2 Required Binding Structure

```yaml
bindings:
  - doc: "ALGO-001"
    governs:
      - art: "IMPL-ALGO-001-PY"
        anchors:
          - doc_anchor: "Block VI"
            art_anchor: "src/algo/train.py::train_loop"
            binding_strength: "HARD"   # HARD|SOFT
            minimum_equivalence: "E1"  # uses EQC E0–E3 semantics
      - art: "PSEUDO-ALGO-001"
        anchors:
          - doc_anchor: "Block VI"
            art_anchor: "procedure::Main"
            binding_strength: "HARD"
            minimum_equivalence: "E0"
    validates:
      - art: "TEST-ALGO-001"
        traces:
          - "GOLDEN-TRACE-ALGO-001"
```

### 2.3 Anchor Semantics (mandatory)

Anchors MUST be stable identifiers that survive refactors:

* **Doc Anchor:** section identifier (e.g., `Block VI`, `§6.X`, or explicit `AnchorID:` tags in the doc).
* **Artifact Anchor:** symbol-level reference (`path::symbol`, function name, class name, or explicit `# EQC_ANCHOR:<id>` tag in code).

Anchors are part of **MetadataDigest**, but changes to anchors can become functional if they break bindings (validation error).

---

## 3. Implementation Dependency Graph (mandatory)

Stored in `sib-graph.yaml` with explicit edge types:

* **IMPORTS** (code-level dependency)
* **USES** (operator-level use)
* **PROVIDES** (exported stable API/operator set)
* **DERIVES** (generated targets)
* **VALIDATES** (test/trace runner validates target)
* **REFERENCES** (non-binding)
* **RECOGNIZES** (metadata-only; excluded from cycle detection)

Cycle detection excludes RECOGNIZES, but all other edges are included.

---

## 4. Trace & Validation Binding (mandatory)

EQC-SIB requires that implementation artifacts are validated through traces consistent with EQC-ES.

### 4.1 Trace Artifacts

Implementation validation uses one or more of:

* **Golden trace sets** (shared with EQC-ES)
* **Shadow-traces per environment profile** (must match EQC-ES environment_profiles)
* **Equivalence diffs** (E0–E3, plus profile drift rules)

### 4.2 Clean-Room Contract (mandatory)

Shadow-traces MUST run in a clean-room workspace:

* fresh workspace directory
* only declared inputs mounted read-only
* outputs written under `./run_artifacts/<run_id>/` only
* cache dirs cleared and cache-disabling env vars applied per profile
* start-state must match the paired portfolio checkpoint (§9)

---

## 5. Bidirectional Change Propagation Protocol (mandatory)

EQC-SIB defines a symmetric protocol. A change event is classified, then impact is computed across **both** registries and both graphs:

* implementation side: `sib-registry.yaml` + `sib-graph.yaml`
* document side: EQC-ES `ecosystem-registry.yaml` + `ecosystem-graph.yaml`
* bridge: `sib-bindings.yaml`

### 5.1 Change Types and Required Actions

| Change Source       | Change Type                                         | Classified As | Affected                                      | Required Actions                                                                                       | Version Impact (most severe wins)         |
| ------------------- | --------------------------------------------------- | ------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Code                | refactor only (no FD change)                        | METADATA      | bound docs (anchor check) + dependents        | update anchors if needed; rerun validate; no doc semantic edits                                        | PATCH                                     |
| Code                | functional behavior change (FD change)              | FUNCTIONAL    | bound docs (GOVERNS targets) + tests + traces | update docs where semantics changed; update compatibility; rerun golden/shadow traces; EQC-ES validate | MINOR (or MAJOR if breaking API/operator) |
| Pseudocode          | procedure semantics change                          | FUNCTIONAL    | bound docs + code that DERIVES/IMPLEMENTS it  | update docs + code; rerun traces                                                                       | MINOR/MAJOR                               |
| Document            | editorial only (no FD change under EQC-ES)          | METADATA      | bound artifacts                               | anchor checks only; no functional rebuild required                                                     | PATCH                                     |
| Document            | functional semantic change (FD change under EQC-ES) | FUNCTIONAL    | bound pseudocode + code artifacts             | update pseudocode/code; rerun traces; update implementation registry digests                           | MINOR/MAJOR                               |
| Operator spec (Doc) | operator contract change                            | FUNCTIONAL    | operator impls + consumers + tests            | update operator code; update dependent algorithms; run targeted traces                                 | MAJOR if breaking                         |
| Test/Trace assets   | golden trace update                                 | VALIDATION    | validators + governed docs                    | rerun validations; update drift report if within tolerance                                             | PATCH unless breaking                     |

---

## 6. Canonical Hashing & Digest Classes (mandatory)

EQC-SIB mirrors EQC-ES digest separation.

### 6.1 Canonicalization (mandatory)

Before hashing:

* UTF-8, LF line endings, no BOM
* trailing whitespace stripped
* language-specific normalization allowed only if explicitly declared in Tooling Manifest
* YAML/JSON parsed and re-emitted canonical (sorted keys, stable floats)

### 6.2 Digest Definitions

* **FunctionalDigest FD(Art):** must change iff functional meaning changes.
* **MetadataDigest MD(Art):** may change for non-functional edits (comments, formatting, anchors, owners).

FD includes dependency closure for IMPORTS/USES edges and any declared exported PROVIDES API/operator lists (Tooling Manifest defines how).

MD includes REFERENCES/RECOGNIZES edges and anchors.

---

## 7. Tooling (mandatory)

EQC-SIB requires tooling that can compute impact and enforce symmetry.

Required sidecars at implementation root:

* `sib-registry.yaml`
* `sib-graph.yaml`
* `sib-bindings.yaml`
* `sib-validation-log.md`
* `sib-compatibility-aggregate.yaml`
* `sib-release-notes-template.md`

### 7.1 Mandatory Commands (formal schema)

* `eqc-sib validate`
* `eqc-sib impact --change ARTID@vX.Y.Z`
* `eqc-sib sync --from code|pseudocode|docs`
* `eqc-sib regenerate-sidecars`
* `eqc-sib generate-migration-plan`

### 7.2 JSON Output Schemas (mandatory)

`eqc-sib validate` MUST output:

```json
{
  "eqc_sib_version": "v1.0",
  "workspace_ref": "<string>",
  "status": "PASS|FAIL",
  "errors": [{"code":"...", "art":"ARTID", "message":"..."}],
  "warnings": [{"code":"...", "art":"ARTID", "message":"..."}],
  "resolved_sib_graph_hash": "<sha256>",
  "broken_bindings": [{"doc":"DOCID","art":"ARTID","anchor":"..."}],
  "affected_docs_via_bindings": ["DOCID","..."]
}
```

`eqc-sib impact` MUST output:

```json
{
  "change": "ARTID@vX.Y.Z",
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

---

## 8. Equivalence & Round-Trip Consistency Rules (mandatory)

EQC-SIB uses EQC E0–E3 equivalence semantics as the shared language of drift control, and adds a round-trip rule set:

### 8.1 Round-Trip Rule (mandatory)

If a binding is **HARD**, then:

* **Doc functional change ⇒ implementation FD must change (or explicit waiver).**
* **Implementation FD change ⇒ doc must change (or explicit waiver).**

Waivers are permitted only when:

* the change is proven within declared equivalence tolerances, and
* a signed entry exists in both `ecosystem-validation-log.md` (EQC-ES) and `sib-validation-log.md` (EQC-SIB) referencing the same evidence.

### 8.2 Binding Strength

* **HARD:** required bidirectional updates; missing update is a blocking error.
* **SOFT:** impact is reported as warning; used only for prototypes/experimental artifacts.

### 8.3 Strictest Tolerance Inheritance (paired rule)

For a governed artifact, effective tolerance is derived from the paired documents:

* If the doc declares strict tolerance scope for that artifact/provider, that strict value applies.
* Otherwise local scope applies.

This prevents “one tiny EPS locks the world” while still allowing strict governance where required.

---

## 9. Paired Portfolio Checkpoint (mandatory)

EQC-SIB extends the EQC-ES portfolio checkpoint with implementation state:

A paired checkpoint snapshot includes:

* EQC-ES: registry, graph, environment profiles, data registry, document FD/MD, resolved graph hash
* EQC-SIB: registry, graph, bindings map, artifact FD/MD, resolved SIB graph hash
* tool manifests and validation logs
* a combined **Atomic Paired Checkpoint Hash**

**Atomic Paired Checkpoint Hash:** MUST include:

* hash of EQC-ES file
* hash of EQC-SIB file
* resolved graph hashes for both sides
* the bindings map hash

Detached checkpoints are invalid.

---

## 10. Governance & Lifecycle (mandatory)

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
* Frozen artifacts generate warnings only but must not be depended on by new ACTIVE artifacts via functional edges.

---

## 11. Glossary

* **Artifact (ArtID):** any code/pseudocode/test/schema item governed by EQC-SIB.
* **Bindings Map:** `sib-bindings.yaml`, the bidirectional bridge to EQC-ES.
* **HARD Binding:** bidirectional functional coupling with blocking enforcement.
* **FunctionalDigest (FD):** hash that must change iff functional meaning changes.
* **MetadataDigest (MD):** hash for non-functional changes.
* **Paired Checkpoint:** atomic snapshot of both document and implementation portfolios.
* **Round-Trip Rule:** functional changes must propagate across doc ↔ implementation unless waived with evidence.
