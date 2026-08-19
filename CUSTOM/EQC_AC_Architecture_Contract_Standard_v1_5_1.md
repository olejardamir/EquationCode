# EQC-AC: EQC Architecture Contract Standard

**Version:** EQC-AC v1.5.1  
**Date:** 2026-08-19  
**Status:** Draft standard candidate for architecture-level governance; completeness re-review found no new architecture-scope gap. Patch release fixes self-versioning examples so concrete contracts bind to the declared EQC-AC version instead of stale hard-coded standard versions.
**Purpose:** Define a governed, implementation-independent architecture contract that sits between system goals / semantic specifications and subsystem / file-level implementation contracts, so architecture can be designed, compared, validated, changed, and handed to coding agents without hidden assumptions, accidental topology, ownership ambiguity, unbounded coupling, or implementation-driven architectural drift.

---

# 0. Executive Summary

EQC-AC fills the architecture-level gap in the EquationCode ecosystem.

The existing ecosystem already separates several concerns:

```text
EQC
  = algorithm semantics, operators, state, procedure, reproducibility

EQC-ES
  = governance of document portfolios, versions, dependencies, traceability,
    compatibility, and change propagation

EQC-SIB
  = bidirectional binding between governed documents and implementation artifacts

EQC-FIC
  = file-level implementation contracts used immediately before code generation
```

EQC-FIC explicitly assumes the existence of higher-level architecture documents and places an architecture document above FIC in the authority hierarchy. The FIC workflow also calls for an `architecture_contract` before implementation units are generated. What is missing is a standard defining what such an architecture contract must contain, how it is validated, how architecture decisions are compared, how system-wide constraints are composed, and how the result constrains downstream subsystem and file-level implementation.

EQC-AC defines that missing layer.

Its core rule is:

```text
No governed implementation architecture may be treated as approved
until its components, boundaries, interactions, state ownership,
interfaces, quality-attribute budgets, assumptions, failure behavior,
deployment topology, and material architectural decisions are explicitly
declared and have passed the architecture readiness gate.
```

EQC-AC is not ordinary architecture documentation. It is a governed architecture contract.

It turns:

```text
requirements + system goal + semantics
```

into:

```text
a bounded, traceable, falsifiable, implementation-ready system structure
```

that can safely govern:

```text
subsystem specifications
shared interface documents
unit dependency graphs
whole-system / unit pseudocode
EQC-FIC documents
implementation artifacts through EQC-SIB
```

The intended transformation is:

```text
system goal / product requirements / EQC semantics
  -> architecture candidates
  -> composition-aware architecture comparison
  -> EQC-AC architecture contract
  -> shared types and interface contracts
  -> subsystem / unit contracts
  -> whole-system and bounded pseudocode
  -> EQC-FIC documents
  -> code
  -> tests / traces / operational evidence
  -> architecture conformance evidence
  -> governed evolution
```

EQC-AC prevents a coding agent, developer, or downstream document from silently inventing architecture.

---

## 0.1 Position in the EQC Ecosystem

EQC-AC operates at a different level from the existing standards.

```text
                 SYSTEM / PRODUCT INTENT
                          |
                          v
                    EQC / PROJECT GOAL
            semantics, invariants, objectives
                          |
                          v
                       EQC-AC
            architecture, boundaries, topology,
             ownership, budgets, interfaces,
              assumptions, design decisions
                          |
              +-----------+-----------+
              |                       |
              v                       v
       subsystem contracts      whole-system /
       shared interfaces        unit pseudocode
              |                       |
              +-----------+-----------+
                          |
                          v
                       EQC-FIC
                 file implementation
                      contracts
                          |
                          v
                        CODE
                          |
                          v
                  TESTS / TRACES / OPS
```

Cross-cutting governance:

```text
EQC-ES
  governs the document portfolio

EQC-SIB
  binds governed documents to pseudocode / code / implementation artifacts
```

EQC-AC MUST NOT duplicate the responsibilities of EQC, EQC-ES, EQC-SIB, or EQC-FIC.

---

## 0.2 Reliability Claim

EQC-AC does not claim that architecture documentation guarantees a correct system.

It claims that architecture should not be an implicit mixture of:

- diagrams;
- implementation accidents;
- framework defaults;
- undocumented assumptions;
- infrastructure guesses;
- conversations;
- tribal knowledge;
- hidden coupling;
- performance folklore.

A governed architecture can still be wrong. EQC-AC makes it possible to know **what architecture was actually approved**, which requirements and assumptions support it, what its failure modes are, and which evidence could falsify it.

---

## 0.3 Conformance Levels

A project or architecture document may claim one of the following levels:

| Level | Name | Meaning | Minimum Requirement |
|---:|---|---|---|
| 0 | advisory | Architecture document exists | narrative architecture only |
| 1 | structured | Required EQC-AC sections exist | manual completeness review |
| 2 | governed | Components, interfaces, ownership, dependencies, decisions, assumptions, and budgets are explicit | architecture readiness gate |
| 3 | validated | Key architecture claims are backed by tests, benchmarks, traces, calculations, or authoritative external evidence | validation report |
| 4 | enforced | CI/tooling rejects downstream artifacts that violate the architecture contract | machine-readable registries + automated checks |
| 5 | audited | Bidirectional traceability exists from requirements → architecture → FIC/code → evidence and back | release / operational evidence + change history |

A normal production architecture SHOULD target Level 2 or 3 before implementation. High-risk or long-lived systems SHOULD target Level 4 or 5.

---

# 1. Scope

EQC-AC applies to architecture-level decisions that materially affect the structure or behavior of a system.

It governs:

- system context and external actors;
- architecture boundaries;
- components / services / modules / processes;
- component responsibilities;
- allowed and forbidden dependencies;
- interaction patterns;
- data flow and control flow;
- public architectural interfaces;
- event / message contracts at architecture level;
- state ownership;
- consistency model;
- persistence responsibilities;
- caching responsibilities;
- deployment topology;
- runtime placement;
- geographic / regional topology;
- failure domains;
- availability and recovery strategy;
- scalability and backpressure strategy;
- performance budgets;
- resource and capacity budgets;
- cost budgets;
- trust boundaries;
- security constraints;
- data-classification boundaries;
- observability responsibilities;
- external provider assumptions;
- architecture decisions and rejected alternatives;
- unresolved assumptions;
- architecture-critical risks;
- architecture validation evidence;
- architecture-to-FIC handoff constraints;
- architecture evolution and migration.

---

## 1.1 Out of Scope

EQC-AC does not replace:

- EQC algorithm semantics;
- detailed mathematical operator definitions;
- business requirements;
- product requirements;
- detailed file-level contracts;
- source-code implementation details;
- unit tests;
- detailed API schema documents when a dedicated schema document owns them;
- infrastructure-as-code;
- deployment scripts;
- detailed runbooks;
- security threat models when a dedicated threat-model document is required;
- formal proof systems;
- human architectural judgment.

EQC-AC may **reference** these artifacts and place constraints on them.

---

## 1.2 Architecture vs Implementation

EQC-AC MAY name a concrete technology when the technology itself is an approved architectural decision.

Example:

```text
Allowed:
"Durable event log is implemented with Apache Kafka because ordered
partitioned replay is part of the approved architecture."

Not sufficient:
"Use Kafka because Kafka is scalable."
```

EQC-AC SHOULD avoid file-level implementation details unless they are architecturally significant.

Example:

```text
Architecture-level:
"Match events are partitioned by match_id and processed by exactly one
logical state owner per partition."

File-level:
"src/match/state_owner.py exposes class MatchStateOwner."
```

The first belongs in EQC-AC. The second belongs in EQC-FIC or downstream implementation documentation.

---

# 2. Normative Language

The following terms are normative:

- **MUST** — required for conformance.
- **MUST NOT** — prohibited.
- **SHOULD** — recommended; deviation requires rationale.
- **MAY** — optional, but declared if used.
- **UNKNOWN** — information required for architecture is missing.
- **ASSUMPTION** — proposition used by the architecture but not established as a governing fact.
- **EXTERNAL FACT** — fact owned outside the project and supported by an authoritative source.
- **INFERENCE** — conclusion drawn from evidence but not directly measured or guaranteed.
- **DECISION** — approved architectural choice among alternatives.
- **INVARIANT** — condition that must remain true across valid system states.
- **BUDGET** — quantified maximum, minimum, or allocation attached to a quality attribute or resource.
- **HARD CONSTRAINT** — violation invalidates the architecture.
- **SOFT PREFERENCE** — used to compare architectures after hard constraints pass.
- **BLOCKED** — architecture cannot advance because required information is absent or contradictory.
- **VALIDATED** — declared architecture checks passed with acceptable evidence.
- **WAIVED** — a requirement has been explicitly relaxed through a recorded exception.
- **DEGRADED** — architecture still operates but with a declared reduction in service level.
- **ARCHITECTURAL SURFACE** — behavior or interface visible across architecture boundaries.
- **HIDDEN PATH** — undeclared component-to-component communication or dependency.
- **STATE OWNER** — the unique architecture unit authorized to mutate a given authoritative state domain.
- **FAILURE DOMAIN** — set of components that may fail together due to a shared dependency or placement.
- **ARCHITECTURE EVIDENCE** — test, benchmark, trace, calculation, authoritative source, model, or reviewed observation supporting an architectural claim.

---

# 3. Core Principles

## 3.1 Architecture Before Implementation

Implementation MUST NOT be allowed to invent system structure that the architecture has not authorized.

If a coding task requires a new:

- service;
- shared database;
- queue;
- cross-component dependency;
- public protocol;
- state owner;
- retry channel;
- cache;
- region;
- trust boundary;
- persistence side effect;

and that choice is architecturally material, the architecture contract MUST be updated before the implementation is accepted.

---

## 3.2 Requirement-to-Architecture Traceability

Every material architecture element MUST answer:

```text
Why does this exist?
```

Every hard architecture requirement MUST map to one or more:

- components;
- interfaces;
- invariants;
- budgets;
- topology rules;
- validation obligations;
- explicit non-architectural owners.

No orphan components. No orphan hard requirements.

---

## 3.3 Explicit Boundaries

Every architecture unit MUST declare:

- what it owns;
- what it does not own;
- what it may call;
- what may call it;
- what state it may mutate;
- what data it may persist;
- what side effects it may produce.

Unbounded “shared services” are prohibited unless the shared surface is explicitly governed.

---

## 3.4 One Owner for Authoritative Mutable State

Every authoritative mutable state domain MUST have exactly one logical owner.

Multiple replicas MAY exist, but mutation authority and reconciliation rules MUST be explicit.

If multiple writers are permitted, the architecture MUST define:

- conflict semantics;
- ordering semantics;
- idempotency;
- consistency model;
- recovery behavior.

---

## 3.5 No Hidden Communication Paths

All material communication paths MUST appear in the architecture interaction registry or dependency graph.

Examples of hidden paths:

- direct database access bypassing an owning service;
- undeclared event publication;
- ad hoc shared storage;
- unregistered callbacks;
- environment-variable coupling across services;
- undocumented cross-region replication;
- direct third-party API usage from an unauthorized component.

Hidden paths are architecture violations.

---

## 3.6 Interface Before Integration

A component-to-component dependency MUST have an owned interface contract before implementation integrates the components.

The interface contract MAY live in:

- the EQC-AC document;
- a governed shared-interface document;
- a schema artifact;
- a subsystem specification.

But there MUST be exactly one authoritative owner.

---

## 3.7 Smallest Sufficient Architecture

Architecture MUST NOT add a component merely because it is common, fashionable, or potentially useful later.

Every component must justify its existence against:

- a hard requirement;
- a quality attribute;
- a failure-isolation requirement;
- a scaling requirement;
- a security boundary;
- a clear simplification of the system.

If removing a component preserves all approved architecture requirements with no unacceptable trade-off, the component SHOULD be removed.

---

## 3.8 Composition-Aware Decision Making

A component that appears optimal in isolation may be inferior once system-wide constraints are composed.

Therefore architecture alternatives MUST be compared against the complete relevant requirement vector, not a single local metric.

Examples:

```text
lowest component latency
  may lose after:
    regional delivery + cost + failure recovery

cheapest database
  may lose after:
    consistency + write amplification + recovery

simplest protocol
  may lose after:
    reconnect + backpressure + ordering + fan-out
```

EQC-AC explicitly prohibits premature architectural pruning based only on isolated local optimality.

---

## 3.9 Facts, Assumptions, Measurements, and Inferences Must Remain Distinct

Every architecture-relevant proposition MUST be classifiable as one of:

```text
GOVERNING_REQUIREMENT
EXTERNAL_FACT
ARCHITECTURE_DECISION
ASSUMPTION
MEASUREMENT
INFERENCE
WAIVER
```

An assumption MUST NOT be presented as a fact.

A local benchmark MUST NOT be presented as production proof.

A vendor capability MUST NOT be assumed without evidence when the architecture depends on it.

---

## 3.10 Failure Is Part of the Architecture

A valid architecture describes not only the happy path but also:

- what can fail;
- what fails together;
- what retries;
- what does not retry;
- what is durable;
- what is reconstructable;
- what is lost;
- what becomes degraded;
- how recovery occurs;
- what the user observes.

A design with no failure model is incomplete.

---

## 3.11 Budgets Are End-to-End Contracts

Latency, cost, capacity, availability, recovery, and other quality attributes MUST be treated as system-level budgets.

Sub-budgets MAY be allocated to components, but the architecture MUST preserve the distinction between:

```text
component measurement
and
end-to-end requirement
```

Quantiles such as p95 MUST NOT be naively added across independent stages and represented as a mathematically valid end-to-end p95 unless the derivation justifies it.

---

## 3.12 Evidence Over Architectural Confidence

A diagram is not evidence.

A vendor slogan is not evidence.

An LLM recommendation is not evidence.

Material architecture claims SHOULD be supported by one or more of:

- authoritative documentation;
- capacity calculation;
- benchmark;
- proof of concept;
- load test;
- trace;
- failure-injection test;
- cost calculation;
- replay;
- model with declared assumptions;
- production observation;
- reviewed engineering evidence.

---

# 4. EQC-AC Document Set

A governed architecture MAY be represented by one document, but non-trivial systems SHOULD separate machine-readable registries and evidence.

Recommended structure:

```text
docs/architecture/
  ARCHITECTURE.eqc-ac.md
  component-registry.yaml
  interface-registry.yaml
  dependency-graph.yaml
  deployment-topology.yaml
  requirement-index.yaml
  budget-ledger.yaml
  assumption-ledger.yaml
  decision-log.md
  risk-ledger.yaml
  validation-plan.md
  generated/
    architecture-lockfile.yaml
    architecture-readiness-report.md
    requirement-coverage-report.md
    architecture-validation-report.md
```

The main document is normative.

Generated artifacts MUST NOT become hidden sources of architectural intent.

---

## 4.1 Required Identity

Every EQC-AC document MUST declare:

```yaml
architecture_id: ARCH-...
title: ...
eqc_ac_version: "<declared EQC-AC version>"
architecture_version: "vX.Y.Z"
status: draft | review | approved | frozen | migrating | deprecated
system_id: ...
owner: ...
last_updated: YYYY-MM-DD
governing_requirements: [...]
governing_eqc_documents: [...]
governing_eqc_es_root: ...
```

---

## 4.2 Registration Under EQC-ES

Until EQC-ES natively defines an `architecture-contract` document type, EQC-AC documents SHOULD be registered in EQC-ES as:

```text
Type: other
```

with metadata declaring:

```yaml
profile: EQC-AC
architecture_id: ARCH-...
```

A future EQC-ES revision MAY add `architecture-contract` as a first-class document type through normal versioned change propagation.

EQC-AC MUST NOT silently mutate EQC-ES registry semantics.

## 4.3 Controlled Vocabulary and Model Conventions

Architecture terms that affect interpretation MUST be defined once.

A non-trivial EQC-AC portfolio SHOULD maintain a controlled vocabulary containing, where relevant:

- domain terms;
- component kinds;
- interface kinds;
- state-domain terms;
- consistency terms;
- failure/recovery terms;
- deployment/environment names;
- security/data-classification terms;
- metric names and units.

Template:

```yaml
term_id: TERM-CANONICAL-MATCH-STATE
term: canonical_match_state
definition: authoritative accepted state derived from the governed event stream
synonyms:
  - current_match_state
forbidden_ambiguous_uses:
  - cache_state
owner: ARCH-...
```

Rules:

- One term MUST NOT have two materially different meanings inside the same governed architecture.
- Different terms that intentionally refer to the same concept SHOULD declare aliases.
- Units, time bases, percentile notation, and identifiers SHOULD use consistent conventions.
- Diagrams, registries, budgets, FIC bindings, and validation evidence MUST use compatible identifiers and terminology.

A glossary is not required for trivial architectures, but semantic ambiguity is never acceptable.

## 4.4 Applicability Matrix and Conformance Statement

Because EQC-AC is intended to govern architectures ranging from small software systems to distributed platforms, a concrete architecture MUST distinguish **not applicable** from **forgotten**.

Each major EQC-AC concern SHOULD be assigned one status:

```text
REQUIRED
SATISFIED
NOT_APPLICABLE_WITH_REASON
DEFERRED_BLOCKING
DEFERRED_NONBLOCKING
WAIVED
```

Recommended sidecar:

```yaml
eqc_ac_conformance:
  standard: EQC-AC
  standard_version: "<declared EQC-AC version>"
  architecture_id: ARCH-...
  architecture_version: vX.Y.Z
  claimed_level: 0 | 1 | 2 | 3 | 4 | 5

  applicability:
    deployment_topology:
      status: SATISFIED
      source: "§10"
    data_residency:
      status: NOT_APPLICABLE_WITH_REASON
      reason: "No residency requirement exists in the governing scope."
    disaster_recovery:
      status: DEFERRED_BLOCKING
      owner: ...
      resolution: ...

  open_waivers:
    - ...
  evidence_bundle:
    - ...
```

Rules:

- `NOT_APPLICABLE_WITH_REASON` MUST contain a reason.
- A `DEFERRED_BLOCKING` concern prevents `ARCH_READY`.
- `DEFERRED_NONBLOCKING` is allowed only when the deferred concern cannot invalidate a hard requirement or architecture-critical invariant.
- A conformance claim MUST identify the exact EQC-AC version and architecture version.
- A higher conformance level MUST NOT be claimed merely because sections exist; its required governance/evidence must actually be present.
- The applicability matrix MUST NOT be used to waive a governing requirement. Requirement waivers follow §33.
- Conformance output SHOULD list all unresolved blockers and waivers rather than collapsing them into a single pass/fail label.

This makes completeness auditable without forcing irrelevant sections into every architecture.

---

# 5. System Context Contract

The architecture MUST define the boundary of the system.

Required:

- users / actors;
- upstream systems;
- downstream systems;
- external providers;
- administrative/control-plane actors;
- data sources;
- data sinks;
- trust boundaries;
- responsibility boundary.

Template:

```yaml
system_context:
  system: MATCH-CENTRE
  actors:
    - id: FAN
      role: anonymous read-only user
  externals:
    - id: FEED-PROVIDER
      relationship: upstream event source
      guarantees: [...]
      assumptions: [...]
  boundary:
    owns:
      - accepted-event processing
      - application-visible state
    does_not_own:
      - upstream event generation
```

The architecture MUST distinguish:

```text
outside failure
from
inside failure
```

## 5.1 Stakeholder and Concern Registry

A governed architecture exists to resolve stakeholder concerns, not only to inventory components.

Every material architecture concern SHOULD have a stable ID and an accountable stakeholder or stakeholder class.

Recommended stakeholder classes include, where applicable:

- end users;
- product/business owner;
- engineering/development;
- operations/SRE;
- security;
- privacy/legal/compliance;
- data/platform owners;
- support;
- finance/cost owner;
- external integration owner.

Template:

```yaml
concern_id: CONCERN-AVAILABILITY
stakeholders:
  - STAKEHOLDER-OPS
statement: service must remain usable during a single-instance failure
priority: hard | high | medium | low
quality_attributes:
  - availability
architecture_bindings:
  - COMP-...
  - BUDGET-...
  - ADR-ARCH-...
validation:
  - VAL-ARCH-...
```

A material stakeholder concern with no architecture response MUST be recorded as `UNRESOLVED` or explicitly declared out of scope.

## 5.2 Architecture Viewpoints and Views

Non-trivial architectures SHOULD define the minimum set of views required to answer material concerns.

A **viewpoint** defines what a view must show and which concerns it addresses.  
A **view** is the concrete representation of this architecture from that viewpoint.

Recommended viewpoints, used only when relevant:

```text
context
logical/component
interaction/runtime
data/state
deployment
failure/recovery
security/trust
operability
cost/capacity
```

Each view SHOULD declare:

```yaml
view_id: VIEW-DEPLOYMENT
viewpoint: deployment
concerns:
  - CONCERN-LATENCY
  - CONCERN-FAILURE-ISOLATION
model_elements:
  - COMP-...
  - DEPLOY-...
  - FD-...
source_of_truth:
  - deployment-topology.yaml
```

### View Correspondence Rule

Different views MUST describe the same governed architecture.

If the deployment view shows a communication path, state owner, replica, region, or dependency that the component/interaction registries do not authorize, validation MUST fail or the discrepancy MUST be explicitly marked as a draft inconsistency.

Diagrams are projections of governed model elements, not independent sources of truth.

## 5.3 Model-Kind and Notation Registry

Each non-trivial architecture view SHOULD declare the **model kind** or representation rules it uses.

A model kind defines what element types and relationship types are legal in that model and what they mean.

Examples may include:

```text
component-and-connector
deployment
data-flow
state-ownership
sequence/interaction
trust-boundary
failure-domain
dependency
cost/capacity
```

Template:

```yaml
model_kind_id: MK-COMPONENT-CONNECTOR
purpose: represent runtime components and allowed communication
allowed_elements:
  - component
  - external
  - datastore
allowed_relations:
  - calls
  - publishes_to
  - subscribes_to
semantics:
  calls: synchronous request/response dependency
notation:
  component: rectangle
  external: boundary-marked rectangle
validation_rules:
  - every relation must map to an interaction_id
```

Rules:

- A notation symbol MUST NOT silently change meaning between views.
- A view MUST identify its model kind(s).
- Model kinds SHOULD bind to governed registries rather than create separate anonymous identifiers.
- A diagram containing an element or relationship that cannot map to the governed architecture model is invalid or explicitly illustrative/non-normative.
- If a project uses C4, UML, ArchiMate, Mermaid, custom YAML, or another notation, EQC-AC governs the **meaning and correspondence**, not the drawing syntax itself.

## 5.4 Cross-Model Correspondence Rules

Where the same architecture fact appears in multiple model kinds, declare correspondence rules.

Examples:

```text
logical component
  -> one or more deployment units

state owner
  -> component with authorized write path

interaction
  -> dependency-graph edge
  -> interface contract

trust boundary crossing
  -> security-control obligation
```

Template:

```yaml
correspondence_id: CORR-COMP-DEPLOY
source_model_kind: MK-COMPONENT-CONNECTOR
target_model_kind: MK-DEPLOYMENT
rule: every deployable runtime component maps to >=1 deployment unit
exceptions:
  - browser-only component
validation: ...
```

A correspondence violation MUST be surfaced as architecture inconsistency.

---

# 6. Requirement and Quality-Attribute Index

Every material requirement MUST receive a stable ID.

Recommended forms:

```text
ARCH-REQ-FUNC-...
ARCH-REQ-LAT-...
ARCH-REQ-CAP-...
ARCH-REQ-AVAIL-...
ARCH-REQ-CONS-...
ARCH-REQ-SEC-...
ARCH-REQ-COST-...
ARCH-REQ-DEPLOY-...
```

Minimum fields:

```yaml
req_id: ARCH-REQ-LAT-GOAL-v1
source: ...
statement: ...
type: hard | soft
owner: architecture | product | external
verification:
  method: test | calculation | benchmark | review | external_evidence
  artifact: ...
status: satisfied | partial | blocked | waived
architecture_bindings:
  - COMP-...
  - BUDGET-...
  - INV-...
```

A hard requirement with no architecture binding MUST block approval unless it is explicitly owned outside architecture.

## 6.1 Quality-Attribute Scenario Contract

Quality attributes SHOULD be expressed as testable scenarios rather than adjectives.

For any material quality attribute, use:

```yaml
qa_id: QA-LATENCY-GOAL
quality_attribute: performance
source: external provider event
stimulus: goal event accepted at ingest
environment: peak viewer load
artifact: live-match delivery path
response: event becomes visible to fan
measure: p95 <= 2s
priority: hard
```

Recommended fields:

```text
source
stimulus
environment
affected artifact/path
expected response
quantified response measure
priority
validation method
```

Statements such as:

```text
scalable
secure
maintainable
resilient
fast
loosely coupled
```

are insufficient when they represent material requirements unless their meaning is made operational or explicitly marked as a design preference.

## 6.2 Architecture Principles and Constraints

