# EQC-SR — Symbolic Regression Profile for EquationCode

**Spec Version:** `EQC-SR-v1.0.3 | 2026-05-25 | Final Closure Patch`

**Status:** Final closure patch for governed symbolic-regression source extraction, EQC specification, Python backend implementation, validation, release claims, errata handling, post-1.0 change control, and v1.0-line closure discipline.

**Extends:**

- `EQC-v1.1`
- `EQC-ES-v1.9.1`
- `EQC-SIB-v1.2.1`

**Purpose:** EQC-SR defines a symbolic-regression-specific profile for translating, specifying, validating, and eventually reimplementing a symbolic-regression engine such as `SymbolicRegression.jl` in a language-independent EQC form, then implementing a Python-native backend with PySR-style API compatibility.

**Design Target:** This profile is intended to support a parallel development path: keep the Julia/PySR backend as the behavioral reference, extract subsystem semantics into EQC-SR, implement a Python-native backend from the EQC-SR documents, and validate the Python backend against golden problems and reference traces.

**Core v1.0 Principle:** EQC-SR specifications MUST be executable as a governed porting program whose semantics, evidence, implementation artifacts, validation traces, compatibility claims, and release packages are frozen, inspectable, reproducible, and change-controlled.

---

## 0. Scope and Non-Scope

### 0.1 In Scope

EQC-SR governs symbolic-regression systems that search over mathematical expressions represented as trees, graphs, or equivalent structured expression objects.

It covers:

- expression tree semantics;
- operator registry semantics;
- protected math behavior;
- dataset and batching semantics;
- candidate expression evaluation;
- loss, complexity, score, and ranking semantics;
- random expression generation;
- mutation and crossover;
- population evolution;
- migration between populations;
- Pareto frontier behavior;
- hall-of-fame behavior;
- constant initialization and optimization;
- constraints and nested constraints;
- simplification and canonicalization;
- custom operator registration;
- trace, replay, and checkpointing;
- performance rules for Python backends;
- compatibility tiers for PySR-like APIs.

### 0.2 Out of Scope

EQC-SR does not define:

- a complete symbolic-regression algorithm by itself;
- a mandatory search strategy;
- a mandatory Python package layout;
- a required dependency stack;
- a full clone of any existing implementation.

Instead, it defines the semantic contracts that a symbolic-regression EQC document must specify before it can be treated as reproducible, comparable, and portable.

---



## 0.3 Normative Language and Requirement Levels

EQC-SR uses the following requirement words:

- **MUST**: required for EQC-SR compliance.
- **MUST NOT**: prohibited for EQC-SR compliance.
- **SHOULD**: recommended default; deviation requires an explicit rationale.
- **MAY**: optional behavior that must still be declared if used.

Each concrete symbolic-regression specification MUST mark every requirement as one of:

```text
required
recommended
optional
not_applicable_with_reason
```

A concrete spec MUST NOT silently inherit ambiguous behavior from an implementation. If behavior is unknown during extraction from a source implementation, it MUST be marked as `unresolved_semantics` with an owner, evidence target, and resolution status.


## 0.3.1 Normative Conflict Resolution

If EQC-SR conflicts with base EQC, EQC-ES, or EQC-SIB, the governing rule is:

```text
EQC core semantics govern generic algorithm requirements.
EQC-SR governs symbolic-regression-specific semantics.
EQC-ES governs document portfolio registry, graph, digest, and propagation requirements.
EQC-SIB governs implementation, pseudocode, code, trace binding, and artifact conformance requirements.
```

A concrete portfolio MUST NOT resolve conflicts informally. It MUST record one of:

```text
no_conflict
sr_specialization_of_eqc
es_governance_takes_precedence
sib_artifact_governance_takes_precedence
explicit_waiver_with_evidence
```

Any conflict affecting behavior, trace keys, sidecar schema, public API, or equivalence testing is a blocker for `SR-RELEASE-READY` unless resolved through an explicit waiver and recorded decision.

## 0.3.2 Requirement Index Rule

Every release-relevant requirement MUST have a stable requirement ID.

Recommended ID form:

```text
SR-REQ-<section>-<short-name>-v<major>
```

Example:

```text
SR-REQ-EVAL-SHAPE-v1
SR-REQ-HOF-IMMUTABLE-v1
SR-REQ-OP-PROTECTED-DIV-v1
```

Each requirement ID MUST appear in exactly one normative source location. Cross-references MAY repeat the ID, but they MUST point to the canonical location.

A portfolio claiming `SR-PORT-READY` or higher MUST include a machine-readable requirement index:

```yaml
requirements:
  - req_id: "SR-REQ-EVAL-SHAPE-v1"
    title: "Evaluator output shape is deterministic"
    source_doc: "SR-EVAL-001"
    source_anchor: "8.4"
    level: "MUST"
    status: "implemented|validated|waived|not_applicable_with_reason"
    owner: "subsystem-or-person"
    evidence:
      tests: ["TEST-EVAL-SHAPE-001"]
      traces: ["TRACE-EVAL-001"]
      implementation_artifacts: ["IMPL-EVAL-PY"]
```

Missing requirement index entries are blockers for `SR-RELEASE-READY`.

## 0.3.3 Requirement-to-Test Minimum Rule

Each `MUST` requirement in release scope MUST bind to at least one validation artifact:

```text
static_lint
unit_test
property_test
metamorphic_test
golden_trace
benchmark
manual_review_record
```

A `manual_review_record` alone is insufficient for behavior that can be executed and tested. It is acceptable only for source extraction provenance, design decisions, and human judgment boundaries.

## 0.4 Primary Use Cases

EQC-SR supports four use cases:

1. **Reference extraction:** document the behavior of an existing symbolic-regression backend.
2. **Clean-room reimplementation:** implement a new backend from EQC-SR rather than from implementation intuition.
3. **Compatibility validation:** compare a new backend to a reference backend by traces, metrics, and golden problems.
4. **Custom math extension:** safely add domain-specific operators, protected functions, and export mappings.

The default recommended project path is:

```text
Reference backend remains working
  ↓
Subsystem behavior is extracted into EQC-SR
  ↓
Python backend implements EQC-SR
  ↓
Compatibility layer exposes PySR-like API
```

## 0.5 Compliance Modes

EQC-SR distinguishes four compliance modes so that early research work does not falsely claim release readiness.

| Mode | Name | Meaning | Minimum Evidence |
|---|---|---|---|
| `SR-DRAFT` | Draft Spec | Semantics are being captured; unresolved items allowed. | unresolved semantics list, owner, evidence target |
| `SR-SPEC-READY` | Specification Ready | A subsystem spec can guide implementation. | complete operator/state/procedure/trace definitions for that subsystem |
| `SR-PORT-READY` | Port Ready | A subsystem can be implemented in Python from the EQC-SR spec. | extraction crosswalk, tests, equivalence target, no blocker unresolved semantics |
| `SR-RELEASE-READY` | Release Ready | A backend or subsystem can be shipped as governed. | golden tests, trace validation, checkpoint/restore, SIB bindings, sidecars, performance profile |

A document MUST declare one compliance mode. A portfolio MUST NOT claim `SR-PORT-READY` or `SR-RELEASE-READY` if any blocker unresolved semantics remain in the governed scope.

## 0.6 Clean-Room Boundary for Reimplementation

A Python backend that claims to implement EQC-SR SHOULD be implemented from EQC-SR specifications and governed pseudocode, not by line-by-line mechanical copying of Julia source.

For a clean-room or semi-clean-room port, each subsystem MUST declare one of:

```text
REFERENCE_OBSERVED     behavior inferred from running the reference implementation
SOURCE_EXTRACTED       behavior extracted from source code and documented in EQC-SR
SPEC_DEFINED           behavior intentionally defined by EQC-SR where source behavior is not reused
DOCUMENTED_DEVIATION   behavior intentionally differs from source and has tests
UNRESOLVED             behavior not yet known; not release-ready
```

Every `DOCUMENTED_DEVIATION` MUST include rationale, expected effect, equivalence level, and validation evidence.

## 0.7 Profile Integrity and Document Boundary Rules

EQC-SR documents MUST keep three kinds of content separated:

```text
SPEC_SEMANTICS      normative behavior that implementations must follow
EXTRACTION_EVIDENCE facts observed from a reference implementation or source audit
IMPLEMENTATION_PLAN intended Python implementation choices
```

A single Markdown file MAY contain all three, but every section MUST be labeled with one of these content classes. Release-targeted documents SHOULD split them into separate files:

```text
sr-spec-*.eqc.md           SPEC_SEMANTICS
sr-extraction-*.md         EXTRACTION_EVIDENCE
sr-implementation-*.md     IMPLEMENTATION_PLAN
```

A Python backend MUST NOT claim conformance based only on implementation notes. Conformance requires binding to `SPEC_SEMANTICS`, validation against declared tests, and a recorded result in the conformance matrix.

### 0.7.1 Blocking Ambiguity Rule

Any behavior that affects outputs, traces, candidate validity, equation ranking, reproducibility, or public API results MUST be one of:

```text
specified
explicitly_deferred
not_applicable_with_reason
documented_deviation
```

A release-targeted scope MUST NOT contain `explicitly_deferred` behavior unless the behavior is unreachable under the declared compatibility level.

### 0.7.2 Document Size and Modularity Rule

A concrete EQC-SR algorithm spec SHOULD NOT become a single unbounded document. Once a section exceeds approximately one implementation subsystem, split it into a governed subdocument and register the dependency through EQC-ES.

Recommended split:

```text
SR-EXPR-*       expression model
SR-OPLIB-*      operator registry
SR-EVAL-*       evaluator and numeric policy
SR-RANK-*       loss/complexity/Pareto/HOF
SR-SEARCH-*     generation/mutation/crossover/evolution
SR-CONSTOPT-*   constants and optimization
SR-EXPORT-*     SymPy/NumPy/Python/PySR exports
SR-TRACE-*      trace and golden tests
```

## 0.8 Reference Snapshot Discipline

When EQC-SR is used to extract behavior from `SymbolicRegression.jl`, the reference implementation MUST be pinned as a snapshot. The snapshot record MUST include:

```text
reference_name
repository_url
commit_hash
package_version_if_any
submodule_or_dependency_versions_if_known
Julia_version
Python_frontend_version_if_used
extraction_date
extractor_identity_or_tool
scope_of_files_reviewed
scope_of_behavior_tested
known_unreviewed_files
```

A reference comparison is invalid if it compares against an unpinned branch name such as `main`, `master`, or `dev`.

### 0.8.1 Reference Snapshot Change Rule

Changing the reference snapshot is a functional event for extraction documents. The affected subsystem specs MUST be reclassified as one of:

```text
unchanged_after_review
changed_behavior
unknown_requires_review
source_only_change_no_behavior_evidence
```

If the new reference changes observed behavior, dependent EQC-SR specs MUST either update semantics or declare a documented deviation.

### 0.8.2 Source-to-Spec Coverage Status

Every source file included in extraction scope MUST have one coverage status:

```text
mapped_to_spec
mapped_to_nonfunctional_support
intentionally_ignored_with_reason
unresolved
```

A portfolio MUST NOT claim `SR-PORT-READY` for a subsystem if any source file required for that subsystem remains `unresolved`.



## 0.9 Requirement Identity and Evidence Rule

Every normative EQC-SR requirement that can affect implementation behavior, validation, trace output, API behavior, or release readiness MUST have a stable requirement identifier.

Recommended form:

```text
REQ-SR-<SUBSYSTEM>-<NUMBER>
```

Examples:

```text
REQ-SR-EXPR-001   expression canonical serialization is defined
REQ-SR-EVAL-004   non-finite prediction policy is deterministic
REQ-SR-ARCH-003   hall-of-fame duplicate behavior is deterministic
REQ-SR-COMPAT-007 unsupported PySR option fails with structured error
```

Each requirement MUST map to at least one evidence path before `SR-RELEASE-READY` is claimed:

```text
spec_section
pseudocode_artifact, if applicable
implementation_artifact, if applicable
test_artifact
trace_or_benchmark_artifact, if applicable
conformance_matrix_row
```

A release-targeted requirement without an evidence path is a BLOCKER finding, not a documentation gap.

### 0.9.1 Requirement Status Vocabulary

Concrete EQC-SR projects MUST use these requirement statuses in conformance matrices:

```text
satisfied
partial
missing
not_applicable_with_reason
deferred_unreachable_at_claimed_level
documented_deviation
```

`partial` is not allowed for BLOCKER requirements at `SR-PORT-READY` or `SR-RELEASE-READY`.

### 0.9.2 Evidence Strength Ranking

When multiple evidence types exist, stronger evidence SHOULD be preferred. Default ranking:

```text
formal_trace_or_golden_test
property_based_test
negative_test
unit_test
source_crosswalk
manual_review_note
inference
```

A release-targeted subsystem SHOULD NOT depend only on `manual_review_note` or `inference` for any requirement that affects candidate validity, ranking, reproducibility, or public API behavior.

## 1. Relationship to Core EQC

EQC-SR is a domain profile, not a replacement for EQC.

All EQC-SR documents MUST still satisfy base EQC requirements:

- global objective semantics;
- reproducibility contract;
- numeric policy;
- ordering and tie-break policy;
- operator manifest;
- explicit persistent state;
- explicit initialization;
- control-flow-only procedure;
- trace schema;
- validation rules;
- equivalence rules;
- checkpoint/restore rules.

EQC-SR adds symbolic-regression-specific required sections and operator categories.

A valid symbolic-regression specification SHOULD use the following document chain:

```text
EQC.md
  ↓ extends
EQC-SR.md
  ↓ instantiates
ALGO-SR-*.eqc.md
  ↓ binds through
EQC-SIB artifacts: pseudocode / Python / tests / traces
```

---

## 2. Required Document Types

A complete release-targeted EQC-SR portfolio MUST contain or explicitly mark not applicable these document types. Research-only portfolios SHOULD contain the subset relevant to the current subsystem.

| DocID Pattern | Type | Purpose |
|---|---|---|
| `SR-PROFILE-*` | profile | Symbolic-regression profile/version policy |
| `SR-OPLIB-*` | operator-library | Math operators, protected operators, custom operators |
| `SR-EXPR-*` | algorithm-spec | Expression model and canonicalization rules |
| `SR-EVAL-*` | algorithm-spec | Evaluation and numeric behavior |
| `SR-SEARCH-*` | algorithm-spec | Main evolutionary search loop |
| `SR-RANK-*` | algorithm-spec | Loss, complexity, Pareto, hall-of-fame ranking |
| `SR-CONSTOPT-*` | algorithm-spec | Constant optimization rules |
| `SR-TRACE-*` | test-suite / golden-trace-set | Trace schemas and golden runs |
| `SR-PYSR-COMPAT-*` | compatibility spec | PySR-style API compatibility |
| `SR-PYBACKEND-*` | derived-impl / pseudocode binding | Python backend implementation binding |

All documents MUST be registered in the EQC-ES registry if used in a governed portfolio.

---



## 2.1 Required Registry Mapping

Every EQC-SR document in a governed portfolio MUST be registered through EQC-ES and every pseudocode or implementation artifact MUST be bound through EQC-SIB. At minimum, the registry mapping MUST declare:

```text
DocID
Document type
Layer
Status
Current version
FunctionalDigest
MetadataDigest
IMPORTS / EXTENDS / USES edges
Provided operator namespaces, if any
Bound pseudocode artifacts, if any
Bound implementation artifacts, if any
Bound validation artifacts, if any
```

For a Julia-to-Python port, the following binding pattern is recommended:

```text
SR-EXTRACT-*       REFERENCES source Julia files and commits
SR-EXPR-*          GOVERNS expression model pseudocode and code
SR-OPLIB-*         GOVERNS operator registry and operator implementations
SR-EVAL-*          GOVERNS evaluator implementations
SR-SEARCH-*        GOVERNS search-loop pseudocode and Python implementation
SR-TRACE-*         VALIDATES all governed implementation artifacts
SR-PYSR-COMPAT-*   GOVERNS the public Python compatibility API
```

A document that changes functional behavior MUST trigger EQC-ES impact analysis, and a code/pseudocode artifact that changes functional behavior MUST trigger EQC-SIB impact analysis.

## 3. EQC-SR Compatibility Levels

EQC-SR defines staged compatibility so the project can advance without pretending to be a full PySR replacement immediately.

| Level | Name | Meaning |
|---:|---|---|
| SR-L0 | Standalone Minimal Engine | Can discover simple formulas with basic operators and scalar loss. |
| SR-L1 | Governed EQC Engine | Has complete EQC-SR semantics, traces, reproducibility, and checkpoints. |
| SR-L2 | Custom Operator Engine | Supports user-defined mathematical operators with registry metadata and safe export. |
| SR-L3 | PySR-Like API | Provides `fit`, `predict`, `equations_`, symbolic export, and common options. |
| SR-L4 | Broad PySR Compatibility | Supports common PySR operators, constraints, loss customization, warm start, and model selection. |
| SR-L5 | Near Drop-In Backend | Intended to replace PySR's backend for a large subset of workflows. |

A document MUST declare its target compatibility level.

Default first target for a Python port:

```text
Target: SR-L2 first, then SR-L3.
```

### 3.1 Compatibility Claim Rule

A compatibility claim MUST include:

```text
compatibility_level
covered_public_surface
unsupported_options
known_deviations
equivalence_target
validation_evidence
```

A project MUST NOT describe itself as PySR-compatible without naming the exact SR-L level and the unsupported option set.

### 3.2 Minimum Viable Port Scope

The first useful Python-native port SHOULD target this scope before any broad compatibility work:

```text
Expression tree + canonical serialization
Operator registry with protected math
Vectorized evaluation
MSE loss
Complexity scoring
Random expression generation
Mutation-only search
Hall of fame
JSONL trace
Checkpoint with RNG + archive
Custom operator registration
```

Crossover, multi-population migration, distributed execution, broad PySR option compatibility, and advanced simplification SHOULD remain out of scope until the minimum port passes golden tests.

---

