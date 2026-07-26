---
name: decision-matrix
description: Canonical engineering standard for transforming review results into engineering decisions. Defines approval states, decision rules, conditional pass guidance, confidence assessment, escalation paths, exception handling, and risk-based decisions. Consistency methodology is defined in references/review-consistency.md. Does not define scoring calculation, implementation simulation, gap analysis, or severity classification.
---

# Decision Matrix

## Purpose

This document defines how engineering reviews are transformed into engineering decisions. It is the single source of truth for all decision methodology within the framework.

A score is information. A decision is engineering judgment. The decision matrix does not simply map score to decision. It evaluates score, ceiling, confidence, evidence quality, critical findings, implementation blockers, uncertainty, architectural risk, and repository understanding to arrive at a sound engineering conclusion.

Review Scoring (`references/review-scoring.md`) produces measurements. This document consumes those measurements and produces a decision. Implementation Readiness (`references/implementation-readiness.md`) defines simulation, gap analysis, and severity classification — the inputs that feed into scoring and then into this decision model.

## Decision Philosophy

### What a Decision Represents

A decision is an engineering judgment about whether implementation should proceed, be revised, or be rejected. It answers one question: can a competent engineer build this Work Package correctly using only the Work Package, the repository, and project documentation — without requiring additional clarification?

### Score vs. Decision

A score is a measurement. A decision is a judgment. The same score may lead to different decisions depending on:

| Factor | How It Affects the Decision |
|---|---|
| Ceiling | A score of 85 with a ceiling of 100 is different from a score of 85 with a ceiling of 89 — the latter means simulation already identified significant gaps. |
| Critical findings | A single Critical finding reduces the score to 0 regardless of other scores. |
| Confidence | Low confidence with Medium findings may justify revision even if the numerical score is passing. |
| Evidence quality | Findings based on weak evidence carry less weight than findings based on code evidence. |
| Architectural risk | A medium-severity architectural drift finding may be more significant than a high-severity documentation gap. |
| Reversibility | Decisions that create irreversible changes (schema migrations, API contracts) warrant higher scrutiny. |

### Deterministic Decisions

The decision model is designed so that different reviewers applying the same methodology to the same inputs reach the same conclusion. When reviewers disagree, the disagreement must be traceable to a difference in findings, severity, scoring, or a methodological gap.

### Certainty and Decisions

Certainty is rare in engineering decisions. The decision model accounts for uncertainty through confidence assessment and escalation paths. A reviewer who expresses absolute certainty about a decision is likely unaware of their assumptions.

---

## Part 1: Approval States

The framework defines three approval states:

| State | Meaning |
|---|---|
| PASS | Implementation may begin. All readiness conditions are met, or any remaining issues are minor and documented. |
| FAIL | Implementation must not begin. The Work Package requires revision and re-review before any implementation work. |
| CONDITIONAL PASS | Implementation may begin on well-defined portions while unresolved issues are addressed. The implementer receives clear guidance on what is blocked versus unblocked. |

### PASS

PASS means the Work Package is implementation-ready. A competent engineer can build it correctly using only the Work Package, the repository, and project documentation — without requiring additional clarification.

PASS does not mean the Work Package is perfect. Minor issues (Low findings) may exist. It means no issue prevents correct implementation.

### FAIL

FAIL means the Work Package is not implementation-ready. Implementation must not begin until the identified issues are resolved and the Work Package is re-reviewed.

FAIL does not mean the Work Package is irredeemable. It means the current version would lead to incorrect implementation, architectural damage, or unacceptable risk.

### CONDITIONAL PASS

CONDITIONAL PASS means the Work Package is partially ready. The implementer may proceed with well-understood portions while the Work Package author addresses unresolved issues.

CONDITIONAL PASS is not a weak PASS. It is a precise engineering instruction about what can proceed and what cannot. A CONDITIONAL PASS without clear implementation guidance is a failed decision.

---

## Part 2: Decision Rules

The decision model converts the severity-weighted finding set, the readiness score, and the simulation ceiling into a PASS / FAIL / CONDITIONAL PASS determination.

### Inputs

Every decision evaluates:

- **Readiness Score**: A numerical measurement from `references/review-scoring.md` (0-100).
- **Simulation Ceiling**: The maximum possible score determined by the worst simulation gap (from `references/implementation-readiness.md`).
- **Severity-weighted findings**: Critical, High, Medium, and Low findings classified per the severity model in `references/implementation-readiness.md`.
- **Confidence**: High, Medium, or Low (see Part 3).
- **Repository understanding**: Thorough, Adequate, or Partial (from `references/repository-analysis.md`).

### Decision Rules

```
PASS if:
  - No Critical findings
  - Simulation ceiling >= 90
  - Score >= 70
  - All High findings have clear remediation guidance
  - Confidence is not Low

FAIL if:
  - Any Critical findings exist
  - Simulation ceiling < 70
  - Score < 70
  - High findings exist without sufficient independent-resolution guidance
  - Confidence is Low and the uncertainty affects the decision

CONDITIONAL PASS if:
  - No Critical findings
  - Simulation ceiling >= 70
  - Score 70-89
  - All findings have documented workarounds or remediation steps
  - Confidence is at least Medium
```

### Rule Application Order

Apply in order:

1. **Check for Critical findings**: If any exist, the decision is FAIL. Score is 0.
2. **Check the simulation ceiling**: If ceiling < 70, the decision is FAIL regardless of other factors.
3. **Calculate the score**: Follow `references/review-scoring.md`.
4. **Apply the PASS and CONDITIONAL PASS rules**: Both require ceiling >= 70, score >= 70, and no Critical findings.
5. **Distinguish PASS from CONDITIONAL PASS**: PASS requires ceiling >= 90 and score >= 70. CONDITIONAL PASS applies when ceiling is 70-89 or score is 70-89.
6. **Apply confidence as a modifier**: Low confidence may downgrade a PASS to CONDITIONAL PASS or a CONDITIONAL PASS to FAIL, depending on what the uncertainty affects.
7. **Evaluate High findings**: If High findings exist without remediation guidance, downgrade the decision one level (PASS → CONDITIONAL PASS → FAIL).

### Escalation Rules

When the decision falls into uncertainty — for example, borderline scores (70 ± 3), conflicting findings, or low confidence in a critical area — the reviewer should escalate rather than decide:

- **Score within 3 points of a boundary**: Consult a second reviewer before finalizing. If unavailable, prefer the lower decision state.
- **Low confidence in a critical subsystem**: Escalate to the subsystem owner for review before making a final decision.
- **Disagreement between score and engineering judgment**: If the score suggests PASS but the reviewer's judgment suggests otherwise, document the discrepancy and escalate. Do not override the score without documented justification.
- **Novel or unprecedented change**: Escalate to an architecture review board if the Work Package introduces patterns, dependencies, or architectural changes that the framework's methodology does not cover.

### Exception Handling

The following situations override the normal decision rules:

| Situation | Override |
|---|---|
| Security vulnerability introduced | Immediate FAIL regardless of score. Escalate to security team. |
| Data loss or corruption risk | Immediate FAIL regardless of score. Escalate to data engineering. |
| Irreversible architectural change with no migration path | FAIL unless the Work Package includes a coexistence strategy. |
| Work Package explicitly marked as draft | Mark as incomplete with Low confidence. Do not score. Gate criteria are defined in `references/quality-gates.md`. |
| Repository analysis could not be completed | Decision confidence is Low. Decision is provisional. Incomplete analysis handling is defined in `references/quality-gates.md`. |
| Simulation could not be completed | Decision confidence is Low. Decision is provisional. Incomplete analysis handling is defined in `references/quality-gates.md`. |

---

## Part 3: Confidence Assessment

### Confidence Levels

| Confidence | Meaning | When to Use |
|---|---|---|
| High | The repository is well-understood and the findings are unambiguous. Different reviewers would reach the same decision. | Repository analysis was thorough per `references/repository-analysis.md`, evidence is strong per `references/evidence-and-justification.md`, simulation was complete, no significant unknowns remain. |
| Medium | Most of the repository is understood, but some areas were not fully explored. Findings are likely correct but some assumptions remain untested. | Repository is large and the reviewer prioritized the most relevant areas. Simulation depth was adequate but not deep in all areas. |
| Low | Repository analysis was limited or simulation uncovered significant unknowns. The decision is provisional. | Repository access was constrained, the Work Package touches unfamiliar subsystems, time prevented thorough analysis, or simulation revealed gaps that could not be fully evaluated. |