Projects MAY maintain stable architecture principles that constrain multiple decisions.

Examples:

- one owner for authoritative mutable state;
- no direct datastore bypass of owning component;
- public interfaces are versioned;
- externally supplied data is validated at the trust boundary;
- stateless serving components are preferred where state ownership does not require otherwise.

Each principle MUST declare whether it is:

```text
MANDATORY
DEFAULT_WITH_JUSTIFIED_EXCEPTION
ADVISORY
```

A principle MUST NOT silently override a governing product/system requirement.

## 6.3 Requirement Conflict and Feasibility Gate

EQC-AC MUST NOT silently resolve contradictory hard requirements by weakening one.

When two or more hard requirements cannot all be satisfied under the declared system boundary, budget, or external constraints, architecture status MUST become:

```text
ARCH_BLOCKED_INFEASIBLE_REQUIREMENTS
```

until one of the following occurs:

- the governing requirement is formally changed by its owner;
- the system boundary changes;
- an external constraint changes;
- a documented waiver is approved;
- evidence shows the apparent conflict was based on a false assumption.

A feasibility record SHOULD include:

```yaml
feasibility_id: FEAS-ARCH-...
conflicting_requirements:
  - ARCH-REQ-...
  - ARCH-REQ-...
reason: ...
evidence: [...]
candidate_resolutions: [...]
owner: ...
status: blocked | resolved | waived
```

Architecture MAY optimize among soft preferences. It MUST NOT optimize away a hard requirement.

---

# 7. Component Registry

Every material architecture component MUST have a stable ID.

Examples:

```text
COMP-INGEST
COMP-MATCH-STATE
COMP-HISTORY
COMP-FANOUT
COMP-WEB
COMP-EDGE
```

Each entry MUST define:

```yaml
component_id: COMP-MATCH-STATE
name: Match State Processor
kind: service | process | module | datastore | broker | edge | client | external | control_plane
purpose: ...
responsibilities:
  - ...
non_responsibilities:
  - ...
owns_state:
  - STATE-MATCH-CANONICAL
provides_interfaces:
  - IFACE-MATCH-EVENTS
consumes_interfaces:
  - IFACE-INGESTED-EVENTS
allowed_dependencies:
  - COMP-EVENT-LOG
forbidden_dependencies:
  - COMP-WEB
failure_domain: FD-...
scaling_unit: ...
security_zone: ...
deployment_unit: ...
requirements:
  - ARCH-REQ-...
```

---

## 7.1 Component Admission Rule

A component MUST NOT enter the approved architecture unless:

1. its purpose is distinct;
2. at least one requirement or architectural invariant justifies it;
3. ownership does not duplicate another component without explicit synchronization;
4. dependencies are bounded;
5. failure behavior is understood at the required level;
6. its cost/resource effect is known or bounded;
7. its interface obligations are known.

---

## 7.2 No Architecture by Product Name

A cloud service or framework name is not a component definition.

Invalid:

```text
"Redis"
```

Valid:

```text
COMP-CURRENT-STATE-CACHE
Purpose: serve current match state under the late-join latency budget
Technology decision: Redis / ElastiCache
Reason: ...
```

This separates architectural responsibility from implementation technology.

## 7.3 Architecture Invariant Registry

Architecture-level invariants MUST be governed explicitly when downstream contracts or validation depend on them.

Each invariant MUST have a stable ID.

Template:

```yaml
invariant_id: INV-ARCH-SINGLE-MATCH-WRITER
statement: exactly one logical owner may mutate canonical state for a given match partition
scope:
  - COMP-MATCH-STATE
  - STATE-MATCH-CANONICAL
rationale: prevents divergent concurrent state mutation
derived_from:
  - ARCH-REQ-CONSISTENCY
enforcement:
  static:
    - dependency_lint
  runtime:
    - owner_epoch_check
validation:
  - VAL-ARCH-...
downstream_bindings:
  - FIC-...
status: active | deprecated | waived
```

Rules:

- An invariant MUST be stated in a falsifiable or lintable form where practical.
- Invariants MUST NOT exist only as prose inside diagrams or decision rationale.
- Downstream FICs MAY bind to invariant IDs rather than restating them.
- An implementation change that invalidates an active architecture invariant MUST trigger architecture impact analysis.
- If two invariants conflict, the architecture is blocked until the conflict is resolved or formally waived.

This registry is the authoritative source for IDs such as `INV-...` referenced by FIC architecture bindings.

## 7.4 Cross-Cutting Architecture Policy Registry

Some behavior spans many components but still needs exactly one architecture-level source of truth.

Examples include, when material:

- authentication / authorization policy;
- retry and timeout defaults;
- idempotency policy;
- caching policy;
- rate limiting / admission control;
- serialization / envelope conventions;
- correlation / trace propagation;
- tenancy / isolation policy;
- error classification;
- data-validation boundaries;
- encryption / key-use policy;
- circuit breaking / load shedding;
- audit-event requirements.

A cross-cutting policy MUST have a stable ID when multiple downstream components or FICs depend on it.

Template:

```yaml
policy_id: POLICY-RETRY-DEFAULT
scope:
  - COMP-...
purpose: ...
rule: ...
exceptions:
  - ...
owner: ...
architecture_bindings:
  - ADR-ARCH-...
downstream_bindings:
  - FIC-...
validation:
  - VAL-ARCH-...
```

Rules:

- A cross-cutting policy MUST NOT be independently redefined by each component.
- Exceptions MUST be explicit and justified.
- Framework defaults MUST NOT silently become architecture policy.
- If a policy materially affects latency, cost, consistency, security, or failure behavior, it MUST bind to the corresponding requirement/budget/risk.
- A policy may be implemented by libraries, middleware, gateways, platform services, or component code; the implementation location does not change its architecture ownership.
- A file-level FIC may refine implementation details but MUST NOT silently weaken an architecture-level cross-cutting policy.

---

# 8. Interaction Registry

Every cross-component interaction MUST be declared.

Required fields:

```yaml
interaction_id: FLOW-INGEST-STATE
source: COMP-INGEST
target: COMP-MATCH-STATE
interface: IFACE-INGESTED-EVENTS
mode: sync | async | stream | batch | read | write | publish | subscribe
direction: one_way | request_response | bidirectional
ordering: ...
delivery: ...
timeout: ...
retry: ...
backpressure: ...
idempotency: ...
failure_behavior: ...
security_context: ...
observability: ...
```

---

## 8.1 No Implicit Retry Semantics

Retries MUST be architecture-owned when they affect:

- duplicate processing;
- cost;
- latency;
- ordering;
- load;
- external provider behavior;
- user-visible failure.

Default framework retry behavior MUST NOT become architecture accidentally.

---

## 8.2 Backpressure Contract

Streaming, fan-out, queueing, and producer-consumer interactions MUST declare:

- buffer location;
- buffer capacity or bounding rule;
- overflow policy;
- producer behavior under saturation;
- consumer lag semantics;
- user-visible degradation;
- metrics.

Unbounded buffering is prohibited unless explicitly justified.

---

# 9. Interface and Data Contract Ownership

Every architecture-level interface MUST have exactly one authoritative owner.

An interface may define:

- request / response shape;
- event schema;
- message envelope;
- sequence identifier;
- correlation identifier;
- timestamp semantics;
- version;
- compatibility policy;
- error semantics;
- authentication context;
- idempotency key;
- required observability fields.

The architecture MAY reference a dedicated schema document rather than duplicate details.

---

## 9.1 Schema Evolution

Each externally or cross-component visible schema MUST declare compatibility rules.

Examples:

```text
backward-compatible additive
versioned envelope
consumer-driven compatibility
strict lockstep
translation gateway
```

The architecture MUST define what happens during mixed-version deployment when live upgrades are required.

## 9.2 Data Lifecycle and Governance Contract

For material data domains, architecture MUST define lifecycle responsibilities when they affect correctness, cost, privacy, compliance, or recovery.

Template:

```yaml
data_domain_id: DATA-MATCH-EVENTS
classification: public | internal | confidential | restricted | domain_specific
owner: ...
source: ...
system_of_record: ...
retention: ...
deletion: ...
archival: ...
residency: ...
encryption:
  in_transit: ...
  at_rest: ...
backup: ...
restore: ...
lineage:
  inputs: [...]
  derived_outputs: [...]
schema_owner: ...
```

The architecture SHOULD distinguish:

```text
authoritative data
derived data
cache
ephemeral data
audit/trace data
```

A cache MUST NOT silently become the only recovery source for authoritative data.

Retention and deletion rules MUST be explicit when required by product, privacy, compliance, or cost constraints.

---

# 10. State Ownership and Consistency Contract

Every state domain MUST be declared.

Template:

```yaml
state_id: STATE-MATCH-CANONICAL
description: authoritative accepted state for one match
owner: COMP-MATCH-STATE
writers:
  - COMP-MATCH-STATE
readers:
  - COMP-API
  - COMP-HISTORY
persistence: ...
durability: ...
consistency: ...
ordering_key: match_id
recovery_source: ...
retention: ...
mutation_invariants:
  - ...
```

---

## 10.1 Single Logical Writer Rule

If a state domain has more than one active writer, the architecture MUST define a deterministic convergence or conflict rule.

Examples:

- optimistic concurrency with version check;
- leader ownership;
- partition ownership;
- CRDT;
- transaction coordinator;
- deterministic merge;
- consensus.

“Eventually consistent” is insufficient without defining what may diverge and how convergence occurs.

---

## 10.2 Snapshot-to-Live Handoff

Systems that serve historical/current state and then continue with live updates MUST define the handoff boundary.

The contract MUST answer:

- What version/sequence identifies the snapshot?
- From what sequence does live delivery continue?
- How are events arriving during snapshot acquisition handled?
- How are duplicates prevented?
- How are gaps detected?
- What happens on reconnect?

This is a first-class architecture concern.

## 10.3 Time, Concurrency, and Transaction Semantics

Systems whose correctness depends on time or concurrent mutation MUST declare the relevant semantics.

Time semantics MAY include:

- event time;
- ingest time;
- processing time;
- wall-clock time;
- monotonic elapsed time;
- client-display time;
- clock-skew tolerance;
- authoritative clock source.

Concurrency semantics MUST define, where applicable:

- serialization boundary;
- optimistic/pessimistic concurrency;
- transaction boundary;
- compare-and-swap/version checks;
- duplicate concurrent request behavior;
- race-resolution rule.

If a business operation spans multiple components and cannot be atomic, the architecture MUST declare the consistency/recovery model, for example:

```text
saga / compensating action
outbox/inbox
idempotent retry
eventual convergence
manual reconciliation
```

The term `transactional` MUST NOT be used across architecture boundaries without defining the actual transaction scope.

---

# 11. Dependency Graph and Layering

EQC-AC MUST define allowed architecture dependencies.

Recommended machine-readable form:

```yaml
nodes:
  - COMP-INGEST
  - COMP-MATCH-STATE
  - COMP-WEB
edges:
  - from: COMP-INGEST
    to: COMP-MATCH-STATE
    kind: publishes_to
  - from: COMP-WEB
    to: COMP-API
    kind: calls
```

Each edge MUST correspond to a declared interaction/interface.

---

## 11.1 Cycle Policy

Cycles are forbidden by default across architectural ownership boundaries.

A cycle MAY be approved only if the architecture records:

- why the cycle is required;
- initialization semantics;
- failure semantics;
- retry semantics;
- deployment/version coupling;
- test evidence.

---

## 11.2 Forbidden Dependencies

The contract SHOULD record negative edges.

Example:

```yaml
forbidden:
  - from: COMP-WEB
    to: DATASTORE-AUTHORITATIVE
    reason: bypasses state owner
```

Negative architecture is important because absence of a prohibition often becomes accidental coupling later.

---

# 12. Runtime and Deployment Topology

The architecture MUST distinguish logical architecture from runtime placement.

For each deployment unit declare:

```yaml
deployment_unit: DEPLOY-MATCH-PROCESSOR
component: COMP-MATCH-STATE
runtime: container | function | vm | browser | edge | managed_service
regions:
  - ...
replicas: ...
autoscaling: ...
placement_constraints: ...
failure_domain: ...
rollout_strategy: ...
drain_behavior: ...
rollback_strategy: ...
stateful: true | false
```

---

## 12.1 Logical vs Physical Architecture

The contract MUST make clear when:

```text
one logical component
```

is deployed as:

```text
many physical replicas
```

and how those replicas preserve ownership and consistency.

---

## 12.2 Mixed-Version Operation

If live or zero-downtime deployment is required, the architecture MUST define whether adjacent versions can coexist.

Required declaration:

```text
MIXED_VERSION_ALLOWED
MIXED_VERSION_FORBIDDEN
MIXED_VERSION_ALLOWED_WITH_COMPATIBILITY_WINDOW
```

If allowed, interfaces and state schemas must support it.

## 12.3 Environment Profiles and Architecture Drift

Where development, test, staging, disaster-recovery, and production environments differ in architecture-relevant ways, those differences MUST be declared.

Template:

```yaml
environment_id: ENV-PROD
purpose: production
regions: [...]
component_overrides: [...]
capacity_profile: WORKLOAD-PEAK
external_dependencies: [...]
security_controls: [...]
configuration_profile: CONFIG-PROD
known_differences_from_reference: [...]
```

Rules:

- Test/staging MAY be smaller than production, but architecture-significant differences MUST be visible.
- Validation evidence MUST state the environment in which it was obtained.
- A non-production result MUST NOT be silently promoted to production equivalence.
- Environment-specific topology or dependency changes participate in architecture impact analysis.

## 12.4 Dynamic Configuration and Feature-Control Contract

Configuration that can change architecture behavior MUST be treated as a governed control surface.

Architecture-significant configuration includes, where applicable:

- routing;
- region enablement;
- retry/backoff policy;
- consistency mode;
- queue/buffer limits;
- feature flags affecting protocol/state behavior;
- failover mode;
- traffic percentages;
- data retention;
- security enforcement.

Template:

```yaml
config_id: CONFIG-FANOUT-MODE
owner: ...
scope:
  - COMP-FANOUT
allowed_values: [...]
default: ...
change_mode: deploy | dynamic | emergency_only
validation: ...
rollback: ...
audit_required: true
```

Rules:

- Dynamic configuration MUST have an owner and validation boundary.
- Safe defaults MUST be explicit.
- Configuration rollback semantics MUST be defined for high-impact controls.
- Secrets MUST NOT be represented as ordinary configuration values; secret ownership and retrieval belong to the security/deployment contract.
- A feature flag MUST NOT create an undeclared alternative architecture. If both states are materially different architectures, both states must be governed or the flag is invalid.

## 12.5 Runtime Lifecycle Contract

Architecture-significant component lifecycle behavior MUST be declared when startup, readiness, shutdown, draining, restart, or dependency order can affect correctness or availability.

For each relevant deployment/runtime unit, define:

```yaml
lifecycle_id: LIFE-FANOUT
component: COMP-FANOUT
startup:
  prerequisites: [...]
  safe_before_ready: false
readiness:
  condition: ...
  dependencies_required: [...]
shutdown:
  mode: graceful | immediate | stateful_handoff
  drain_condition: ...
  maximum_drain_time: ...
restart:
  recovery_source: ...
  duplicate_or_replay_handling: ...
```

Rules:

- Process existence is not equivalent to readiness.
- Readiness MUST represent the conditions required to receive intended production traffic.
- Graceful shutdown/drain semantics MUST be explicit when terminating in-flight work or connections could violate requirements.
- Startup ordering SHOULD be avoided as a hidden correctness dependency; where unavoidable, it MUST be declared.
- Stateful restart MUST identify the recovery source and any fencing/ownership-transfer mechanism required to prevent concurrent authority.
- Rolling deploys MUST be compatible with the lifecycle contract.

## 12.6 Operational Modes and Mode Transitions

Systems with materially different runtime behavior under failure, maintenance, overload, migration, or recovery SHOULD define explicit operational modes.

Possible modes include:

```text
STARTING
NORMAL
DEGRADED
READ_ONLY
OVERLOAD_PROTECTION
MAINTENANCE
FAILOVER
RECOVERY
MIGRATING
DRAINING
```

Only modes actually used by the architecture should be declared.

Template:

```yaml
mode_id: MODE-DEGRADED
entry_conditions:
  - ...
allowed_capabilities:
  - ...
disabled_capabilities:
  - ...
quality_changes:
  - ...
state_rules:
  - ...
exit_conditions:
  - ...
operator_actions:
  - ...
observability:
  - ...
```

Rules:

- Mode transitions that affect externally visible behavior MUST be deterministic or governed by an explicit decision policy.
- A degraded mode MUST state which guarantees remain and which are relaxed.
- Overload protection / load shedding MUST preserve declared priority and correctness rules.
- Maintenance mode MUST NOT silently bypass security, consistency, or audit invariants unless a time-bounded waiver explicitly permits it.
- Recovery from a degraded or failover mode MUST define convergence/reconciliation before returning to `NORMAL` where state may have diverged.
- Operational modes MUST correspond to failure, deployment, configuration, and observability contracts rather than existing only as runbook prose.

---

# 13. Failure Domain and Recovery Contract

The architecture MUST enumerate material failures.

Minimum categories, where applicable:

- component process failure;
- node/container failure;
- datastore failure;
- queue/broker failure;
- network partition;
- region failure;
- dependency timeout;
- external provider interruption;
- malformed external data;
- overload;
- deployment failure;
- schema incompatibility;
- corrupted state;
- client reconnect storm.

For each material failure:

```yaml
failure_id: FAIL-...
trigger: ...
affected_components: [...]
user_effect: ...
detect: ...
contain: ...
recover: ...
data_loss: none | bounded | possible
recovery_time_target: ...
fallback: ...
evidence: ...
```

---

## 13.1 Failure-Domain Independence

Two redundant components do not provide meaningful redundancy if they share the same failure domain.

The architecture MUST identify shared dependencies that defeat intended redundancy.

---

## 13.2 Recovery Source Rule

Every recoverable state MUST name its recovery source.

Examples:

- durable event log;
- authoritative database;
- upstream replay;
- replicated snapshot;
- deterministic recomputation.

If no recovery source exists, the architecture MUST state the loss boundary honestly.

## 13.3 Recovery Objectives and Disaster-Recovery Contract

When recovery objectives are material, declare them explicitly.

```yaml
recovery_objective_id: RECOV-PRIMARY
scope: ...
rto: ...
rpo: ...
recovery_source: ...
failover_mode: automatic | manual | none
restore_validation: ...
last_tested: ...
```

Definitions:

- **RTO** — maximum acceptable time to restore the declared capability.
- **RPO** — maximum acceptable amount of state/data loss measured in time or another explicit unit.

If RTO/RPO are not product requirements, the architecture MAY derive design targets, but they MUST be labelled as architecture decisions rather than governing facts.

Backup existence is not recovery evidence. Restore/failover procedures SHOULD be testable, and high-impact recovery claims SHOULD identify the evidence that demonstrates they work.

A region-level or site-level resilience claim MUST identify:

- control-plane dependencies;
- data replication/recovery path;
- DNS/routing/failover dependency;
- secret/configuration availability;
- operational authority for failover and failback.

---

# 14. Performance, Capacity, and Resource Budgets

## 14.0 Workload and Demand Model

Capacity, latency, and cost reasoning MUST be grounded in an explicit workload model when those qualities are material.

A workload profile SHOULD declare:

```yaml
workload_id: WORKLOAD-PEAK
source: governing requirement | measurement | forecast | assumption
time_horizon: ...
concurrency: ...
arrival_rate:
  steady: ...
  burst: ...
burst_duration: ...
payload_distribution: ...
read_write_mix: ...
fanout_factor: ...
geographic_distribution: ...
retention_effect: ...
growth_assumption: ...
seasonality: ...
background_work: ...
confidence: low | medium | high
```

Rules:

- Separate **observed**, **required**, **forecast**, and **assumed** demand.
- Do not use average traffic as a substitute for a peak or burst requirement.
- Cost and capacity models MUST reference the workload profile they use.
- Benchmarks MUST state which workload profile, or scaled analogue, they implement.
- If a scaling relationship is assumed, record the assumption explicitly.
- A workload change large enough to invalidate a budget or design-capacity margin MUST trigger architecture impact analysis.

The workload model is not a performance result. It defines the demand against which performance, capacity, and cost are evaluated.

Each material quality attribute MUST have a stable budget ID.

Examples:

```text
BUDGET-LAT-END2END
BUDGET-LAT-INGEST
BUDGET-CAP-CONNECTIONS
BUDGET-EVENT-THROUGHPUT
BUDGET-MEMORY
BUDGET-CPU
BUDGET-RECOVERY
```

Template:

```yaml
budget_id: BUDGET-LAT-GOAL
requirement: ARCH-REQ-LAT-GOAL-v1
metric: end_to_end_latency
target: p95 <= ...
scope: ...
measurement_boundary: ...
allocation:
  - owner: COMP-INGEST
    target: ...
  - owner: COMP-FANOUT
    target: ...
margin: ...
validation: ...
```

---

## 14.1 Headroom Rule

Capacity architecture SHOULD not target exactly the nominal peak.

The contract SHOULD declare:

- nominal peak;
- design capacity;
- expected headroom;
- saturation indicator;
- degradation point.

---

## 14.2 Quantile Integrity Rule

Component percentile latencies MUST NOT be summed and mislabeled as an end-to-end percentile unless mathematically justified.

The architecture SHOULD instead:

- measure end-to-end directly where possible;
- use conservative bounds;
- use simulation/modeling with stated assumptions;
- treat sub-budgets as engineering allocations, not probabilistic identities.

---

## 14.3 Load-Generator Honesty

POCs and benchmarks MUST distinguish system saturation from generator saturation.

Host limitations such as:

- file descriptor limits;
- ephemeral ports;
- CPU scheduler saturation;
- loopback networking;
- client process limits;

must be disclosed when material.

---

# 15. Availability and Service-Level Contract

If availability or continuity is a requirement, declare:

- service-level indicator;
- target;
- measurement window;
- excluded/ included failure classes;
- failover behavior;
- maintenance behavior;
- deployment behavior;
- degraded modes.

Terms such as:

```text
highly available
fault tolerant
zero downtime
```

are prohibited as sole requirements without operational definition.

## 15.1 SLI / SLO / Error-Budget Contract

Where service levels matter, distinguish:

```text
SLI = what is measured
SLO = target for the SLI
SLA = external contractual commitment, if any
```

A project MAY define an operational error budget:

```yaml
slo_id: SLO-LIVE-DELIVERY
sli: successful_live_event_delivery
target: ...
window: ...
error_budget: ...
burn_alerts: ...
owner: ...
```

Architecture MUST NOT invent an SLA where none exists.

If an error budget is used, the architecture SHOULD state what operational or delivery decisions it informs rather than treating it as a decorative metric.

---

# 16. Security and Trust Boundaries

EQC-AC MUST identify trust boundaries when the system crosses them.

For each boundary declare:

- caller identity expectations;
- authentication requirement;
- authorization requirement;
- data classification;
- encryption requirement;
- rate/abuse controls;
- validation/sanitization owner;
- secret-handling owner.

Architecture-level security MUST include public exposure and abuse considerations where applicable.

Detailed threat modeling MAY live in a separate governed document.

## 16.1 Privacy, Compliance, and Data-Residency Contract

When applicable, the architecture MUST identify requirements that constrain where data may flow, where it may persist, and who may access it.

Potential concerns include:

- personal information / PII;
- health/financial/regulatory data;
- data residency;
- retention/deletion;
- auditability;
- legal hold;
- cross-border transfer;
- encryption/key ownership;
- least privilege;
- privileged administrative access;
- third-party subprocessors.

Template:

```yaml
governance_constraint_id: GOV-DATA-RESIDENCY
applies_to:
  - DATA-...
requirement_source: ...
allowed_locations: [...]
forbidden_locations: [...]
controls:
  - ...
evidence:
  - ...
```

Compliance requirements MUST come from an identified governing source. An architecture agent MUST NOT invent regulatory obligations.

---

# 17. Observability and Operability Contract

Each critical architecture claim SHOULD have a corresponding runtime signal.

The architecture MUST define ownership for material:

- request/event latency;
- error rate;
- throughput;
- queue lag;
- dropped work;
- retry rate;
- saturation;
- connection count;
- state divergence;
- dependency health;
- deployment health;
- recovery status.

For critical flows, trace/correlation identifiers SHOULD cross component boundaries.

Observability MUST serve architecture validation and operations, not exist as decorative tooling.

## 17.1 Operational Ownership and Runbook Binding

Every production-critical component or capability SHOULD have an operational owner.

For material failure modes, the architecture SHOULD identify:

- detection signal;
- first-response owner;
- runbook/playbook reference;
- escalation path;
- safe manual intervention boundary;
- rollback/failback authority.

Template:

```yaml
operational_capability_id: OPS-MATCH-FANOUT
owner: TEAM-...
alerts:
  - ALERT-...
runbooks:
  - RUNBOOK-...
manual_actions:
  allowed: [...]
  forbidden: [...]
```