## 3.3 PySR Option Coverage Matrix

A PySR-style compatibility claim MUST include an option coverage matrix. Each public option or behavior that users may reasonably expect MUST be classified as:

```text
supported
supported_with_deviation
accepted_but_ignored_with_warning
rejected_with_clear_error
not_in_scope_for_level
unresolved
```

Minimum fields:

```text
option_name
source_api
compatibility_level_required
status
semantic_owner_spec
implementation_artifact
test_coverage
error_or_warning_code
notes
```

The backend MUST NOT silently ignore a PySR-style option unless the compatibility spec explicitly classifies it as `accepted_but_ignored_with_warning` and declares the warning code.

### 3.4 Compatibility Claim Labels

Use precise labels instead of vague compatibility claims:

```text
PySR-inspired API          similar user-facing style, no drop-in claim
PySR-subset compatible     supports declared subset only
PySR-option compatible     specific options supported by matrix
PySR-behavior comparable   validated on declared golden/reference problems
PySR-drop-in compatible    broad surface support, strict error policy, and migration notes
```

`PySR-drop-in compatible` MUST NOT be claimed before SR-L4 and SR-RELEASE-READY evidence exist.



### 3.5 Capability Profile Contract

Every backend claim MUST declare a capability profile independent of its API compatibility level. Compatibility level says how much public behavior is exposed; capability profile says what the engine can actually do.

Minimum fields:

```yaml
capability_profile:
  profile_id: "SR-CAP-MINIMAL-MUTATION-v1"
  expression_form: "tree"
  typed_expressions: false
  supported_regression_modes: ["scalar_real_regression"]
  search_strategy: ["mutation_only"]
  populations: "single"
  constants: "ephemeral_and_mutated"
  constant_optimizer: "none|deterministic_local|external_declared"
  evaluation_backends: ["numpy_vectorized"]
  parallelism: "disabled|deterministic_declared"
  custom_operators: "registry_required"
  simplification: "none|declared_operator"
  exports: ["canonical", "python_callable", "sympy_optional"]
```

A compatibility layer MUST NOT expose options that imply unavailable capabilities unless it rejects them or marks them as accepted-but-ignored with a warning code in the option coverage matrix.

### 3.6 Minimum Honest Claim Set

A project at early stage SHOULD use one of these honest public claims:

```text
EQC-SR-specified symbolic-regression prototype
Python-native symbolic-regression backend, SR-L0
Python-native custom-operator symbolic-regression backend, SR-L2
PySR-inspired API, declared option subset only
```

The profile SHOULD avoid terms such as `drop-in`, `full PySR`, `complete port`, or `feature parity` until the option matrix, conformance matrix, benchmark profile, and release gates support those claims.

## 4. Global Symbolic Regression Semantics

### 4.1 Optimization Sense

Unless overridden, symbolic regression minimizes a vector objective:

```text
objective(candidate) = (primary_loss, complexity, tie_break_key)
```

Default comparison:

1. lower primary loss is better;
2. if losses are equal within `EPS_LOSS_EQ`, lower complexity is better;
3. if both are tied, deterministic structural key wins;
4. if still tied, lowest insertion index wins.

For Pareto mode, the objective is partially ordered by loss and complexity, but every selection step MUST define a deterministic total fallback.

### 4.2 Invalid Candidate Policy

A candidate is invalid if any mandatory validation check fails:

- invalid expression structure;
- arity mismatch;
- unresolved operator;
- type/domain violation;
- max size exceeded;
- max depth exceeded;
- forbidden nested operator pattern;
- evaluation failure not handled by protected operators;
- non-finite objective under the declared numeric policy.

Default invalid ranking:

```text
valid candidate ≺ invalid candidate
invalid candidates ordered by deterministic invalid_reason_code, then structural key, then insertion index
```

Invalid candidates MAY still be logged for diagnostics, but MUST NOT enter the hall of fame unless the algorithm explicitly supports invalid-archive debugging.

---



## 4.3 Default Reproducibility and Numeric Defaults

Unless a concrete spec overrides these values, EQC-SR recommends the following defaults:

```text
Seed space: uint64
PRNG family: PCG64 or explicitly declared equivalent
Floating dtype: IEEE-754 binary64
Fast-math: forbidden for governed reproducibility runs
Index base in canonical forms: 0-based
Sorts: stable
Argmin/argmax ties: lowest deterministic key wins
NaN/Inf objective: ranked worse than every valid finite objective
Transcendental equality: tolerance-based, not bitwise, unless profile declares otherwise
Parallel reduction: fixed deterministic order or disabled
```

A production implementation MAY use faster profiles, but any profile that enables fast-math, nondeterministic GPU kernels, unordered parallel reductions, or approximate compiler rewrites MUST declare a lower equivalence target unless evidence proves stronger equivalence.

## 5. Expression Tree Contract

### 5.1 Expression Object

Every expression MUST be representable as:

```text
Expression := Node
Node := Variable | Constant | OperatorCall
OperatorCall := OperatorID(children...)
```

Each node MUST expose:

- `node_kind`;
- `operator_id` if applicable;
- `arity`;
- ordered children;
- numeric value if constant;
- variable index if variable;
- declared output type;
- structural hash;
- size;
- depth;
- complexity;
- canonical string or canonical serialization.

### 5.2 Node Identity

EQC-SR distinguishes:

- **object identity:** implementation-specific memory identity;
- **structural identity:** same tree shape, same operators, same variables, same constants according to constant equality policy;
- **semantic identity:** same evaluated function under declared numeric and domain policy;
- **canonical identity:** same canonical serialization.

Unless explicitly stated, equality in search, caching, duplicate detection, and hall-of-fame logic uses canonical identity.

### 5.3 Canonical Serialization

A canonical expression serialization MUST define:

- prefix/infix/tree encoding;
- operator namespace and version encoding;
- variable index encoding;
- constant encoding, precision, and rounding;
- child order;
- commutative operator child sorting policy;
- associative flattening policy;
- unary minus policy;
- protected operator names;
- custom operator names;
- version tag.

Recommended canonical text format:

```text
op.namespace.name_vN(child_0,child_1,...)
var[0]
const[float64:0x1.921fb54442d18p+1]
```

Human-readable forms MAY differ from canonical forms.

### 5.4 Structural Hash

Structural hash MUST be computed from canonical serialization.

Default:

```text
structural_hash = SHA256(canonical_expression_bytes)
```

Hash collisions MUST be handled by canonical serialization comparison before treating expressions as duplicates.

---



### 5.5 Semantic Equality Is Not a Default Duplicate Rule

Semantic identity is generally undecidable for arbitrary symbolic expressions. Therefore, EQC-SR MUST NOT use semantic equality as the default duplicate-detection rule.

Default duplicate detection order:

1. compare structural hash;
2. if hashes match, compare canonical serialization;
3. if both match, treat as duplicate;
4. optional semantic equivalence checks MAY be used only as a secondary simplification or archive-compaction step.

If semantic equivalence is used, the concrete spec MUST declare:

- the equivalence engine;
- timeout;
- allowed rewrites;
- failure behavior;
- whether numerical sampling is allowed as evidence;
- whether the result can affect hall-of-fame membership.

## 6. Operator Registry Contract

### 6.1 Operator Record

Every operator MUST declare:

- `operator_id`;
- `namespace`;
- `name`;
- `version`;
- `arity`;
- `input_types`;
- `output_type`;
- `purity_class`;
- `determinism`;
- `complexity_cost`;
- `domain_policy`;
- `nan_inf_policy`;
- `numpy_impl` or equivalent reference;
- `numba_impl` if production Python backend uses Numba;
- `sympy_export` if symbolic export is supported;
- `latex_export` if LaTeX export is supported;
- `python_callable_export` if runtime prediction export is supported;
- test vectors;
- edge cases;
- dependency list;
- version compatibility rules.

### 6.2 Operator Categories

EQC-SR recognizes these operator categories:

- arithmetic;
- transcendental;
- protected math;
- comparison-like mathematical functions;
- domain-specific custom operators;
- ephemeral constants;
- feature transforms;
- aggregation operators if n-ary expressions are supported;
- internal-only helper operators.

Only operators declared as `search_visible: true` may appear in generated candidate expressions.

### 6.3 Protected Operator Requirement

Risky operators MUST have protected variants or a declared rejection policy.

Risky operators include:

- division;
- reciprocal;
- logarithm;
- square root;
- power;
- exponential;
- tangent;
- arcsine;
- arccosine;
- user-defined operators with restricted domains.

Default protected definitions:

```text
protected_div(x, y) = x / y              if abs(y) >= EPS_DENOM else SIGN_POLICY(x, y)
safe_log(x)        = log(abs(x) + EPS_DENOM)
safe_sqrt(x)       = sqrt(abs(x))
safe_exp(x)        = exp(clamp(x, EXP_MIN, EXP_MAX))
safe_inv(x)        = 1 / x               if abs(x) >= EPS_DENOM else INV_FALLBACK
```

Each protected operator MUST specify its exact fallback behavior. The placeholder names above are not sufficient in a concrete algorithm spec.

### 6.4 Custom Operator Policy

A custom operator is valid only if it declares:

- stable name;
- arity;
- vectorized implementation;
- scalar fallback if required;
- domain and numeric safety policy;
- complexity cost;
- export behavior;
- test vectors;
- whether it is allowed during mutation;
- whether it is allowed during simplification;
- whether it is allowed in final output;
- whether it is deterministic and side-effect-free.

Custom operators used during search MUST be `PURE` and deterministic.

Operators that perform file I/O, network access, database access, mutable state updates, hidden random sampling, or external nondeterministic calls MUST NOT be search-visible.

---



### 6.5 Machine-Readable Operator Schema

Every operator library SHOULD provide a machine-readable schema. Minimum required fields:

```yaml
operator_id: "sr.math.safe_log_v1"
namespace: "sr.math"
name: "safe_log"
version: "v1"
arity: 1
input_types: ["float64"]
output_type: "float64"
search_visible: true
purity_class: "PURE"
determinism: "DETERMINISTIC"
complexity_cost: 2
domain_policy:
  finite_inputs_required: true
  protected_domain: "all_real_finite"
numeric_policy:
  nan_policy: "invalid_candidate"
  inf_policy: "invalid_candidate"
  eps_denom_ref: "NumericPolicy.EPS_DENOM"
implementations:
  numpy: "sr_backend.ops.safe_log_numpy"
  numba: "sr_backend.ops.safe_log_numba"
exports:
  sympy: "log(abs(x) + EPS_DENOM)"
  latex: "\\log(|x| + \varepsilon)"
tests:
  - input: ["0.0"]
    expected: "log(EPS_DENOM)"
```

The schema MUST reject operators with undeclared side effects, missing complexity cost, missing arity, missing export policy for final-output use, or missing test vectors for custom operators.

### 6.6 Operator Grammar for String-Based Configuration

If a compatibility layer accepts string-defined operators, the grammar MUST be explicitly declared. The default safe grammar is:

```text
operator_definition := identifier "(" arg_list ")" "=" expression
arg_list            := identifier | identifier "," identifier | ...
expression          := literals, arguments, allowed arithmetic operators, allowed registered calls
forbidden           := assignment other than top-level definition, import, eval, exec, attribute access, file I/O, network I/O, mutation of external state
```

String operator definitions MUST be parsed, validated, and converted into registry records. They MUST NOT be executed as unrestricted host-language code.

## 7. Dataset and Input Contract

### 7.1 Dataset Record

A symbolic-regression run MUST declare a dataset record:

- `X` shape and dtype;
- `y` shape and dtype;
- feature names;
- target name;
- sample weights if used;
- train/validation split if used;
- hashes of raw and preprocessed data;
- missing-value policy;
- scaling/normalization policy;
- batching policy;
- row ordering policy.

### 7.2 Missing and Non-Finite Input Policy

The spec MUST declare one of:

- reject dataset if any non-finite values exist;
- impute through a named preprocessing operator;
- allow non-finite values only if all operators define behavior for them.

Default:

```text
reject non-finite X or y before search starts
```

### 7.3 Batch Evaluation Policy

If candidates are evaluated on batches or subsamples, the spec MUST define:

- batch selection method;
- whether selection is deterministic or stochastic;
- PRNG ownership for stochastic batches;
- batch size;
- full-dataset reevaluation frequency;
- whether hall-of-fame candidates require full-dataset scoring;
- how batch loss is aggregated.

Default for reproducible first implementation:

```text
full dataset evaluation for all candidates
```

---



### 7.4 Train/Validation/Test Semantics

If model selection or generalization scoring is used, the spec MUST declare:

- train split;
- validation split;
- test split, if any;
- split generation seed;
- row order after split;
- whether the search sees validation loss;
- whether hall-of-fame ranking uses train loss, validation loss, or a composite;
- whether final reported score is computed on held-out data.

Default for SR-L0/SR-L1:

```text
single declared scoring dataset, no hidden validation split
```

### 7.5 Feature and Target Type Semantics

The dataset contract MUST specify whether the run is:

```text
scalar regression
multiple-output regression
classification-like symbolic modeling
weighted regression
unit-aware regression
```

EQC-SR v0.5 defaults to scalar real-valued regression. Other modes are allowed only when their loss, prediction type, export behavior, and metric schema are explicitly specified.

## 8. Evaluation Semantics

### 8.1 Candidate Evaluation

Candidate evaluation maps:

```text
Evaluate(Expression, X, OperatorRegistry, NumericPolicy) -> y_pred | EvaluationError
```

The spec MUST define:

- traversal order;
- operator dispatch;
- vectorization behavior;
- constant handling;
- dtype promotion;
- broadcasting rules;
- temporary allocation policy if observable;
- non-finite handling;
- error code behavior.

### 8.2 Evaluation Backends

Permitted backend classes:

- `debug_python`: pure Python, correctness/debug only;
- `numpy_vectorized`: default minimal production backend;
- `numba_compiled`: preferred Python performance backend;
- `jax_compiled`: optional;
- `torch_compiled`: optional;
- `external_backend`: allowed only if declared through SIB and dependency manifests.

Production Python backend rule:

```text
No pure Python inner evaluation loop is allowed for production scoring unless explicitly classified as SR-L0 debug mode.
```

### 8.3 Non-Finite Prediction Policy

The spec MUST define one of:

- reject candidate;
- assign invalid objective;
- replace non-finite predictions with clamp value;
- compute finite mask loss only.

Default:

```text
if any y_pred is NaN or Inf, candidate evaluation fails with EVAL_NONFINITE_PREDICTION and candidate is invalid
```

---



### 8.4 Evaluation Shape and Type Contract

Every evaluator MUST declare a shape contract:

```text
X: shape (n_samples, n_features)
y_pred: shape (n_samples,) for scalar regression
operator input arrays: shape (n_samples,) unless scalar broadcasting is explicitly allowed
constants: scalar broadcast to (n_samples,) during vectorized evaluation
```

The evaluator MUST declare dtype promotion. Default:

```text
inputs converted to float64 before governed evaluation
operator outputs must be float64 arrays or scalar float64 constants
boolean or integer operators are unsupported unless a typed-expression profile is active
```

### 8.5 Compiled Backend Eligibility

A candidate expression is eligible for a compiled backend only if:

- every operator has a compatible compiled implementation;
- every constant can be represented in the compiled dtype;
- no unsupported export or dynamic dispatch is required inside the hot loop;
- the compiled backend preserves the declared numeric policy;
- fallback behavior is deterministic.

If compiled evaluation fails, the spec MUST declare whether to:

```text
fall back to numpy_vectorized
mark candidate invalid
mark backend run failed
```

Fallbacks MUST be recorded in the trace.

### 8.6 Evaluation Cache Contract

If evaluation caching is used, the cache key MUST include at minimum:

```text
structural_hash
constant encoding
dataset digest or batch digest
operator manifest digest
numeric policy digest
evaluation backend identity
```

Cache hits MAY affect performance counters but MUST NOT change candidate ranking, trace semantics, or archive eligibility unless cache events are part of the declared trace schema.



### 8.7 Evaluator Determinism Test Matrix

A governed evaluator MUST be tested across a deterministic matrix of expression classes. Minimum matrix:

```text
terminal_variable
terminal_constant
binary_arithmetic
protected_division
unary_transcendental
custom_unary_operator
custom_binary_operator
invalid_domain_expression
expression_with_repeated_subtree
expression_at_max_depth
```

For each class, tests MUST verify:

```text
shape contract
dtype contract
non-finite behavior
structural hash stability
loss reproducibility
export equivalence if exported
```

A backend with multiple evaluators MUST compare each production evaluator against the reference evaluator on the same matrix under declared tolerances.

### 8.8 Evaluation Failure Boundary

The spec MUST distinguish between:

```text
candidate_invalid        one candidate is invalid; search continues
operator_invalid         operator declaration is invalid; run cannot start
dataset_invalid          input data violates contract; run cannot start
backend_invalid          evaluator cannot satisfy declared profile; run cannot start or must downgrade with trace
internal_error           implementation failure; run terminates with final trace
```

Candidate-level failures MUST NOT be escalated to run-level failures unless the concrete spec declares that policy. Run-level failures MUST emit a final trace record.

## 9. Loss, Complexity, and Ranking

### 9.1 Loss Contract

A loss operator MUST declare:

- input shapes;
- sample-weight behavior;
- reduction rule;
- dtype;
- non-finite target/prediction behavior;
- tolerance;
- whether lower or higher is better.

Default loss:

```text
MSE(y_pred, y) = mean((y_pred - y)^2)
```

### 9.2 Complexity Contract

Every expression MUST have a complexity score.

Default:

```text
complexity(expr) = sum(complexity_cost(node) for node in expression tree)
```

The spec MUST declare:

- variable cost;
- constant cost;
- operator-specific cost;
- repeated subtree behavior;
- n-ary operator cost if supported;
- simplification effect;
- maximum allowed complexity.

### 9.3 Score Contract

If a scalar score is used:

```text
score = loss + parsimony_weight * complexity
```

The spec MUST declare:

- `parsimony_weight`;
- whether loss is normalized;
- whether complexity penalty is linear, logarithmic, or custom;
- tie-breaking;
- invalid score behavior.

### 9.4 Pareto Contract

If Pareto ranking is used, the spec MUST define:

- objective dimensions;
- dominance relation;
- equality tolerance per dimension;
- duplicate detection;
- frontier update order;
- maximum frontier size;
- tie-break fallback;
- removal policy when full.

Default dimensions:

```text
(loss, complexity)
```

---



### 9.5 Model Selection Contract

If multiple equations are retained, the spec MUST declare how the final selected model is chosen. Allowed model-selection strategies include:

```text
best_loss
best_score
best_validation_loss
pareto_knee
simplest_within_loss_tolerance
user_selected_from_equations_table
```

Default:

```text
simplest candidate whose loss is within EPS_MODEL_SELECT of the best valid loss; ties resolved by canonical key then insertion index
```

If this default is not desired, the concrete spec MUST override it.

### 9.6 Complexity Must Be Stable Under Export

The complexity reported for an equation MUST correspond to the canonical EQC expression unless the spec declares a separate exported complexity. If simplification changes the expression, both pre-simplification and post-simplification complexity SHOULD be logged.

## 10. Random Expression Generation

### 10.1 Generator Contract

Random expression generation MUST declare:

- max depth;
- max size;
- variable sampling distribution;
- constant sampling distribution;
- operator sampling distribution;
- terminal probability;
- arity handling;
- retry budget;
- failure policy.

All randomness MUST occur inside versioned generator operators.

### 10.2 Initial Population Contract

Initial population generation MUST declare:

- population size;
- number of populations;
- seed derivation per population;
- uniqueness policy;
- invalid candidate handling;
- guaranteed terminal expressions, if any;
- initial constant policy;
- baseline expressions, if any.

Default baseline expressions SHOULD include:

- each variable alone;
- constant mean predictor if regression target is numeric;
- simple linear forms if enabled by compatibility level.

### 10.3 Seed Derivation Contract

If multiple stochastic components exist, child seeds MUST be derived deterministically from the run seed and stable component identifiers. Recommended form:

```text
child_seed = Hash64(run_seed || component_id || population_id || stream_name)
```

The spec MUST define whether RNG streams are independent per population, per worker, per proposal operator, or globally threaded. Hidden calls to process-global RNG state are prohibited.

---

## 11. Mutation Contract

### 11.1 Required Mutation Metadata

Each mutation operator MUST declare:

- mutation type;
- preconditions;
- postconditions;
- affected subtree policy;
- random choices;
- retry budget;
- constraint checks;
- invalid result behavior;
- trace fields.

### 11.2 Standard Mutation Types

EQC-SR recognizes:

- replace subtree;
- insert unary operator;
- insert binary operator;
- delete operator;
- replace operator with same arity;
- replace variable;
- perturb constant;
- add constant;
- simplify subtree;
- wrap subtree;
- splice subtree from another expression.

A concrete algorithm MAY use a subset, but MUST declare mutation weights over the enabled set.

### 11.3 Mutation Validity

A mutation is valid only if the result:

- has valid tree structure;
- respects operator arity;
- respects type rules;
- respects max depth;
- respects max size;
- respects max complexity;
- respects nested constraints;
- has all operators registered;
- passes optional evaluation sanity check if enabled.

If no valid mutation is found within retry budget, the operator MUST return either:

- deterministic no-op with reason code; or
- structured error with reason code.

Default:

```text
return no-op mutation with MUTATION_NO_VALID_PROPOSAL
```

---

## 12. Crossover Contract

Crossover operators MUST declare:

- parent selection inputs;
- subtree selection policy;
- arity/type compatibility rules;
- child construction rules;
- retry budget;
- constraint checks;
- duplicate policy;
- invalid child policy.

Default first implementation MAY omit crossover and use mutation-only search, but the omission MUST be explicit.

---


## 12.1 Parent and Child Lineage Contract

Any crossover-enabled spec MUST define lineage fields so that a candidate can be traced back to its source parents and proposal operator. Minimum lineage fields:

```text
child_hash
parent_hashes
proposal_operator_id
proposal_step
population_id
rng_fingerprint_before
rng_fingerprint_after
```

## 13. Population, Selection, and Evolution Loop

### 13.1 Population State

Persistent population state MUST declare:

- population id;
- candidate list;
- candidate metadata;
- current scores;
- age or generation if used;
- local hall of fame if used;
- random state ownership;
- migration inbox/outbox if used.

### 13.2 Selection Contract

Selection operators MUST declare:

- selection method;
- tournament size if tournament selection is used;
- fitness transformation;
- tie-breaking;
- invalid candidate policy;
- stochastic sampling behavior;
- trace fields.

### 13.3 Evolution Step Contract

Each evolution step MUST define:

1. parent selection;
2. proposal generation through mutation or crossover;
3. validation;
4. evaluation;
5. scoring;
6. replacement or insertion;
7. archive update;
8. trace emission;
9. explicit persistent state update.

The main procedure MUST remain control-flow-only and call named operators for all semantic work.

### 13.3.1 Minimal Control-Flow Skeleton

A mutation-only EQC-SR procedure SHOULD be expressible with this control-flow skeleton. Concrete specs may rename operators but MUST preserve explicit state updates.

```text
Procedure SR_Search(seed, dataset, options):

  rng_state ← Random.InitializePRNG_v#(seed)
  operator_registry ← Operators.LoadRegistry_v#(options.operator_manifest)
  dataset_state ← Data.PrepareDataset_v#(dataset, options.data_policy)

  population ← Init.GenerateInitialPopulation_v#(dataset_state, operator_registry, options, rng_state)
  evaluated_population ← Eval.EvaluatePopulation_v#(population, dataset_state, operator_registry, options)
  archive ← Archive.InitializeHallOfFame_v#(evaluated_population, options)
  trace ← Trace.StartRun_v#(seed, dataset_state, operator_registry, options)
  t ← 0

  WHILE NOT Termination.Terminated_v#(t, population, archive, options):

      (parent, rng_state) ← Select.SelectParent_v#(population, archive, rng_state, options)
      (candidate, proposal_record, rng_state) ← Propose.Mutate_v#(parent, operator_registry, rng_state, options)
      validity ← Validate.Candidate_v#(candidate, operator_registry, options.constraints)

      IF validity.status == valid:
          evaluation ← Eval.EvaluateCandidate_v#(candidate, dataset_state, operator_registry, options)
          score_record ← Rank.ScoreCandidate_v#(candidate, evaluation, options)
      ELSE:
          evaluation ← Eval.InvalidEvaluation_v#(validity)
          score_record ← Rank.InvalidScore_v#(candidate, validity, options)

      population ← Replace.UpdatePopulation_v#(population, candidate, score_record, options)
      archive ← Archive.UpdateHallOfFame_v#(archive, candidate, score_record, options)
      trace ← Trace.LogSearchStep_v#(trace, t, proposal_record, validity, evaluation, score_record, archive, rng_state)
      t ← t + 1

  checkpoint ← Checkpoint.Emit_v#(population, archive, rng_state, trace, options)
  RETURN Results.Finalize_v#(archive, checkpoint, trace, options)
```

This skeleton is not the only valid algorithm, but any concrete algorithm MUST expose the same semantic categories if it claims SR-L1 or higher.

---



### 13.4 Replacement Policy

The population replacement operator MUST define:

- whether population size is fixed;
- whether invalid candidates can remain in population;
- whether duplicate candidates are allowed;
- whether age is considered;
- whether lower-ranking candidates can survive for diversity;
- how ties are resolved;
- how replacement is logged.

Default:

```text
fixed population size; valid candidates preferred over invalid; deterministic worst-candidate replacement under the declared total preorder; duplicates rejected unless all retry attempts fail
```

### 13.5 Termination Contract

Termination MUST be handled by a versioned operator. Termination reasons MUST be enumerated. Minimum reason codes:

```text
MAX_ITERATIONS
MAX_EVALUATIONS
TIME_BUDGET_RECORDED_ONLY
TARGET_LOSS_REACHED
STALL_LIMIT_REACHED
NO_VALID_CANDIDATES
USER_STOP_REQUESTED_IF_SUPPORTED
ERROR_TERMINATION
```

Time-budget termination is allowed only when the time source is declared under the environment/profile policy. For reproducibility traces, max-iteration or max-evaluation termination is preferred.

## 14. Migration Between Populations

If multiple populations are used, migration MUST declare:

- migration interval;
- migrant selection rule;
- destination rule;
- insertion/replacement rule;
- deterministic ordering;
- tie-breaking;
- whether migration is synchronous or asynchronous;
- trace fields.

Default for reproducible first implementation:

```text
single population, no migration
```

---

## 15. Hall of Fame and Archive Contract

### 15.1 Hall-of-Fame Entry

Each hall-of-fame entry MUST contain:

- canonical expression;
- structural hash;
- loss;
- complexity;
- score if used;
- origin step;
- origin population;
- operator manifest digest;
- dataset digest;
- evaluation backend;
- numeric policy digest;
- export representations if materialized.

### 15.2 Update Rule

The hall-of-fame update operator MUST define:

- eligibility;
- duplicate detection;
- dominance or score comparison;
- max entries;
- replacement rule;
- simplification timing;
- full-dataset reevaluation requirement;
- tie-breaking.

Default:

```text
Only valid candidates evaluated on the full declared scoring dataset may enter the hall of fame.
```

---



### 15.3 Equation Table Contract

A PySR-like equation table MUST have a declared schema. Minimum fields:

```text
equation_id
canonical_expression
human_expression
loss
complexity
score_or_selection_metric
structural_hash
origin_step
selected_flag
validity_status
export_status
```

Rows MUST be ordered deterministically. Default order:

```text
loss ascending, complexity ascending, canonical key ascending, insertion index ascending
```

If Pareto ordering is exposed, the table MUST still define a deterministic display order.



### 15.4 Archive Revalidation Rule

Any candidate that enters a public archive, final equation table, or exported result MUST be revalidated under the declared archive evaluation policy.

Minimum revalidation fields:

```text
archive_eval_dataset_digest
archive_eval_backend
archive_eval_numeric_policy_digest
archive_eval_timestamp_policy
archive_eval_loss
archive_eval_complexity
archive_eval_status
```

If search used batches or approximate scoring, public archive entries MUST be rescored on the declared final scoring dataset unless the concrete spec explicitly marks the archive as approximate/debug-only.

### 15.5 Archive Immutability and Revision Rule

Once an archive entry is emitted in a governed trace, later simplification, constant optimization, or export rewriting MUST create either:

```text
a new archive entry linked to the previous one, or
a revision record preserving the original canonical expression and original score
```

Overwriting archive entries in place is forbidden for `SR-RELEASE-READY` traces.

## 16. Constant Handling and Optimization

### 16.1 Constant Representation

Constants MUST declare:

- dtype;
- canonical encoding;
- initialization distribution;
- equality tolerance;
- bounds;
- mutation behavior;
- export formatting.

Recommended canonical float representation:

```text
IEEE-754 binary64 hexadecimal float string
```

### 16.2 Constant Optimization Contract

If constant optimization is used, the spec MUST declare:

- optimizer family;
- trigger condition;
- max iterations;
- tolerance;
- bounds;
- retry policy;
- random initialization policy;
- failure behavior;
- effect on expression identity;
- trace fields.

Permitted optimizer classes:

- no optimization;
- random perturbation only;
- coordinate search;
- SciPy local optimizer;
- gradient-based optimizer;
- hybrid optimizer.

Default for SR-L0:

```text
constant mutation only, no external optimizer
```

Default for SR-L2 and above:

```text
constant optimizer MAY be enabled but must have deterministic seed, deterministic options, and traceable failure behavior
```

---

## 17. Constraint Contract

### 17.1 Structural Constraints

The spec MUST declare:

- max depth;
- max size;
- max complexity;
- allowed operators;
- forbidden operators;
- allowed variable indices;
- maximum constants;
- arity rules.

### 17.2 Nested Constraints

Nested constraints MUST define forbidden ancestor/descendant patterns.

Example:

```text
forbid: exp(exp(x))
forbid: log(log(x))
forbid: operator_A inside operator_B
```

Concrete specs MUST express these patterns in a machine-checkable form.

### 17.3 Dimensional or Unit Constraints

If physical units are used, the spec MUST declare:

- units per feature;
- operator unit rules;
- allowed output unit;
- behavior on unit violation.

Default:

```text
no dimensional constraints
```

---


### 17.4 Constraint Failure Codes

Constraint failures MUST use stable reason codes. Minimum recommended codes:

```text
CONSTRAINT_MAX_DEPTH
CONSTRAINT_MAX_SIZE
CONSTRAINT_MAX_COMPLEXITY
CONSTRAINT_FORBIDDEN_OPERATOR
CONSTRAINT_NESTED_PATTERN
CONSTRAINT_TYPE_MISMATCH
CONSTRAINT_UNIT_MISMATCH
CONSTRAINT_TOO_MANY_CONSTANTS
CONSTRAINT_UNREGISTERED_OPERATOR
```

## 18. Simplification and Canonicalization



### 18.1 Canonicalization Versus Simplification

Canonicalization and simplification MUST be separate concepts.

- **Canonicalization** produces stable identity and hashing.
- **Simplification** changes the expression according to rewrite rules.

Canonicalization MAY reorder commutative children or normalize encodings, but MUST NOT perform algebraic rewrites unless those rewrites are explicitly classified as canonical rewrites.

Simplification MAY change the candidate and therefore MUST trigger reevaluation before the simplified candidate is used for ranking, export, or archive insertion.

Simplification MUST be treated as a semantic operator, not an informal cleanup step.

A simplification operator MUST declare:

- rewrite rules;
- rewrite order;
- fixed-point behavior;
- max rewrite steps;
- numeric tolerance;
- effect on constants;
- effect on structural hash;
- effect on hall-of-fame entries;
- trace fields.

If SymPy is used, the spec MUST declare:

- SymPy version;
- exact functions called;
- timeout policy;
- failure policy;
- whether simplified expression must be reevaluated before archive insertion.

Default:

```text
simplification is optional, but any simplified candidate entering the hall of fame must be reevaluated and rescored
```

---

## 19. Export Contract

Each export format MUST be defined separately.

### 19.1 Export Equivalence Test

Every final-output export path MUST have an equivalence test. Minimum requirement:

```text
for each final equation and export backend:
    evaluate canonical EQC expression on test input matrix
    evaluate exported expression on same test input matrix
    compare predictions under declared EPS_EXPORT
```

Unsupported operators MUST produce structured export errors. Silent dropping, rewriting, or substituting operators is prohibited.


Supported export categories:

- canonical EQC expression;
- Python callable;
- NumPy expression;
- Numba-compatible expression;
- SymPy expression;
- LaTeX expression;
- PyTorch expression;
- JAX expression;
- JSON expression tree.

Each export operator MUST declare:

- supported operators;
- unsupported operator behavior;
- numeric equivalence target;
- constant formatting;
- protected operator mapping;
- custom operator mapping;
- test vectors.

Default required exports for SR-L2:

```text
canonical EQC expression
Python callable or NumPy callable
SymPy expression where possible
```

---

## 20. Trace Schema

### 20.1 Required Run-Level Trace Fields

Every run trace MUST include:

- `run_id`;
- `spec_version`;
- `compatibility_level`;
- `seed`;
- `prng_family`;
- `operator_manifest_digest`;
- `dataset_digest`;
- `numeric_policy_digest`;
- `environment_profile`;
- `evaluation_backend`;
- `start_time_policy` as fixed or recorded;
- `completion_status`;
- `termination_reason`.

### 20.2 Required Iteration-Level Trace Fields

Each iteration or search step MUST include:

- `t`;
- `rng_fingerprint`;
- `population_id`;
- `selected_parent_hashes`;
- `proposal_operator`;
- `mutation_or_crossover_type`;
- `candidate_hash_before` if applicable;
- `candidate_hash_after`;
- `validity_status`;
- `invalid_reason_code` if invalid;
- `loss` if evaluated;
- `complexity`;
- `score` if used;
- `accepted_or_inserted`;
- `archive_update_status`;
- `best_hash_after_step`;
- `best_loss_after_step`;
- `best_complexity_after_step`.

### 20.3 Candidate-Level Trace Fields

For debug or golden traces, candidate records SHOULD include:

- canonical expression;
- operator list;
- depth;
- size;
- constants;
- prediction checksum;
- loss components;
- structural hash;
- parent lineage.

---



### 20.4 Minimal JSONL Trace Record Shapes

A concrete implementation SHOULD emit JSONL traces using stable keys. Recommended minimal run record:

```json
{
  "record_type": "run_start",
  "run_id": "...",
  "spec_version": "...",
  "compatibility_level": "SR-L1",
  "seed": "...",
  "operator_manifest_digest": "...",
  "dataset_digest": "...",
  "numeric_policy_digest": "...",
  "evaluation_backend": "numpy_vectorized"
}
```

Recommended minimal step record:

```json
{
  "record_type": "search_step",
  "t": 0,
  "rng_fingerprint": "...",
  "population_id": 0,
  "proposal_operator": "sr.mutation.replace_subtree_v1",
  "candidate_hash_after": "...",
  "validity_status": "valid",
  "loss": "0.0",
  "complexity": 3,
  "archive_update_status": "inserted"
}
```

Numerically sensitive values SHOULD be encoded as strings in canonical trace artifacts.



### 20.5 Trace Digest and Replay Token Rule

A governed run SHOULD emit trace digests so that large traces can be compared without reading every record. Minimum digests:

```text
run_header_digest
operator_manifest_digest
dataset_digest
policy_digest
step_trace_digest
archive_digest
final_result_digest
```

The `step_trace_digest` SHOULD be computed from canonical JSONL step records in deterministic order.

A replay token MUST bind at least:

```text
rng_fingerprint
iteration_index
population_id
operator_manifest_digest
policy_digest
```

If a backend cannot expose complete RNG state, it MUST declare the strongest replay level it can honestly support.

### 20.6 Trace Redaction Rule