### How Confidence Affects the Decision

Confidence modifies the decision determined by the decision rules:

- **High confidence**: The decision stands as determined by the rules.
- **Medium confidence**: The decision stands, but the Decision Rationale must document what would need to change for confidence to reach High.
- **Low confidence**: The decision is provisional. If the rules produce PASS with Low confidence, the reviewer should downgrade to CONDITIONAL PASS or FAIL depending on what the uncertainty affects. If the rules produce FAIL with Low confidence, the FAIL stands — Low confidence does not override a FAIL.

### Communicating Confidence

Every decision must include a confidence label and a rationale explaining the confidence level. The Decision Rationale must answer: "What would need to change for the reviewer to reach High confidence?"

### Factors That Reduce Confidence

- Repository analysis was not thorough per `references/repository-analysis.md`.
- Simulation was shallow or incomplete.
- Evidence is primarily convention-based or implication-based (weak evidence per `references/evidence-and-justification.md`).
- The Work Package touches subsystems the reviewer has not fully explored.
- The reviewer has identified unknown unknowns — areas where the Work Package's impact cannot be fully assessed.
- Time pressure prevented complete analysis.

---

## Part 4: Conditional Pass Guidance

When issuing a conditional pass, the reviewer must specify:

### Required Output

1. **What can proceed immediately**: The well-defined portions of the Work Package that the implementer can start working on without risk.
2. **What requires clarification**: The specific items that need resolution before the implementer can complete the affected portions.
3. **Recommended order of operations**: A suggested sequence — implement the well-defined parts first, resolve ambiguities before reaching them.

### Guidance Standards

- Be specific about what the implementer should do when blocked: "Contact the database team to confirm the migration plan" not "resolve the schema issue."
- Distinguish between blocked and unblocked work clearly.
- Note whether unresolved items require Work Package revision or can be resolved during implementation.
- If a finding has multiple valid resolutions, recommend the one that best aligns with project conventions.

---

Decision consistency methodology — including intra-review consistency, inter-review consistency, and decision validation — is defined in `references/review-consistency.md`. All decision consistency concepts are owned by that document.

---

## Part 6: Risk-Based Decisions

### Risk Factors

The following risk factors should influence the decision beyond the numerical score:

| Risk Factor | Impact on Decision |
|---|---|
| Architectural drift | A Work Package that accelerates architecture degradation may warrant FAIL even with a passing score. |
| Security boundary crossing | A Work Package that crosses a security boundary requires FAIL unless the security implications are fully documented. |
| Data migration | A Work Package with irreversible data changes requires higher scrutiny. CONDITIONAL PASS is not sufficient — the migration plan must be reviewed separately. |
| Cross-team dependency | If the Work Package depends on another team's deliverables, the dependency must be documented. If the dependency is not yet specified, the decision should be CONDITIONAL PASS at most. |
| Third-party dependency introduction | A new external dependency requires justification. If the dependency is not justified, FAIL. |
| Performance-critical path | If the Work Package modifies a performance-critical path and does not specify performance requirements, the decision should be CONDITIONAL PASS or FAIL. |

### When to Escalate Instead of Decide

Escalate to a senior engineer, architect, or review board when:

- The Work Package introduces a new architectural pattern not present in the repository.
- The Work Package removes or deprecates a widely-used API or module.
- The Work Package changes the security model.
- The reviewer has Low confidence and cannot resolve the uncertainty.
- The Work Package proposes a change that contradicts an explicit architectural decision record (ADR) without addressing it.
- The decision requires business judgment that the reviewer cannot make (e.g., accepting a known security risk).

---

## Part 7: Common Decision Anti-Patterns

### Score-Driven Decisions

**Why it happens**: The reviewer starts with a target decision in mind and adjusts findings or severity to produce that decision. This may be conscious (the reviewer wants the Work Package to pass) or unconscious (the reviewer believes the Work Package is ready and subconsciously classifies findings to produce a passing score).

**How to recognize it**:
- The decision and the written decision rationale describe different levels of readiness.
- Severity assignments seem influenced by how close the score is to a boundary.
- The reviewer mentions the target outcome during the review ("this should pass").
- Deductions cluster near the minimum of the range for every finding.

