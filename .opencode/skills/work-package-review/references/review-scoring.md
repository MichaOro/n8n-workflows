---
name: review-scoring
description: Canonical engineering standard for scoring implementation readiness in Work Package Reviews. Defines scoring methodology, score calculation, score interpretation, and common scoring mistakes. Consistency and calibration methodology is defined in references/review-consistency.md. Decision methodology is defined in references/decision-matrix.md.
---

# Review Scoring

## Purpose

This document defines how implementation readiness is measured numerically. It is the single source of truth for all scoring concepts within the Work Package Review framework.

Scoring converts qualitative findings into a quantitative readiness measure. A score does not replace engineering judgment — it encodes it in a reproducible form. Two reviewers applying this methodology to the same findings must produce the same score.

This document defines **how scores are produced**. It does not define review methodology, engineering principles, or decision rules. Decision rules (PASS / FAIL / CONDITIONAL PASS) are defined in `references/decision-matrix.md`. The decision matrix transforms the score produced by this document into an engineering decision.

## Scoring Philosophy

### What a Score Represents

A readiness score represents the degree to which a Work Package can be implemented correctly using only the Work Package, the repository, and project documentation. It does not measure:

- **The quality of the Work Package as a document.** A score of 40 does not mean the Work Package is poorly written. It means the implementer lacks sufficient information to proceed safely.
- **The complexity of the implementation.** A feature that is difficult to implement should not receive a lower readiness score unless the Work Package fails to account for the complexity.
- **The value of the feature.** A score reflects implementation readiness, not business priority.
- **The reviewer's confidence.** Confidence is a separate assessment (see `references/decision-matrix.md`).

### Why Numeric Scoring

A numeric score provides:

- **Precision**: "Scored 65" communicates more than "not ready."
- **Comparability**: Scores across different Work Packages and reviewers can be compared to identify systemic issues.
- **Threshold clarity**: The boundary between PASS and FAIL is unambiguous.
- **Trend tracking**: Score distributions over time reveal whether the Work Package process is improving or degrading.

### Score Integrity

A score is valid only when the methodology has been followed. A score produced without repository analysis, simulation, or gap analysis is meaningless regardless of the number. Score validation — including methodology compliance verification — is defined in the Scoring Gate of `references/quality-gates.md`.

## Score Components

The readiness score is composed of the following components, applied in order:

### 1. Simulation Ceiling

The simulation ceiling is the maximum possible score determined by the worst gap found during implementation simulation. It is the primary constraint on the final score.

The ceiling is determined during the simulation phase defined in `references/implementation-readiness.md`. The mapping from simulation gaps to ceilings is:

| Worst Gap | Score Ceiling |
|---|---|
| No gaps | 100 |
| Minor gaps (implementer can proceed with brief clarification) | 89 |
| Significant gaps (implementer must guess or ask) | 69 |
| Critical gaps (implementer would produce wrong code) | 39 |

The ceiling is a hard upper bound. No score may exceed it.

### 2. Severity-Based Deductions

After the ceiling is established, deductions are applied based on the severity of findings in each review dimension (defined in `references/review-process.md`).

| Finding Severity | Deduction Effect |
|---|---|
| Critical | Score is reduced to 0 regardless of ceiling. A single Critical finding makes the Work Package unscorable. |
| High | Significant deduction proportional to the finding's impact on implementation correctness and maintainability. Each High finding reduces the score by 10-25 points depending on impact. |
| Medium | Minor deduction. Each Medium finding reduces the score by 3-8 points. |
| Low | Minimal or no deduction. Low findings are suggestions and may not affect the score. |

Deductions are not purely additive. Multiple findings from the same root cause are treated as a single deduction at the severity of the worst finding.

### 3. Ceiling Constraint

After deductions are applied, the score is capped at the simulation ceiling. If the preliminary score is 75 but the ceiling is 69, the final score is capped to 69.

## Score Calculation

### Formula

```
final_score = min(ceiling, max(0, 100 - deductions))
```

Where:
- `ceiling` = simulation ceiling from `references/implementation-readiness.md`
- `deductions` = total points deducted based on severity-weighted findings

### Simplified Procedure

1. Start with a preliminary score of 100.
2. Apply the simulation ceiling: if the ceiling is below 100, reduce the preliminary score to the ceiling.
3. For each Critical finding: set the score to 0. Stop further calculation.
4. For each High finding: subtract 10-25 points depending on impact.
5. For each Medium finding: subtract 3-8 points depending on impact.
6. For each Low finding: subtract 0-2 points (typically 0).
7. If the score falls below 0, clamp to 0.
8. The final score is the result of step 7.

### Deduction Calibration

Reviewers should calibrate deductions within the specified ranges based on:

- **Impact scope**: A finding that affects a single internal function has lower impact than one that affects a public API contract consumed by multiple services.
- **Remediation effort**: A finding that requires minor clarification has lower impact than one that requires architectural changes.
- **Risk profile**: A finding that could cause silent data corruption has higher impact than one that causes a compilation error (which would be caught immediately).
- **Reversibility**: A finding that creates irreversible schema changes has higher impact than one that adds an additional parameter to an internal function.

When uncertain, use the midpoint of the range. For example, default High deductions to 17-18 points, Medium to 5-6 points.

## Score Interpretation

