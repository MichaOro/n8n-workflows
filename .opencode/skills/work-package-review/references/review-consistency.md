---
name: review-consistency
description: Canonical engineering standard for review consistency in Work Package Reviews. Defines deterministic reviews methodology, intra-review and inter-review consistency, reviewer calibration, cross-LLM consistency, consistency validation, drift detection, and consistency metrics. Reviews are deterministic when methodology and evidence determine the outcome — not the specific reviewer.
---

# Review Consistency

## Purpose

This document is the single source of truth for all consistency concepts in the review framework. Consistency means that two reviewers applying the same methodology to the same Work Package and repository arrive at the same findings, scores, confidence, and decision.

The goal is consistent engineering decisions — not identical reviews. Different wording and different LLMs are acceptable as long as findings, scores, confidence, and decisions converge.

The **Deterministic Reviews** principle is owned by `references/review-principles.md`. This document defines the methodology for achieving and verifying consistency.

## Why Consistency Matters

A review that depends on the specific reviewer is not an engineering gate — it is a personality test. The framework exists because individual judgment varies. Standardized principles, methodology, and evidence requirements make outcomes reproducible.

Disagreement between reviewers exposes a gap in the methodology, not a difference in taste. Every inconsistency should be traced to a specific principle, evidence gap, or methodological ambiguity — then the framework is updated to eliminate the ambiguity.

## Reviewer Calibration

### Purpose

Calibration ensures that different reviewers apply severity ranges, deduction ranges, and decision rules consistently. Without calibration, the framework produces reviewer-dependent outcomes.

### Calibration Reviews

Periodically, completed reviews should be independently re-evaluated:

1. Select a previously reviewed Work Package with findings.
2. Have a second reviewer independently score the same findings.
3. Compare scores. If the difference exceeds 5 points, identify the source of the discrepancy.
4. Determine whether the discrepancy is due to:
   - Different findings (one reviewer identified a finding the other did not)
   - Different severity classification (one rated a finding High, the other Medium)
   - Different deduction calibration (both rated it High but deducted different amounts)
   - Different decision interpretation
5. Update calibration guidance if the discrepancy reveals a systematic interpretation issue.

### Cross-Review Calibration

Periodic calibration reviews identify systematic bias patterns across reviewers. When a pattern is detected:

- Document the pattern in the framework's calibration log.
- Update methodology or guidance to reduce ambiguity.
- Re-calibrate affected reviewers.

### Calibration Log

Maintain a log of calibration discrepancies. Each entry records:

- The Work Package and review date
- The discrepancy (score difference, severity difference, decision difference)
- The root cause (methodology gap, reviewer interpretation, evidence difference)
- The resolution (methodology update, guidance clarification, reviewer training)
- The date the resolution was applied

## Finding Consistency

### Intra-Review Consistency

Within a single review:

- All findings must use consistent language patterns. If one finding starts with "The Work Package omits...", all findings should follow the same pattern.
- Similar severity must be assigned to similar issues. A missing error path for SMTP and a missing error path for database timeouts should receive the same severity if the impact is similar.
- Deductions must be consistent with severity. Two High findings with different deduction amounts must document why the impacts differ.
- No finding contradicts another finding.
- Severity assignments are consistent across similar findings.

### Inter-Review Consistency

Across different reviews:

- The same issue found in different Work Packages must produce findings with similar wording, severity, and structure.
- Two reviewers evaluating the same Work Package should produce findings that overlap in content, even if wording differs.
- If two reviewers produce fundamentally different findings for the same Work Package, the framework methodology or the reviewers' understanding of the methodology requires examination. The disagreement should be traced to a specific principle, evidence gap, or methodological ambiguity.

### Language Consistency

Use consistent language across all findings:

- **Active voice**: "The Work Package omits error handling" not "Error handling has been omitted by the Work Package."
- **Engineering terms**: "contradicts the existing schema" not "is wrong."
- **Specific references**: "src/services/order.ts:45" not "the order service file."
- **Severity language**: Use the exact severity labels (Critical, High, Medium, Low) — never synonyms like "major," "minor," "important."