**Why it is dangerous**: The decision model exists to encode engineering judgment in a reproducible form. When the desired decision drives the evidence rather than the reverse, the decision loses meaning. The review becomes a justification for a pre-determined outcome rather than an honest assessment.

**How to recover**: Calculate the decision only after all findings are classified and scored. If the calculated decision differs from the expected outcome, investigate whether findings were missed or severity was misapplied.

**How to prevent it**: Classify severity before calculating the score. Calculate the score before determining the decision. The decision is a derivation, not an input.

### Checklist-Driven Decisions

**Why it happens**: The reviewer treats the decision rules as a mechanical checklist — computing decision from score without exercising engineering judgment about whether the Work Package is actually implementable.

**How to recognize it**: The decision rationale restates the rules without explaining why this particular Work Package's situation warrants this decision. The reviewer cannot explain the decision without reading from the rules.

**Why it is dangerous**: Decision rules encode typical patterns, but every Work Package is unique. Rules alone cannot capture architectural risk, subsystem maturity, or team context. A rule-compliant decision can still be wrong.

**How to recover**: Put the rules aside and ask: "Does this Work Package pass or fail based on my engineering judgment?" If the answer differs from the rule result, investigate why. The rules may need updating.

### Certainty in Decisions

Expressing absolute certainty about a decision that involves trade-offs, uncertainty, or insufficient evidence.

**Resolution**: Acknowledge uncertainty in the decision rationale. "This Work Package passes with Medium confidence because the notification module's test infrastructure was not fully explored. Full exploration would raise confidence to High."

---

## Part 8: Examples

### Example 1: PASS

**Input**: Score 92, Ceiling 100. No Critical findings. Two Low findings (suggestions for formatting). High confidence. Thorough repository analysis.

**Decision**: PASS.

**Rationale**: Score is well within the PASS range. The ceiling is unconstrained. Findings are minor and do not affect implementation correctness. High confidence is supported by thorough analysis.

### Example 2: FAIL — Critical Finding

**Input**: Score 0 (set to 0 by Critical finding). Ceiling 69 (one Critical simulation gap). One Critical finding: the Work Package proposes a schema change that would cause data loss in the existing customer table. Medium confidence.

**Decision**: FAIL.

**Rationale**: The Critical finding about data loss makes the Work Package unscorable regardless of other merits. The Work Package must be revised to include a migration plan with rollback, then re-reviewed.

### Example 3: CONDITIONAL PASS

**Input**: Score 78, Ceiling 89. No Critical findings. One High finding: missing error handling for SMTP failures. Medium confidence. Repository analysis was adequate.

**Decision**: CONDITIONAL PASS.

**Rationale**: The score and ceiling are in the CONDITIONAL PASS range. The missing error handling is High but has clear remediation guidance. The implementer can proceed with non-email portions while the error handling specification is added. Re-review is not required if the remediation is followed.

### Example 4: FAIL — Low Confidence

**Input**: Score 75, Ceiling 89. No Critical findings. Two Medium findings. Low confidence because the reviewer was unable to fully explore the notification subsystem due to access constraints.

**Decision**: FAIL (downgraded from CONDITIONAL PASS due to Low confidence).

**Rationale**: While the score and ceiling support CONDITIONAL PASS, Low confidence in the notification subsystem means the reviewer cannot guarantee that no blocking issues exist in that area. The Work Package should be re-reviewed after notification subsystem access is obtained.

### Example 5: Escalation

**Input**: Score 85, Ceiling 100. No Critical findings. One High finding: the Work Package proposes a new event-driven architecture pattern that does not exist in the repository. Medium confidence.

**Decision**: Cannot decide. Escalate to architecture review board.

**Rationale**: The Work Package introduces a new architectural pattern not present in the repository. The framework permits this but requires escalation because architectural decisions have cross-cutting impact that a single reviewer should not authorize. The architecture review board will evaluate the pattern fit and either approve with migration guidance or reject.

### Example 6: Exception Override

**Input**: Score 88, Ceiling 100. No Critical findings. The Work Package introduces a new HTTP API dependency on an unsecured external service without specifying authentication.

**Decision**: FAIL. Security exception override activated.

**Rationale**: The numerical inputs would support CONDITIONAL PASS or even PASS. However, the security vulnerability (unauthenticated external API access) triggers the exception handling override. Immediate FAIL. Escalate to security team.
