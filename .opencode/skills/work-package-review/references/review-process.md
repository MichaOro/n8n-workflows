---
name: review-process
description: Canonical engineering methodology for performing implementation-readiness reviews of Work Packages. Defines the complete review lifecycle, evaluation dimensions, evidence collection, and quality standards. Defers simulation and severity to references/implementation-readiness.md, scoring to references/review-scoring.md, and decision methodology to references/decision-matrix.md.
---

# Review Process

## Purpose

This document defines the methodology for performing a complete implementation-readiness review of a Work Package.

An implementation-readiness review determines whether a competent engineer could build the Work Package correctly using only the Work Package, the repository, and project documentation — without requiring additional clarification.

The purpose of this methodology is to make every review deterministic, repository-aware, evidence-based, and reproducible across different reviewers and different LLMs.

This document is the canonical reference for how reviews are performed. It may be referenced by any skill that needs to evaluate implementation readiness.

## Review Principles

This methodology implements the principles defined in `references/review-principles.md`. Every phase, dimension, and quality standard in this document is derived from those principles.

Reviewers must understand the principles before applying this methodology. When a principle conflict arises during a review, resolve it using the hierarchy and conflict resolution method in `references/review-principles.md`.

## Review Lifecycle

The review follows seven sequential phases. Each phase produces outputs that feed the next phase. Skipping or reordering phases produces unreliable results.

```
Context Acquisition
    │
    ▼
Repository Intelligence
    │
    ▼
Implementation Simulation
    │
    ▼
Gap Analysis
    │
    ▼
Engineering Validation
    │
    ▼
Decision Making
    │
    ▼
Review Validation
```

### Phase 1: Context Acquisition

**Purpose**: Understand the Work Package's intent before forming judgments about the codebase.

**Why this exists**: Reviewers who jump into code without understanding the full scope of the Work Package form premature conclusions. They judge individual statements out of context and miss the overall design intent. Reading the Work Package in full first prevents this.

**When this applies**: Every review. This is always the first phase.

**Steps**:

1. Read the Work Package from beginning to end without examining any code. The goal is to understand the overall intent, scope, and approach.
2. Identify which parts of the repository are likely relevant based on domain, module names, file paths, and type names mentioned in the Work Package.
3. Form three to five hypotheses about what the review will find. Examples:
   - "The Work Package proposes creating a service that may already exist in src/services."
   - "The data model described does not match the existing database schema."
   - "The acceptance criteria are too vague to write tests."
   These hypotheses will be validated or refuted in Phase 2.

**Output**: A list of hypotheses to test during repository analysis.

**Common mistakes**:
- Skimming the Work Package instead of reading it fully. Every requirement, constraint, and assumption must be absorbed before forming judgments.
- Forming conclusions about feasibility before understanding the existing codebase. Hypotheses are guesses, not conclusions.
- Silently filling in ambiguous requirements. If the Work Package is ambiguous, note the ambiguity as a hypothesis to validate.

---

### Phase 2: Repository Intelligence

**Purpose**: Build a structural understanding of the repository that will be used to evaluate every statement in the Work Package.

**Why this exists**: The most common source of low-quality reviews is insufficient repository knowledge. Reviewers who do not understand the codebase cannot detect architectural inconsistencies, missed reuse opportunities, or incorrect assumptions. This phase is mandatory because it establishes the ground truth against which the Work Package is measured.

**When this applies**: Every review. The depth of analysis may vary with repository size, but the phase is never skipped.

**Method**: Perform repository analysis following the methodology defined in `references/repository-analysis.md`. This includes:

- **Project topography**: Language, build system, structure, configuration conventions.
- **Architectural contour**: Pattern identification, boundary discovery, data flow analysis.
- **Convention discovery**: Naming, error handling, imports, testing, API, state management.
- **Reuse inventory**: Existing types, functions, components, patterns overlapping with the Work Package.
- **Dependency mapping**: Internal modules, external libraries, data storage, APIs, configuration, infrastructure.

**Output**: Architecture summary, convention catalog, reuse inventory, and dependency map as defined in `references/repository-analysis.md`.

---

### Phase 3: Implementation Simulation

**Purpose**: Identify information gaps by mentally executing the implementation step by step.

**Why this exists**: A Work Package can read well but be unimplementable. The only way to discover this is to simulate implementation. Static reading reveals surface-level issues; simulation reveals gaps in data contracts, integration points, error handling, and edge cases. This phase is the core discovery mechanism of the entire review methodology.

**When this applies**: Every review.

**Method**: Execute the simulation methodology defined in `references/implementation-readiness.md`. This covers:
- The simulation procedure (name the unit, trace reads, write connections, handle errors and edge cases)
- Simulation depth levels (shallow, medium, deep) and when to apply each
- Recording gaps with location, missing information, engineer response, and consequence
- Determining the simulation ceiling from the worst gap

**Output**: A complete list of simulation gaps, each with location, missing information, engineer response, consequence, and the overall simulation ceiling.

**Common mistakes**:
- Treating the review as a document evaluation rather than an implementation dry-run.
- Applying shallow depth to complex operations. Consult `references/implementation-readiness.md` for depth guidance.
- Skipping error path and edge case simulation. Happy-path-only simulation misses the majority of implementation gaps.