If datasets or custom operators are sensitive, traces MAY redact raw values, but redaction MUST preserve comparability. Redacted traces MUST retain:

```text
content hashes
shape/dtype metadata
policy digests
candidate hashes
loss/complexity metrics
error codes
archive updates
```

A redacted trace MUST declare which fields were removed and what digest substitutes them.

## 21. Validation and Golden Tests

### 21.1 Required Lint Rules

An EQC-SR spec MUST fail lint if:

1. an operator has no arity;
2. an operator has no complexity cost;
3. a search-visible operator has no numeric safety policy;
4. a custom operator has no test vectors;
5. expression canonicalization is undefined;
6. structural hash is undefined;
7. invalid candidate ranking is undefined;
8. non-finite prediction policy is undefined;
9. mutation retry/failure behavior is undefined;
10. hall-of-fame duplicate behavior is undefined;
11. Pareto tie fallback is undefined when Pareto is enabled;
12. checkpoint does not include RNG and operator manifest bindings;
13. export mapping is missing for any final-output operator;
14. production backend permits pure Python hot-loop evaluation without explicit SR-L0 debug classification.

### 21.2 Required Test Families

A governed implementation SHOULD include:

- expression construction tests;
- operator test vectors;
- protected math tests;
- canonicalization tests;
- structural hash tests;
- evaluation tests;
- invalid candidate tests;
- complexity tests;
- mutation legality tests;
- crossover legality tests if crossover enabled;
- hall-of-fame tests;
- Pareto frontier tests;
- constant optimization tests if enabled;
- export equivalence tests;
- checkpoint/restore tests;
- golden trace tests.

### 21.3 Minimum Golden Problems

A first EQC-SR backend SHOULD include these golden datasets:

```text
y = x0
y = x0 + x1
y = x0 * x1
y = x0^2 + 1
y = sin(x0)
y = safe_log(abs(x0) + c)
y = x0^2 + sin(x1)
```

Each golden problem MUST declare:

- dataset generation seed;
- feature range;
- noise level;
- expected acceptable expression class;
- loss threshold;
- max complexity;
- allowed operators;
- expected reproducibility level.

---



### 21.4 Lint Severity Classes

EQC-SR lint findings SHOULD use these severities:

```text
BLOCKER: spec cannot be implemented reproducibly
MAJOR: spec is implementable but likely not comparable or safe to port
MINOR: spec is usable but missing useful validation or metadata
INFO: recommendation only
```

The following are BLOCKER findings:

- missing objective preorder;
- missing numeric policy;
- missing expression canonicalization;
- missing operator manifest;
- hidden randomness;
- missing invalid candidate policy;
- missing trace schema for governed runs;
- search-visible custom operator with side effects;
- production backend without declared evaluation backend;
- final-output operator without export behavior.

### 21.5 Reference Comparison Test Set

For a Julia-to-Python port, each subsystem extraction SHOULD define one of the following comparison targets:

```text
EXACT: output must match exactly
TOLERANCE: numeric output must match within tolerance
PROPERTY: invariants must match
DISTRIBUTIONAL: seed-set distribution must match
DOCUMENTED_DEVIATION: intentional difference recorded and tested against new rule
```

No subsystem may remain `unresolved_semantics` in a release-targeted SR-L2 or higher backend.

## 21.6 Property-Based and Metamorphic Test Requirements

EQC-SR validation SHOULD include property-based tests because symbolic regression contains too many candidate expressions to test only by examples.

Required property families for SR-RELEASE-READY scopes:

```text
TREE_ARITY_PROPERTY        every node has the declared number of children
TREE_CLOSURE_PROPERTY      mutation/crossover returns a valid expression or structured failure
EVALUATOR_SHAPE_PROPERTY   prediction shape is stable across valid inputs
FINITE_POLICY_PROPERTY     non-finite values follow the declared policy
HASH_STABILITY_PROPERTY    structural hash is stable under serialization round trip
EXPORT_ROUNDTRIP_PROPERTY  exported expression preserves predictions within tolerance
RANK_TOTALITY_PROPERTY     every candidate pair is comparable under the declared preorder
SEED_REPLAY_PROPERTY       same seed and profile produce declared replay class
```

Metamorphic tests SHOULD include:

```text
commutative_child_ordering for + and * if canonicalized
constant_serialization_roundtrip
feature_column_permutation when variable names are remapped consistently
dataset_row_order_invariance when the loss is order-independent
batch_partition_invariance when deterministic reduction policy is fixed
```

Any failed property MUST map to an EQC-SR error code and a governed subsystem.

## 21.7 Negative Test Requirements

A release-targeted backend MUST test invalid or hostile cases, including:

```text
unknown operator
operator arity mismatch
invalid operator domain
division by zero input
all-NaN candidate prediction
wrong target shape
empty dataset
single-row dataset
constant-only expression where disallowed
max complexity exceeded
invalid checkpoint manifest
unsupported PySR option
```

Each negative test MUST assert either a structured rejection or the declared penalty behavior. Silent fallback is forbidden unless explicitly declared and traced.



## 21.8 Golden Problem Acceptance Bands

Golden problems MUST avoid only exact-expression expectations unless exact recovery is the stated purpose. Each golden problem SHOULD declare one or more accepted result bands:

```text
loss_band              maximum loss threshold
complexity_band        maximum complexity threshold
semantic_family        accepted expression family
prediction_band        prediction error threshold on validation grid
operator_presence      required or forbidden operators
reproducibility_band   replay or metric equivalence expectation
```

This prevents rejecting a valid mathematically different expression while still controlling result quality.

## 21.9 Mutation Coverage Accounting

For any enabled mutation type, validation MUST include at least one test proving:

```text
valid input can produce valid output
invalid or constrained input produces structured failure or no-op
max depth/size/complexity checks are enforced
trace records identify the mutation type and result
```

A mutation type with no validation coverage MUST be disabled for release-targeted runs.

## 22. Equivalence Levels for EQC-SR

EQC-SR inherits EQC equivalence levels and specializes them.

### E0-SR Trace Equivalent

Two runs are E0-SR if iteration traces match according to the declared trace schema, including candidate hashes, choices, and archive updates.

### E1-SR Metric Equivalent

Two runs are E1-SR if final metrics match within declared tolerances:

- best loss;
- best complexity;
- prediction checksum or prediction metric;
- number of valid candidates;
- archive size;
- termination reason.

### E2-SR Distribution Equivalent

Two implementations are E2-SR if over declared seed sets they produce statistically comparable distributions of:

- best loss;
- best complexity;
- success rate on golden problems;
- runtime class if performance is part of the claim.

### E3-SR Invariant Equivalent

Two implementations are E3-SR if they preserve declared invariants but are not expected to match traces, metrics, or distributions exactly.

Default requirement for Julia-to-EQC extraction:

```text
E1-SR for extracted subsystem behavior where practical.
E2-SR for stochastic search behavior.
E3-SR only for exploratory experimental ports.
```

---

## 23. Checkpoint and Restore

A symbolic-regression checkpoint MUST include:

- current iteration;
- all population states;
- all candidates or reconstructable candidate references;
- hall of fame;
- Pareto frontier;
- RNG state;
- operator manifest;
- dataset digest;
- numeric policy digest;
- evaluation backend identity;
- simplification policy;
- constant optimization state if active;
- trace schema version;
- compatibility level;
- implementation artifact digests if bound through EQC-SIB.

Restore guarantee MUST declare one of:

- restore to E0-SR trace continuity;
- restore to E1-SR metric continuity;
- restore to E2-SR distribution continuity.

Default for early Python backend:

```text
restore to E1-SR unless full RNG and scheduler state are complete enough for E0-SR
```

---

## 24. Python Backend Performance Profile

### 24.1 Production Rules

A Python implementation targeting SR-L2 or higher SHOULD satisfy:

- vectorized candidate evaluation;
- no pure Python loop over rows inside candidate scoring;
- operator dispatch resolved before hot evaluation where possible;
- structural hashes cached;
- complexity cached;
- repeated subtree evaluation cached if enabled;
- hall-of-fame candidates reevaluated deterministically;
- deterministic parallel reduction if parallel evaluation is used.

### 24.2 Recommended Stack

Recommended but not mandatory:

```text
Python package shell
NumPy for vectorized arrays
Numba for compiled expression evaluation when operator set permits
SciPy for optional constant optimization
SymPy for symbolic export and optional simplification
pytest for tests
golden JSONL traces for reproducibility checks
```

### 24.3 Performance Metrics

Performance claims MUST specify:

- dataset size;
- number of features;
- operator set;
- population size;
- number of evaluated candidates;
- evaluation backend;
- hardware profile;
- wall-clock time;
- candidates per second;
- valid candidates per second;
- peak memory.

Performance improvements MUST not change semantic equivalence level without a declared version impact.

---



### 24.4 Minimum Performance Gate for Python Backend

A Python backend targeting SR-L2 or higher SHOULD meet this minimum practical gate on a declared CPU profile:

```text
Can evaluate at least 10,000 simple arithmetic candidates over 1,000 rows without using a pure Python row loop.
Can run the minimum golden problem suite under the declared timeout.
Can report candidates_per_second and valid_candidates_per_second.
```

The exact numeric threshold MAY be adjusted by hardware profile, but a governed implementation MUST declare the threshold before benchmarking.

### 24.5 Deterministic Parallelism Gate

If parallelism is used in production, the implementation MUST prove that the following are deterministic under the declared equivalence level:

- candidate evaluation order or equivalent ordering key;
- archive update order;
- reduction order;
- random seed assignment;
- trace record order.

If this cannot be proven, parallelism MUST be disabled for E0-SR/E1-SR validation runs.



### 24.6 Benchmark Profile Classes

Performance claims MUST declare one benchmark class:

```text
BENCH-SMOKE       tiny run proving the backend works
BENCH-DEV         local development benchmark for regressions
BENCH-RELEASE     release gate benchmark with fixed profile and thresholds
BENCH-COMPARE     comparison against reference backend or previous version
```

A release benchmark MUST pin:

```text
hardware class
Python version
dependency lock digest
thread count
BLAS/threading environment
dataset generator
operator set
search options
benchmark timeout policy
```

### 24.7 Performance Versus Quality Rule

A faster backend is not an improvement unless result quality remains within the declared equivalence target. Benchmark reports MUST include both:

```text
performance metrics
quality metrics
```

Quality metrics SHOULD include best loss, best complexity, success rate on golden problems, invalid candidate rate, and archive size.

## 25. PySR-Style Compatibility Contract

A PySR-style API compatibility document MUST declare which fields are supported.

### 25.1 Minimum SR-L3 Surface

Required minimum:

```text
fit(X, y)
predict(X)
equations_
best_expression_
best_loss_
best_complexity_
sympy()
latex()
```

### 25.2 Common Configuration Fields

A compatibility layer SHOULD map or reject explicitly:

- `niterations`;
- `population_size`;
- `populations`;
- `binary_operators`;
- `unary_operators`;
- `constraints`;
- `nested_constraints`;
- `maxsize`;
- `maxdepth`;
- `loss`;
- `elementwise_loss`;
- `parsimony`;
- `random_state`;
- `warm_start`;
- `model_selection`;
- `verbosity`.

Unsupported options MUST produce deterministic structured errors, not silent behavior changes.

### 25.3 Julia String Compatibility

If accepting PySR-style Julia operator strings, the compatibility layer MUST declare:

- parser behavior;
- accepted grammar subset;
- rejection behavior;
- mapping to Python operator registry;
- safety policy.

Default recommendation:

```text
Do not execute Julia-style strings. Parse only a strict safe subset or require Python operator registration objects.
```

---



### 25.4 Compatibility Error Policy

A compatibility layer MUST fail deterministically for unsupported PySR options. Minimum error fields:

```text
error_code
unsupported_option
received_value_type
compatibility_level
message
suggested_supported_alternative_if_any
```

Silent acceptance of an unsupported option is a compatibility bug.

### 25.5 API Stability Surface

A PySR-style compatibility document MUST define the public surface digest for:

```text
constructor parameters
fit signature
predict signature
attributes populated after fit
export methods
error classes
configuration parsing behavior
```

Any breaking change to this surface is a major compatibility change.

## 26. Julia-to-EQC Extraction Policy

When extracting behavior from `SymbolicRegression.jl`, do not translate files mechanically.

Use subsystem extraction:

```text
Julia source files
  ↓ mapped to
EQC-SR subsystem specs
  ↓ bound to
pseudocode artifacts
  ↓ implemented as
Python backend modules
```

Each extraction record MUST include:

- source repository URL;
- source commit hash;
- source file paths;
- extracted subsystem name;
- behavior summary;
- intentional deviations;
- unresolved semantics;
- equivalence target;
- test evidence.

Recommended subsystem order:

1. expression model;
2. operator registry;
3. evaluation;
4. loss and complexity;
5. random expression generation;
6. mutation;
7. hall of fame;
8. Pareto frontier;
9. main search loop;
10. constant optimization;
11. parallel/migration behavior;
12. PySR compatibility.

### 26.1 Source Crosswalk Record

Every extracted subsystem MUST have a source crosswalk record. Minimum fields:

```yaml
subsystem_id: "SR-EXTRACT-MUTATION-001"
source_repo: "https://github.com/astroautomata/SymbolicRegression.jl"
source_commit: "<full commit hash>"
source_files:
  - path: "src/..."
    role: "mutation behavior"
extraction_method: "SOURCE_EXTRACTED|REFERENCE_OBSERVED|SPEC_DEFINED|DOCUMENTED_DEVIATION"
covered_behaviors:
  - "replace subtree"
  - "operator replacement"
unresolved_semantics:
  - id: "UNRESOLVED-MUT-RETRY-ORDER"
    severity: "MAJOR"
    owner: "<name>"
    resolution_evidence_needed: "trace comparison or source confirmation"
intentional_deviations: []
equivalence_target: "E1-SR"
validation_assets:
  - "SR-GOLDEN-MUTATION-001"
```

A subsystem without a crosswalk record may be useful as a new design, but it MUST NOT be represented as a faithful extraction from the reference backend.

### 26.2 Extraction Evidence Classes

Extraction evidence SHOULD be classified as:

```text
CODE_PATH       source-code path and function/type references
RUN_TRACE       observed reference run traces
UNIT_TEST       reference tests or recreated tests
DOC_REFERENCE   public documentation or comments
INFERENCE       reasoned interpretation requiring review
DEVIATION       intentional replacement behavior
```

Release-targeted subsystem specs SHOULD avoid relying only on `INFERENCE`.

---

## 26.3 Source Extraction Work Unit

A source extraction task SHOULD be small enough to review independently. Each work unit MUST declare:

```text
work_unit_id
reference_snapshot_id
source_files
source_symbols_or_functions
subsystem_target
extraction_method
expected_output_docs
evidence_required
review_status
unresolved_semantics_added
```

Recommended work unit order for a Python-native symbolic regression port:

```text
WU-001 expression node/tree model
WU-002 operator registry and protected operators
WU-003 evaluator and non-finite behavior
WU-004 loss and complexity
WU-005 random expression generation
WU-006 mutation legality
WU-007 hall of fame / Pareto archive
WU-008 minimal population loop
WU-009 constant handling
WU-010 export and API surface
```

A work unit is complete only when the target EQC-SR documents, test vectors, and conformance matrix entries are updated.

## 26.4 Anti-Contamination Note for Reimplementation

EQC-SR does not require a legal clean-room process by itself. However, if the project wants a clean-room or semi-clean-room reimplementation claim, then source reviewers and Python implementers SHOULD be separated by role:

```text
source reviewer → writes extraction evidence and EQC-SR semantics
implementer     → implements only from EQC-SR semantics and tests
reviewer        → validates behavior through traces and conformance matrix
```

If the same person performs all roles, the project SHOULD call the result `source-guided reimplementation`, not clean-room.

## 27. Required Sidecars for an EQC-SR Portfolio

A governed EQC-SR portfolio SHOULD include:

```text
ecosystem-registry.yaml
ecosystem-graph.yaml
data-registry.yaml
sr-operator-registry.yaml
sr-expression-schema.yaml
sr-trace-schema.yaml
sr-golden-problems.yaml
sr-compatibility.yaml
sib-registry.yaml
sib-graph.yaml
sib-bindings.yaml
sib-inputs.yaml
```

### 27.1 `sr-operator-registry.yaml` Minimum Shape

```yaml
operators:
  - operator_id: "sr.math.add_v1"
    name: "add"
    version: "v1"
    arity: 2
    input_types: ["float64", "float64"]
    output_type: "float64"
    complexity_cost: 1
    search_visible: true
    purity_class: "PURE"
    determinism: "DETERMINISTIC"
    nan_inf_policy: "propagate_or_invalid_per_eval_policy"
    exports:
      sympy: "Add"
      numpy: "np.add"
    tests: []
```

### 27.2 `sr-golden-problems.yaml` Minimum Shape

```yaml
golden_problems:
  - problem_id: "SR-GOLDEN-IDENTITY-001"
    generator_seed: 1
    features: 1
    samples: 128
    target: "x0"
    allowed_operators: ["sr.math.add_v1", "sr.math.mul_v1"]
    max_complexity: 5
    success:
      max_loss: "1e-12"
      max_complexity: 3
```

---



### 27.3 Sidecar Validation Priority

For a first governed backend, sidecars SHOULD be implemented in this order:

1. `sr-operator-registry.yaml`;
2. `sr-expression-schema.yaml`;
3. `sr-golden-problems.yaml`;
4. `sr-trace-schema.yaml`;
5. `sib-bindings.yaml`;
6. `ecosystem-registry.yaml`;
7. full checkpoint and profile lock sidecars.

This order allows early functional testing without pretending that the full ecosystem tooling already exists.

### 27.4 Error Code Registry

EQC-SR implementations SHOULD maintain `sr-error-codes.yaml`. Minimum error families:

```text
EXPR_*
OPERATOR_*
DATA_*
EVAL_*
LOSS_*
MUTATION_*
CROSSOVER_*
CONSTRAINT_*
ARCHIVE_*
CONSTOPT_*
EXPORT_*
COMPAT_*
CHECKPOINT_*
```