## Score Consistency

### Intra-Review Consistency

All findings in a single review must use consistent deduction ranges. A High finding in one dimension must not receive a 25-point deduction while a similar High finding in another dimension receives 10 points unless there is a documented difference in impact scope.

### Inter-Review Consistency

Two reviewers scoring the same Work Package with the same findings must produce the same score. If reviewers produce different scores, the difference must be traceable to:

- Different findings (one reviewer identified a finding the other did not)
- Different severity classification (one rated a finding High, the other Medium)
- Different deduction calibration (both rated it High but deducted different amounts)

Inter-review inconsistency identifies a gap in the scoring methodology. When discovered, the methodology should be updated to eliminate the ambiguity.

## Decision Consistency

### Intra-Review Consistency

Within a single review:

- The decision must be consistent with the severity-weighted finding set. A review with two High findings about data loss cannot produce a PASS.
- Severity assignments must be consistent across similar findings. A missing error path for SMTP and a missing error path for database timeouts warrant similar severity if the impact is similar.
- The decision must be consistent with the confidence level. Low confidence should not produce a PASS.
- The decision rationale must explain how the decision was reached from the inputs, not restate the findings.

### Inter-Review Consistency

Across different reviews:

- Two reviewers evaluating the same Work Package must reach the same decision if their findings and scoring agree.
- If reviewers reach different decisions, the difference must be traceable to:
  - Different findings (one identified an issue the other did not)
  - Different severity classification (one rated a finding High, the other Medium)
  - Different scoring (one deducted more or less)
  - Different confidence assessment
- Inter-review inconsistency identifies a gap in the methodology. When discovered, the decision methodology should be updated.

### Decision Validation

Before finalizing a decision, verify:

- The decision is consistent with the severity-weighted finding set.
- The decision is consistent with the confidence level.
- The simulation ceiling has been correctly applied.
- The score has been correctly calculated per `references/review-scoring.md`.
- All Critical findings have been correctly identified and accounted for.
- If CONDITIONAL PASS, the implementation guidance is complete and actionable.
- If FAIL, the blocking issues are clearly identified and the path to resolution is described.

## Report Consistency

### Internal Consistency

- All findings must use consistent severity language within a single report.
- Similar findings must receive similar severity classification.
- The decision must be consistent with the score and findings.
- Score boundaries and decision rules must be applied consistently.

### Cross-Report Consistency

- Two reports for the same review type must use identical section ordering.
- Findings with similar severity in different reports must use identical presentation format.
- Score ranges must mean the same thing across reports of the same type.

## Cross-LLM Consistency

When reviews are performed by different LLMs:

- The methodology and output contract must be the primary source of structure, not the LLM's default formatting.
- If two LLMs produce structurally different reports for the same inputs, the output contract needs refinement.
- LLM-specific biases (verbose vs. terse, cautious vs. confident) should be identified during calibration reviews.
- Language consistency rules override LLM default writing patterns.

## Consistency Validation

Consistency validation criteria — including finding consistency, score consistency, decision consistency, report consistency, and simulation consistency — are defined in the Consistency Gate of `references/quality-gates.md`. All consistency validation concepts are owned by that document.

### Validation Methods

Consistency can be validated through:

- **Self-review**: The reviewer checks their own output for consistency before release.
- **Peer review**: A second reviewer validates the output for consistency issues.
- **Calibration review**: A periodic structured comparison of two independent reviews of the same Work Package.

## Consistency Drift Detection

### What Is Drift

Consistency drift occurs when a reviewer's application of the framework changes over time without a corresponding methodology change. Symptoms include:

- Severity inflation or deflation over consecutive reviews
- Deduction creep (gradually higher or lower deductions for the same severity)
- Decision boundary drift (PASS threshold shifting without methodology change)
- Language drift (departure from standard phrasing patterns)