Architecture MUST distinguish the **data plane** that serves product traffic from the **control plane** that configures, deploys, or operates it when a control-plane failure could affect availability or recovery.

A critical runtime path SHOULD NOT depend synchronously on a control-plane operation unless that dependency is explicitly justified.

---

# 18. Cost and Resource Economics

If cost is material, EQC-AC MUST include a cost model.

Required:

- pricing source;
- date checked;
- region;
- workload assumptions;
- unit prices;
- major drivers;
- fixed vs variable cost;
- egress / transfer;
- replication;
- observability;
- headroom;
- expected peak cost;
- sensitivity to dominant variables.

Template:

```yaml
cost_model:
  currency: USD
  pricing_date: YYYY-MM-DD
  region: ...
  workload:
    ...
  items:
    - id: COST-FANOUT
      driver: ...
      unit_price: ...
      source: ...
      monthly_estimate: ...
  total_peak_month: ...
  uncertainty: ...
```

---

## 18.1 No Convenient Omission Rule

A cost model MUST NOT omit a dominant cost category merely to satisfy a budget.

If a cost cannot be estimated, mark it `UNKNOWN` and determine whether that blocks approval.

---

# 19. External Dependency and Provider Contract

For every architecture-critical external dependency declare:

```yaml
external_id: EXT-FEED
provider: ...
capabilities_used:
  - ...
documented_guarantees:
  - ...
assumptions:
  - ...
rate_limits:
  - ...
failure_modes:
  - ...
recovery_capabilities:
  - ...
data_contract:
  - ...
cost_dependency:
  - ...
exit_strategy: ...
evidence_refs:
  - EVID-ARCH-...
last_verified: YYYY-MM-DD
```

Architecture-critical external facts SHOULD bind to the evidence/provenance rules in §32 rather than relying on untracked links or remembered vendor behavior.

---

## 19.1 No Invented Provider Guarantees

Do not assume:

- retries;
- replay;
- idempotency;
- ordering;
- uniqueness;
- regional availability;
- SLA;
- compatibility guarantees;

unless they are supported by authoritative evidence or explicitly declared as assumptions.

---

# 20. Architecture Decision Protocol

Every material architecture decision MUST have a stable decision ID.

Recommended format:

```text
ADR-ARCH-###
```

Minimum decision record:

```yaml
decision_id: ADR-ARCH-012
question: ...
hard_constraints:
  - ...
soft_preferences:
  - ...
candidates:
  - A
  - B
  - C
selected: B
reason: ...
evidence: [...]
assumptions: [...]
risks: [...]
rejected:
  A: ...
  C: ...
revisit_trigger: ...
```

---

## 20.1 Hard-Gate First

Candidates that violate a hard requirement MUST be:

```text
REJECTED
or
REDESIGN_REQUIRED
```

They MUST NOT remain preferred because they score well on softer criteria.

---

## 20.2 Composition-Aware Comparison

After hard-gate filtering, compare candidates under the full relevant system composition.

For candidate `C`, define a qualitative or quantitative evaluation vector:

```text
V(C) =
[
  correctness,
  latency,
  throughput,
  surge behavior,
  consistency,
  recovery,
  deployment compatibility,
  geographic behavior,
  security,
  operability,
  cost,
  complexity
]
```

Not every dimension must be numeric.

The purpose is to prevent:

```text
best local component
```

from being assumed to imply:

```text
best composed architecture
```

---

## 20.3 Preference-Reversal Check

For each consequential choice, ask:

```text
Does the preferred option change after adding the surrounding
constraints and interactions?
```

If yes, the composed result governs.

---

## 20.4 Dominance / Retention Rule

A candidate may be pruned when another candidate is:

- no worse on every material dimension;
- strictly better on at least one;
- and there is no unresolved interaction likely to reverse the result.

Non-dominated candidates SHOULD remain until the decisive system-level trade-off is understood.

## 20.5 Sensitivity and Trade-Off Point Registry

For architecture-significant decisions, identify variables or choices whose small change can materially change quality outcomes.

A **sensitivity point** is an architecture parameter or decision with disproportionate influence on one or more quality attributes.

A **trade-off point** is a sensitivity point that materially affects multiple quality attributes in competing directions.

Template:

```yaml
analysis_point_id: ATP-FANOUT-BUFFER
kind: sensitivity | tradeoff
decision_or_parameter: fanout_buffer_limit
affected_quality_attributes:
  - latency
  - loss_behavior
  - memory
  - recovery
direction:
  larger_buffer:
    improves:
      - transient_burst_absorption
    worsens:
      - memory
      - stale_delivery_risk
known_thresholds:
  - ...
evidence:
  - VAL-ARCH-...
revisit_trigger: ...
```

Rules:

- Material sensitivity/trade-off points SHOULD be linked to the decision log and risk ledger.
- A critical threshold SHOULD be validated or explicitly labelled as an assumption.
- A decision that is robust only within a narrow parameter range SHOULD expose that range rather than present the decision as universally safe.
- Sensitivity analysis MAY be qualitative when reliable quantitative models are unavailable.

This supplements composition-aware comparison by identifying **which assumptions and parameters actually control the architecture outcome**.

---

## 20.6 Revisit Trigger

Every material decision SHOULD state what new evidence would reopen it.

Examples:

- traffic exceeds threshold;
- provider changes API guarantee;
- cost doubles;
- benchmark misses budget;
- region added;
- consistency requirement tightened.

Architecture is governed evolution, not permanent doctrine.

## 20.7 Technology Choice and Lifecycle Contract

A technology choice becomes architecture-significant when changing it would materially affect:

- interfaces;
- state model;
- topology;
- quality budgets;
- operational model;
- security/compliance;
- cost;
- downstream FICs.

Architecture-significant technology decisions SHOULD declare:

```yaml
technology_decision_id: TECH-ARCH-...
capability: ...
selected: ...
version_or_service_class: ...
reason: ...
portability_boundary: ...
support_lifecycle: ...
known_limits: [...]
exit_or_migration_trigger: ...
```

Avoid both extremes:

```text
technology is completely unspecified
```

when its properties are required by the architecture, and:

```text
every library/version is architectural
```

when it is merely a file-level implementation detail.

Deprecated, end-of-support, or materially changed external technology MUST trigger architecture impact analysis when the architecture depends on its behavior.

## 20.8 Reference Architecture and Pattern Adoption

A project MAY adopt an external or internal reference architecture, pattern, platform blueprint, or organizational standard.

Adoption MUST NOT turn an external pattern into hidden authority.

For each materially adopted source, record:

```yaml
reference_architecture_id: REFARCH-...
name: ...
source: ...
version_or_date: ...
adoption_scope:
  - ...
inherited_constraints:
  - ...
intentional_deviations:
  - id: DEV-...
    reason: ...
assessed_effect: ...
validation:
  - ...
```

Rules:

- The concrete EQC-AC architecture remains authoritative for the governed system.
- Imported patterns MUST be translated into explicit components, interfaces, constraints, decisions, or principles where they materially affect the system.
- A pattern name such as `event-driven`, `hexagonal`, `CQRS`, `microservices`, or `zero trust` is not a complete architecture decision by itself.
- Deviations from an adopted mandatory organizational/reference architecture MUST be explicit.
- Changes to an external reference architecture do not silently mutate the approved concrete architecture; they trigger impact review.

---

# 21. Assumption Ledger

Every architecture-critical assumption MUST have a stable ID.

Template:

```yaml
assumption_id: ASM-ARCH-...
statement: ...
supports:
  - ADR-ARCH-...
  - COMP-...
confidence: low | medium | high
impact_if_false: low | medium | high | architecture_invalidating
evidence:
  - ...
testability: local | staging | production | external_only
status: open | supported | falsified | accepted_risk | retired
owner: ...
revisit_trigger: ...
```

---

## 21.1 Architecture-Invalidating Assumption Rule

If an assumption is both:

```text
high impact
and
low confidence
```

it MUST appear in the risk ledger and validation plan.

Where practical, it SHOULD become a proof-of-concept or experiment target before architecture approval.

---

# 22. Risk Ledger and Falsification Protocol

Architecture risk SHOULD be ranked using at least:

- impact if false;
- uncertainty;
- detectability;
- local/staging testability;
- cost of late discovery.

A useful selection model:

```text
RiskPriority =
  ArchitecturalImpact
  × Uncertainty
  × LateDiscoveryCost
```

Exact arithmetic is optional. Relative ranking is mandatory for material risks.

---

## 22.1 Falsification Before Promotion

Before promoting an architecture, actively ask:

- What assumption would invalidate this design?
- What dependency is least trustworthy?
- What happens under peak and surge?
- What happens during mixed-version deployment?
- What happens when the external provider misbehaves?
- What happens under duplication, delay, reordering, or partial loss?
- What is the actual recovery source?
- Which cost term dominates at scale?
- Which claim is only inferred rather than measured?
- Which component becomes the bottleneck first?
- Which failure domain defeats redundancy?
- What can create silent state divergence?

Unanswered high-impact questions block validation or require a waiver.

---

# 23. Architecture Validation Plan

Every approved architecture MUST define how major claims are validated.

Validation methods include:

```text
STATIC_REVIEW
REQUIREMENT_TRACE
DEPENDENCY_LINT
INTERFACE_LINT
CAPACITY_CALCULATION
COST_CALCULATION
BENCHMARK
POC
LOAD_TEST
FAILURE_INJECTION
REPLAY
GOLDEN_TRACE
SECURITY_REVIEW
VENDOR_DOCUMENTATION
PRODUCTION_OBSERVATION
```

Each validation entry:

```yaml
validation_id: VAL-ARCH-...
claim: ...
method: ...
environment: ...
acceptance_criteria: ...
evidence_artifact: ...
status: planned | passed | failed | inconclusive | waived
```

---

## 23.1 Pre-Evidence Criteria Rule

When validation produces measured evidence, acceptance criteria SHOULD be frozen before the final measurement.

Changing criteria after observing results creates a new validation run.

---

## 23.2 Measurement Boundary Rule

Each measured architecture metric MUST state its exact boundary.

Examples:

```text
ingest -> queue publish
queue publish -> client receive
client receive -> browser render
end-to-end
```

A partial measurement MUST NOT be promoted to an end-to-end claim.

## 23.3 Architecture Fitness Functions

Architecture-critical invariants and budgets SHOULD be converted into continuously executable checks where practical.

Examples:

- dependency graph has no forbidden edge;
- only declared writers mutate an authoritative state domain;
- schema compatibility remains valid;
- p95 latency remains within an architecture budget;
- cost forecast remains below an approved threshold;
- replica placement preserves failure-domain separation;
- dependency versions remain supported;
- recovery test succeeds within target.

A fitness function MUST bind to a declared architecture requirement, invariant, budget, or decision. It SHOULD NOT create a new hidden requirement merely because tooling can measure it.

Recommended status:

```text
PASS
FAIL
INCONCLUSIVE
NOT_RUN
```

A failing architecture fitness function MUST identify the affected architecture IDs and trigger impact analysis.

---

# 24. Architecture Readiness Gate

Before an architecture reaches `approved`, it MUST pass the readiness gate.

Possible statuses:

```text
ARCH_READY
ARCH_READY_WITH_WAIVERS
ARCH_BLOCKED_MISSING_REQUIREMENT
ARCH_BLOCKED_UNOWNED_STATE
ARCH_BLOCKED_UNDEFINED_INTERFACE
ARCH_BLOCKED_HIDDEN_DEPENDENCY
ARCH_BLOCKED_BUDGET
ARCH_BLOCKED_EXTERNAL_ASSUMPTION
ARCH_BLOCKED_FAILURE_GAP
ARCH_BLOCKED_COST_UNKNOWN
ARCH_BLOCKED_SECURITY_BOUNDARY
ARCH_BLOCKED_UNVALIDATED_CRITICAL_ASSUMPTION
ARCH_BLOCKED_TRACEABILITY_GAP
```

---

## 24.1 Mandatory Readiness Checks

The architecture MUST pass:

1. System boundary is explicit.
2. Hard requirements are indexed.
3. Every hard requirement has an architecture response or explicit external owner.
4. Every component has a justified purpose.
5. Every cross-component path is declared.
6. Public architectural interfaces have an owner.
7. Authoritative state has an owner.
8. No unauthorized multiple writers exist.
9. Dependency graph contains no unexplained cycle.
10. Failure domains are declared.
11. Material failure modes have recovery or explicit loss semantics.
12. Deployment topology is defined.
13. Quality-attribute budgets exist where required.
14. Critical external assumptions are identified.
15. Cost/resource constraints are evaluated.
16. Trust boundaries are identified.
17. Critical architecture decisions have alternatives/rationale.
18. High-impact low-confidence assumptions have validation or waiver.
19. Downstream subsystem/FIC constraints are derivable.
20. Architecture version and approval state are frozen.
21. Material stakeholder concerns are represented or explicitly out of scope.
22. Required architecture views correspond to the same governed model.
23. Material quality attributes have operational scenarios or explicit rationale.
24. Material data domains have ownership and lifecycle rules where required.
25. Time/concurrency/transaction semantics are declared where correctness depends on them.
26. RTO/RPO or explicit recovery targets exist where recovery objectives are material.
27. Privacy/compliance/residency constraints are represented where applicable.
28. Production-critical capabilities have operational ownership.
29. Architecture-significant technology choices have lifecycle/revisit triggers where material.
30. Critical architecture fitness checks are identified where continuous validation is practical.
31. Architecture invariants have stable IDs and authoritative ownership where downstream bindings depend on them.
32. Capacity/cost/performance claims reference an explicit workload model where demand is material.
33. Architecture-significant environment differences and dynamic configuration controls are governed.
34. Contradictory hard requirements have been resolved, waived, or explicitly block approval.
35. Transitional/migration architecture is defined when target-state change is non-atomic.
36. Governance roles and approval/waiver authority are declared for material decisions.
37. Architecture semantic version impact is determined and downstream stale bindings are identified.
38. Controlled vocabulary/model conventions eliminate material semantic ambiguity.
39. Subsystem handoff boundaries are explicit when subsystem documents are used.
40. Applicability/conformance status distinguishes irrelevant concerns from unresolved omissions.
41. Required architecture views declare model kinds/notation semantics and cross-model correspondences where needed.
42. Material sensitivity/trade-off points are identified for decisions whose outcome depends strongly on parameters or competing quality attributes.
43. Adopted reference architectures/patterns are translated into explicit local constraints/decisions and material deviations are recorded.
44. Material cross-cutting policies have one architecture-level source of truth and explicit exceptions.
45. Runtime startup/readiness/shutdown/drain semantics are defined where they affect correctness or availability.
46. Material degraded/maintenance/failover/recovery modes and their transition semantics are governed where used.
47. Architecture-critical mutable evidence has provenance, applicability context, and freshness/revalidation rules.
48. Stale, superseded, disputed, or unavailable evidence has been propagated to affected decisions, budgets, assumptions, and downstream bindings.

---

# 25. Architecture Lockfile

When architecture is approved for implementation, create a lockfile.

Example:

```yaml
eqc_ac_lock_version: 1
architecture_id: ARCH-MATCH-CENTRE
architecture_version: v1.0.0
status: ARCH_READY
created_at: YYYY-MM-DD

digests:
  architecture_document: sha256:...
  component_registry: sha256:...
  interface_registry: sha256:...
  dependency_graph: sha256:...
  deployment_topology: sha256:...
  requirement_index: sha256:...
  budget_ledger: sha256:...
  assumption_ledger: sha256:...
  decision_log: sha256:...
  validation_report: sha256:...

downstream_authority:
  fic_must_reference_architecture_version: true
```

FIC generation SHOULD be blocked against a stale architecture lock unless the architecture explicitly permits compatible downstream work.

---

# 26. Downstream Handoff Contract

EQC-AC MUST provide enough information for downstream documents to be generated without architectural invention.

For each subsystem / FIC-bound unit, architecture MUST expose:

- component ID;
- responsibility;
- non-responsibility;
- allowed dependencies;
- forbidden dependencies;
- owned interfaces;
- consumed interfaces;
- state ownership;
- invariants;
- security zone;
- performance/resource budgets;
- failure obligations;
- observability obligations;
- deployment constraints;
- architecture decision references;
- architecture version.

A FIC SHOULD contain:

```text
Architecture Binding:
  ARCH-...@vX.Y.Z
  Component: COMP-...
  Interfaces: IFACE-...
  State: STATE-...
  Budgets: BUDGET-...
  Decisions: ADR-ARCH-...
```

A FIC MUST NOT override architecture-level ownership or topology unless an architecture delta is approved.

## 26.1 Subsystem Contract Boundary

EQC-AC may hand architecture responsibility to a subsystem document before FIC generation.

A subsystem contract SHOULD contain, at minimum:

- governing architecture/component IDs;
- subsystem purpose and non-goals;
- public subsystem interfaces;
- internal state domains inherited from or delegated by architecture;
- allowed/forbidden external dependencies;
- subsystem invariants;
- subsystem quality budgets;
- failure/recovery obligations;
- security/data obligations;
- observability obligations;
- downstream unit/FIC decomposition.

A subsystem contract MAY refine internal structure. It MUST NOT:

- create a new cross-system dependency;
- move authoritative state ownership across architecture components;
- weaken an architecture invariant;
- change an architecture-level interface;
- alter a hard architecture budget;

without an approved architecture delta.

This creates the governed chain:

```text
EQC-AC
  -> SUBSYSTEM contract (optional for non-trivial components)
  -> bounded unit/pseudocode
  -> EQC-FIC
  -> code
```

---

# 27. Relationship to EQC-SIB

EQC-SIB remains the bidirectional bridge between documents and implementation artifacts.

EQC-AC adds architecture semantics to the document side of that bridge.

Implementation changes that materially alter:

- component boundaries;
- dependencies;
- public architecture interfaces;
- state ownership;
- deployment topology;
- consistency model;
- budgets;
- security boundaries;

MUST trigger architecture impact analysis.

The implementation MUST NOT become the de facto architecture simply because it exists.

---

# 28. Relationship to EQC-ES

EQC-ES governs:

- architecture document registration;
- versioning;
- dependency edges;
- compatibility;
- change propagation;
- digests;
- portfolio reachability.

EQC-AC governs:

- the architecture content and architecture-specific validation rules.

An EQC-AC document SHOULD be part of the EQC-ES graph and MUST participate in change propagation.

---

# 29. Change Classification

Architecture changes MUST be classified.

Recommended classes:

```text
ARCH_METADATA
ARCH_COMPATIBLE
ARCH_STRUCTURAL
ARCH_BREAKING
ARCH_EMERGENCY
```

### ARCH_METADATA
No functional or structural effect.

Examples:

- wording;
- diagram layout;
- typo;
- clarified rationale with no changed decision.

### ARCH_COMPATIBLE
Architecture behavior is extended without breaking existing downstream contracts.

Examples:

- additive optional interface field;
- extra replica with unchanged ownership;
- observability improvement.

### ARCH_STRUCTURAL
Component topology or deployment changes, but governed public behavior may remain compatible.

Examples:

- split service;
- merge services;
- move state store;
- change region topology.

### ARCH_BREAKING
Downstream contracts or system guarantees change.

Examples:

- state owner changes;
- interface compatibility breaks;
- consistency model changes;
- required dependency removed;
- latency/cost guarantee weakened.

### ARCH_EMERGENCY
Temporary exceptional architecture change required to restore service or contain risk.

Emergency changes MUST be documented retroactively within the declared governance window and either normalized or rolled back.

## 29.1 Architecture Semantic Versioning

Architecture versions SHOULD use semantic versioning:

```text
MAJOR.MINOR.PATCH
```

Recommended mapping:

- `PATCH` — `ARCH_METADATA` and clarification-only changes with no changed architectural contract.
- `MINOR` — `ARCH_COMPATIBLE` changes and some `ARCH_STRUCTURAL` changes that preserve all downstream compatibility contracts.
- `MAJOR` — `ARCH_BREAKING` changes or structural changes that invalidate downstream bindings, ownership, interfaces, consistency, topology assumptions, or required migration behavior.

`ARCH_EMERGENCY` describes urgency, not compatibility severity. An emergency change still receives PATCH/MINOR/MAJOR impact according to its actual contract effect.

Rules:

- Version impact MUST be based on downstream observable architectural meaning, not author intent.
- Any FIC bound to an incompatible architecture version becomes stale.
- A version bump MUST update the architecture lockfile after approval.
- Migration from one MAJOR architecture version to another SHOULD use the transitional architecture contract in §30.1.

---

# 30. Change Propagation

A material architecture change MUST identify impact on:

- subsystem documents;
- shared interfaces;
- unit DAG;
- pseudocode;
- EQC-FIC documents;
- implementation artifacts through EQC-SIB;
- tests;
- traces;
- deployment config;
- cost model;
- validation plan;
- risk ledger.

Example:

```text
Change:
  COMP-MATCH-STATE no longer owns snapshot persistence

Impact:
  STATE-MATCH-SNAPSHOT ownership changes
  interface contract changes
  dependency graph changes
  relevant FICs stale
  recovery validation stale
  cost model changes
```

No architecture change is complete until downstream stale artifacts are identified.

## 30.1 Transitional and Migration Architecture

When the approved architecture cannot change atomically, EQC-AC MUST govern the transition as explicitly as the target state.

A migration record SHOULD define:

```yaml
migration_id: MIG-ARCH-...
from_architecture: ARCH-...@v1
to_architecture: ARCH-...@v2
stages:
  - id: STAGE-1
    topology: ...
    temporary_dependencies: [...]
    temporary_state_owners: [...]
    compatibility_rules: [...]
entry_criteria: ...
exit_criteria: ...
data_migration: ...
dual_read_or_write: ...
cutover: ...
rollback: ...
decommission: ...
maximum_transition_window: ...
validation: [...]
```

Rules:

- Temporary dependencies and temporary multiple-writer arrangements MUST be declared; they are not exempt from governance.
- Dual-write/dual-read migrations MUST define divergence detection and reconciliation.
- Data migration MUST define source of truth, verification, retry/idempotency, and rollback or forward-only semantics.
- Cutover criteria MUST be measurable where practical.
- Deprecated components/interfaces MUST have a decommission condition.
- A target architecture MUST NOT be marked fully active while the system is materially operating in an undocumented transitional topology.

A migration architecture MAY relax a target-state invariant only through an explicit time-bounded migration waiver with compensating controls.

---

# 31. Architecture Drift Detection

Architecture drift exists when implementation or operational reality differs from the approved architecture contract.

Examples:

- unregistered dependency appears;
- new shared datastore is accessed directly;
- a component mutates state it does not own;
- deployment topology changes;
- actual retry behavior differs;
- public interface changes;
- cost driver changes materially;
- production saturation point invalidates capacity model.

EQC-AC validation SHOULD include drift checks where practical.

---

# 32. Architecture Evidence Levels

Architecture claims SHOULD declare evidence strength.

Recommended scale:

```text
E0 — assertion only
E1 — authoritative documentation / reviewed calculation
E2 — local or synthetic experiment
E3 — staging / representative environment evidence
E4 — production observation
E5 — repeated production evidence across expected operating envelope
```

Higher evidence does not automatically mean a better architecture. It communicates confidence and provenance.

## 32.1 Evidence Provenance Registry

Every architecture-critical evidence item SHOULD have a stable ID when it supports a hard requirement, architecture decision, budget, provider capability, compliance constraint, or validation conclusion.

Template:

```yaml
evidence_id: EVID-ARCH-AWS-WS-PRICING
claim: ...
evidence_type:
  authoritative_documentation | regulation | contract | measurement |
  benchmark | calculation | trace | production_observation | review
source_owner: ...
source_reference: ...
source_version_or_date: ...
retrieved_or_observed_at: YYYY-MM-DD
applicable_region: ...
applicable_environment: ...
applicable_version: ...
authority_level: primary | secondary | internal_observation | inference
evidence_level: E0 | E1 | E2 | E3 | E4 | E5
supports:
  - ADR-ARCH-...
  - BUDGET-...
  - EXT-...
  - ARCH-REQ-...
limitations:
  - ...
freshness_policy: ...
status: current | stale | superseded | disputed | unavailable
```

Rules:

- Evidence MUST be traceable to what was actually observed or published.
- A URL alone is not sufficient provenance when version, date, region, edition, or environment materially changes the meaning.
- Primary/authoritative sources SHOULD be preferred for mutable external facts.
- Secondary sources MAY support discovery or context but SHOULD NOT be the sole basis for a high-impact mutable claim when an authoritative source exists.
- Measurements MUST identify environment, workload, boundary, and relevant tool/runtime version.
- Calculations MUST identify their input evidence and assumptions.
- An inference MUST remain labelled as an inference even when supported by strong evidence.
- Evidence supporting a hard constraint MUST NOT silently disappear from the review packet.

## 32.2 Evidence Freshness and Expiry

Evidence strength and evidence freshness are independent.

