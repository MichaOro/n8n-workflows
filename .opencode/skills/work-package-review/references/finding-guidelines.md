---
name: finding-guidelines
description: Canonical engineering standard for creating review findings. Defines finding structure, quality criteria, actionability standards, common mistakes, finding-specific anti-patterns, and classification. Consistency methodology is defined in references/review-consistency.md. Evidence and justification methodology is owned by references/evidence-and-justification.md.
---

# Finding Guidelines

## Purpose

This document defines how engineering findings are created within the review framework. It is the single source of truth for finding construction, quality, and consistency.

A finding is a formal statement that a specific part of the reviewed artifact (Work Package, Epic, ADR, design document) contains a problem that must be addressed. Findings transform raw observations into actionable engineering conclusions.

This document defines **how findings are created**. It does not define engineering principles, methodology, scoring, decision rules, or report layout.

## What Constitutes a Finding

A finding exists when the reviewer can answer all five questions of the Engineering Justification Chain, defined in `references/evidence-and-justification.md`:

1. **What is the specific problem?** A precise statement of what is wrong or missing.
2. **Where is the evidence?** A verifiable citation from the repository or reviewed artifact.
3. **Which engineering principle is violated?** A specific principle from `references/review-principles.md` that this finding contravenes.
4. **What is the impact?** A concrete consequence of not addressing this finding.
5. **What should change?** An actionable recommendation that an implementer can follow.

If any of the five questions cannot be answered, the observation is not yet a finding. It is a note, a question, or an incomplete observation that requires further investigation.

### What Findings Are Not

- **Not personal opinions**: "I don't like this approach" is not a finding. "This approach creates a third data-access pattern when the project standardizes on two" is a finding.
- **Not style preferences**: "Use tabs instead of spaces" is not a finding unless the project has an explicit formatting convention.
- **Not pre-existing issues**: "The repository has poor test coverage" is not a finding unless the Work Package directly affects testing.
- **Not speculations**: "This might cause performance issues" is not a finding unless the reviewer can cite evidence of the performance constraint.
- **Not compliments**: "Good use of existing patterns" is a positive observation, not a finding. Positive observations are encouraged but are not findings.

## Finding Lifecycle

```
Observation ──→ Investigation ──→ Finding ──→ Classification ──→ Presentation ──→ Resolution
```

1. **Observation**: During simulation, analysis, or dimension evaluation, the reviewer identifies a potential problem.
2. **Investigation**: The reviewer gathers evidence, traces dependencies, and verifies the observation against the repository.
3. **Finding**: The observation is formalized into a finding following the Engineering Justification Chain.
4. **Classification**: The finding is assigned a severity (Critical, High, Medium, Low) per `references/implementation-readiness.md` and mapped to a dimension.
5. **Presentation**: The finding is formatted per `references/review-output-contract.md` and included in the review output.
6. **Resolution**: The implementer addresses the finding during implementation or Work Package revision.

A finding may be discarded during Investigation if evidence does not support the observation. Discarded observations should be noted in the reviewer's working notes but not included in the output.

## Finding Structure

Every finding must follow the Engineering Justification Chain defined in `references/evidence-and-justification.md`. This is mandatory. Findings that skip links are incomplete. The chain has five links:

```
Finding ──→ Evidence ──→ Engineering Principle ──→ Impact ──→ Recommendation
```

Evidence requirements, quality standards, hierarchy, and validation are defined in `references/evidence-and-justification.md`. The engineering principle must reference a specific principle from `references/review-principles.md`.

### Finding Identifier

Findings are assigned identifiers during presentation (per `references/review-output-contract.md`). Identifiers follow the format `F-NNN` (F-001, F-002, ...) ordered by severity.

## Evidence in Findings

Evidence methodology — including requirements, hierarchy, strength, validation, traceability, and common errors — is defined in `references/evidence-and-justification.md`. Every finding follows the evidence standards defined there.

## Finding Quality

Every finding must satisfy all of the following quality criteria.

### Be Specific

Every finding must reference exact file paths, line numbers, and code excerpts. Generic statements are insufficient.

**Poor**: "The data model is incomplete."
**Excellent**: "The data model at `src/models/order.ts:30` must add a `shippingAddress` field of type `Address` to satisfy acceptance criterion 3, which requires shipping address capture."

### Be Complete

Deliver all findings in a single pass. Do not drip findings across multiple updates. The implementer needs the complete picture to decide whether to proceed.

**Poor**: Sending one finding, then another finding in a follow-up message, then a third.
**Excellent**: All findings delivered together in a single review output.

### Be Actionable

For every finding that blocks implementation, describe exactly what needs to change.

**Poor**: "The error handling is insufficient."
**Excellent**: "Add error handling for SMTP connection failures: implement 3 retries with exponential backoff, log authentication failures with the SMTP response code, and route undeliverable messages to the existing dead-letter queue at `src/queues/dead-letter.ts`."

