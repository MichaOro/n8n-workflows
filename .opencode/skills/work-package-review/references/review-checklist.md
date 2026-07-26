---
name: review-checklist
description: Execution verification checklist for Work Package Reviews. Verifies that every methodology phase was executed before output release. Quality gate validation is defined in references/quality-gates.md. Defers methodology to references/review-process.md, references/repository-analysis.md, references/implementation-readiness.md, references/review-scoring.md, references/decision-matrix.md, references/review-output-contract.md.
---

# Review Checklist

## Purpose

This checklist verifies that every methodology phase has been executed before the review output is released. It is the execution verification step in the review lifecycle (Phase 7 of `references/review-process.md`).

This document verifies **that phases were executed**. It does not define **how** to review, score, or decide — those are defined in the methodology documents. Quality gate validation — whether the completed review satisfies framework standards — is defined in `references/quality-gates.md`.

## Preparation (Phase 1)

- The Work Package was read in full before examining any code.
- Initial hypotheses were formed about what the review would find.
- The relevant sections of the repository were identified based on domain, module names, and file paths.

## Repository Analysis (Phase 2)

- Repository analysis was executed per `references/repository-analysis.md`.
- Key findings documented: architecture pattern, conventions, relevant existing code, reuse candidates.
- Initial hypotheses from Phase 1 were validated or refuted with evidence.

## Implementation Readiness (Phase 3)

- Implementation Simulation was executed per `references/implementation-readiness.md`.
- Every simulation gap was recorded with location, missing information, and consequence.
- The simulation ceiling was determined from the worst gap.
- Repository Gap Analysis (four-layer comparison) was executed per `references/implementation-readiness.md`.
- Every discrepancy from gap analysis is represented in the finding set.

## Dimension Evaluation (Phase 4)

- Every Review Dimension was evaluated systematically.
- Each dimension produced findings with the Engineering Justification Chain (per `references/evidence-and-justification.md`).
- Severity was assigned per `references/implementation-readiness.md`.
- Findings were cross-checked across dimensions for root cause relationships.

## Readiness Determination (Phase 5)

- All findings were aggregated into the final finding list.
- The readiness score was calculated per `references/review-scoring.md`.
- Confidence was assessed per `references/decision-matrix.md`.
- The decision (PASS / FAIL / CONDITIONAL PASS) was determined per `references/decision-matrix.md`.
- The decision was verified against the quality gates in `references/quality-gates.md`.

## Output (Phase 6)

- The review output was produced following `references/review-output-contract.md`.
- Every finding has evidence, every severity is justified, and every blocked item has a clear path forward.

## Quality Gate Validation

- The completed review has been validated against the quality gates in `references/quality-gates.md`.
- All mandatory gates pass before the review is released.
- Any advisory gate failures are documented.

## Self-Verification

- The checklist itself has been verified by re-reading the relevant findings and methodology outputs, not by assuming correctness.