---

### Phase 4: Gap Analysis

**Purpose**: Find every inconsistency between what the repository contains, what the Work Package proposes, what the engineer would build, and what would actually ship.

**Why this exists**: Individual phase outputs may each appear reasonable in isolation. A gap analysis reveals contradictions between them that no single phase would catch. An architecturally sound simulation may still produce an implementation that contradicts the repository's intent.

**When this applies**: Every review. This phase runs after Implementation Simulation and uses its outputs.

**Method**: Execute the four-layer gap analysis (Repository → Work Package → Expected Implementation → Implementation Reality) defined in `references/implementation-readiness.md`. This covers:
- The four comparison points and what each reveals
- How to trace silent deviation risk
- How to identify architectural drift

**Output**: A list of discrepancies organized by comparison type. Each discrepancy maps to one or more findings that will be formally classified in Phase 5.

**Common mistakes**:
- Stopping after Repository → Work Package comparison. This only catches factual errors.
- Failing to trace Expected Implementation → Implementation Reality — the step that catches the most impactful gaps.
- Treating gaps as independent rather than chained.

---

### Phase 5: Engineering Validation

**Purpose**: Classify each gap and inconsistency into a formal finding using the evaluation dimensions and the Engineering Justification Chain.

**Why this exists**: Gaps found during simulation and analysis are raw observations. They must be classified by dimension, linked to engineering principles, and justified with evidence before they become actionable findings. This phase transforms observations into engineering conclusions.

**When this applies**: After Gap Analysis is complete. Every gap from Phases 3 and 4 must be processed through this phase.

**Method**:

Findings are constructed following the methodology in `references/finding-guidelines.md`. Every finding must follow the Engineering Justification Chain defined in `references/evidence-and-justification.md` — this is mandatory and findings that skip links are incomplete.

Evidence collection for findings follows `references/evidence-and-justification.md`.

#### Evaluation Dimensions

Classify each finding into one of the following dimensions. Evaluate all six dimensions systematically — even if the finding for a dimension is "no issues found."

##### 1. Specification Completeness

Can the implementer determine what to build without guessing?

Evaluate:
- Is the purpose stated explicitly and precisely?
- Are functional requirements unambiguous? Could two implementers build different things?
- Are non-functional requirements (performance, scalability, security) stated where relevant?
- Are boundary conditions and edge cases discussed?
- Are scope boundaries explicit? What is explicitly not included?

Common mistakes:
- Assuming intent is obvious from context. Flag any requirement that could be interpreted in multiple ways.
- Confusing implementation details with requirements. The Work Package should describe what, not how — unless specific implementation constraints are required for consistency.

##### 2. Data Contract Completeness

Are all data shapes, types, and schemas fully specified or inferable from context?

Evaluate:
- Are data models, types, interfaces, or schemas fully specified?
- If not fully specified, can the implementer infer them from existing code without risk of mismatch?
- Are database migrations, schema changes, or data format changes fully enumerated?
- Are new derived or computed fields explained (derivation rules, update timing)?
- Are serialization and deserialization boundaries defined (JSON shape, protocol buffers, etc.)?

Common mistakes:
- Assuming the implementer will infer a data model that matches the existing schema. Without explicit specification, the implementer may choose a shape that conflicts with existing consumers.
- Specifying data shapes inconsistently between the acceptance criteria and the described implementation.

##### 3. Integration Surface Completeness

Are all touch points between new and existing code enumerated?

Evaluate:
- Are API contracts (request/response shapes, status codes, error formats) defined for new endpoints?
- Are configuration changes enumerated (environment variables, feature flags, application settings)?
- Are all existing callers and consumers of modified code identified?
- Are import paths, module registration, and dependency injection wiring specified?
- Are deployment or infrastructure changes enumerated (new services, new queues, new storage)?

Common mistakes:
- Omitting configuration changes because they seem minor. Every configuration surface change should be enumerated — an unlisted environment variable is a runtime failure waiting to happen.
- Describing API changes without specifying the contract. "Add a user preferences endpoint" is not a contract.

##### 4. Error and Observability Completeness

What happens when each operation fails, and how will operators know?

Evaluate:
- Are error scenarios documented for each operation? What happens when a step fails?
- Are error responses specified (status codes, error shapes, error messages)?
- Are logging, observability, and monitoring expectations stated?
- Are retry policies, fallback behavior, and circuit-breaking specified where appropriate?
- Are audit trails or event logs specified where data changes occur?
- Are debugging and troubleshooting expectations stated?

Common mistakes:
- Assuming the implementer will "figure out" error handling. Error handling affects API contracts, database schemas, and user experience. It must be specified.
- Specifying happy-path observability while ignoring error observability. "Log on success" without "log on failure with context" leaves operators blind.

##### 5. Architectural Fit

Does the Work Package align with the repository's architecture?

Evaluate:
- Does the proposed change fit within existing architectural layers?
- Does it introduce new patterns that contradict established conventions?
- Does it respect existing module boundaries? Would it create circular dependencies or inappropriate cross-layer access?
- Does it introduce new dependencies that the project currently avoids?
- If it proposes architectural changes, are migration paths and coexistence strategies defined?