### Be Proportionate

Allocate finding depth proportionally to the finding's impact.

**Poor**: A five-paragraph analysis of a one-character typo in documentation.
**Excellent**: A one-line note about a typo, and a full analysis of a missing API contract.

### Be Objective

Every finding must be framed in engineering terms, not personal or subjective terms.

**Poor**: "The developer forgot to include the migration script."
**Excellent**: "The Work Package describes a schema change but does not include a corresponding migration script. The project convention at `CONTRIBUTING.md:15` requires every schema change to include a migration."

### Be Traceable

Every finding must be traceable back to specific evidence in the repository or reviewed artifact. Traceability requirements are defined in `references/evidence-and-justification.md`. An implementer must be able to verify the finding by following the evidence trail.

## Finding Classification

### Severity

Every finding is classified into one of four severity levels per `references/implementation-readiness.md`:

| Severity | Effect on Readiness |
|---|---|
| Critical | Score set to 0. Work Package must be revised and re-reviewed. |
| High | Significant score deduction. Work Package should be revised before implementation. |
| Medium | Minor score deduction. Should be addressed but does not block. |
| Low | Minimal or no score impact. Suggestion for improvement. |

### Dimension

Every finding must be mapped to one of the evaluation dimensions defined in `references/review-process.md` (e.g., Specification Completeness, Data Contract Completeness, Integration Surface Completeness, Error and Observability Completeness, Architectural Fit, Reuse Efficiency).

Mapping to a dimension ensures that finding distribution across dimensions can be analyzed. A finding that maps to multiple dimensions should be assigned to the primary dimension affected; note the secondary dimension in the finding text if relevant.

## Finding Prioritization

Findings are prioritized by severity within the review output. Within the same severity, findings should be ordered by:

1. **Impact scope**: Findings that affect more consumers or systems come first.
2. **Reversibility**: Findings that create irreversible damage come before reversible ones.
3. **Implementation order**: Findings that must be resolved before other work can proceed come first.

The reviewer may reorder findings within the same severity to reflect implementation dependencies. For example, a High finding about the data model should appear before a High finding about a feature that depends on that data model.

## Finding Granularity

### When to Merge Findings

Merge findings when they share:

- **Same root cause**: Multiple symptoms of the same underlying issue should be a single finding at the severity of the worst symptom.
  - *Example*: Three missing validation rules all stem from the same missing validation specification → one finding.
- **Same remediation**: Multiple issues that are resolved by a single change should be a single finding.
  - *Example*: Five unused imports are all resolved by removing a single deprecated module → one finding.

### When to Split Findings

Split findings when they require:

- **Different remediation**: Two issues that require different changes must be separate findings, even in the same dimension.
  - *Example*: Missing error handling for SMTP and missing error handling for database timeouts are different issues requiring different code.
- **Different decision impact**: Issues that affect the decision differently must be separate.
  - *Example*: A Critical security vulnerability and a Medium naming inconsistency cannot be the same finding.
- **Different ownership**: Issues that would be resolved by different teams or individuals should be separate findings.

### Avoiding Duplicate Findings

Before creating a finding, verify it is not already covered by another finding:

1. Search existing findings for the same root cause.
2. If the same root cause exists, extend that finding rather than creating a new one.
3. If the evidence overlaps by more than 50%, the findings are likely duplicative.
4. If the remediation overlaps, the findings should likely be merged.

A finding is duplicate not when it describes the same issue verbatim, but when addressing one finding would automatically resolve the other.

Finding consistency methodology — including intra-review consistency, inter-review consistency, and language consistency — is defined in `references/review-consistency.md`. All finding consistency concepts are owned by that document.

### Duplicate Findings Across Dimensions

A finding identified in one dimension may also be relevant to another dimension. When this occurs, reference the original finding rather than duplicating the analysis. Each finding should appear once in the finding set, with its dimension classification reflecting the primary dimension. Cross-dimension impacts are noted in the finding's impact description.

## Common Finding Mistakes

### Missing Evidence

The finding states a problem but provides no way to verify it. The implementer cannot confirm the issue exists.

**Mitigation**: Before finalizing a finding, verify that the evidence satisfies the standards in `references/evidence-and-justification.md`.

### Missing Principle

The finding states a problem and cites evidence but does not explain why it matters in engineering terms. The implementer sees what is wrong but not why it is wrong.

**Mitigation**: Reference a specific principle from `references/review-principles.md`.

### Missing Impact

The finding states a problem but does not describe the consequence. The implementer cannot prioritize without understanding severity.

**Mitigation**: Describe what will happen if the finding is not addressed. Be concrete.

### Missing Recommendation

The finding identifies a problem but does not suggest a fix. The implementer must determine the solution independently, which may deviate from the reviewer's intent.

**Mitigation**: Always include a recommendation. If multiple valid solutions exist, recommend the one that best aligns with project conventions.