| Score Range | Meaning |
|---|---|---|
| 90-100 | Ready. The score indicates implementation can begin without clarification. All critical information is present, and any remaining issues are minor. |
| 70-89 | Conditionally ready. The score indicates minor gaps exist that an implementer could resolve independently. The Work Package is fundamentally sound but has non-blocking issues. |
| 40-69 | Not ready. The score indicates significant gaps or inconsistencies that require Work Package revision. The implementer would need to make assumptions likely to be incorrect. |
| 0-39 | Rejected. The score indicates fundamental problems make the Work Package unsuitable as written. |

Score ranges describe readiness. The decision (PASS / FAIL / CONDITIONAL PASS) is determined by the decision model in `references/decision-matrix.md`, which evaluates the score alongside ceiling, severity, confidence, and risk factors.

### Score Boundaries

Boundaries between score ranges are not sharp thresholds. A score of 69 and 70 differ by a single point, but the difference may reflect one minor finding. Reviewers should not treat boundaries as absolute — the score communicates a readiness band, and the decision communicates the engineering judgment.

When a score falls within 3 points of a boundary, the reviewer should verify that the deduction calibration was appropriate. If uncertain, prefer the lower range — overestimating readiness is more harmful than underestimating. Escalation guidance is defined in `references/decision-matrix.md`.

## Score and Ceiling Relationship

The simulation ceiling is the dominant constraint on the score. It ensures that a Work Package with severe simulation gaps cannot receive a passing score regardless of how well other dimensions are specified.

```
Examples:
  Ceiling 100, no findings          → Score 100
  Ceiling 100, 2 Medium findings    → Score 90-94
  Ceiling 89, no findings           → Score 89  (capped by ceiling)
  Ceiling 69, 1 High finding        → Score 44-59 (capped by ceiling at 69, minus High deduction)
  Ceiling 39, any findings          → Score 0-39 (already failing range)
  Ceiling 100, 1 Critical finding   → Score 0  (Critical sets score to 0)
```

If the simulation ceiling maps the worst gap as "significant" (ceiling 69), the score cannot exceed 69 regardless of how well-specified the rest of the Work Package is. This prevents a Work Package with a fundamental simulation gap from passing based on the quality of its other sections.

## Missing Information and Uncertainty

### Missing Dimensions

If a review dimension has not been evaluated (e.g., repository analysis could not be completed for all areas), the score must account for the uncertainty. Incomplete evaluation handling — including score adjustments and confidence requirements — is defined in the Scoring Gate of `references/quality-gates.md`.

### Uncertainty in Findings

When a finding's impact is uncertain (the finding could be Critical under specific conditions but is not guaranteed), classify it at the lower severity per the severity assignment rules in `references/implementation-readiness.md` and document the conditions that would escalate it.

When the reviewer is uncertain about a finding's severity, choose the lower severity. Over-classification erodes credibility.

### Evidence Gaps

When evidence for a finding is weak per the evidence hierarchy in `references/evidence-and-justification.md`, the deduction should be at the lower end of the range for that severity. Strong evidence supports higher deductions within the range.

Score consistency and reviewer calibration methodology are defined in `references/review-consistency.md`. All consistency concepts — intra-review consistency, inter-review consistency, calibration reviews, drift detection, and consistency metrics — are owned by that document.

## Common Scoring Mistakes

### Ignoring the Ceiling

Calculating the readiness score without referencing the simulation ceiling. The ceiling is a hard constraint — the score cannot exceed it regardless of how well-specified the rest of the Work Package is.

### Over-Deduction from Multiple Related Findings

Applying full deductions for multiple findings that stem from the same root cause. If three High findings all trace to the same missing specification, they should be treated as a single High deduction, not three.

### Under-Deduction from High-Impact Findings

Deducting only the minimum for a High finding that affects a public API, data schema, or cross-service contract. Impact scope must guide the deduction amount within the range, not the minimum by default.

### Treating Score as Precision

Interpreting a score of 73 as meaningfully different from 72. Scores within the same range are functionally equivalent. The score communicates a band, not a precise measurement. Focus on the range and the decision, not the exact number.

### Scoring Without Simulation

Producing a score without executing the simulation. A score without a simulation ceiling is invalid because the ceiling is the primary constraint. Simulation is mandatory before scoring.

### Ceiling Violation by Dimension Score

Calculating per-dimension scores that exceed the simulation ceiling. If the ceiling is 69, no dimension score may exceed 69 — the ceiling applies to the overall score and is a property of the simulation, not of individual dimensions.

## Scoring Example

### Work Package: "Add Email Notification Service"

**Simulation results**:
- 1 Significant gap: Error handling for SMTP connection failures is not specified
- Ceiling: 69

**Findings**:
- F-001 [High]: No error handling for SMTP failures (error handling dimension)
- F-002 [Medium]: Missing configuration documentation for SMTP host (specification dimension)
- F-003 [Low]: Suggestion to use the project's existing logger instead of a new one (reuse dimension)

**Deductions**:
- Ceiling caps score at 69
- F-001: High deduction = -18 points (SMTP failure handling affects reliability)
- F-002: Medium deduction = -5 points (config is documented elsewhere, just not in the WP)
- F-003: Low deduction = 0 points (suggestion only)

**Calculation**:
```
preliminary = min(100, 69) = 69
after_high = 69 - 18 = 51
after_medium = 51 - 5 = 46
after_low = 46 - 0 = 46
final = max(0, 46) = 46
```

**Result**: Score 46 (Not ready). The ceiling of 69 was already in the failing range, and the High finding reduced the score further. The Work Package needs revision.