Errors that affect candidate validity MUST be stable because they can participate in deterministic ranking and trace comparison.

### 27.5 Machine-Readable Schema Gate

For `SR-RELEASE-READY`, the following sidecars MUST have schemas or schema-equivalent validators:

```text
sr-operator-registry.yaml
sr-expression-schema.yaml
sr-trace-schema.yaml
sr-golden-problems.yaml
sr-error-codes.yaml
sib-bindings.yaml
```

Validation MUST fail if a sidecar contains unknown required-field names, missing required fields, duplicate identifiers, invalid operator references, invalid error codes, or unresolved document/artifact bindings.

### 27.6 Sidecar Identifier Stability

IDs in sidecars MUST be stable across refactors. Renaming an operator, trace field, golden problem, error code, or subsystem ID is a functional change unless the old ID is retained as a deprecated alias with a migration note.



### 27.7 Required Sidecar Cross-References

Sidecars MUST be mutually checkable. Minimum cross-reference rules:

```text
sr-golden-problems.yaml allowed_operators must resolve in sr-operator-registry.yaml
sr-trace-schema.yaml error-code fields must resolve in sr-error-codes.yaml
sr-compatibility.yaml option statuses must resolve to COMPAT_* error/warning codes where applicable
sib-bindings.yaml artifacts must resolve in sib-registry.yaml
ecosystem-graph.yaml DocIDs must resolve in ecosystem-registry.yaml
```

A sidecar that cannot be cross-validated MAY be used for drafting but MUST NOT support a release-readiness claim.

### 27.8 Minimal Schema Fragments

A first implementation SHOULD define schema fragments for these reusable objects:

```text
ExpressionNode
OperatorID
OperatorRecord
DatasetRecord
CandidateRecord
ScoreRecord
ArchiveEntry
TraceRecord
CompatibilityOptionRecord
ErrorRecord
```

These fragments SHOULD be reused across sidecars and implementation tests to prevent schema drift.

## 28. First Implementation Acceptance Criteria

A first EQC-SR-compliant Python backend is acceptable at SR-L0/SR-L1 if:

- expression trees are canonicalized;
- operator registry exists;
- `+`, `-`, `*`, protected division, `sin`, `cos`, `safe_log`, and constants are supported;
- MSE loss is implemented;
- complexity scoring is implemented;
- random expression generation is seeded;
- mutation-only search works;
- invalid candidates are handled deterministically;
- hall of fame exists;
- at least three golden problems pass;
- trace schema is emitted;
- checkpoint contains RNG state and hall of fame;
- pure Python evaluation is marked debug-only or vectorized NumPy is used.

A first SR-L2 backend is acceptable if, in addition:

- custom operators can be registered;
- custom operators require test vectors;
- custom operators export to at least Python/NumPy callable form;
- unsupported export paths fail deterministically;
- custom operator candidates can appear in the hall of fame;
- golden tests include at least one custom operator.

---



## 28.1 Release Gate Checklist

A release-targeted EQC-SR backend MUST pass these gates:

```text
[ ] All search-visible operators have schemas and test vectors.
[ ] Expression canonicalization is deterministic.
[ ] Structural hash collision policy is defined.
[ ] Dataset digest and preprocessing policy are declared.
[ ] Invalid candidate policy is deterministic.
[ ] Mutation failure behavior is deterministic.
[ ] Hall-of-fame duplicate behavior is deterministic.
[ ] Trace schema is emitted and validated.
[ ] Checkpoint includes RNG and archive state.
[ ] Golden problems pass under declared thresholds.
[ ] Unsupported compatibility options produce structured errors.
[ ] Performance profile reports candidates/second.
[ ] SIB bindings exist for governed pseudocode/code artifacts.
```

### 28.2 Minimum v0 Python Backend Module Map

Recommended initial module map:

```text
sr_backend/
  core/expression.py
  core/operators.py
  core/canonical.py
  core/errors.py
  data/dataset.py
  eval/numpy_evaluator.py
  search/random_expr.py
  search/mutation.py
  search/population.py
  search/hall_of_fame.py
  search/loop.py
  export/sympy_export.py
  compat/pysr_like.py
  tracing/jsonl_trace.py
  checkpoint/checkpoint.py
```

The module map is not mandatory, but every implementation MUST have equivalent responsibilities assigned somewhere.

### 28.3 Implementation Conformance Matrix

Every backend implementation SHOULD maintain a conformance matrix with one row per EQC-SR requirement. Minimum columns:

```text
requirement_id
source_section
status: satisfied|partial|not_applicable|missing
implementation_artifact
test_artifact
trace_artifact
equivalence_level
notes
```

A release-targeted backend MUST NOT have `missing` status for any BLOCKER requirement.

### 28.4 Risk Register for Porting

A Julia-to-Python port SHOULD maintain a risk register. Minimum risk categories:

```text
PERFORMANCE_RISK
NUMERIC_DRIFT_RISK
RANDOMNESS_DRIFT_RISK
API_COMPATIBILITY_RISK
OPERATOR_SEMANTICS_RISK
SIMPLIFICATION_DRIFT_RISK
PARALLELISM_NONDETERMINISM_RISK
```

Each active high-risk item SHOULD have a mitigation, test, and owner before SR-L2 or higher is claimed.

## 28.5 Conformance Scoring Rubric

A backend or subsystem MAY be scored out of 10 using this rubric:

| Area | Weight | Evidence |
|---|---:|---|
| Spec completeness | 1.5 | no blocker unresolved semantics, all required contracts present |
| Traceability | 1.0 | EQC-ES registry, SIB bindings, source crosswalk |
| Reproducibility | 1.0 | seed replay, numeric policy, checkpoint/restore |
| Evaluation correctness | 1.2 | evaluator tests, non-finite policy, shape/type tests |
| Search behavior | 1.2 | generation, mutation/crossover, replacement, termination tests |
| Ranking correctness | 1.0 | loss, complexity, Pareto, hall-of-fame tests |
| Export/API compatibility | 0.8 | option matrix, export equivalence, API stability surface |
| Performance readiness | 0.8 | compiled/vectorized path, benchmark gate, deterministic parallelism |
| Validation depth | 1.0 | golden, property-based, negative, reference comparison tests |
| Governance hygiene | 0.5 | versioning, risk register, sidecars, changelog |

A subsystem below 7.0 SHOULD NOT be used as a foundation for implementation. A backend below 8.5 SHOULD NOT claim release readiness. A backend below 9.2 SHOULD NOT claim broad PySR behavior comparability.

## 28.6 Milestone Exit Criteria

Recommended milestones:

```text
M0 Profile accepted
M1 Expression model + operator registry specified
M2 Evaluator + loss + complexity implemented and tested
M3 Mutation-only search discovers simple formulas
M4 Hall-of-fame/Pareto/model selection stable
M5 Custom operators and safe string grammar supported
M6 PySR-inspired API usable for common cases
M7 Reference comparison suite passes declared thresholds
M8 Release-ready governed backend
```

Each milestone MUST define entry criteria, exit criteria, responsible documents, implementation artifacts, and tests.

## 28.7 Performance Regression Policy

Performance changes are functional for release governance when they alter feasibility of the declared backend profile. A release-targeted Python backend MUST maintain a benchmark record with:

```text
benchmark_id
dataset_shape
operator_set
population_size
iterations
backend_path
hardware_profile
runtime_seconds
candidate_evaluations_per_second
peak_memory_mb
result_quality_metric
```

A performance regression greater than the declared drift budget MUST be classified as `PERFORMANCE_DRIFT` and reviewed before release.



## 28.8 Release Readiness Decision Table

A release decision SHOULD use this table rather than a single subjective score.

| Gate | Required for SR-L0/L1 | Required for SR-L2/L3 | Required for SR-L4+ | Fail Result |
|---|---|---|---|---|
| Requirement IDs and conformance matrix | SHOULD | MUST | MUST | cannot claim release-ready |
| Operator registry schema | MUST | MUST | MUST | run blocked |
| Golden problem suite | MUST | MUST | MUST | release blocked |
| Property/negative tests | SHOULD | MUST | MUST | release blocked for missing tested scope |
| PySR option matrix | optional | SHOULD if API exposed | MUST | compatibility claim blocked |
| Benchmark profile | SHOULD | MUST | MUST | performance claim blocked |
| SIB bindings | SHOULD | MUST for governed implementation | MUST | governed release blocked |
| Checkpoint/restore | SHOULD | MUST | MUST | replay claim downgraded or blocked |

## 28.9 Known Limitation Register

Every non-trivial backend SHOULD maintain a limitation register. Minimum fields:

```text
limitation_id
affected_compatibility_level
affected_subsystem
user_visible_effect
workaround_if_any
planned_resolution_or_not_in_scope
error_or_warning_code
```

Known limitations are acceptable. Hidden limitations that cause silent result changes are not.

## 28.10 Porting Stop Rule

If a subsystem repeatedly fails equivalence or reproducibility tests, the project SHOULD stop adding features to that subsystem and either:

```text
reduce compatibility claim
declare documented deviation
replace the subsystem design
return to extraction evidence collection
```

Continuing feature expansion on an unstable subsystem is a governance failure for release-targeted work.


## 28.11 Issue, Decision, and Waiver Governance

A release-targeted EQC-SR portfolio MUST track unresolved behavior as governed issues rather than prose notes.

Minimum issue record:

```yaml
issues:
  - issue_id: "SR-ISSUE-0001"
    title: "Unresolved constant optimization retry semantics"
    affected_requirements: ["SR-REQ-CONSTOPT-RETRY-v1"]
    affected_subsystems: ["constant_optimization"]
    severity: "blocker|major|minor|informational"
    status: "open|investigating|resolved|deferred|not_applicable"
    resolution_owner: "..."
    evidence_needed: ["source_audit", "reference_trace", "unit_test"]
    release_blocking_modes: ["SR-PORT-READY", "SR-RELEASE-READY"]
```

Design decisions that intentionally define behavior differently from the reference implementation MUST use an ADR-style record:

```yaml
decisions:
  - decision_id: "SR-ADR-0001"
    title: "Use structural hash, not symbolic equality, for archive duplicate detection"
    status: "accepted|superseded|rejected"
    context: "..."
    decision: "..."
    consequences: "..."
    affected_requirements: ["SR-REQ-HOF-DUPLICATE-v1"]
    equivalence_level: "E1-SR|E2-SR|E3-SR|documented_deviation"
    validation_evidence: ["TEST-HOF-DUP-001"]
```

A waiver MUST NOT be used to hide missing implementation work. A waiver is valid only when evidence demonstrates that the deviation is understood, bounded, and compatible with the declared compatibility level.

## 28.12 Implementation Handoff Packet

Before implementation begins for a subsystem, the EQC-SR portfolio SHOULD provide a handoff packet containing:

```text
1. subsystem EQC-SR spec
2. source crosswalk records
3. unresolved issue list
4. requirement index slice
5. operator registry slice
6. expected sidecars
7. golden tests and negative tests
8. trace schema
9. implementation constraints
10. performance target
```

A subsystem MUST NOT be marked `SR-PORT-READY` unless its handoff packet exists and all blocker issues are resolved or explicitly out of scope.

## 28.13 Minimal Port Prioritization Rule

When translating a large reference backend, EQC-SR work MUST prioritize semantic bottlenecks before breadth.

Default priority order:

```text
P0 expression representation, operator registry, evaluation, invalid candidate policy
P1 loss, complexity, ranking, hall-of-fame, Pareto frontier
P2 generation, mutation, replacement, termination
P3 constants and local optimization
P4 constraints, nested constraints, custom operator grammar
P5 PySR-style API and export formats
P6 parallelism, migration, distributed execution, advanced performance tuning
```

A project SHOULD NOT spend significant effort on SR-L3/SR-L4 compatibility before P0–P2 pass golden problem and negative-test gates.

## 28.14 Oracle Discipline and Tolerance Budget

Every golden problem MUST declare its oracle type:

```text
exact_equation_oracle
prediction_metric_oracle
ranking_oracle
trace_shape_oracle
statistical_distribution_oracle
reference_backend_oracle
human_review_oracle
```

Every tolerance MUST have a budget owner and rationale:

```yaml
tolerance_budget:
  metric: "mse"
  value: "1e-8"
  rationale: "float64 evaluator parity on deterministic CPU profile"
  owner: "SR-EVAL"
  applies_to: ["E1-SR"]
  expires_or_revalidate_on: ["evaluator_change", "operator_change", "dependency_change"]
```

Tolerances MUST NOT be widened to make failing tests pass unless the change is recorded as a functional decision with impact analysis.

## 28.15 Release Artifact Manifest

A release-targeted portfolio MUST list the exact artifacts that define the release:

```yaml
release_artifacts:
  specs: []
  extraction_records: []
  sidecars: []
  pseudocode: []
  implementation_artifacts: []
  tests: []
  golden_traces: []
  benchmarks: []
  checkpoints: []
```

Any artifact omitted from this manifest is not part of the release claim.


## 28.16 Validator Output Contract

An EQC-SR portfolio that claims `SR-PORT-READY` or `SR-RELEASE-READY` MUST define a validator output format.

The validator output MUST be canonical JSON and MUST include:

```json
{
  "eqc_sr_version": "v0.7.0",
  "portfolio_id": "<string>",
  "validation_profile": "<profile-id>",
  "reference_snapshot_id": "<snapshot-id|null>",
  "claim_level": "SR-DRAFT|SR-SPEC-READY|SR-PORT-READY|SR-RELEASE-READY",
  "status": "PASS|FAIL|WARN_ONLY",
  "score": {
    "overall": "0.0-10.0",
    "spec_completeness": "0.0-10.0",
    "implementation_binding": "0.0-10.0",
    "test_strength": "0.0-10.0",
    "trace_replay": "0.0-10.0",
    "performance_profile": "0.0-10.0"
  },
  "blocking_failures": [
    {
      "req_id": "SR-REQ-...",
      "code": "SR_ERR_...",
      "location": "doc-or-artifact-anchor",
      "message": "...",
      "release_impact": "BLOCKER|MAJOR|MINOR|PATCH"
    }
  ],
  "warnings": [],
  "waivers_used": [],
  "evidence_summary": {
    "requirements_total": 0,
    "requirements_validated": 0,
    "requirements_waived": 0,
    "requirements_unresolved": 0,
    "tests_total": 0,
    "golden_traces_total": 0
  },
  "artifact_manifest_digest": "sha256:<hex>",
  "trace_bundle_digest": "sha256:<hex>",
  "generated_at_policy": "fixed|recorded"
}
```

Rules:

- `status=PASS` is forbidden if any `BLOCKER` exists.
- `status=PASS` is forbidden if any release-scope `MUST` requirement lacks validation evidence.
- `WARN_ONLY` MAY be used only for `SR-DRAFT` or `SR-SPEC-READY` profiles.
- validator output ordering MUST be deterministic.
- validator output MUST NOT contain unredacted local absolute paths unless the path is explicitly declared as a reproducible workspace path.

## 28.17 Source-to-Spec Coverage Accounting

For a source-guided port, source extraction MUST produce coverage accounting that is separate from implementation test coverage.

Each referenced source file, module, or function group MUST have a coverage status:

```text
not_reviewed
reviewed_not_relevant
mapped_to_spec
mapped_to_issue
mapped_to_deviation
blocked_by_dependency
superseded_by_profile_rule
```

A source-to-spec coverage sidecar SHOULD use this form:

```yaml
source_coverage:
  - source_id: "SRJL::<commit>::src/<file>.jl::<symbol-or-region>"
    source_kind: "file|module|type|function|macro|test|doc"
    coverage_status: "mapped_to_spec"
    mapped_docs: ["SR-MUTATION-001"]
    mapped_requirements: ["SR-REQ-MUTATION-LEGALITY-v1"]
    extraction_evidence: ["EXTRACT-MUT-004"]
    unresolved_issues: []
    deviation_records: []
    reviewer: "<name-or-role>"
    reviewed_at_policy: "fixed|recorded"
```

`SR-PORT-READY` requires all source regions in declared scope to be either `mapped_to_spec`, `reviewed_not_relevant`, `mapped_to_issue`, `mapped_to_deviation`, or `superseded_by_profile_rule`. `not_reviewed` is a blocker.

## 28.18 Semantic Drift Triage Protocol

When Python behavior differs from the reference backend, the drift MUST be triaged into exactly one category:

```text
DRIFT_BUG_PYTHON_IMPL       Python implementation violates EQC-SR.
DRIFT_SPEC_GAP              EQC-SR did not specify behavior tightly enough.
DRIFT_REFERENCE_UNSTABLE    reference behavior is nondeterministic or version-sensitive.
DRIFT_INTENTIONAL_DEVIATION Python behavior intentionally differs and is documented.
DRIFT_NUMERIC_TOLERANCE     behavior differs only within declared numeric tolerance.
DRIFT_UNCLASSIFIED_BLOCKER  cannot yet classify.
```

Triage record template:

```yaml
drift_records:
  - drift_id: "DRIFT-0001"
    observed_in: "TRACE-..."
    reference_snapshot_id: "..."
    python_artifact_id: "..."
    affected_requirements: ["SR-REQ-..."]
    category: "DRIFT_SPEC_GAP"
    severity: "BLOCKER|MAJOR|MINOR|PATCH"
    equivalence_level_affected: "E0|E1|E2|E3"
    decision_required: true
    resolution: "open|fixed|waived|accepted_deviation|out_of_scope"
```

`DRIFT_UNCLASSIFIED_BLOCKER` is forbidden in `SR-RELEASE-READY` scope.

## 28.19 Deterministic Toolchain Profile for Python Backend

An EQC-SR Python backend MUST declare a deterministic toolchain profile before claiming release readiness.

Minimum declared fields:

```yaml
python_backend_profile:
  profile_id: "py-sr-cpu-reference-v1"
  python_version: "<exact>"
  package_manager: "uv|pip|conda|poetry|other"
  dependency_lock: "<path-and-digest>"
  numerical_libraries:
    numpy: "<version>"
    scipy: "<version|null>"
    numba: "<version|null>"
    sympy: "<version|null>"
  threading_policy:
    omp_num_threads: 1
    mkl_num_threads: 1
    openblas_num_threads: 1
  hash_seed: "PYTHONHASHSEED=0"
  filesystem_policy: "workspace-relative-only"
  time_policy: "fixed|recorded"
```

Rules:

- A benchmark result is invalid if it lacks the profile ID.
- A golden trace is invalid if it lacks the profile ID.
- Any change to dependency lock, numerical libraries, or threading policy triggers state revalidation.

## 28.20 Public Surface and API Stability Contract

A backend claiming any PySR-style compatibility level MUST declare a public surface manifest.

Minimum surface groups:

```text
estimator_constructor
fit_method
predict_method
equations_table
export_methods
operator_registration
loss_registration
checkpoint_methods
trace_methods
error_types
```

Each public surface entry MUST include:

```yaml
surface:
  - name: "PythonSRRegressor.fit"
    kind: "method"
    signature: "fit(X, y, *, weights=None, variable_names=None) -> self"
    compatibility_claim: "PYAPI-L1|PYAPI-L2|PYAPI-L3|PYAPI-L4"
    stability: "experimental|stable|deprecated"
    governed_by: ["SR-REQ-PYSR-FIT-v1"]
    tests: ["TEST-API-FIT-001"]
```

A breaking public-surface change requires a major compatibility bump or an explicit migration record.

## 28.21 Minimal Release Candidate Package Layout

An `SR-RELEASE-READY` candidate SHOULD publish a release package with this minimum structure:

```text
release-candidate/
  docs/
    EQC-SR.md
    subsystem-specs/
    decisions/
    issues/
  sidecars/
    requirement-index.yaml
    source-coverage.yaml
    operator-registry.yaml
    trace-schema.yaml
    compatibility-matrix.yaml
    release-artifact-manifest.yaml
  implementation/
    python-backend/
  validation/
    tests/
    golden-traces/
    benchmark-results/
    validator-output.json
  checkpoints/
    paired-checkpoint.yaml
  README.md
```

A release package MUST NOT require undeclared files outside the package except for explicitly content-addressed external dependencies.

## 28.22 Human Review Gates

Some EQC-SR claims require human review in addition to automated validation.

Human review is mandatory for:

```text
reference source extraction completeness
documented deviations from source behavior
compatibility claim level changes
waivers for release-scope MUST requirements
performance regressions accepted as intentional
security or sandbox exceptions
```

Human review record:

```yaml
review_records:
  - review_id: "REVIEW-0001"
    scope: "source_extraction|deviation|compatibility|waiver|performance|sandbox"
    reviewed_artifacts: ["..."]
    decision: "approved|rejected|needs_changes"
    rationale: "..."
    required_followups: []
    reviewer_role: "maintainer|domain-reviewer|implementation-reviewer"
```

A human review MAY NOT override a failed executable requirement unless a valid waiver exists.

## 28.23 Security and Side-Effect Boundary

Symbolic-regression systems with custom operators can accidentally execute unsafe code. EQC-SR therefore requires a side-effect boundary.

Operator implementations MUST declare one of:

```text
PURE_NUMERIC
PURE_SYMBOLIC
STATEFUL_DETERMINISTIC
IO_DECLARED
EXTERNAL_NONDETERMINISTIC_FORBIDDEN
```

Release-ready search evaluation MUST allow only `PURE_NUMERIC`, `PURE_SYMBOLIC`, or explicitly sandboxed `STATEFUL_DETERMINISTIC` operators.

Forbidden in release search evaluation unless explicitly waived and sandboxed:

```text
network access
filesystem writes
filesystem reads outside declared inputs
process spawning
reflection-based execution of untrusted strings
runtime package installation
system environment mutation
```

Custom operator strings MUST be parsed through a declared grammar or rejected. Arbitrary `eval` over untrusted operator strings is not EQC-SR compliant.

## 28.24 Deprecation and Migration Policy for SR Requirements

A requirement ID MUST NOT be reused for changed semantics.

Allowed changes:

```text
text clarification without semantic change       PATCH
new optional detail                             MINOR
new required validation evidence                MINOR or MAJOR depending on scope
changed behavior or changed pass/fail result    MAJOR
removed requirement                             MAJOR with migration note
```

Deprecated requirements MUST remain traceable for at least one major profile version and MUST include:

```yaml
deprecated_requirement:
  old_req_id: "SR-REQ-...-v1"
  replacement_req_id: "SR-REQ-...-v2|null"
  deprecation_reason: "..."
  removal_earliest_version: "vX.Y.Z"
  migration_notes: "..."
```

## 28.25 Minimal Executable Toy Problem Suite

Before implementing broad feature parity, every Python backend SHOULD pass a minimal toy problem suite.

Required toy problems:

```text
constant_only:         y = 3.0
linear_single:         y = 2*x0 + 1
linear_two_var:        y = x0 - 0.5*x1
multiplicative:        y = x0*x1
unary_sin:             y = sin(x0)
protected_division:    y = x0 / (abs(x1)+1)
custom_operator:       y = custom_square(x0) + x1
noise_robust_small:    y = x0^2 + noise
```

Each toy problem MUST declare:

```text
allowed operators
sample generation seed
data shape
noise model
expected maximum loss band
expected complexity band
minimum discovery rate over seed set
runtime budget
```

Passing the toy suite does not imply PySR parity. It only establishes that the core backend is alive and minimally useful.

## 28.26 Worked Operator Record Template

Each custom operator SHOULD be specified using this complete record shape:

```yaml
operator:
  operator_id: "SR-OP-safe_log-v1"
  name: "safe_log"
  arity: 1
  input_domain: "real_finite"
  output_domain: "real_finite"
  purity: "PURE_NUMERIC"
  deterministic: true
  numpy_definition: "log(abs(x) + EPS_DENOM)"
  sympy_definition: "log(Abs(x) + EPS_DENOM)"
  numba_eligible: true
  differentiability: "almost_everywhere|none|smooth|piecewise"
  protected_math_policy: "never_nan_for_finite_input"
  complexity_cost: 2
  canonicalization_rule: "safe_log(x) preserved; do not rewrite to log(abs(x)+eps) unless export profile requests expansion"
  edge_cases:
    - input: "x=0"
      expected: "log(EPS_DENOM)"
    - input: "x=NaN"
      expected: "invalid_input_policy"
  tests:
    unit: ["TEST-OP-SAFE-LOG-001"]
    property: ["TEST-OP-SAFE-LOG-FINITE-001"]
  governed_by: ["SR-REQ-OP-REGISTRY-v1"]
```

## 28.27 Worked Trace Record Template

A minimal search trace record SHOULD follow this shape:

```json
{
  "trace_schema_version": "sr-trace-v1",
  "run_id": "RUN-...",
  "iteration": 17,
  "rng_fingerprint_before": "...",
  "rng_fingerprint_after": "...",
  "population_id": 0,
  "event": "mutation_evaluated",
  "parent_expr_id": "EXPR-...",
  "candidate_expr_id": "EXPR-...",
  "mutation_type": "replace_subtree",
  "candidate_complexity": 5,
  "candidate_loss": 0.0123,
  "candidate_score": 0.0173,
  "accepted": true,
  "archive_update": "inserted|rejected|replaced|none",
  "failure_code": null
}
```

Trace fields MAY be extended, but required fields MUST remain stable within a trace schema major version.


## 28.28 Subsystem Dependency Graph Contract

A symbolic-regression port MUST maintain a subsystem dependency graph before claiming `SR-PORT-READY`.

The graph MUST include at least these node classes:

```text
expression_model
operator_registry
evaluator
loss_complexity_ranking
random_generation
mutation
crossover
population_loop
archive_hall_of_fame
pareto_frontier
constant_optimization
constraint_engine
simplification_canonicalization
export_layer
trace_checkpoint
pysr_api_adapter
```

Each edge MUST declare one of:

```text
SEMANTIC_DEPENDENCY     output meaning depends on upstream semantics
DATA_DEPENDENCY         runtime data structure depends on upstream artifact
VALIDATION_DEPENDENCY   tests/traces depend on upstream artifact
PERFORMANCE_DEPENDENCY  performance claim depends on upstream implementation
API_DEPENDENCY          public surface depends on upstream artifact
```

A subsystem MUST NOT be marked `implementation_complete` unless all upstream `SEMANTIC_DEPENDENCY` and `DATA_DEPENDENCY` nodes are at least `SR-PORT-READY`, or an explicit temporary shim is declared.

Required graph sidecar:

```yaml
sr_subsystem_graph:
  version: "sr-subsystem-graph-v1"
  nodes:
    - id: "evaluator"
      class: "evaluator"
      compliance_mode: "SR-PORT-READY"
      owner: "core-runtime"
  edges:
    - src: "operator_registry"
      dst: "evaluator"
      kind: "SEMANTIC_DEPENDENCY"
      reason: "evaluator resolves operator arity, domain, and numeric behavior from registry"
```

## 28.29 Invariant Catalog Contract

Every release-scoped subsystem MUST publish an invariant catalog.

Each invariant MUST have:

```text
invariant_id
subsystem
statement
scope
severity
checked_by
failure_code
release_impact
```

Minimum required invariant categories:

```text
expression_structure
operator_arity
finite_output_policy
complexity_nonnegative
archive_immutability
pareto_non_dominance
rng_threading
trace_append_only
checkpoint_restorability
public_api_stability
```

Example:

```yaml
invariants:
  - invariant_id: "SR-INV-EXPR-ARITY-v1"
    subsystem: "expression_model"
    statement: "Every operator node has exactly the number of children declared by the operator registry."
    scope: "all_candidate_expressions"
    severity: "blocker"
    checked_by: ["TEST-EXPR-ARITY-001", "LINT-EXPR-STRUCTURE-001"]
    failure_code: "SR_ERR_EXPR_ARITY"
    release_impact: "blocks_SR_PORT_READY"
```

A subsystem with missing invariant coverage MUST NOT be marked `SR-RELEASE-READY`.

## 28.30 Validator Report Schema

An EQC-SR validator SHOULD emit a canonical JSON report. A portfolio claiming `SR-RELEASE-READY` MUST define this schema even if the first validator is manual or script-based.

Minimum report shape:

```json
{
  "eqc_sr_version": "v0.8.0",
  "portfolio_id": "SR-PORT",
  "run_id": "RUN-...",
  "reference_snapshot": {
    "repo": "SymbolicRegression.jl",
    "commit": "<full_commit_hash>",
    "source_digest": "<sha256>"
  },
  "status": "PASS|FAIL|INCOMPLETE",
  "compliance_mode_checked": "SR-DRAFT|SR-SPEC-READY|SR-PORT-READY|SR-RELEASE-READY",
  "requirements": [
    {
      "req_id": "SR-REQ-...",
      "status": "pass|fail|waived|not_applicable|not_checked",
      "evidence": ["TEST-...", "TRACE-..."],
      "failure_code": null
    }
  ],
  "subsystems": [
    {
      "id": "evaluator",
      "status": "pass|fail|incomplete",
      "coverage": {
        "source_to_spec": 0.92,
        "requirement_to_test": 1.0,
        "invariant_to_check": 1.0
      }
    }
  ],
  "errors": [],
  "warnings": [],
  "waivers_used": [],
  "artifact_manifest_digest": "<sha256>"
}
```

Validator outputs MUST be deterministic: arrays sorted by stable identifiers, no wall-clock timestamps except recorded run metadata, and no environment-dependent field order.

## 28.31 Deviation Budget and Drift Ledger

Every intentional difference from the reference backend MUST be tracked in a drift ledger.

A deviation record MUST contain:

```text
deviation_id
reference_behavior
new_behavior
reason
affected_subsystems
expected_user_visible_effect
equivalence_level
validation_evidence
approval_status
release_impact
```

Deviation classes:

```text
BUG_COMPATIBLE        preserves known reference bug for compatibility
BUG_FIXED             intentionally fixes known reference issue
PYTHONIC_REDESIGN     same semantics, different implementation structure
PERFORMANCE_VARIANT   different internal path, same accepted outputs within tolerance
SCOPE_DEFERRED        reference feature intentionally not supported yet
SEMANTIC_CHANGE       behavior differs and must be disclosed in compatibility claims
```

A backend MUST NOT claim drop-in PySR compatibility while any release-scoped deviation is classified as `SEMANTIC_CHANGE` or `SCOPE_DEFERRED` for a PySR option included in the claim.

## 28.32 Operator DSL and Unsafe Code Policy

If the Python backend accepts user-provided operator definitions as strings, the string grammar MUST be treated as a governed DSL, not as unrestricted Python execution.

Allowed approaches:

```text
DECLARED_CALLABLE_ONLY       user registers Python callables through a typed registry
SAFE_EXPRESSION_DSL          parser accepts a restricted expression grammar
REFERENCE_COMPAT_STRING      PySR-style strings are parsed and translated through a safe grammar
UNSAFE_EVAL_FORBIDDEN        eval/exec is forbidden in release mode
```

Release mode MUST NOT use unrestricted `eval`, `exec`, dynamic imports, filesystem access, network access, or mutation of global state for operator registration.

Each custom operator MUST declare:

```text
side_effect_free: true
finite_policy
arity
shape_policy
sympy_export
compiled_backend_eligibility
security_class
```

If a custom operator cannot be safely compiled or inspected, the backend MAY run it only in an explicitly declared fallback mode with a degraded performance claim.

## 28.33 Reproducible Benchmark Harness Contract

Performance claims MUST be tied to a benchmark harness, not informal timing.

Each benchmark MUST declare:

```text
benchmark_id
dataset_id
operator_set
n_samples
n_features
n_iterations
population_size
max_complexity
seed_set
hardware_profile
backend_mode
warmup_policy
measurement_policy
acceptance_threshold
```

Benchmark classes:

```text
SMOKE_PERF       completes quickly; validates no catastrophic slowdown
CORE_PERF        validates evaluator/search performance on normal workloads
STRESS_PERF      validates scaling and memory behavior
REFERENCE_COMP   compares Python backend against reference backend where applicable
```

Timing results MUST report at least:

```text
median_runtime
p95_runtime
peak_memory
expressions_evaluated_per_second
best_loss_distribution
successful_runs
failed_runs
```

A performance improvement MUST NOT be accepted if it violates trace, ranking, candidate validity, or checkpoint invariants.

## 28.34 Implementation Handoff Acceptance Checklist

Before a subsystem moves from specification to implementation, it MUST pass an implementation handoff checklist.

Checklist:

```yaml
handoff:
  subsystem_id: "mutation"
  spec_doc: "SR-MUTATION-001"
  compliance_mode: "SR-PORT-READY"
  source_crosswalk_complete: true
  unresolved_blockers: []
  requirement_index_complete: true
  invariant_catalog_complete: true
  operator_dependencies_resolved: true
  trace_fields_declared: true
  tests_declared: true
  golden_oracle_declared: true
  performance_expectation_declared: true
  public_surface_declared: true
  checkpoint_effect_declared: true
  accepted_by: "implementation-owner"
```

A subsystem without a completed handoff checklist SHOULD remain in spec work and SHOULD NOT be implemented as release code.

## 28.35 Release Candidate Assembly Rule

An EQC-SR release candidate MUST be assembled from exact artifacts, not from “latest” files.

The release candidate manifest MUST include:

```text
all EQC-SR documents and digests
all extraction evidence records and digests
all sidecars and schemas
all implementation artifacts and digests
all tests and golden traces
all benchmark reports
all waivers and decisions
all known limitations
all public compatibility claims
```

The release candidate MUST declare one of:

```text
RC_SPEC_ONLY        governed documents only, no backend release
RC_SUBSYSTEM        one or more governed subsystems
RC_BACKEND_CORE     Python backend core without PySR-style compatibility claim
RC_API_COMPAT       backend plus declared PySR-style compatibility level
```

A release candidate MUST NOT mix unpinned draft files with release-scoped artifacts.



## 28.36 Semantic Surface Freeze Rule

A subsystem approaching `SR-RELEASE-READY` MUST declare a frozen semantic surface.

The semantic surface is the set of observable behaviors that users, tests, traces, downstream specs, or compatibility claims may rely on.

At minimum, a semantic surface declaration MUST include:

```yaml
semantic_surface:
  subsystem_id: "SR-EVAL-001"
  version: "v0.1.0"
  frozen: true
  public_behaviors:
    - behavior_id: "EVAL-SHAPE"
      description: "Evaluator returns one prediction per input row."
      requirement_ids: ["SR-REQ-EVAL-SHAPE-v1"]
    - behavior_id: "INVALID-CANDIDATE-POLICY"
      description: "Invalid numeric output is converted to declared invalid objective."
      requirement_ids: ["SR-REQ-EVAL-INVALID-v1"]
  public_inputs:
    - "ExpressionTree"
    - "DatasetBatch"
    - "OperatorRegistry"
  public_outputs:
    - "PredictionVector"
    - "EvaluationStatus"
    - "EvaluationDiagnostics"
  trace_keys:
    - "candidate_hash"
    - "evaluation_status"
    - "nan_count"
    - "inf_count"
  compatibility_level: "SR-L1"
  freeze_scope: "behavior|api|trace|sidecar|all"
```

After freeze, any change to the declared surface MUST be classified as one of:

```text
PATCH_CLARIFICATION       wording only; no behavior change
MINOR_ADDITION            additive behavior or trace key, old behavior preserved
MAJOR_BREAKING_CHANGE     behavior, trace, public API, or sidecar contract changed
DEPRECATION_WITH_SHIM     old behavior preserved through compatibility layer
```

A frozen semantic surface MUST NOT be changed by implementation convenience. Performance improvements are allowed only if they preserve the declared equivalence level and pass the differential, golden, and property tests attached to the surface.

## 28.37 Reference Differential Testing Protocol

If a subsystem claims compatibility with a reference backend such as `SymbolicRegression.jl`, it MUST include a differential testing protocol.

