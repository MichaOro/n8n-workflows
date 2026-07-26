---
name: work-package-review
description: Implementation-readiness review of Work Packages. Determines whether a Work Package is complete, repository-consistent, and safe to implement without requiring additional clarification.
---

# Work Package Review

## Purpose

A Work Package describes what must be built. An implementation-readiness review determines whether a competent engineer could build it correctly using only the Work Package, the repository, and project documentation — without requiring additional clarification.

This skill exists because incomplete or ambiguous Work Packages cause rework, architectural drift, integration failures, and wasted engineering time. The review prevents these outcomes by catching problems before implementation begins.

The review is not a quality assessment of the Work Package author. It is an engineering gate that protects both the implementer and the codebase.

## Scope

This skill covers the end-to-end review of a Work Package against:

- The existing repository structure and conventions
- Project architecture and design decisions
- Existing implementations for reuse opportunities
- Testing strategy and infrastructure
- Maintainability and complexity constraints
- The Work Package's own internal consistency

The review does not cover:

- Business value of the requested feature
- Priority relative to other work
- Personnel allocation or timeline estimation
- Code review of completed implementation

## When to Use

Apply this skill when:

- A Work Package has been drafted and is proposed for implementation
- An engineer requests a second opinion on implementation feasibility
- The project is incorporating a new subsystem with cross-cutting concerns
- The Work Package introduces new architectural patterns or dependencies
- Prior implementations have revealed recurring gaps in the Work Package process

## When Not to Use

Do not apply this skill when:

- The task is exploratory or research-oriented with undefined outcomes
- The change is a trivial bug fix covered by existing patterns
- The Work Package is explicitly marked as draft or incomplete
- The reviewer is also the implementer and will discover gaps during implementation

## Output Format

**The skill returns deterministic JSON output** suitable for both human reading and machine automation. The JSON (`metadata`, `summary`, `findings`) is the canonical source of truth; the markdown inside `report.content` is generated from it, not an independent source — see "Source of Truth" in the output contract.

- **Output Contract**: `references/review-output-contract.md` — Defines JSON schema, field specifications, markdown report structure, and the Source of Truth rules
- **Output Documentation**: `OUTPUT-FORMAT.md` — Consumption patterns, field reference, finding lifecycle, migration notes
- **Migration Guide**: `MIGRATION.md` — Version 2.0 → 2.1 changes (most relevant) and the historical 1.x → 2.0 migration
- **Output Examples**: `examples-output-pass-no-findings.json`, `examples-output-pass-low-findings.json`, `examples-output-fail-high-finding.json`, `examples-output-fail-multiple-findings.json`, `examples-output-conditional-pass.json`
- **Execution Prompt**: `prompt.md` — Orchestration prompt that guides the skill through all 12 phases while generating JSON output

The output JSON contains:
1. **metadata** — Provenance and unique identification: skill name, methodology `version`, contract `schemaVersion`, `reviewId`, `reviewType`, review date, repository, work package
2. **summary** — Machine-readable object for n8n Switch/IF nodes, GitHub comments, dashboards, quality gates: decision, score, ceiling, confidence, readiness, `implementationReady` (boolean), finding counts, and the derived `totalFindings`/`blockingFindings`
3. **findings** — Structured array of all findings with severity, a lifecycle `status` (`"open"` at emission), dimension, evidence, impact, and recommendations
4. **report** — A structured object (`{ format, version, content }`) whose `content` is a complete standalone markdown document suitable for archival and human reading

**Schema version**: 2.1 (refined 2026-07-25). **Methodology version**: 2.0 (unchanged by this refinement — see `MIGRATION.md`).

## Reference Documents

The complete review methodology is defined across thirteen reference documents. Every phase of the orchestration below references the responsible document for its engineering knowledge.

| Document | Purpose |
|---|---|
| `references/review-principles.md` | Engineering philosophy, principles, hierarchy, conflict resolution |
| `references/review-process.md` | Review lifecycle, evaluation dimensions, evidence collection, quality standards |
| `references/repository-analysis.md` | Repository discovery, architecture reconstruction, convention discovery, reuse inventory, dependency mapping |
| `references/repository-reuse.md` | Reuse evaluation, extension vs creation, abstraction evaluation, duplication analysis, reuse confidence, reuse trade-offs |
| `references/implementation-readiness.md` | Implementation simulation, gap analysis, severity model, simulation ceiling |
| `references/evidence-and-justification.md` | Engineering evidence, evidence hierarchy, evidence collection and validation, traceability, justification chain, engineering reasoning, claim validation, burden of proof |
| `references/finding-guidelines.md` | Finding construction, quality criteria, classification, finding-specific anti-patterns, common finding mistakes |
| `references/review-scoring.md` | Readiness scoring, score calculation, interpretation, common scoring mistakes |
| `references/decision-matrix.md` | Decision methodology, approval states, decision rules, confidence assessment, escalation, exception handling, risk-based decisions |
| `references/review-output-contract.md` | **JSON schema, field specifications, markdown report structure, backward compatibility notes** |
| `references/review-consistency.md` | Deterministic reviews methodology, reviewer calibration, intra/inter-review consistency, cross-LLM consistency, drift detection, consistency validation |
| `references/quality-gates.md` | Review quality validation, gate model, gate criteria, completeness verification, publication readiness, integrity validation |
| `references/anti-patterns.md` | Reviewer anti-patterns, cognitive biases, scope and process failures, review smells, rationalizations, LLM-specific failures |
| `references/review-checklist.md` | Execution verification checklist for methodology phases |

