---
name: quality-gates
description: Canonical engineering standard for validating the quality and completeness of engineering reviews. Defines objective, repeatable quality gates that determine whether a completed review satisfies the framework before it is accepted or published. Does not define repository analysis, implementation readiness, engineering principles, scoring, decision making, or report formatting.
---

# Quality Gates

## Purpose

This document defines the quality gates that every completed engineering review must satisfy before it is accepted or published.

A quality gate is an objective, verifiable condition that determines whether a review meets the framework's engineering standards. Gates consume the outputs of the framework's methodology documents and validate them against quality criteria. The gates verify quality — they do not create it.

This document validates the quality of the review itself. It does not evaluate the implementation being reviewed.

A review should only be considered complete when every required quality gate has been satisfied.

## Gate Model

### Gate Types

| Type | Meaning |
|---|---|
| Mandatory | The gate must pass for the review to be accepted. Failure blocks publication. |
| Optional | The gate provides guidance. Failure is noted but does not block. |
| Blocking | A mandatory gate whose failure requires remediation before the review can proceed. |
| Advisory | An optional gate whose failure should be documented but does not prevent publication. |

### Gate Outcomes

| Outcome | Meaning |
|---|---|
| PASS | Gate requirements are satisfied. The review may proceed to the next gate. |
| FAIL | Gate requirements are not satisfied. The review must be remediated before proceeding. If the gate is mandatory, the review cannot be accepted until the gate passes. |
| CONDITIONAL | Gate requirements are partially satisfied. Specific conditions must be met for the review to proceed. The conditions must be documented. |

### Gate Ordering

Gates are evaluated in order. Each gate depends on the previous gate passing. If a gate fails, later gates are not evaluated until the failure is resolved.

```
Completeness → Evidence → Actionability → Scoring → Decision → Consistency → Output Contract → Integrity
```

### Gate Dependencies

| Gate | Depends On | Description |
|---|---|---|
| Completeness | — | No dependencies. Evaluated first. |
| Evidence | Completeness | Findings must exist before their evidence can be validated. |
| Actionability | Evidence | Evidence must be validated before actionability can be assessed. |
| Scoring | Actionability | Findings must be actionable before scoring can be validated. |
| Decision | Scoring, Consistency | Score and consistent findings are required before decision validation. |
| Consistency | Completeness | Can be evaluated in parallel with Evidence through Decision gates. |
| Output Contract | All preceding gates | Output structure validation is the final review gate. |
| Integrity | Any time | Can be evaluated at any point. Integrity failures invalidate the review regardless of other gates. |

---

## Gate 1: Completeness Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that every required methodology phase has been executed and every evaluation dimension has been examined.

### Criteria

- Every Review Dimension has been evaluated (even if the finding is "no issues found").
- Repository Intelligence has been completed per `references/repository-analysis.md` and its findings are referenced in dimension evaluations.
- Implementation Simulation (per `references/implementation-readiness.md`) has been executed and gaps are documented.
- Gap Analysis (four-layer comparison per `references/implementation-readiness.md`) has been executed.
- Scoring (per `references/review-scoring.md`) has been executed.
- All acceptance criteria in the Work Package have been evaluated for testability.
- All data models, schemas, and interfaces mentioned in the Work Package have been verified against existing code.

### Failure

If any criterion is not met, the review is incomplete. The missing phase must be executed and the review re-validated before proceeding.

---

## Gate 2: Evidence Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that every finding is supported by traceable evidence from the repository.

### Criteria

- Every finding cites specific evidence (per `references/evidence-and-justification.md`).
- Evidence is from the repository, not inferred or assumed.
- Evidence citations include file paths and line numbers per `references/evidence-and-justification.md`.
- Every finding follows the Engineering Justification Chain defined in `references/evidence-and-justification.md`.

### Failure

If any finding lacks evidence or the evidence is untraceable, the finding is not valid. The finding must be corrected or removed before the review can be accepted.

---

## Gate 3: Actionability Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that findings enable the implementer to act without additional clarification.

### Criteria

- Every High or Critical finding includes a remediation recommendation (per `references/finding-guidelines.md`).
- Remediation recommendations are specific enough for the implementer to follow.
- Conditional PASS decisions include clear guidance on what is blocked vs. unblocked.