Differential testing compares the governed implementation against either:

```text
REFERENCE_RUNTIME_OUTPUT      outputs from running the reference backend
REFERENCE_TRACE_OUTPUT        traces generated from instrumented reference runs
REFERENCE_SPEC_OUTPUT         behavior extracted into EQC-SR and treated as authoritative
```

A differential test record MUST include:

```yaml
differential_test:
  test_id: "DIFF-EVAL-001"
  subsystem_id: "SR-EVAL-001"
  reference_kind: "REFERENCE_RUNTIME_OUTPUT"
  reference_snapshot_id: "SRJL-<commit-sha>"
  implementation_artifact: "IMPL-EVAL-PY"
  dataset_id: "DATA-TOY-POLY-001"
  operator_set_id: "OPS-BASIC-001"
  seed_set: [0, 1, 2, 3, 4]
  compared_outputs:
    - "prediction_vector"
    - "loss"
    - "invalid_status"
  equivalence_level: "E1"
  tolerance_budget_id: "TOL-EVAL-001"
  failure_classification:
    on_reference_error: "mark_inconclusive"
    on_impl_error: "fail"
    on_both_error: "compare_error_codes"
```

Differential tests MUST separate:

```text
reference disagreement caused by unresolved source behavior
reference disagreement caused by intentional documented deviation
reference disagreement caused by implementation bug
reference disagreement caused by tolerance budget insufficiency
reference disagreement caused by nondeterministic environment
```

A subsystem MUST NOT use a reference differential pass as its only evidence. It must also include direct EQC-SR conformance tests, because the reference backend may contain behavior that the profile intentionally does not adopt.

## 28.38 Candidate Corpus and Benchmark Dataset Contract

EQC-SR portfolios SHOULD maintain a fixed candidate corpus for evaluator, simplifier, canonicalizer, archive, and export testing.

A candidate corpus is different from a training dataset. It is a governed set of expressions designed to exercise semantic edge cases.

A candidate corpus record MUST include:

```yaml
candidate_corpus:
  corpus_id: "CORPUS-SR-CORE-001"
  version: "v0.1.0"
  purpose: "Core expression edge cases for evaluator and exporter"
  expression_format: "canonical_tree_json"
  operator_sets: ["OPS-BASIC-001", "OPS-PROTECTED-001"]
  cases:
    - case_id: "CONST-ONLY-001"
      expression: {kind: "constant", value: "1.0"}
      expected_properties: ["finite", "shape_preserving"]
    - case_id: "PROTECTED-DIV-ZERO-001"
      expression:
        kind: "binary_op"
        op: "protected_div_v1"
        left: {kind: "variable", index: 0}
        right: {kind: "constant", value: "0.0"}
      expected_properties: ["finite_or_declared_invalid", "no_crash"]
```

A release candidate MUST run at least these corpus groups:

```text
constant_only_expressions
single_variable_expressions
deep_unary_chains
commutative_binary_trees
protected_math_edge_cases
invalid_candidate_cases
duplicate_structure_cases
constant_identity_cases
export_roundtrip_cases
constraint_violation_cases
```

Corpus updates are validation asset changes unless they alter the semantic requirement they test. If a corpus change changes expected behavior, it is a functional spec change and MUST trigger impact propagation.

## 28.39 Feature Flag and Experimental Isolation Policy

Experimental symbolic-regression capabilities MUST be isolated behind declared feature flags.

Feature flags are mandatory for:

```text
new search strategies
new mutation families
new crossover families
new optimizer strategies
new simplification rules
new cache semantics
new evaluator backends
new custom operator execution modes
new PySR compatibility options
```

A feature flag record MUST include:

```yaml
feature_flag:
  flag_id: "SR-FEAT-CROSSOVER-001"
  name: "enable_crossover_v1"
  default: false
  status: "experimental|candidate|stable|deprecated"
  affected_subsystems: ["SR-CROSSOVER-001", "SR-SEARCH-001"]
  guarded_requirements: ["SR-REQ-XOVER-VALID-CHILD-v1"]
  trace_keys:
    - "feature_flags.enable_crossover_v1"
  validation_required:
    - "property_test"
    - "golden_trace"
    - "benchmark"
  promotion_rule: "may_promote_only_after_no_regression_on_core_suite"
```

A release candidate MUST declare all enabled feature flags in the release artifact manifest. Undeclared experimental behavior is a blocker for `SR-RELEASE-READY`.

## 28.40 Failure Injection and Recovery Validation

EQC-SR implementations MUST test failure paths, not only successful runs.

At minimum, a release candidate MUST include failure-injection cases for:

```text
invalid operator registration
operator arity mismatch
operator returns wrong shape
operator returns NaN/Inf where not allowed
dataset contains non-finite values
mutation cannot produce a valid child
constraint set rejects all candidates
constant optimizer fails
checkpoint restore has incompatible manifest
trace schema version mismatch
cache digest mismatch
unsupported export target
```

A failure-injection record MUST include:

```yaml
failure_injection_test:
  test_id: "FAIL-OP-ARITY-001"
  injected_failure: "operator_arity_mismatch"
  expected_error_code: "SR_ERR_OPERATOR_ARITY_MISMATCH"
  expected_recovery: "reject_operator_registration"
  trace_required: true
  state_mutation_allowed: false
  checkpoint_allowed_after_failure: false
```

Failure tests MUST verify:

```text
error code is deterministic
failure trace is emitted when required
persistent state is unchanged unless recovery declares otherwise
RNG state handling is declared
no partial archive update occurs after rejected candidate
```

## 28.41 Numerical Envelope and Resource Budget Contract

A Python backend claiming practical usefulness MUST declare numerical and resource envelopes.

A numerical envelope defines where the implementation is expected to behave reliably:

```yaml
numerical_envelope:
  envelope_id: "NUMENV-CORE-001"
  dtype: "float64"
  finite_input_range: [-1.0e6, 1.0e6]
  protected_eps:
    EPS_DENOM: "1.0e-12"
    EPS_LOG: "1.0e-12"
  max_abs_prediction_before_invalid: "1.0e100"
  invalid_on_nan: true
  invalid_on_inf: true
  overflow_policy: "mark_candidate_invalid"
```

A resource budget defines acceptable runtime and memory behavior for validation profiles:

```yaml
resource_budget:
  budget_id: "BUDGET-CPU-SMOKE-001"
  profile: "cpu-local-smoke"
  max_wall_seconds: 60
  max_memory_mb: 1024
  max_dataset_rows: 10000
  max_candidate_evaluations: 50000
  backend_allowed: ["numpy", "numba"]
  pure_python_hot_loop_allowed: false
```

Performance claims MUST NOT be generic. They MUST be tied to:

```text
hardware profile
dataset size
operator set
candidate count
iteration count
backend evaluator
feature flags
```

A regression is blocking if it violates a declared resource budget for a release profile, even if correctness tests pass.

## 28.42 Pre-1.0 Stabilization Rule

Before EQC-SR reaches `v1.0.0`, each new draft SHOULD reduce ambiguity or execution risk. It SHOULD NOT add broad new conceptual scope unless the missing scope blocks implementation.

A pre-1.0 draft MUST classify each addition as:

```text
SEMANTIC_CLARIFICATION
VALIDATION_STRENGTHENING
IMPLEMENTATION_HANDOFF
COMPATIBILITY_DISCIPLINE
PERFORMANCE_DISCIPLINE
SECURITY_DISCIPLINE
GOVERNANCE_ALIGNMENT
```

A draft SHOULD be considered ready for `v1.0.0` when:

```text
all release-relevant MUST requirements have stable IDs
all MUST requirements have validation binding types
the minimal Python backend scope is implementable without unresolved blocker semantics
the sidecar set is complete enough for validator implementation
the compatibility labels prevent overclaiming
the benchmark and golden problem suites are defined
the release candidate assembly rule is actionable
```


## 28.43 v1.0 Semantic Closure Rule

`EQC-SR-v1.0.0` is the first stabilization target for this profile. A portfolio claiming conformance to `EQC-SR-v1.0.0` MUST treat the following as closed semantic surfaces unless a MAJOR version update is issued:

```text
expression node identity and validity semantics
operator registry required fields
protected math declaration requirements
evaluation output shape and invalid-value policy
loss, complexity, ranking, Pareto, and hall-of-fame contracts
mutation/crossover legality contracts
trace schema minimum fields
sidecar identity and cross-reference rules
compatibility claim labels
release artifact manifest rules
```

A later change MAY refine examples, add optional fields, or add stricter validation recommendations, but it MUST NOT silently change the meaning of the closed surfaces above.

### Required closure record

A release package MUST include:

```yaml
semantic_closure:
  eqc_sr_version: "1.0.0"
  closed_surfaces:
    - "expression_node_identity"
    - "operator_registry"
    - "evaluation_contract"
    - "loss_complexity_ranking"
    - "pareto_hof"
    - "trace_schema_minimum"
    - "compatibility_claim_labels"
  exceptions: []
  exception_records: []
```

If exceptions exist, the release MUST NOT claim full `EQC-SR-v1.0.0` conformance. It MAY claim `EQC-SR-v1.0.0-partial` with listed exceptions.

## 28.44 Conformance Claim Grammar

A conformance claim MUST use one of the following forms exactly:

```text
EQC-SR-v1.0.0/SR-DRAFT
EQC-SR-v1.0.0/SR-SPEC-READY
EQC-SR-v1.0.0/SR-PORT-READY
EQC-SR-v1.0.0/SR-RELEASE-READY
EQC-SR-v1.0.0-partial/<mode>/<exception-list-id>
```

A claim MUST bind to:

```text
release artifact manifest
semantic closure record
requirement index
validator report
sidecar validation report
trace bundle digest
benchmark bundle digest, if performance claims are made
compatibility matrix, if PySR-style compatibility claims are made
```

Marketing or README language MUST NOT use vague phrases such as:

```text
fully compatible
same as PySR
drop-in replacement
complete port
production-ready
```

unless the corresponding compatibility and release-readiness evidence is present.

## 28.45 Maintainer Decision Boundary

When source behavior is unclear, conflicting, unstable, or too implementation-specific to port directly, the maintainer MUST choose exactly one decision type:

```text
PORT_AS_REFERENCE
SPECIFY_CLEAN_EQC_BEHAVIOR
DOCUMENTED_DEVIATION
DEFER_OUT_OF_SCOPE
BLOCK_RELEASE_PENDING_EVIDENCE
```

The decision record MUST include:

```yaml
decision_id: "SR-ADR-..."
subsystem: "..."
source_behavior: "observed|source_extracted|unknown|conflicting"
decision_type: "PORT_AS_REFERENCE|SPECIFY_CLEAN_EQC_BEHAVIOR|DOCUMENTED_DEVIATION|DEFER_OUT_OF_SCOPE|BLOCK_RELEASE_PENDING_EVIDENCE"
rationale: "..."
validation_required: ["..."]
compatibility_effect: "none|minor|major|unknown"
release_effect: "non_blocking|blocks_port_ready|blocks_release_ready"
```

A release MUST NOT contain undocumented maintainer decisions that affect behavior.

## 28.46 Regression Triage Matrix

All regressions detected during validation MUST be classified using this matrix:

| Regression Type | Examples | Default Severity | Blocks Release? |
|---|---|---:|---|
| `SEMANTIC` | changed result ordering, invalid expression accepted | Critical | Yes |
| `TRACE` | missing replay token, changed required trace key | Critical | Yes |
| `NUMERIC` | outside declared tolerance, unstable NaN handling | High/Critical | Yes if outside envelope |
| `PERFORMANCE` | exceeds runtime or memory budget | High | Yes for performance claim |
| `COMPATIBILITY` | PySR-style option misreported | High | Yes for affected compatibility level |
| `SECURITY` | unsafe custom operator side effect | Critical | Yes |
| `GOVERNANCE` | missing requirement/test binding | High/Critical | Yes for release scope |
| `DOCUMENTATION` | unclear example or typo | Low/Medium | No unless it changes meaning |

A regression report MUST include:

```yaml
regression_id: "SR-REG-..."
type: "SEMANTIC|TRACE|NUMERIC|PERFORMANCE|COMPATIBILITY|SECURITY|GOVERNANCE|DOCUMENTATION"
severity: "low|medium|high|critical"
affected_requirements: ["SR-REQ-..."]
affected_artifacts: ["..."]
first_detected_in: "..."
evidence: ["test", "trace", "benchmark", "review"]
release_blocker: true
resolution_status: "open|fixed|waived|deferred"
```

## 28.47 Minimal Validator CLI Contract

A portfolio that claims `SR-PORT-READY` or higher SHOULD provide an executable validator. A portfolio that claims `SR-RELEASE-READY` MUST provide one.

Minimum commands:

```text
eqc-sr validate --profile <profile> --out <validator-report.json>
eqc-sr requirements --out <requirement-index.json>
eqc-sr coverage --out <coverage-report.json>
eqc-sr traces --run <suite> --out <trace-report.json>
eqc-sr package --check <release-manifest.yaml>
```

The validator MUST return:

```text
0 = pass
1 = validation failed
2 = validator/tooling error
```

The validator MUST NOT silently skip missing sidecars, missing traces, missing requirement IDs, or missing evidence bindings.

## 28.48 Porting Work Package Definition

Every subsystem implementation task SHOULD be packaged as a porting work package.

```yaml
work_package_id: "SR-WP-..."
subsystem: "expression_tree|operator_registry|evaluation|loss_complexity|mutation|hof|pareto|constants|export|api"
source_scope:
  reference_snapshot_id: "..."
  source_files: ["..."]
eqc_docs: ["..."]
requirements: ["SR-REQ-..."]
implementation_artifacts: ["..."]
validation_artifacts: ["..."]
entry_criteria:
  - "source crosswalk complete"
  - "blocker unresolved semantics cleared"
exit_criteria:
  - "unit/property/metamorphic tests pass"
  - "trace schema validates"
  - "requirement evidence bindings complete"
status: "not_started|extracting|specified|implementing|validating|done|blocked"
```

A work package MUST NOT move to `done` unless its governed requirements are either validated, explicitly not applicable, or waived with evidence.

## 28.49 Minimal 1.0 Release Bundle

A minimally acceptable `EQC-SR-v1.0.0` governed release bundle contains:

```text
/docs
  EQC-SR.md
  subsystem/*.md
/sidecars
  requirement-index.yaml
  sr-registry.yaml
  sr-graph.yaml
  sr-compatibility.yaml
  sr-error-codes.yaml
  semantic-closure.yaml
  release-artifact-manifest.yaml
  validator-report.schema.json
  trace-record.schema.json
/evidence
  source-crosswalk.yaml
  unresolved-semantics.yaml
  decision-log.yaml
  deviation-ledger.yaml
  regression-ledger.yaml
/tests
  golden-problems.yaml
  candidate-corpus.yaml
  property-tests.md
  metamorphic-tests.md
  negative-tests.md
/traces
  golden/*.jsonl
/benchmarks
  benchmark-profile.yaml
  benchmark-results.json
```

If the release is specification-only and does not include implementation artifacts, the manifest MUST explicitly mark implementation fields as `not_included_spec_only_release`.

## 28.50 Post-1.0 Change Control

After `EQC-SR-v1.0.0`, changes MUST follow this rule:

```text
PATCH: clarify wording, fix examples, add non-normative guidance, correct typos.
MINOR: add optional fields, optional sidecars, optional validation checks, or new compatibility levels without changing existing semantics.
MAJOR: change required semantics, required trace fields, required sidecars, compatibility claim meaning, or conformance mode criteria.
```

A post-1.0 change MUST include a migration note when it affects:

```text
requirement IDs
trace schema
operator registry fields
sidecar schemas
compatibility labels
release-readiness gates
validator output schema
```

A portfolio MAY continue using an older EQC-SR major version, but compatibility claims MUST identify the exact version used.


## 28.51 Finalization Audit Contract

A portfolio MUST NOT mark an EQC-SR release as final unless a finalization audit record exists.

The finalization audit record MUST answer every item below with `pass`, `fail`, `not_applicable_with_reason`, or `waived_with_evidence`:

```yaml
finalization_audit:
  eqc_sr_version: "1.0.0"
  audit_id: "SR-FINAL-AUDIT-..."
  semantic_closure_record_present: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  requirement_index_complete: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  all_must_requirements_bound_to_evidence: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  sidecar_schemas_present: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  validator_report_present: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  source_crosswalk_present: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  unresolved_blockers_absent: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  compatibility_claims_checked: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  release_artifact_manifest_checked: "pass|fail|not_applicable_with_reason|waived_with_evidence"
  reviewer: "..."
  reviewed_at: "YYYY-MM-DD"
  decision: "final|not_final|final_with_waivers"
```

A `final_with_waivers` decision MUST NOT be used for missing semantic closure, missing validator report, missing requirement index, or unresolved blocker semantics in the claimed scope.

## 28.52 Normative Glossary and Term Consistency Rule

Every EQC-SR portfolio SHOULD include a glossary sidecar for terms that affect behavior.

The following terms are normative in this profile and MUST NOT be redefined incompatibly in subsystem documents:

| Term | Meaning |
|---|---|
| candidate expression | A structured expression object being evaluated or transformed by the search process. |
| expression identity | The rule used to decide whether two expressions are the same for hashing, caching, lineage, or archive purposes. |
| semantic equivalence | A declared equivalence relation over expression behavior, never assumed from structural similarity alone. |
| protected operator | An operator with explicit handling for invalid domains, overflow, underflow, NaN, Inf, or denominator hazards. |
| invalid candidate | A candidate expression that violates syntax, arity, type, domain, resource, constraint, or numerical policy. |
| hall of fame | A governed archive of selected candidate expressions retained across search iterations. |
| Pareto frontier | A governed non-dominated set under declared objective components and tie-breaking rules. |
| compatibility claim | A public or internal statement about what external API or reference behavior is supported. |
| release scope | The exact subsystem, API surface, dataset, trace suite, benchmark profile, and sidecar set covered by a release claim. |