### Scope Creep in Findings

Adding findings about code quality issues that are unrelated to the reviewed artifact. The review evaluates the artifact, not the overall health of the repository.

**Mitigation**: Pre-existing issues should only be mentioned if the reviewed artifact directly affects them.

### False Precision

Assigning severity without considering actual impact. Severity must be proportional to implementation and maintenance consequences, not to the reviewer's certainty.

**Mitigation**: Evaluate severity based on impact, not confidence. When uncertain, choose the lower severity.

### Over-Granularity

Splitting a single root cause into multiple findings, creating the impression of more issues than actually exist. This dilutes the impact of genuine issues and makes the review harder to act on.

**Mitigation**: Before splitting, verify that each proposed finding has a distinct root cause, remediation, and decision impact.

### Under-Granularity

Merging distinct issues into a single finding, hiding the scope of required work. The implementer may address one part of the finding and miss the rest.

**Mitigation**: Before merging, verify that all merged issues share root cause, remediation, and ownership.

## Finding Anti-Patterns

### The Kitchen Sink Finding

A single finding that lists every problem in a module. It is impossible to act on because it describes multiple unrelated issues.

**Example**: "The user service has incorrect error handling, missing validation, wrong return types, no tests, and uses the wrong logger."
**Resolution**: Split into separate findings per root cause.

### The Opinion Disguised as Finding

A finding that reflects personal preference rather than engineering principle.

**Example**: "I prefer React hooks over class components."
**Resolution**: Verify that the finding references a specific principle from `references/review-principles.md`. If it cannot, it is not a finding.

### The Unsolvable Finding

A finding that identifies a problem without any viable remediation, leaving the implementer with no path forward.

**Example**: "The entire architecture needs to be redesigned."
**Resolution**: Either provide a specific, incremental recommendation or downgrade to a note about technical debt. A finding without a realistic recommendation is not actionable.

### The Certainty Trap

A finding stated with absolute certainty about a complex judgment that involves trade-offs. See the evidence confidence framework in `references/evidence-and-justification.md` for how to express uncertainty in findings.

**Example**: "This approach will definitely cause performance problems."
**Resolution**: Acknowledge uncertainty: "This approach may cause performance problems under high load because..." and provide evidence for the risk.

### The Severity Inflation

Assigning Critical or High severity to findings that are minor inconsistencies. Over-classification erodes the credibility of the severity model and makes it harder for implementers to prioritize.

**Resolution**: Evaluate severity objectively per `references/implementation-readiness.md`. When uncertain, choose the lower severity.

### The Inconsistent Standards Finding

A finding that flags a pattern in one location while accepting the same pattern elsewhere. This undermines the credibility of the review and confuses the implementer about what the project actually requires.

**Example**: Flagging direct database access in a controller without noting that three existing controllers use the same pattern.
**Resolution**: Before writing a finding, search for the same pattern in the repository. If the pattern exists and is accepted elsewhere, either the finding is invalid or the project convention needs to change — document which.

## Examples

### Excellent Finding

```
#### F-001 [High] Missing Error Handling for SMTP Connection Failures
- **Dimension**: Error and Observability Completeness
- **Evidence**: `src/services/email.ts:45-60` shows all existing email services implement error handling with retry logic and logging. The Work Package section "Email Notification Service" contains no error handling specification.
- **Engineering Principle**: Explicit over Implicit — requirements affecting correctness must be specified explicitly, not left to the implementer to infer.
- **Problem**: Without error handling, an SMTP connection failure during a critical order notification will fail silently. The implementer must guess the error handling strategy, which may not match the project's established patterns.
- **Impact**: Production incidents where order notifications are lost without any alert to operators or users. The existing pattern at `src/services/email.ts:45-60` shows retry with exponential backoff; the implementer might choose a simpler approach that is incompatible with the project's reliability requirements.
- **Recommendation**: Add an error handling specification to the Email Notification Service covering: (1) connection timeout retry with exponential backoff (matching `src/services/email.ts`), (2) authentication failure logging including SMTP response codes, (3) undeliverable message routing to the dead-letter queue at `src/queues/dead-letter.ts`.
```

**Why this finding is excellent**:
- Specific evidence with file paths and line numbers
- References a concrete engineering principle
- Explains the problem in engineering terms
- Describes concrete consequences
- Provides an actionable, specific recommendation
- Traces the recommendation to existing project patterns

### Poor Finding

```
#### Issue with Error Handling
- **Problem**: The error handling is not good enough.
- **Evidence**: I looked at the code and it seems incomplete.
- **Impact**: This could cause problems.
- **Recommendation**: Fix the error handling.
```

**Why this finding is poor**:
- No specific evidence (no file paths, no excerpts)
- No engineering principle referenced
- Problem statement is subjective ("not good enough")
- Impact is vague ("could cause problems")
- Recommendation is not actionable ("Fix the error handling")
- Missing severity and dimension