## Engineering Principles

The framework follows the principles defined in `references/review-principles.md`. Every reviewer must understand them before producing output.

## Review Lifecycle

The review follows a deterministic 12-phase lifecycle. Each phase references the document responsible for its engineering knowledge. Phases must be executed in order. Skipping or reordering phases produces unreliable results.

```
Initialize
    │
    ▼
Repository Analysis
    │
    ▼
Repository Reuse Evaluation
    │
    ▼
Implementation Readiness Assessment
    │
    ▼
Evidence Collection & Justification
    │
    ▼
Finding Generation
    │
    ▼
Review Scoring
    │
    ▼
Engineering Decision
    │
    ▼
Review Report Generation
    │
    ▼
Consistency Validation
    │
    ▼
Quality Gate Validation
    │
    ▼
Publish Review
```

### Phase 1: Initialize

| Field | Value |
|---|---|
| **Purpose** | Set up the review context, understand the Work Package intent, and form initial hypotheses. |
| **Responsible Reference** | `references/review-process.md` (Phase 1: Context Acquisition) |
| **Required Inputs** | Work Package document, repository access |
| **Expected Output** | List of hypotheses to test during analysis, identified impact zones |
| **Entry Conditions** | Review has been requested. Work Package is not draft or incomplete. |
| **Exit Criteria** | Work Package read in full. Three to five hypotheses formed. |
| **Next Phase** | Repository Analysis |

### Phase 2: Repository Analysis

| Field | Value |
|---|---|
| **Purpose** | Build a structural understanding of the repository — architecture, conventions, existing code, and dependencies. |
| **Responsible Reference** | `references/repository-analysis.md` |
| **Required Inputs** | Work Package, repository access, hypotheses from Phase 1 |
| **Expected Output** | Architecture summary, convention catalog, reuse inventory, dependency map |
| **Entry Conditions** | Phase 1 complete. Hypotheses documented. |
| **Exit Criteria** | Repository analysis complete per `references/repository-analysis.md`. All hypotheses validated or refuted with evidence. |
| **Next Phase** | Repository Reuse Evaluation |

### Phase 3: Repository Reuse Evaluation

| Field | Value |
|---|---|
| **Purpose** | Evaluate whether the repository already contains code that should be extended or composed rather than recreated. |
| **Responsible Reference** | `references/repository-reuse.md` |
| **Required Inputs** | Reuse inventory from Phase 2, Work Package proposals for new code |
| **Expected Output** | Reuse decisions for every proposed creation (extend, compose, reference, or create) |
| **Entry Conditions** | Phase 2 complete. Reuse inventory available. |
| **Exit Criteria** | Every proposed creation evaluated for reuse opportunity. Reuse confidence assessed. |
| **Next Phase** | Implementation Readiness Assessment |

### Phase 4: Implementation Readiness Assessment

| Field | Value |
|---|---|
| **Purpose** | Identify information gaps by simulating implementation and analyzing discrepancies between the Work Package, repository, expected implementation, and implementation reality. |
| **Responsible Reference** | `references/implementation-readiness.md` |
| **Required Inputs** | Work Package, repository analysis outputs, reuse evaluation outputs |
| **Expected Output** | Simulation gaps with location and consequence, gap analysis discrepancies, simulation ceiling |
| **Entry Conditions** | Repository understanding established. Reuse decisions documented. |
| **Exit Criteria** | Simulation executed per methodology. Simulation ceiling determined. Four-layer gap analysis complete. |
| **Next Phase** | Evidence Collection & Justification |

### Phase 5: Evidence Collection & Justification

| Field | Value |
|---|---|
| **Purpose** | Collect traceable repository evidence for every gap and justify conclusions using the Engineering Justification Chain. |
| **Responsible Reference** | `references/evidence-and-justification.md` |
| **Required Inputs** | Simulation gaps, gap analysis discrepancies, repository analysis outputs |
| **Expected Output** | Evidence-backed conclusions for every gap. Each conclusion satisfies the Engineering Justification Chain. |
| **Entry Conditions** | Gaps and discrepancies identified in Phase 4. |
| **Exit Criteria** | Every gap has traceable evidence. Evidence hierarchy standards satisfied. |
| **Next Phase** | Finding Generation |

### Phase 6: Finding Generation

| Field | Value |
|---|---|
| **Purpose** | Classify each evidence-backed conclusion into a formal finding using evaluation dimensions and severity classification. |
| **Responsible Reference** | `references/finding-guidelines.md` (finding construction), `references/review-process.md` (evaluation dimensions) |
| **Required Inputs** | Evidence-backed conclusions from Phase 5 |
| **Expected Output** | Severity-graded findings classified by evaluation dimension, each following the Engineering Justification Chain |
| **Entry Conditions** | Evidence collected and justified for every gap. |
| **Exit Criteria** | All findings classified by dimension and severity. Severity assignment consistent with severity model. |
| **Next Phase** | Review Scoring |