If a subsystem needs a narrower term, it MUST introduce a new term rather than overloading a normative term.

## 28.53 Spec-Only Versus Implementation-Included Release Claims

A release MUST declare whether it is:

```text
SPEC_ONLY
PSEUDOCODE_INCLUDED
REFERENCE_IMPLEMENTATION_INCLUDED
PRODUCTION_IMPLEMENTATION_INCLUDED
```

A `SPEC_ONLY` release MAY claim `SR-SPEC-READY`, but MUST NOT claim `SR-PORT-READY` or `SR-RELEASE-READY` for an implementation.

A `PSEUDOCODE_INCLUDED` release MAY claim that governed pseudocode is ready for implementation only if:

```text
all pseudocode procedures are bound to EQC-SR subsystem documents
all operator calls resolve to a manifest entry
all unresolved semantics are non-blocking for the declared pseudocode scope
```

An implementation-included release MUST include SIB bindings, implementation artifact digests, trace outputs, and validator outputs for the claimed implementation scope.

## 28.54 Minimal Human Reviewer Checklist

Before accepting an EQC-SR subsystem as complete, a reviewer MUST check:

```text
1. Does every public behavior have a requirement ID?
2. Does every MUST requirement have evidence?
3. Are unresolved semantics explicitly listed rather than hidden in prose?
4. Are source-extracted behaviors separated from clean EQC decisions?
5. Are invalid expressions handled deterministically?
6. Are random choices localized and replayable?
7. Are archive and Pareto update rules deterministic?
8. Are compatibility claims limited to the tested scope?
9. Are custom operators safe, pure, and exportable under declared rules?
10. Are performance claims tied to a declared benchmark profile?
```

A reviewer MUST NOT approve a subsystem by readability alone. Approval requires evidence alignment.

## 28.55 Scope Lock and Future Work Parking Lot

After `EQC-SR-v1.0.0`, new ideas that are not required for the declared release scope SHOULD be placed in a future-work parking lot rather than added to the normative profile.

Future-work records SHOULD use:

```yaml
future_work:
  item_id: "SR-FW-..."
  title: "..."
  category: "algorithmic|compatibility|performance|tooling|documentation|security|research"
  reason_not_in_v1: "..."
  dependency: "..."
  proposed_target: "v1.1|v2.0|unscheduled"
  release_blocker: false
```

A future-work item MAY become a release blocker only if it exposes a correctness, safety, traceability, compatibility, or governance failure in the current release scope.




## 28.56 Final Errata and Patch Amendment Rule

After `EQC-SR-v1.0.0`, changes to the profile MUST be classified as one of:

```text
ERRATA_ONLY          wording, numbering, spelling, broken reference repair
CLARIFICATION_PATCH  resolves ambiguity without changing required behavior
NORMATIVE_PATCH      changes a requirement but remains backward compatible
MINOR_EXTENSION      adds new optional capability or sidecar without affecting existing compliance
MAJOR_REVISION       changes compliance meaning, trace meaning, or release claim meaning
```

A final-stabilization release such as `v1.0.1` MUST NOT add broad new scope. It MAY add errata handling, missing validation linkage, claim wording precision, or auditability improvements.

Patch releases MUST include a patch record:

```yaml
patch_record:
  profile_version: "EQC-SR-v1.0.3"
  patch_class: "CLARIFICATION_PATCH"
  affected_sections: ["28.56", "28.57", "28.58", "28.59", "28.60"]
  behavior_change: false
  compatibility_effect: "no_change_to_v1_0_compliance_claims"
  requires_revalidation: false
  rationale: "close final audit and errata handling gaps"
```

A patch record is a blocker for release packaging when the profile version is greater than `v1.0.0`.

## 28.57 Normative Reference Lockfile Rule

A release package SHOULD include a normative reference lockfile so that the governing standards are not guessed from filenames or prose.

Minimum shape:

```yaml
sr_normative_refs:
  eqc_sr:
    version: "v1.0.3"
    path: "EQC-SR-v1.0.3-final.md"
    digest_sha256: "<sha256>"
  eqc:
    version: "v1.1"
    path: "EQC.md"
    digest_sha256: "<sha256>"
  eqc_es:
    version: "v1.9.1"
    path: "ECOSYSTEM.md"
    digest_sha256: "<sha256>"
  eqc_sib:
    version: "v1.2.1"
    path: "BRIDGE.md"
    digest_sha256: "<sha256>"
```

If a concrete portfolio uses different governing versions, it MUST declare the difference and the conflict-resolution status from Section 0.3.1.

## 28.58 Acceptance Evidence Table Rule

Each release claim MUST include a compact acceptance evidence table in human-readable Markdown and machine-readable sidecar form.

Minimum Markdown columns:

```text
Requirement ID | Evidence Type | Evidence Artifact | Status | Reviewer | Blocking Notes
```

Allowed status values:

```text
pass
pass_with_waiver
fail
not_applicable_with_reason
not_in_release_scope
```

A release claim MUST NOT rely only on narrative statements such as "tested" or "validated". It MUST point to named evidence artifacts.

## 28.59 Final Cross-Reference Audit Rule

Before publishing an EQC-SR governed profile, subsystem, or backend release, the portfolio MUST run a cross-reference audit.

The audit MUST verify:

```text
all section references resolve
all requirement IDs resolve to one canonical source location
all sidecar names mentioned in prose appear in the release artifact manifest or are explicitly future-work
all trace keys mentioned in tests appear in the trace schema
all public API claims appear in the compatibility claim grammar
all unresolved semantics are outside release scope or blocked
all waiver IDs resolve to waiver records
all implementation artifacts have SIB bindings when implementation-included release is claimed
```

A failed cross-reference audit blocks `SR-RELEASE-READY`.

## 28.60 Human-Readable Release Note Contract

A final or patch release SHOULD include short release notes. The release notes MUST be factual and MUST NOT expand conformance claims beyond the evidence.

Minimum release note structure:

```text
Release name
Release type: spec-only | pseudocode-included | implementation-included
Compatibility claim
Included artifacts
Known limitations
Waivers used
Validation summary
Breaking changes
Migration notes
```

For `v1.0.3`, the intended release type is `spec-only`, and the intended compatibility effect is `no_change_to_v1_0_compliance_claims`.


## 28.61 Non-Normative Example Marking Rule

Examples are useful for implementation, but they MUST NOT accidentally become hidden requirements.

Every example, template, sample sidecar, pseudo-record, or illustrative YAML/JSON fragment MUST be marked as one of:

```text
NORMATIVE_TEMPLATE       required structure; fields and status values are binding unless marked optional
ILLUSTRATIVE_EXAMPLE     explanatory only; not a compliance requirement
REFERENCE_SHAPE          recommended minimal structure; deviations require rationale
TEST_FIXTURE             executable validation input or expected output
```

A concrete portfolio MUST NOT infer conformance obligations from an `ILLUSTRATIVE_EXAMPLE`. If an example is intended to be binding, it MUST be promoted to `NORMATIVE_TEMPLATE` and assigned a requirement ID or schema ID.

## 28.62 Compliance Claim Badge Rule

A release MAY expose a short compliance badge, but the badge MUST be backed by the release artifact manifest and acceptance evidence table.

Allowed badge forms:

```text
EQC-SR v1.0.x / SPEC_ONLY / SR-SPEC-READY
EQC-SR v1.0.x / PSEUDOCODE_INCLUDED / SR-PORT-READY-SCOPE:<subsystem>
EQC-SR v1.0.x / IMPLEMENTATION_INCLUDED / SR-RELEASE-READY-SCOPE:<subsystem-or-package>
```

A badge MUST NOT omit the release type. A badge MUST NOT claim global release readiness unless the entire declared backend scope, compatibility surface, traces, benchmarks, sidecars, and SIB bindings are included in the release evidence.

## 28.63 Release Artifact Filename and Digest Table Rule

A release package MUST include a stable artifact table so that humans and tools can identify the exact files being governed.

Minimum table columns:

```text
Artifact Path | Artifact Type | Version | Digest | Required For | Included In Release | Notes
```

Allowed artifact types:

```text
profile_spec
subsystem_spec
operator_registry
sidecar_schema
pseudocode
implementation
trace
benchmark
candidate_corpus
release_note
waiver
reference_lockfile
acceptance_table
```

A missing digest is allowed only for draft artifacts that are explicitly marked `not_in_release_scope`. Release-critical artifacts MUST have digests before the package can claim `SR-RELEASE-READY`.

## 28.64 Final Patch Closure Rule

`EQC-SR-v1.0.3` is intended as the closure patch of the v1.0 line.

Further v1.0.x patches SHOULD be made only for:

```text
broken internal references
contradictory normative statements
missing required release evidence fields
security/safety wording defects
schema examples that cannot be parsed
```

New symbolic-regression capabilities, new compatibility tiers, new required sidecars, or new backend behavior requirements SHOULD be deferred to `v1.1` or `v2.0` according to Section 29.

## 28.65 v1.0-Line Closure and No-Op Patch Rule

After `EQC-SR-v1.0.3`, a proposed patch to the v1.0 line MUST first pass a no-op patch review.

A no-op patch review asks whether the issue can be resolved without changing the profile file. Acceptable no-op resolutions include:

```text
release note clarification
portfolio-local waiver
portfolio-local future-work record
implementation-side SIB update
subsystem-specific EQC document update
validator/tooling update that preserves profile semantics
```

A further `v1.0.x` file patch is justified only when at least one of the following is true:

```text
A normative contradiction exists inside EQC-SR itself.
A required release artifact cannot be produced because the profile omits a required field.
A compliance claim can be honestly misread in a way that changes release meaning.
A security or side-effect boundary is ambiguous enough to permit unsafe custom operators.
A schema or template marked normative is internally invalid.
A reference to EQC, EQC-ES, or EQC-SIB creates an unresolved governance conflict.
```

If none of these conditions apply, the change MUST be recorded outside the v1.0 profile as future work, portfolio-local policy, implementation documentation, or a v1.1 proposal.

This rule is intended to prevent the v1.0 line from growing indefinitely after the core symbolic-regression profile has stabilized.


## 29. Versioning Rules

EQC-SR follows semantic versioning.

Patch changes:

- wording clarification;
- non-functional examples;
- typo fixes;
- optional recommendations that do not change requirements.

Minor changes:

- new optional sections;
- new compatibility level details;
- new optional operator metadata fields;
- additional validation recommendations.

Major changes:

- required field changes;
- altered default invalid-candidate policy;
- altered equivalence semantics;
- changed trace requirements;
- changed canonical expression rules;
- changed required sidecars.

---

## 30. Minimal EQC-SR Spec Template

```text
# ALGO-SR-Example

Spec Version: ALGO-SR-v0.1.0
Extends: EQC-v1.1, EQC-SR-v1.0.0
Target Compatibility: SR-L1

1. Header and Global Semantics
2. Dataset Contract
3. Expression Tree Contract
4. Operator Manifest
5. Operator Definitions
6. Evaluation Semantics
7. Loss, Complexity, Ranking
8. Random Initialization
9. Mutation and/or Crossover
10. Population Procedure
11. Hall of Fame / Pareto Frontier
12. Trace Schema
13. Validation and Golden Problems
14. Equivalence Rules
15. Checkpoint and Restore
16. Requirement IDs and Conformance Matrix
17. SIB Bindings
18. Sidecar Schemas and Cross-References
19. Requirement Index
20. Issue / Decision / Waiver Records
21. Release Artifact Manifest
22. Subsystem Dependency Graph
23. Invariant Catalog
24. Validator Report Schema
25. Drift Ledger
26. Operator DSL Security Policy
27. Benchmark Harness
28. Release Candidate Manifest
29. Semantic Surface Freeze Declaration
30. Reference Differential Testing Plan
31. Candidate Corpus and Benchmark Dataset Contract
32. Feature Flag / Experimental Isolation Plan
33. Failure Injection and Recovery Validation
34. Numerical Envelope and Resource Budget
```

---



## 31. Change Log

### EQC-SR-v1.0.3 Final Closure Patch

- Updated final profile status from `v1.0.2 Final Patch` to `v1.0.3 Final Closure Patch`.
- Added a v1.0-line closure and no-op patch rule to prevent unnecessary future patch churn.
- Clarified when future corrections belong outside the profile file as release notes, portfolio-local policy, implementation documentation, or v1.1 proposals.
- Updated reference lockfile examples and patch-record examples to use the v1.0.3 filename/version.
- Preserved v1.0 semantic scope; no new symbolic-regression subsystem scope was added.

### EQC-SR-v1.0.2 Final Patch

- Updated final profile status from `v1.0.1 Final Stabilization` to `v1.0.2 Final Patch`.
- Added non-normative example marking rules to prevent examples from becoming hidden requirements.
- Added compliance badge wording constraints so release claims remain scoped and evidence-backed.
- Added release artifact filename and digest table requirements.
- Added final patch closure rules for the v1.0 line.
- No symbolic-regression semantic scope expansion.

### EQC-SR-v1.0.1 Final Stabilization

- Updated final profile status from `v1.0.0 Final` to `v1.0.1 Final Stabilization`.
- Added final errata and patch amendment rule.
- Added normative reference lockfile rule.
- Added acceptance evidence table rule.
- Added final cross-reference audit rule.
- Added human-readable release note contract.
- Preserved v1.0 semantic scope; no new symbolic-regression subsystem scope was added.



### EQC-SR-v1.0.0 Final

Finalized the release candidate by adding stabilization-only closure sections:

- finalization audit contract;
- normative glossary and term consistency rule;
- spec-only versus implementation-included release claim distinction;
- minimal human reviewer checklist;
- scope lock and future-work parking lot.

These additions are intended to prevent overclaiming and uncontrolled post-release scope expansion without changing the closed v1.0 semantic surfaces.

### EQC-SR-v1.0.0 Release Candidate

Strengthened v0.9 by stabilizing the profile as a first release candidate and adding:

- semantic closure rule for the v1.0 surface;
- exact conformance claim grammar;
- maintainer decision boundary records;
- regression triage matrix;
- minimal validator CLI contract;
- porting work package definition;
- minimal v1.0 release bundle layout;
- post-1.0 change control rules;
- updated template target to extend `EQC-SR-v1.0.0`.

### EQC-SR-v0.9.0

Strengthened v0.8 by adding:

- semantic surface freeze rules;
- reference differential testing protocol;
- candidate corpus and benchmark dataset contract;
- feature flag and experimental isolation policy;
- failure injection and recovery validation;
- numerical envelope and resource budget contract;
- pre-1.0 stabilization rule;
- updated template and profile status for final pre-1.0 execution readiness.

### EQC-SR-v0.8.0

Strengthened v0.7 by adding:

- subsystem dependency graph contract;
- invariant catalog contract;
- canonical validator report schema;
- deviation budget and semantic drift ledger;
- safe operator DSL and unsafe-code policy;
- reproducible benchmark harness contract;
- implementation handoff acceptance checklist;
- release candidate assembly rule;
- updated template and profile status for operational port execution.

### EQC-SR-v0.6.0

Strengthened v0.5 by adding:

- normative conflict-resolution rules across EQC, EQC-SR, EQC-ES, and EQC-SIB;
- stable machine-readable requirement index rules;
- requirement-to-test minimum binding rules;
- governed issue records for unresolved semantics;
- ADR-style decision records for documented deviations;
- implementation handoff packet requirements;
- minimal port prioritization order;
- oracle discipline and tolerance budget governance;
- release artifact manifest rules;
- updated status and template for implementation handoff readiness.

### EQC-SR-v0.5.0

Strengthened v0.4 by adding:

- profile integrity and document boundary rules;
- blocking ambiguity rule;
- reference snapshot discipline;
- source-to-spec coverage statuses;
- PySR option coverage matrix;
- compatibility claim labels;
- property-based and metamorphic validation requirements;
- negative test requirements;
- source extraction work unit template;
- anti-contamination note for clean-room versus source-guided implementation;
- conformance scoring rubric;
- milestone exit criteria;
- performance regression policy.

### EQC-SR-v0.3.0

Strengthened v0.2 by adding:

- explicit compliance modes: `SR-DRAFT`, `SR-SPEC-READY`, `SR-PORT-READY`, `SR-RELEASE-READY`;
- clean-room and semi-clean-room source boundary classification;
- compatibility claim rule to prevent vague PySR-compatible claims;
- minimum viable port scope;
- concrete mutation-only control-flow skeleton;
- deterministic seed derivation contract;
- source crosswalk record template;
- extraction evidence classes;
- machine-readable sidecar schema gate;
- sidecar identifier stability rule;
- implementation conformance matrix;
- porting risk register.

### EQC-SR-v0.2.0

Strengthened the first draft by adding:

- normative requirement language;
- primary use-case framing;
- required registry and SIB binding mapping;
- reproducibility and numeric defaults;
- explicit warning against semantic equality as default duplicate detection;
- machine-readable operator schema;
- safe string-operator grammar;
- train/validation/test semantics;
- shape/type contracts for evaluators;
- compiled backend eligibility and fallback rules;
- evaluation cache key requirements;
- model-selection contract;
- seed derivation rules;
- replacement and termination contracts;
- equation table schema;
- constant identity policy;
- constraint failure codes;
- canonicalization-versus-simplification distinction;
- export equivalence tests;
- JSONL trace record shapes;
- lint severity classes;
- source inventory and extraction evidence records;
- sidecar validation priority;
- error code registry;
- release gate checklist;
- initial Python backend module map.

### EQC-SR-v0.1.0

First usable symbolic-regression profile draft.


## 32. Summary

EQC-SR turns symbolic regression from an implementation-specific engine into a portable, testable specification layer.

Its intended pipeline is:

```text
SymbolicRegression.jl behavior
  ↓
EQC-SR subsystem specifications
  ↓
Governed pseudocode
  ↓
Python-native symbolic-regression backend
  ↓
PySR-style compatibility layer
```

The first goal is not full parity. The first goal is a stable, explicit, reproducible symbolic-regression core that can later grow toward PySR compatibility without losing semantic control.