Common mistakes:
- Approving a pattern that contradicts the project's conventions because it seems reasonable in isolation. Every architectural decision must be evaluated against what exists.
- Assuming that because no explicit architecture document exists, there is no architecture. Reconstruct the architecture from the code.

##### 6. Reuse Efficiency

Did the Work Package optimize for reuse, or does it duplicate existing capabilities?

Reuse evaluation follows the methodology in `references/repository-reuse.md`. The evaluation framework covers existence, fitness, extension, coupling, and confidence for every proposed creation in the Work Package.

This dimension raises the standard from passive awareness (not missing existing code) to active efficiency (optimizing reuse by extending rather than creating).

**Output**: A complete set of findings, each following the Engineering Justification Chain and classified by dimension and evidence type.

---

### Phase 6: Decision Making

**Purpose**: Classify findings by severity, calculate the readiness score, and determine whether the Work Package is ready for implementation.

**Why this exists**: A long list of findings is not a decision. The severity model and decision rules convert findings into an actionable determination that an implementer can act on.

**When this applies**: After all findings are classified by dimension.

**Method**: Follow the severity model defined in `references/implementation-readiness.md`, scoring defined in `references/review-scoring.md`, and the decision methodology (PASS / FAIL / CONDITIONAL PASS rules, conditional pass guidance, confidence assessment) defined in `references/decision-matrix.md`. This covers:
- The four severity levels (Critical, High, Medium, Low) with criteria and required responses (per `references/implementation-readiness.md`)
- Severity assignment rules (per `references/implementation-readiness.md`)
- Readiness score calculation with simulation ceiling constraint (per `references/review-scoring.md`)
- Decision rules for PASS, FAIL, and CONDITIONAL PASS (per `references/decision-matrix.md`)
- Conditional pass guidance (per `references/decision-matrix.md`)
- Confidence assessment (High, Medium, Low) with criteria (per `references/decision-matrix.md`)

**Output**: Severity-classified findings, readiness score, decision, confidence level, and (if conditional pass) implementation guidance.

---

### Phase 7: Review Validation

**Purpose**: Verify the review's completeness, consistency, evidence quality, and actionability before releasing it.

**Why this exists**: Reviewers make mistakes. This phase is a self-check that catches incomplete evaluations, contradictory findings, unsupported conclusions, and unactionable recommendations before they reach the implementer.

**When this applies**: Every review, after decision making is complete, before the review output is released.

**Method**: Evaluate the completed review against the quality gates defined in `references/quality-gates.md`. Each gate must pass before the review can proceed to output formatting.

The quality gates validate:
- **Completeness**: All methodology phases executed, all dimensions evaluated.
- **Evidence**: Every finding has traceable evidence from the repository.
- **Actionability**: High and Critical findings have specific remediation guidance.
- **Scoring**: Score is correctly calculated and constrained by the simulation ceiling.
- **Decision**: Decision follows the decision model and is consistent with findings.
- **Consistency**: No contradictory findings, severity is consistent, score and decision align.
- **Output Contract**: Report structure and presentation follow `references/review-output-contract.md`.
- **Integrity**: No signs of methodology bypass or integrity failure.

The execution checklist in `references/review-checklist.md` verifies that methodology phases were executed. The quality gates in `references/quality-gates.md` verify that the completed review satisfies framework standards.

**Output**: A validated review ready for output formatting. Gate validation follows `references/quality-gates.md`. Output structure follows `references/review-output-contract.md`.

**Common mistakes**:
- Skipping this phase entirely. Without validation, the review may contain errors that undermine the implementer's trust.
- Running the checklist superficially. Each item should be verified by re-reading the relevant finding, not by assuming it was done correctly.

---

## Quality Principles

Finding quality criteria are defined in `references/finding-guidelines.md`.

## Anti-Patterns

All review anti-patterns, failure modes, and rationalizations are defined in `references/anti-patterns.md`. Reviewers should consult that document to recognize and prevent the recurring engineering failure modes that degrade review quality.

---

## Review Success Criteria

A successful review is one where the Review Success Criteria in `references/quality-gates.md` are met. These criteria define whether the review has achieved its purpose and are evaluated after all quality gates pass.

The six success criteria are:

1. **The intent is clear.** An implementer can understand what to build without guessing.
2. **The implementation is simulable.** A dry-run of the implementation completes without stopping due to missing information.
3. **The Work Package is consistent with the repository.** Every statement has been validated against the codebase following the methodology in `references/repository-analysis.md`.
4. **The findings are complete.** All gaps, inconsistencies, and risks are documented.
5. **The findings are justified.** Every finding follows the Engineering Justification Chain defined in `references/evidence-and-justification.md` with evidence, principle, impact, and recommendation.
6. **The decision is deterministic.** A different reviewer applying this methodology to the same inputs would reach the same conclusion, per the consistency methodology in `references/review-consistency.md`.

If these criteria are met, the review serves its purpose: protecting both the implementer and the codebase from incomplete, inconsistent, or ambiguous Work Packages.