Examples:

```text
an authoritative cloud-pricing page may be strong but stale;
a production benchmark may be recent but apply to the wrong workload;
a regulation may remain authoritative for years;
a service quota may change by region or account.
```

Each mutable architecture-critical evidence item SHOULD declare either:

```text
freshness_policy: stable_until_superseded
```

or a revalidation rule such as:

```yaml
freshness_policy:
  max_age_days: 30
  revalidate_on:
    - provider_version_change
    - region_change
    - workload_change
    - architecture_major_change
```

Evidence MUST be revalidated when a known change can materially affect the architecture claim it supports.

Examples include:

- pricing changes;
- quota/limit changes;
- provider SLA changes;
- service capability changes;
- framework/runtime version changes;
- region availability changes;
- security/compliance rule changes;
- workload changes;
- benchmark environment changes.

An expired evidence item does not automatically make the architecture wrong. It makes the dependent claim **stale until revalidated**.

## 32.3 Evidence Invalidation and Propagation

When architecture-critical evidence becomes `stale`, `superseded`, `disputed`, or `unavailable`, the system MUST identify affected architecture artifacts.

Propagation SHOULD follow:

```text
evidence item
  -> supported fact / assumption / measurement
  -> architecture decision / requirement / budget / provider contract
  -> component/interface/topology implications
  -> subsystem / FIC bindings
  -> implementation / validation artifacts through EQC-SIB
```

Possible resulting statuses include:

```text
NO_ARCHITECTURE_IMPACT
REVALIDATION_REQUIRED
DECISION_REOPEN_REQUIRED
BUDGET_RECALCULATION_REQUIRED
PROVIDER_ASSUMPTION_REOPEN_REQUIRED
ARCHITECTURE_BLOCKED
```

A changed external fact MUST NOT silently mutate the approved architecture. It triggers impact analysis first.

## 32.4 Evidence Conflict Rule

If two credible evidence items materially disagree:

1. do not silently choose the more convenient result;
2. record both;
3. compare scope, version, region, environment, date, and measurement boundary;
4. determine whether the disagreement is resolvable;
5. if architecture outcome depends on the unresolved difference, mark the affected claim `DISPUTED` and block approval or require a waiver.

Evidence conflict is an architecture uncertainty, not a documentation formatting issue.

---

# 33. Unknowns and Waivers

Unknown architecture information MUST NOT be silently guessed.

For each `UNKNOWN`:

```yaml
unknown_id: UNK-ARCH-...
question: ...
impact: ...
owner: ...
resolution_evidence: ...
deadline: ...
blocking: true | false
```

A waiver MUST include:

- requirement being waived;
- reason;
- approver;
- duration;
- risk;
- compensating control;
- revisit condition.

## 33.1 Governance Roles and Approval Authority

A governed architecture SHOULD declare who may author, review, approve, waive, freeze, and reopen architecture decisions.

Recommended roles:

```text
ARCHITECTURE_OWNER
REQUIREMENT_OWNER
SECURITY_REVIEWER
OPERATIONS_REVIEWER
DATA/PRIVACY_REVIEWER
COST_OWNER
IMPLEMENTATION_OWNER
WAIVER_APPROVER
RELEASE_APPROVER
```

One person MAY hold multiple roles in a small project.

Template:

```yaml
governance:
  architecture_owner: ...
  approvers: [...]
  waiver_authority:
    security: ...
    cost: ...
    availability: ...
  required_reviewers:
    - ...
  reopen_rules:
    - trigger: ...
      authority: ...
```

Rules:

- A reviewer MUST NOT implicitly change a governing requirement; the requirement owner must approve that change.
- Waiver authority MUST be explicit for high-impact constraints.
- Approval status MUST identify the architecture version being approved.
- Emergency changes MAY use an expedited authority path, but must still produce an auditable record and normalization/rollback plan.

---

# 34. Architecture Review Packet

Before approval, a reviewer SHOULD be able to inspect a bounded packet containing:

```text
1. System goal / requirements
2. Main EQC-AC document
3. Architecture diagram
4. Requirement coverage report
5. Component registry
6. Interface registry
7. Dependency graph
8. State ownership map
9. Deployment topology
10. Budget ledger
11. Decision log
12. Assumption + risk ledger
13. Validation report
14. Open waivers / unknowns
15. Stakeholder / concern registry
16. Required architecture views
17. Data lifecycle / governance constraints where applicable
18. Operational ownership and recovery objectives where applicable
19. Applicability / conformance statement
20. Model-kind / notation / correspondence definitions where used
21. Sensitivity / trade-off points for material decisions
22. Adopted reference architectures/patterns and deviations where used
23. Cross-cutting architecture-policy registry where material
24. Runtime lifecycle / operational-mode contracts where material
25. Evidence provenance/freshness registry for architecture-critical mutable facts
26. Stale/disputed evidence and impact dispositions
```

Review SHOULD not require reconstruction from chat history.

---

# 35. Lint Rules

An EQC-AC linter SHOULD fail on:

1. duplicate component IDs;
2. duplicate state ownership;
3. unowned state;
4. undeclared cross-component dependency;
5. interface with no owner;
6. interface consumer referencing missing interface;
7. dependency edge with no interaction;
8. unapproved cycle;
9. component with no requirement/decision binding;
10. hard requirement with no architecture response;
11. budget with no owner;
12. hard budget with no validation path;
13. external dependency with undocumented assumption set;
14. architecture-invalidating assumption with no risk entry;
15. approved architecture with unresolved blocker;
16. FIC binding to stale architecture version;
17. hidden direct datastore access;
18. mixed-version deployment requirement with no compatibility rule;
19. redundant components sharing undeclared common failure domain;
20. recovery claim with no recovery source;
21. stateful component with no recovery semantics;
22. cost-constrained architecture with no cost model;
23. public entry point with no trust/security boundary;
24. architecture decision with no alternatives when the choice is material;
25. implementation-critical technology choice with no rationale;
26. measurement claim with unspecified boundary;
27. material concern with no architecture response or explicit disposition;
28. architecture views that contradict registries/topology;
29. material quality attribute expressed only as an undefined adjective;
30. authoritative/regulated data with no required lifecycle owner;
31. time-dependent correctness with no clock/time semantics;
32. multi-component mutation workflow claiming transactionality with no transaction/convergence boundary;
33. recovery target with no declared RTO/RPO or equivalent measure when required;
34. data-residency/compliance constraint with no source or enforcement binding;
35. production-critical capability with no operational owner;
36. architecture-significant technology with no decision/revisit trigger when lifecycle risk is material;
37. continuously enforceable critical invariant that is claimed enforced but has no fitness/check binding;
38. downstream FIC references an undefined architecture invariant ID;
39. performance/cost/capacity claim has no workload profile where workload is material;
40. architecture-significant environment difference is undeclared;
41. dynamic configuration can change architecture behavior without owner/validation/rollback semantics;
42. contradictory hard requirements are silently traded off rather than blocked/resolved;
43. non-atomic architecture change has no governed transitional architecture;
44. dual-write/dual-read migration has no divergence/reconciliation rule;
45. approved architecture has no identifiable approval authority/version record;
46. architecture version impact is incompatible with downstream bindings but stale artifacts are not reported;
47. model terminology creates materially ambiguous ownership/interface/state meaning;
48. subsystem contract overrides architecture-level ownership/interface/budget without approved architecture delta;
49. conformance claim omits exact EQC-AC/architecture versions or uses NOT_APPLICABLE without rationale;
50. required view uses undefined or inconsistent model semantics;
51. cross-model element/relationship cannot map to the governed registries;
52. material architecture decision has an unrecorded sensitivity/trade-off threshold that can reverse the decision;
53. adopted reference architecture/pattern introduces material constraints or dependencies that are not represented in the concrete EQC-AC contract;
54. cross-cutting behavior is independently redefined across components without an authoritative policy or explicit exception;
55. framework/middleware default materially affects system behavior but is not represented as architecture policy;
56. process is treated as ready before architecture-required dependencies/state are usable;
57. graceful shutdown/drain is required by availability/correctness constraints but has no lifecycle contract;
58. operational mode changes externally visible guarantees without declared entry/exit and quality semantics;
59. recovery from degraded/failover state can return to normal without required reconciliation/convergence evidence;
60. architecture-critical mutable fact has no recoverable evidence provenance;
61. evidence scope omits material region/version/environment context;
62. stale/superseded/disputed evidence still supports an approved architecture claim without revalidation, waiver, or impact analysis;
63. conflicting credible evidence is silently resolved without a recorded disposition;
64. architecture-critical calculation or benchmark cannot be traced to its input assumptions/workload/environment.

---

# 36. Minimum EQC-AC Template

A minimal conforming architecture contract MUST contain:

```markdown
# Architecture Identity

# System Context

# Stakeholders, Concerns, and Required Views

# Model Kinds / Notation / Correspondence Rules (when needed)

# Governing Requirements and Quality-Attribute Scenarios

# Controlled Vocabulary / Model Conventions (when needed)

# Architecture Overview

# Component Registry

# Architecture Invariant Registry

# Cross-Cutting Architecture Policies (when material)

# Interaction / Dependency Model

# Interface Ownership

# State Ownership and Consistency

# Data Lifecycle / Governance (when applicable)

# Time / Concurrency / Transaction Semantics (when applicable)

# Deployment Topology

# Environment / Dynamic Configuration Profiles (when architecture-significant)

# Runtime Lifecycle / Operational Modes (when material)

# Failure and Recovery Model

# Workload / Demand Model

# Performance / Capacity Budgets

# Security / Trust Boundaries

# Privacy / Compliance / Residency (when applicable)

# Observability and Operational Ownership

# Cost / Resource Model

# External Dependencies and Assumptions

# Architecture Decisions and Alternatives

# Sensitivity / Trade-Off Points (when material)

# Reference Architecture / Pattern Adoption (when used)

# Risk / Assumption Ledger

# Validation Plan and Evidence

# Evidence Provenance / Freshness / Invalidation (for mutable critical evidence)

# Downstream FIC Constraints

# Change / Version Governance

# Transitional / Migration Architecture (when required)

# Governance / Approval Authority

# Applicability / Conformance Statement

# Architecture Readiness Status
```

---

# 37. Recommended Full EQC-AC Document Skeleton

```markdown
# EQC-AC Architecture Contract

## 0. Identity
- Architecture ID
- System ID
- Architecture version
- EQC-AC version
- Owner
- Status
- Governing requirements
- Governing EQC / EQC-ES documents

## 1. System Goal and Non-Goals

## 2. System Context
- Actors
- External systems
- Trust boundaries
- Responsibility boundary

## 2A. Stakeholders, Concerns, and Views
- Stakeholder registry
- Concern registry
- Required viewpoints/views
- Model-kind/notation registry
- Cross-model correspondence rules

## 3. Requirement Index
- Functional
- Latency
- Capacity
- Availability
- Consistency
- Security
- Cost
- Deployment
- Compliance
- Quality-attribute scenarios

## 4. Architecture Overview
- Main diagram
- Architectural style
- Key decisions
- Controlled vocabulary / model conventions when needed

## 5. Component Contracts
For each component:
- ID
- Purpose
- Responsibilities
- Non-responsibilities
- Interfaces
- State
- Dependencies
- Scaling
- Failure behavior
- Security
- Observability

## 5A. Architecture Invariant Registry

## 5B. Cross-Cutting Architecture Policies

## 6. Interaction Contracts
- Data flows
- Control flows
- Sync/async
- Ordering
- Retry
- Backpressure
- Idempotency

## 7. Interface and Schema Ownership

## 8. State and Consistency
- State registry
- Owners
- Writers/readers
- Consistency
- Ordering
- Snapshot/live handoff
- Retention
- Recovery
- Time/clock semantics
- Concurrency/transaction boundaries

## 8A. Data Lifecycle and Governance
- Classification
- Retention/deletion
- Residency
- Backup/restore
- Lineage
- Derived/cache/ephemeral distinction

## 9. Dependency Graph
- Allowed edges
- Forbidden edges
- Cycles

## 10. Deployment Topology
- Runtime units
- Regions
- Replicas
- Failure domains
- Scaling
- Rollout
- Rollback
- Mixed-version behavior
- Environment profiles
- Architecture-significant dynamic configuration
- Runtime lifecycle: startup/readiness/shutdown/drain
- Operational modes and transitions

## 11. Failure and Recovery
- Failure matrix
- Data-loss boundaries
- Recovery sources
- Degraded modes
- RTO/RPO where material
- Disaster-recovery/failover path

## 11A. Workload / Demand Model

## 12. Performance and Capacity
- End-to-end budgets
- Component allocations
- Headroom
- Saturation
- Validation

## 13. Availability

## 14. Security and Trust
- Privacy/compliance/residency where applicable

## 15. Observability and Operability
- Operational ownership
- Runbook bindings
- Control-plane/data-plane dependencies

## 16. Cost Model

## 17. External Dependencies
- Guarantees
- Assumptions
- Limits
- Failure behavior

## 18. Decision Log
- Candidates
- Hard gates
- Composed comparison
- Sensitivity/trade-off points
- Selected option
- Revisit trigger
- Technology lifecycle where architecture-significant
- Reference architecture/pattern adoption where used

## 19. Assumption Ledger

## 20. Risk and Falsification Plan

## 21. Validation Plan and Evidence
- Architecture fitness functions where practical
- Evidence provenance registry
- Freshness/revalidation policy
- Evidence invalidation propagation
- Conflicting-evidence disposition

## 22. Downstream Contract
- Shared interfaces
- Subsystems
- FIC bindings

## 23. Version / Migration / Change Propagation
- Semantic version impact
- Transitional architecture
- Cutover/rollback/decommission

## 23A. Governance / Approval Authority

## 23B. Applicability / Conformance Statement

## 24. Readiness Gate
```