### Failure

If a finding in the actionable severity range lacks a specific recommendation, the review is incomplete. The missing recommendation must be added.

---

## Gate 4: Scoring Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that the readiness score is calculated correctly and constrained appropriately.

### Criteria

- The score is within the [0-100] range.
- The score does not exceed the simulation ceiling.
- If any Critical findings exist, the score is 0.
- Deduction ranges follow `references/review-scoring.md` (10-25 for High, 3-8 for Medium).
- Related findings from the same root cause were not over-deducted.
- The simulation ceiling matches the worst simulation gap from Phase 3.

### Score Integrity

A score is valid only when the methodology has been followed. A score produced without repository analysis, simulation, or gap analysis is meaningless regardless of the number. The Scoring Gate must verify that all preceding methodology phases were executed before validating the score.

### Incomplete Evaluations

If a review dimension could not be fully evaluated:

- If 1-2 dimensions could not be fully evaluated: the score should be reduced by 5-10 points and the limitation noted in the decision rationale.
- If 3+ dimensions could not be evaluated: the score is provisional. The decision confidence must be set to Low per `references/decision-matrix.md`.

### Failure

If the score violates any criterion, the score calculation must be corrected. If the methodology was not followed, the score is invalid and must be recalculated after the methodology is executed.

---

## Gate 5: Decision Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that the decision is correctly derived from the findings, score, ceiling, and confidence.

### Criteria

- The decision follows the decision model in `references/decision-matrix.md` and is consistent with the score, ceiling, and findings.
- Confidence is assessed and appropriate for the analysis depth.
- The decision rationale explains how the decision was reached from the inputs (per `references/review-output-contract.md`).

### Incomplete Analysis Handling

The following conditions trigger decision limitations that must be validated:

| Condition | Gate Outcome |
|---|---|
| Repository analysis could not be completed | Confidence is Low. Decision is provisional. |
| Simulation could not be completed | Confidence is Low. Decision is provisional. |
| Work Package is explicitly marked as draft | Mark as incomplete with Low confidence. Do not score. |

### Failure

If the decision contradicts the decision model or evidence, the decision must be re-evaluated. If required methodology phases were not executed, the decision is invalid.

---

## Gate 6: Consistency Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that findings, scores, decisions, and output are internally consistent.

### Criteria

- No finding contradicts another finding.
- Severity assignments are consistent across similar findings.
- The score is consistent with the findings (no score-finding mismatch).
- The decision is consistent with the severity-weighted finding set.
- The simulation ceiling matches the worst simulation gap.
- The readiness score was calculated per `references/review-scoring.md` and is consistent with the findings.
- All findings use consistent severity language (per `references/review-consistency.md`).

### Failure

Any inconsistency must be resolved before the review can be accepted. If findings contradict each other, the contradictory findings must be corrected or reconciled. If the score or decision contradicts the findings, the calculation or decision must be re-evaluated.

---

## Gate 7: Output Contract Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that the review output follows the structural and presentation standards defined in the output contract.

### Criteria

- The output is exactly one valid, RFC 8259-compliant JSON object — no surrounding markdown, code fences, or explanatory text.
- All four top-level fields (`metadata`, `summary`, `findings`, `report`) are present. `findings` is an array (empty is acceptable; missing is not).
- `report` is an object (`{ format, version, content }`), never a bare string.
- `metadata` includes `skill`, `version`, `schemaVersion`, `reviewId`, `reviewType`, `reviewDate`, `repository`, `workPackage`.
- `summary`'s derived fields match their formulas exactly: `totalFindings === critical + high + medium + low === findings.length`; `blockingFindings === critical + high`; `implementationReady === (decision === "PASS")`; `readiness === "Implementation Ready"` iff `implementationReady === true`.
- Every finding includes all five links of the Engineering Justification Chain, plus `id`, `severity`, and `status` (`"open"` at emission).
- Every evidence citation includes file paths and line numbers.
- Every High and Critical finding includes a remediation recommendation.
- Conditional PASS decisions include implementation guidance in `report.content`.
- The decision is explicitly stated in `summary.decision` and consistent with the evidence in `findings`.
- `report.content`'s mandatory sections are present, correctly ordered (per `references/review-output-contract.md`), and every fact stated in it is traceable to `metadata`/`summary`/`findings` — see the Source of Truth rules in that document. No finding is described in the markdown without a matching entry in `findings[]`, and vice versa.