### Phase 7: Review Scoring

| Field | Value |
|---|---|
| **Purpose** | Calculate a numerical readiness score from severity-weighted findings, constrained by the simulation ceiling. |
| **Responsible Reference** | `references/review-scoring.md` |
| **Required Inputs** | Severity-graded findings, simulation ceiling |
| **Expected Output** | Readiness score (0-100), ceiling-constrained |
| **Entry Conditions** | All findings finalized and severity-classified in Phase 6. |
| **Exit Criteria** | Score calculated per scoring methodology. Ceiling constraint applied. |
| **Next Phase** | Engineering Decision |

### Phase 8: Engineering Decision

| Field | Value |
|---|---|
| **Purpose** | Determine whether the Work Package is ready for implementation by applying decision rules, confidence assessment, and escalation criteria. |
| **Responsible Reference** | `references/decision-matrix.md` |
| **Required Inputs** | Readiness score, simulation ceiling, severity-weighted findings, confidence assessment |
| **Expected Output** | Decision (PASS / FAIL / CONDITIONAL PASS), confidence level, implementation guidance |
| **Entry Conditions** | Score calculated in Phase 7. All findings finalized. |
| **Exit Criteria** | Decision made per decision methodology. Confidence documented. Implementation guidance provided if CONDITIONAL PASS. |
| **Next Phase** | Review Report Generation |

### Phase 9: Review Report Generation

| Field | Value |
|---|---|
| **Purpose** | Format the complete review output following the structural and presentation standards of the output contract. |
| **Responsible Reference** | `references/review-output-contract.md` |
| **Required Inputs** | All review outputs: findings, score, ceiling, decision, confidence, implementation guidance |
| **Expected Output** | Formatted review report with all mandatory and conditional sections |
| **Entry Conditions** | Decision made in Phase 8. All review artifacts available. |
| **Exit Criteria** | All mandatory sections present. Section ordering follows the output contract. Evidence citations include file paths and line numbers. |
| **Next Phase** | Consistency Validation |

### Phase 10: Consistency Validation

| Field | Value |
|---|---|
| **Purpose** | Verify that findings, scores, decisions, and report presentation are internally consistent and cross-review consistent. |
| **Responsible Reference** | `references/review-consistency.md` |
| **Required Inputs** | Completed review report, all review artifacts |
| **Expected Output** | Consistency-validated review |
| **Entry Conditions** | Report generated in Phase 9. |
| **Exit Criteria** | No contradictory findings. Severity consistent across similar findings. Score and decision consistent with findings. Simulation ceiling matches worst gap. |
| **Next Phase** | Quality Gate Validation |

### Phase 11: Quality Gate Validation

| Field | Value |
|---|---|
| **Purpose** | Validate the completed review against framework quality standards before publication. |
| **Responsible Reference** | `references/quality-gates.md` |
| **Required Inputs** | Completed review report, all methodology outputs |
| **Expected Output** | Gate-validated review. All mandatory gates pass. |
| **Entry Conditions** | Consistency validated in Phase 10. |
| **Exit Criteria** | All mandatory quality gates pass (Completeness, Evidence, Actionability, Scoring, Decision, Consistency, Output Contract, Integrity). Any advisory gate failures documented. |
| **Next Phase** | Publish Review |

### Phase 12: Publish Review

| Field | Value |
|---|---|
| **Purpose** | Release the completed, gate-validated review as a single valid JSON object to downstream consumers. |
| **Responsible Reference** | `references/review-checklist.md` (execution verification), `references/review-output-contract.md` (JSON schema) |
| **Required Inputs** | Gate-validated review, all artifacts from Phases 1-11 |
| **Expected Output** | Single valid JSON object with metadata, summary, findings, report fields (see `references/review-output-contract.md`) |
| **Entry Conditions** | All mandatory quality gates pass in Phase 11. |
| **Exit Criteria** | JSON object emitted. No text, markdown, or explanations outside the JSON. Execution checklist verified. |
| **Next Phase** | (none — lifecycle complete) |

## Execution Verification

Before final publication, verify that every methodology phase was executed. The execution checklist in `references/review-checklist.md` covers all phases.

## Anti-Patterns

All review anti-patterns, failure modes, cognitive biases, and integrity failure symptoms are defined in `references/anti-patterns.md`. If any integrity failure symptom is present, the review should not be trusted.

## Interaction Notes

- When a finding references repository code, read the actual code. Do not rely on search results alone.
- When searching for reuse candidates, use both semantic search (Grep) and file name search (Glob) per `references/repository-analysis.md`. Reuse evaluation follows `references/repository-reuse.md`.
- If the Work Package references non-existent files or directories, verify by checking the repository rather than assuming the Work Package is wrong.
- Execute the Implementation Readiness assessment (simulation and gap analysis per `references/implementation-readiness.md`) before writing findings. The simulation is the primary discovery mechanism.