---

# 38. Example Architecture Binding into FIC

An EQC-FIC document generated under EQC-AC SHOULD include:

```yaml
architecture_binding:
  architecture_id: ARCH-LIVE-MATCH
  architecture_version: v1.2.0
  component_id: COMP-MATCH-STATE
  architecture_invariants:
    - INV-EVENT-IDEMPOTENT
    - INV-SCORE-HISTORY-COHERENT
  interfaces:
    provides:
      - IFACE-CURRENT-STATE
    consumes:
      - IFACE-INGESTED-EVENT
  state:
    owns:
      - STATE-MATCH-CANONICAL
  dependencies:
    allowed:
      - COMP-EVENT-LOG
      - COMP-SNAPSHOT-STORE
    forbidden:
      - COMP-WEB
  budgets:
    - BUDGET-PROCESSING-LATENCY
  decisions:
    - ADR-ARCH-012
```

This lets a coding agent implement a file without redesigning the system.

---

# 39. Example Architecture Decision

```yaml
decision_id: ADR-ARCH-007
question: How should live updates reach 100k anonymous clients?

hard_constraints:
  - goal updates must meet end-to-end p95 target
  - viewer surge must not overload origin
  - monthly infrastructure must stay under budget

candidates:
  - websocket_origin_fanout
  - managed_websocket_gateway
  - sse_edge_distribution

evaluation:
  websocket_origin_fanout:
    strengths:
      - direct control
    weaknesses:
      - high connection-management burden
  managed_websocket_gateway:
    strengths:
      - managed connection scaling
    weaknesses:
      - message/connection cost sensitivity
  sse_edge_distribution:
    strengths:
      - simple one-way semantics
    weaknesses:
      - must validate reconnect and edge behavior

selected: pending_validation

critical_assumption:
  - ASM-ARCH-FANOUT-CAPACITY

validation:
  - VAL-ARCH-FANOUT-POC
```

The example is illustrative only. It is not a recommendation for a particular product architecture.

---

# 40. Tooling Model

Future tooling MAY provide:

```text
eqc-ac init
eqc-ac lint
eqc-ac graph
eqc-ac coverage
eqc-ac assumptions
eqc-ac budgets
eqc-ac impact
eqc-ac validate
eqc-ac lock
eqc-ac diff
eqc-ac drift
eqc-ac fic-packet
```

Expected outputs MAY include:

```text
architecture-readiness-report.md
requirement-coverage-report.md
unowned-state-report.md
hidden-dependency-report.md
budget-validation-report.md
assumption-risk-report.md
architecture-impact-report.md
fic-binding-packet.yaml
```

Tooling is not required for Level 1 or Level 2 conformance unless a project declares it mandatory.

---

# 41. Adoption Path

A project adopting EQC-AC SHOULD proceed in this order:

```text
1. Register the architecture document.
2. Import system requirements.
3. Define system boundary and context.
4. Inventory current components.
5. Inventory communication paths.
6. Inventory state owners.
7. Build dependency graph.
8. Record deployment topology.
9. Record hard quality budgets.
10. Record external assumptions.
11. Record major architecture decisions.
12. Identify drift / hidden paths.
13. Validate risks and architecture-invalidating assumptions.
14. Freeze architecture.
15. Generate or update subsystem / FIC bindings.
```

Existing systems SHOULD begin with observed architecture, clearly labeling:

```text
OBSERVED
INTENDED
DEVIATION
UNKNOWN
```

The migration process MUST NOT pretend that the intended architecture is already the observed implementation.

---

# 42. Compatibility with Existing EQC Documents

EQC-AC is designed to complement, not replace, the existing repository.

### EQC
EQC remains authoritative for algorithm-level mathematical semantics, state, operators, reproducibility, and algorithm procedure.

### EQC-ES
EQC-ES remains authoritative for document portfolio governance, graph structure, version compatibility, hashing, and change propagation.

### EQC-SIB
EQC-SIB remains authoritative for document ↔ pseudocode ↔ implementation binding and implementation-artifact conformance.

### EQC-FIC
EQC-FIC remains authoritative for file-level coding contracts, implementation boundaries, test obligations, and code-generation gating.

### Domain Profiles
Domain profiles such as EQC-SR MAY extend EQC-AC when the domain has architecture-specific requirements.

Example:

```text
EQC-AC-SR
```

could define architecture contracts specific to distributed symbolic-regression engines without modifying generic EQC-AC semantics.

---

# 43. What EQC-AC Is Not

EQC-AC is not:

- a cloud architecture checklist;
- a replacement for C4 diagrams;
- a UML standard;
- a mandatory microservices methodology;
- a framework selection guide;
- a formal verification system;
- an infrastructure-as-code language;
- an API description language;
- a substitute for FIC;
- an instruction to over-document trivial systems.

Its purpose is narrower:

```text
make architecture explicit enough that downstream implementation
cannot silently redesign the system.
```

---

# 44. Final Architecture Invariants

A governed EQC-AC architecture MUST preserve these meta-invariants:

1. **No orphan hard requirement.**
2. **No unjustified component.**
3. **No hidden dependency.**
4. **No unowned authoritative mutable state.**
5. **No architecture-level interface without an owner.**
6. **No undeclared multiple-writer semantics.**
7. **No reliability claim without a failure/recovery model.**
8. **No recoverability claim without a recovery source.**
9. **No performance claim without a measurement/model boundary.**
10. **No cost-constrained approval without a cost model.**
11. **No architecture-critical provider guarantee without evidence or assumption label.**
12. **No downstream FIC override of approved architecture.**
13. **No implementation-driven architecture change without architecture impact analysis.**
14. **No architecture approval with unresolved architecture-invalidating low-confidence assumptions unless explicitly waived.**
15. **No local-optimum decision accepted without checking its composed system effect where interactions are material.**
16. **No material stakeholder concern silently omitted from architecture disposition.**
17. **No contradictory architecture views.**
18. **No material quality attribute left as an undefined adjective when it can be operationalized.**
19. **No material data lifecycle, residency, or retention obligation without an owner when applicable.**
20. **No time/concurrency-dependent correctness claim without declared temporal and concurrency semantics.**
21. **No disaster-recovery claim without measurable recovery objectives where those objectives are material.**
22. **No production-critical capability without operational ownership.**
23. **No architecture-significant technology dependency treated as permanent without a revisit/lifecycle trigger when lifecycle risk is material.**
24. **No claim of continuously enforced architecture conformance without executable fitness/check evidence.**
25. **No downstream reference to an architecture invariant that lacks an authoritative registry entry.**
26. **No material capacity, latency, or cost model detached from its workload assumptions.**
27. **No architecture-significant environment/configuration difference hidden outside the contract.**
28. **No silent trade-off between contradictory hard requirements.**
29. **No non-atomic architecture migration performed through undocumented temporary topology or ownership.**
30. **No architecture approval without identifiable authority and exact approved version.**
31. **No compatibility-breaking architecture change hidden behind an insufficient version bump.**
32. **No material semantic ambiguity in governed model terminology.**
33. **No subsystem refinement allowed to redefine architecture-level ownership, interfaces, or hard budgets without an architecture delta.**
34. **No conformance claim allowed to hide omitted concerns behind unexplained non-applicability.**
35. **No normative architecture view allowed to use undefined model semantics or contradict another governed view.**
36. **No materially sensitive architecture choice presented as robust without exposing the controlling parameter/threshold where known.**
37. **No external reference architecture or named pattern allowed to become hidden architectural authority.**
38. **No material cross-cutting behavior allowed to fragment into contradictory component-local policy.**
39. **No availability/correctness-sensitive runtime unit allowed to rely on undeclared startup, readiness, shutdown, or drain semantics.**
40. **No degraded/failover/maintenance mode allowed to change system guarantees without explicit mode and transition semantics.**
41. **No architecture-critical mutable fact allowed to become permanent truth without recoverable provenance and freshness rules.**
42. **No stale, superseded, disputed, or unavailable evidence allowed to silently continue validating architecture decisions.**
43. **No conflicting credible evidence allowed to be resolved by convenience rather than explicit scope/evidence analysis.**

---

# 45. Architecture Finalization Threshold

EQC-AC itself should not grow indefinitely.

A release is structurally complete when it can:

1. define an architecture unambiguously;
2. expose ownership and boundaries;
3. prevent hidden dependencies;
4. govern interfaces and state;
5. express deployment and failure behavior;
6. express quality budgets and cost constraints;
7. compare architecture alternatives;
8. expose assumptions and risks;
9. require validation evidence;
10. bind architecture to FIC;
11. propagate architecture changes downstream;
12. support deterministic readiness decisions;
13. represent stakeholders, concerns, and architecture views coherently;
14. express testable quality-attribute scenarios;
15. govern material data lifecycle and temporal/concurrency semantics;
16. express measurable recovery objectives where applicable;
17. bind privacy/compliance/residency constraints without inventing them;
18. assign operational ownership;
19. govern architecture-significant technology lifecycle;
20. support continuous architecture fitness checks where practical;
21. govern architecture invariants as first-class IDs;
22. ground capacity/cost/performance in explicit workload models;
23. distinguish architecture-relevant environment/configuration variants;
24. block unresolved hard-requirement infeasibility;
25. govern transitional/migration architectures;
26. identify approval and waiver authority;
27. apply architecture semantic versioning and stale-binding detection;
28. maintain controlled architectural vocabulary where ambiguity is material;
29. hand architecture safely into subsystem contracts before FIC where needed;
30. make section applicability and conformance claims explicit;
31. govern architecture model kinds, notations, and cross-view correspondences;
32. expose sensitivity and trade-off points in material design decisions;
33. govern adoption/deviation from reference architectures and named patterns;
34. govern cross-cutting architecture policies without duplicating component/FIC implementation detail;
35. define runtime lifecycle and operational-mode behavior where it affects architecture guarantees;
36. preserve provenance and applicability of architecture-critical evidence;
37. detect evidence staleness/conflict and propagate invalidation into architecture impact analysis.

Future versions SHOULD add only capabilities that close a demonstrated governance gap, not merely add more checklist items.

---

# 46. Core Operating Principle

```text
Requirements define what must be true.
EQC-AC defines what system structure is allowed to make it true.
Interfaces define how architecture units may interact.
State ownership defines who may change authoritative truth.
Budgets define how much latency, capacity, cost, and failure the system may consume.
Decisions record why this architecture won.
Assumptions expose what could still invalidate it.
Evidence tests those assumptions.
FIC turns the approved architecture into bounded implementation contracts.
Code implements the FIC.
Operations reveal whether the architecture remains true.
```

---

# 47. Summary

EQC-AC provides the missing architecture contract layer between high-level governed intent and file-level implementation.

It creates an explicit chain from stakeholder concerns and governing requirements into an enforceable architecture:

```text
requirements / EQC semantics
  -> EQC-AC architecture contract
  -> subsystem / interface / pseudocode contracts
  -> EQC-FIC
  -> code
  -> evidence
```

The practical objective is not more documentation.

The objective is to eliminate the architecture gap in which implementation agents are otherwise forced to decide:

- what components exist;
- who owns state;
- how components communicate;
- what topology is intended;
- what failures are acceptable;
- what budgets apply;
- what external assumptions are trusted;
- what architecture trade-offs were approved.

Under EQC-AC, those decisions and architecture-wide runtime policies are explicit, versioned, reviewable, falsifiable, migration-aware, and enforceable downstream.