### Failure

If any mandatory field is missing, the JSON is not parseable as a single object, `report` is a bare string instead of an object, a derived `summary` field doesn't match its formula, or `report.content` diverges from the structured data, the output must be corrected before the review is accepted.

---

## Gate 8: Integrity Gate

**Type**: Mandatory, Blocking

**Purpose**: Verify that the review was conducted in good faith following the framework methodology, without signs of integrity failure.

### Criteria

- Repository analysis was executed per `references/repository-analysis.md`.
- At least one evidence citation exists in the findings.
- Engineering recommendations do not contradict the project's existing patterns.
- The review identified gaps or risks (not only positive observations).
- Confidence is proportional to analysis depth. High confidence requires thorough repository analysis.
- The review does not exhibit integrity failure symptoms as defined in `references/anti-patterns.md`.

### When Integrity Is Compromised

A review is invalid when any of the following is true:

- Repository analysis was not executed.
- No evidence citations exist in any finding.
- Engineering recommendations contradict the project's existing patterns.
- The reviewer accepted "will be handled during implementation" for a Critical finding.
- The review mentions only positive observations without identifying any gaps.
- Decision confidence is High but repository analysis covered fewer than three source files.

If any of these conditions is present, the review must not be trusted. Decision confidence must be set to Low and the review flagged as incomplete.

### Failure

Integrity failure is irrecoverable. The review must be restarted from the beginning following the full methodology.

---

## Review Success Criteria

These higher-level criteria define whether a review has achieved its purpose. They are evaluated after all gates pass.

A successful review is one where:

1. **The intent is clear.** An implementer can understand what to build without guessing.
2. **The implementation is simulable.** A dry-run of the implementation completes without stopping due to missing information.
3. **The Work Package is consistent with the repository.** Every statement has been validated against the codebase following the methodology in `references/repository-analysis.md`.
4. **The findings are complete.** All gaps, inconsistencies, and risks are documented.
5. **The findings are justified.** Every finding follows the Engineering Justification Chain defined in `references/evidence-and-justification.md` with evidence, principle, impact, and recommendation.
6. **The decision is deterministic.** A different reviewer applying this methodology to the same inputs would reach the same conclusion, per the consistency methodology in `references/review-consistency.md`.

If these criteria are met, the review serves its purpose: protecting both the implementer and the codebase from incomplete, inconsistent, or ambiguous Work Packages.

---

## Gate Failures and Re-review

### When a Gate Fails

| Gate Type | Failure Response |
|---|---|
| Mandatory, Blocking | Review must be remediated. The failing gate and all subsequent gates must be re-evaluated after remediation. |
| Advisory | Failure is documented. Review may proceed. |

### When a Gate Passes Conditionally

The conditions must be documented in the review output. The review may proceed, but the conditions must be tracked until resolved.

### Re-review Triggers

A review must be re-validated when:

- New findings are discovered after a gate has passed.
- A Critical or High finding's remediation changes the scope of the affected dimension.
- The review methodology was not followed for any phase.
- An integrity failure symptom is detected after the review was accepted.

### Remediation Path

When a gate fails:

1. Identify the specific criterion that failed.
2. Determine the root cause: methodology omission, reviewer error, or framework ambiguity.
3. Correct the issue.
4. Re-evaluate the failing gate.
5. Re-evaluate all downstream gates that depend on it.

---

## Related Documents

- `references/review-process.md` — Defines Phase 7: Review Validation, which executes these gates as part of the review lifecycle.
- `references/review-checklist.md` — Execution verification tool that references these gates. Verifies that methodology phases were executed.
- `references/review-consistency.md` — Defines consistency methodology validated by the Consistency Gate.
- `references/review-scoring.md` — Defines scoring methodology validated by the Scoring Gate.
- `references/decision-matrix.md` — Defines decision methodology validated by the Decision Gate.
- `references/review-output-contract.md` — Defines output structure validated by the Output Contract Gate.
- `references/anti-patterns.md` — Defines integrity failure symptoms and review smells referenced by the Integrity Gate.