### Drift Detection Methods

- Compare a reviewer's recent reviews against their earlier reviews for pattern changes.
- Compare a reviewer's output against calibration benchmarks.
- Track deduction amounts per severity level over time and flag outliers.
- Monitor decision frequency (ratio of PASS to FAIL reviews) for unexplained shifts.

### Drift Correction

When drift is detected:

1. Determine whether the drift reflects improved understanding (good) or methodology degradation (bad).
2. If degradation, re-calibrate the reviewer against the current methodology.
3. Update methodology guidance if drift was caused by ambiguity in the methodology itself.

## Consistency Metrics

### Quantitative Metrics

- **Inter-reviewer score agreement**: Maximum 5-point difference for calibration reviews.
- **Decision agreement rate**: Percentage of calibration reviews where independent reviewers reach the same decision.
- **Severity classification agreement**: Percentage of findings where independent reviewers assign the same severity.
- **Finding overlap**: Percentage of findings in two independent reviews that describe the same issues (regardless of wording).

### Qualitative Metrics

- **Language consistency audit**: Are all findings using active voice, standard severity labels, and engineering terms?
- **Structure consistency audit**: Do all reports follow the same section ordering and presentation format?
- **Rationale consistency audit**: Do decision rationales explain how the decision was reached from inputs, rather than restating findings?

### Interpreting Metrics

- Metrics identify patterns, not individual errors. A single discrepancy is a data point; a trend is a methodology gap.
- When metrics indicate inconsistency, investigate the root cause before making methodology changes.

## Consistency Anti-Patterns

### Severity Inflation

Raising severity to compensate for uncertainty. A finding that is truly Medium is classified as High because the reviewer wants to ensure it is addressed.

**Correction**: Severity must be based on impact, not urgency or uncertainty. Follow the severity model in `references/implementation-readiness.md`.

### Deduction Creep

Gradually increasing deduction amounts within severity ranges over consecutive reviews. The first review deducts 10 points for a High finding; the tenth review deducts 25 points for a similar High finding — with no methodology change.

**Correction**: Track deduction ranges per severity level. Review calibration benchmarks before every review.

### Decision Boundary Drift

The PASS threshold shifting without methodology change. A set of findings that produced a FAIL in January produces a PASS in June — with no methodology update.

**Correction**: Decision methodology is defined in `references/decision-matrix.md`. Follow it literally. Do not override based on intuition.

### Over-Calibration

So much calibration that reviewers lose independence. If every review produces identical output because reviewers have been trained to think identically, the calibration has gone too far.

**Correction**: Calibration ensures consistency of outcomes, not identity of thought. Different paths to the same conclusion are acceptable.

### Calibration Avoidance

Skipping calibration reviews because they take time. This is the most common consistency anti-pattern.

**Correction**: Schedule calibration reviews as part of the regular review process. A team that does not calibrate cannot guarantee deterministic outcomes.

## Framework Evolution without Drift

When the framework methodology is updated:

1. Document the change in the revision history of the affected document.
2. Run a calibration review comparing a pre-change review against the post-change methodology to verify the change does not unintentionally alter outcomes for previously reviewed Work Packages.
3. If the change alters outcomes for previously reviewed Work Packages, assess whether the change is a correction (good) or drift (bad). Corrections override prior outcomes; drift must be corrected.
4. Update calibration baselines after methodology changes to reflect the new methodology.

## Related Documents

- `references/review-principles.md` — Defines the Deterministic Reviews principle that this document implements.
- `references/review-scoring.md` — Defines scoring calculation; consistency of scoring is covered here.
- `references/decision-matrix.md` — Defines decision methodology; consistency of decisions is covered here.
- `references/finding-guidelines.md` — Defines finding construction; consistency of findings is covered here.
- `references/review-output-contract.md` — Defines output structure; consistency of reports is covered here.
- `references/anti-patterns.md` — Lists calibration avoidance and other anti-patterns.
