---
name: evidence-and-justification
description: Canonical engineering standard for evidence and justification in Work Package Reviews. Defines what constitutes engineering evidence, evidence quality and hierarchy, evidence collection and validation, traceability, engineering justification, the justification chain, engineering reasoning, claim validation, and burden of proof. Does not define repository analysis, implementation readiness, review workflow, scoring, decision making, or report formatting.
---

# Evidence and Justification

## Purpose

This document defines how engineering conclusions are justified within the review framework. It is the single source of truth for all evidence and justification concepts.

Every engineering conclusion must be supported. Every recommendation must be explainable. Every finding must be traceable. This document teaches reviewers how to build engineering arguments rather than simply collect evidence.

Evidence supports reasoning. Reasoning supports conclusions. Conclusions support decisions.

Repository Analysis (`references/repository-analysis.md`) discovers evidence. This document determines how that evidence supports engineering conclusions. The Decision Matrix (`references/decision-matrix.md`) consumes justified conclusions to make readiness determinations.

## Evidence Principles

The following principles from `references/review-principles.md` govern every evidence and justification decision:

- **Evidence Based**: Every finding must cite observable evidence from the repository or the Work Package. Subjective statements, impressions, and unsupported assertions have no place in a review.
- **Verifiability**: Every finding, recommendation, and conclusion must be independently verifiable by reading the repository or the Work Package.
- **Challenge Assumptions**: Treat every statement in the Work Package as a hypothesis that must be validated against the repository.

This document defines the methodology for applying these principles.

---

## Part 1: Engineering Evidence

### 1.1 What Constitutes Engineering Evidence

Engineering evidence is observable, verifiable information from the repository or the reviewed artifact that supports or contradicts an engineering claim. Evidence must be:

- **Observable**: The reviewer has read the source and can cite specific content.
- **Verifiable**: An independent reviewer can confirm the evidence by reading the same source at the same location.
- **Relevant**: The evidence directly relates to the engineering claim it supports or contradicts.
- **Specific**: The evidence includes exact file paths, line numbers, and relevant excerpts.

Evidence is not:

- Personal experience or prior knowledge of similar projects.
- Assumptions about what the code probably does without reading it.
- Generalizations from other codebases or technologies.
- The reviewed artifact's own statements about itself (the Work Package cannot validate itself).

### 1.2 Evidence Quality

Evidence quality is determined by three dimensions:

| Dimension | High Quality | Low Quality |
|---|---|---|
| Specificity | Exact file path, line numbers, and relevant excerpt | Vague reference to "the codebase" or "the service layer" |
| Verifiability | An independent reviewer can confirm by reading the same location | Requires domain knowledge the reviewer may not have |
| Directness | Directly supports or contradicts the claim | Requires additional inference to connect to the claim |

Every evidence citation must satisfy all three dimensions at an acceptable level. If any dimension is low, the evidence is weak and should be noted as such in the finding.

### 1.3 Evidence Hierarchy

Evidence is classified by strength. Prefer stronger evidence types. When only weak evidence is available, note the evidence limitation in the finding.

| Type | Strength | Description |
|---|---|---|
| Code | Strongest | The repository contains code that directly supports or contradicts the finding. Example: an existing function at `src/utils/format.ts:45` validates email addresses; the Work Package proposes creating a new validation function. |
| Documentation | Strong | Project documentation explicitly states a rule, convention, or requirement. Example: `CONTRIBUTING.md:22` states "all API changes require OpenAPI spec updates." |
| Configuration | Strong | Configuration files confirm or deny a dependency or setting. Example: `package.json:15` shows no dependency on lodash. |
| Convention | Medium | A pattern observed across multiple independent locations in the repository. Requires verification across at least three locations. Example: all repository services follow the repository pattern. |
| Absence | Medium | The absence of expected code, tests, or documentation. Example: no test files exist for the module the Work Package modifies. |
| Implication | Weak | The reviewed artifact's acceptance criteria or requirements imply a contradiction with existing code. Example: the Work Package's acceptance criteria imply a data model that contradicts `src/models/user.ts:30`. |

Evidence strength determines how much weight a finding carries. A finding based on code evidence is more reliable than one based on implication. When using medium or weak evidence, acknowledge the limitation.

### 1.4 Evidence Collection

Evidence collection is the systematic process of gathering observable information from the repository and the reviewed artifact. It is guided by the repository analysis methodology in `references/repository-analysis.md` and occurs during the following review lifecycle phases:

1. **Repository Intelligence (Phase 2)**: Evidence is collected about project topography, architecture, conventions, reuse inventory, and dependencies.
2. **Implementation Simulation (Phase 3)**: Evidence is collected about information gaps, ambiguous requirements, and missing specifications.
3. **Engineering Validation (Phase 5)**: Evidence is organized, validated, and linked to findings via the Engineering Justification Chain.

#### Collection Standards

- Read the actual code. Search result snippets are insufficient for confirming behavioral equivalence.
- Verify every claim across multiple independent locations. A pattern observed in one file may be an individual choice; a pattern observed across three locations is likely a convention.
- When a finding references repository code, cite the exact file paths and line numbers.
- When citing conventions, provide at least three example locations.
- Search for disconfirming evidence as rigorously as confirming evidence. The goal is to find what the Work Package got wrong, not to confirm what it got right.

#### Common Collection Mistakes

- **Citing without reading**: Citing a file without reading its contents. File names alone are insufficient evidence.
- **Convention from one file**: Citing a convention observed in a single file as project-wide. Verify across at least three independent locations.
- **Over-reliance on naming**: Inferring behavior from file names, directory names, or function names without reading the implementation. Names describe intent, not behavior.
- **Confirmation bias in collection**: Searching only for evidence that confirms the Work Package's correctness rather than actively searching for contradictions.

### 1.5 Evidence Validation

Every piece of evidence must be validated before it can support a finding. Validation answers:

1. **Is the evidence accurate?** Does the code actually say what the reviewer claims?
2. **Is the evidence current?** Does the evidence reflect the current state of the repository, or is it from deprecated or migrated code?
3. **Is the evidence representative?** Does a single occurrence represent a project-wide pattern, or is it an isolated case?
4. **Is the evidence sufficient?** Does the evidence alone support the conclusion, or does additional evidence need to be gathered?

Validation happens during investigation (step 2 of the Finding Lifecycle). Evidence that cannot be validated is insufficient to support a finding.

### 1.6 Evidence Traceability

Every piece of evidence must be traceable. Traceability means an independent reviewer can:

1. Follow the evidence trail from the finding back to the source.
2. Verify the evidence by reading the same file at the same location.
3. Understand how the evidence supports the conclusion without additional explanation from the reviewer.

#### Traceability Requirements

- Every evidence citation must include the exact file path (absolute or relative to repository root).
- Every evidence citation must include line numbers or line ranges.
- Every evidence citation must include a relevant excerpt — enough context to verify without cross-referencing multiple files.
- Every finding must explain the relationship between the evidence and the conclusion. "This code exists" is not sufficient; "this code at `src/utils/validators.ts:45` does what the Work Package proposes, so the Work Package should reference it rather than create a new function" connects evidence to conclusion.

### 1.7 Evidence Confidence and Uncertainty

Not all evidence carries the same certainty. Evidence confidence expresses how reliable the reviewer considers the evidence to be.

| Factor | Reduces Confidence |
|---|---|
| No test coverage for the cited code | The code's behavior may not match expectations |
| No callers for the cited code | The code may be deprecated or incomplete |
| The behavioral match was inferred from naming, not from reading code | The name may not reflect actual behavior |
| The cited code has recent churn or bug fixes | The implementation may be unstable |
| The cited code uses deprecated APIs | The code may be removed or require migration |
| The evidence is from documentation or conventions rather than code | Documentation may be outdated; conventions may have exceptions |

#### Expressing Uncertainty in Findings

When evidence confidence is low, the finding must acknowledge the uncertainty:

- **High confidence**: "The existing function at `src/utils/validators.ts:45` has the same behavior, has tests, and has callers. The Work Package should reference it."
- **Medium confidence**: "The function at `src/utils/validators.ts:45` appears to match based on naming. No tests exist. The implementer should verify behavioral equivalence before referencing it."
- **Low confidence**: "The function at `src/utils/validators.ts:45` may be related based on naming. Further investigation is needed during implementation."

Medium and low confidence findings are still findings — they signal to the implementer that additional verification is needed.

### 1.8 Supporting, Contradictory, and Missing Evidence

Findings are built from three types of evidence:

| Type | Role |
|---|---|
| Supporting evidence | Evidence that confirms a claim. Example: the repository contains the types the Work Package references. |
| Contradictory evidence | Evidence that refutes a claim. Example: the repository uses a different error handling pattern than what the Work Package proposes. |
| Missing evidence | Expected evidence that does not exist. Example: the Work Package describes a schema change but no migration file exists. |

A finding may involve all three types. For example, a claim that "the existing email service should be extended" has:
- **Supporting evidence**: The email service exists and has relevant behavior.
- **Contradictory evidence**: The email service is tightly coupled to a specific transport that the new requirement does not support.
- **Missing evidence**: No existing abstraction layer separates transport logic from business logic, making extension costly.

When contradictory evidence exists, the finding must reconcile it. When evidence is missing, the finding must state what was expected but not found.

### 1.9 Burden of Proof

The burden of proof in a review rests on the party making the claim:

- **The Work Package** must demonstrate that its proposals are consistent with the repository.
- **The reviewer** must demonstrate that findings are supported by evidence.

A finding without evidence fails its burden of proof. An implementer should not act on a finding that cannot be verified.

The standard of proof is preponderance of evidence: a finding is valid when the evidence supporting it outweighs the evidence contradicting it. Certainty is not required — engineering decisions involve trade-offs — but the evidence must clearly support the conclusion.

---

## Part 2: Engineering Justification

### 2.1 Engineering Justification Chain

Every finding must follow the Engineering Justification Chain. This is mandatory. Findings that skip links are incomplete.

```
Finding ──→ What is the specific problem?
    │
    ▼
Evidence ──→ Where in the repository or reviewed artifact is this visible?
    │
    ▼
Engineering Principle ──→ Which engineering principle does this violate?
    │
    ▼
Impact ──→ What will happen if this is not addressed?
    │
    ▼
Recommendation ──→ What exactly needs to change?
```

#### Link Definitions

| Link | Required | Content |
|---|---|---|
| Finding | Yes | A precise, one-sentence statement of the problem. "The Work Package omits error handling for SMTP connection failures." |
| Evidence | Yes | Verifiable citation from the repository or artifact. Must include file path, line number, and relevant excerpt. "`src/services/email.ts:45-60` shows all existing services implement error handling; the Work Package has no equivalent section." |
| Engineering Principle | Yes | A specific principle from `references/review-principles.md`. Generic references to "good practice" are not acceptable. |
| Impact | Yes | A concrete consequence of not addressing the finding. Must describe what will break, degrade, or increase cost. "Without SMTP error handling, a temporary mail server outage will cause silent order failures with no notification to the user or operator." |
| Recommendation | Yes | An actionable instruction. Must be specific enough for an implementer to follow without additional clarification. "Add an error handling section to the email service specification covering: connection timeout retry (3 attempts, exponential backoff), authentication failure logging, and a dead-letter queue for undeliverable messages." |

The chain ensures that no finding exists in isolation. Every link must be present and must connect logically to the next. A missing link breaks the chain and invalidates the finding.

#### Chain Validation

Before finalizing a finding, verify each link:

1. **Finding → Evidence**: Does the evidence clearly demonstrate the problem exists?
2. **Evidence → Principle**: Does the evidence violate a specific engineering principle?
3. **Principle → Impact**: Does violating this principle lead to a concrete negative outcome?
4. **Impact → Recommendation**: Does the recommendation directly address the impact?

If any connection is weak or missing, the finding is incomplete.

### 2.2 Engineering Reasoning

Engineering reasoning is the logical connection between evidence and conclusions. The chain is the structure; reasoning is the content that fills it.

#### Reasoning Standards

- **Explicit connections**: Do not assume the reader will infer how evidence supports the conclusion. State it explicitly.
- **Causal links**: When evidence reveals a problem, trace the causal path from problem to impact. "The missing error handling causes silent failures because the SMTP library throws exceptions that are caught by a generic handler and logged without context."
- **Principle-based justification**: All reasoning must trace back to engineering principles. "This is wrong because" must be followed by a specific principle from `references/review-principles.md`, not by personal preference.
- **Trade-off recognition**: Engineering decisions involve trade-offs. Acknowledge when the recommendation has costs. "Extending the existing service avoids duplication but increases coupling. The coupling is acceptable because both features belong to the same domain."

#### Reasoning Anti-Patterns

| Anti-Pattern | Example | Resolution |
|---|---|---|
| Circular reasoning | "The Work Package is ready because it has no issues." The Work Package cannot validate itself. | Evidence must come from the repository or other independent sources. |
| Assertion without connection | "This is wrong." Without explaining why or citing evidence. | State the evidence, the principle, and the logical connection. |
| False cause | "The Work Package does not specify error handling, therefore it will cause production incidents." Without tracing the path from omission to incident. | Trace the causal path: missing specification → implementer guesses → wrong guess → incorrect behavior in production. |
| Appeal to authority | "The senior architect approved this pattern." Without evidence that the pattern fits the repository. | Evaluate the pattern against the repository, not against the author's reputation. |

### 2.3 Engineering Claims

An engineering claim is a statement in the Work Package or a finding that asserts a fact about the repository, the implementation, or the relationship between them.

#### Claim Types

| Type | Example | Validation Required |
|---|---|---|
| Claim of fact | "The repository has a customer service module at `src/services/customer.ts`." | Verify the file exists and contains the described functionality. |
| Claim of relationship | "Extending the existing service is cheaper than creating a new one." | Estimate extension cost vs. creation cost based on code structure. |
| Claim of impact | "This omission will cause production incidents." | Trace the causal path from omission to incident. |
| Claim of principle | "This violates Separation of Responsibilities." | Demonstrate that the proposed change blurs a defined boundary. |

#### Claim Validation

Every engineering claim must be validated before it can support a decision. Validation follows these steps:

1. **Identify the claim**: What exactly is being asserted?
2. **Determine the claim type**: What kind of evidence is needed to support it?
3. **Gather evidence**: Collect supporting, contradictory, and missing evidence.
4. **Evaluate evidence sufficiency**: Does the evidence clearly support the claim?
5. **Document the conclusion**: State whether the claim is supported, refuted, or uncertain.

A claim that cannot be validated with available evidence must be noted as unvalidated. Unvalidated claims cannot support findings.

### 2.4 Causal Reasoning

Causal reasoning traces the chain from root cause to observable impact. It is essential for severity determination — impact is the primary factor in severity classification.

#### Causal Chain Structure

```
Root Cause ──→ Direct Effect ──→ Observable Impact ──→ Engineering Cost
```

**Example**:
- **Root Cause**: The Work Package does not specify error handling for SMTP failures.
- **Direct Effect**: The implementer does not know what happens when SMTP connection fails.
- **Observable Impact**: The implementer uses a generic try-catch that logs "error" without context.
- **Engineering Cost**: Production incidents where email failures are not observable, causing silent order failures.

#### Causal Reasoning Standards

- Every finding must trace at least one causal path from evidence to impact.
- When multiple causal paths exist, document the most likely one and acknowledge alternative paths.
- When the causal path involves assumptions (e.g., "the implementer will guess X"), state the assumption explicitly.
- A finding whose causal path depends on unlikely assumptions should have reduced severity or confidence.

### 2.5 Root Cause Justification

Root cause justification identifies the underlying source of a finding rather than treating symptoms.

#### Finding Root Causes

When multiple symptoms trace to the same root cause, they should be a single finding at the severity of the worst symptom.

**Example**:
- **Symptoms**: Missing SMTP error handling, missing database error handling, missing API error handling.
- **Root Cause**: The Work Package has no error handling specification section.
- **Finding**: One finding about missing error handling specification, not three findings about each missing error handler.

#### Root Cause vs. Symptom Severity

A finding's severity is determined by the worst symptom, not by the root cause itself. A missing specification (root cause) may be Medium, but if it causes a Critical impact (data loss when error path is unspecified), the finding is Critical.

#### Documenting Root Cause

Every finding should trace to its root cause. The finding text must distinguish between the root cause and the symptoms. "The missing error handling specification (root cause) means the implementer will not handle SMTP failures, database timeouts, or API errors (symptoms)."

---

## Part 3: Evidence in the Review Lifecycle

### Phase 2: Repository Intelligence

Repository analysis (`references/repository-analysis.md`) discovers evidence. The analysis produces architecture summaries, convention catalogs, reuse inventories, and dependency maps — all of which are evidence sources for subsequent phases.

Evidence collection during this phase follows the standards in Part 1 of this document.

### Phase 3: Implementation Simulation

Implementation simulation (`references/implementation-readiness.md`) produces evidence about information gaps. Each simulation gap is a piece of evidence that the Work Package is incomplete.

### Phase 5: Engineering Validation

Engineering validation transforms raw evidence into justified findings. Each finding follows the Engineering Justification Chain. Evidence from Phases 2 and 3 is validated, organized, and linked to principles, impacts, and recommendations.

### Phase 7: Review Validation

The review validation phase verifies evidence quality per this document. Every finding is checked for traceability, specificity, and sufficiency.

---

## Part 4: Evidence Anti-Patterns

### Circular Evidence

Using the reviewed artifact as evidence for its own correctness. The Work Package cannot validate itself.

**Example**: "The Work Package states that the data model is correct" as evidence that the data model is correct.

**Resolution**: Evidence must come from the repository, project documentation, or other independent sources.

### Evidence from Assumption

Stating a conclusion based on what the reviewer assumes rather than what they have read.

**Example**: "The service probably handles this case because it looks complete."

**Resolution**: Read the actual code. If the code has not been read, the evidence is insufficient.

### Citation Without Reading

Citing a file path without reading the file's contents.

**Example**: Citing `src/services/email.ts` without having read the file to verify its behavior.

**Resolution**: Read every file before citing it. File names alone are insufficient.

### Single-Instance Convention

Citing a convention observed in one file as project-wide.

**Resolution**: Verify conventions across at least three independent locations before citing them as evidence.

### The Certainty Trap

Stating a finding with absolute certainty about a complex judgment that involves trade-offs. Engineering decisions are rarely certain.

**Example**: "This approach will definitely cause performance problems."

**Resolution**: Acknowledge uncertainty: "This approach may cause performance problems under high load because..." and provide evidence for the risk.

### Missing Burden of Proof

Asserting a finding without sufficient evidence to support it.

**Resolution**: Before finalizing a finding, verify that the evidence clearly supports the conclusion. If the evidence is insufficient, gather more or downgrade the finding.

### Severity Based on Confidence

Assigning severity based on how certain the reviewer is rather than on the impact of the issue.

**Resolution**: Severity is determined by impact, not by confidence. A likely minor issue with high confidence is still minor. An unlikely Critical issue with low confidence may still warrant attention.

---

## Part 5: Examples

### Strong Evidence and Justification

**Finding**: The Work Package proposes creating a new `validateEmail` function.

**Evidence**: `src/utils/validators.ts:45-60` exports `isValidEmail(email: string): boolean` with the same behavior, used by three modules, with unit tests at `src/utils/validators.test.ts:30-55`. The function handles format validation, disposable domain detection, and DNS lookup — matching the Work Package's requirements.

**Causal Reasoning**: The Work Package creates unnecessary duplication. The implementer would write a new function that duplicates existing behavior, increasing maintenance burden. When the email validation logic needs to change, both implementations must be updated — a missed update causes a bug.

**Conclusion**: Clear missed reuse opportunity. High confidence.

### Medium Evidence and Justification

**Finding**: The Work Package proposes a new configuration format.

**Evidence**: `config/default.yaml:10-40` shows the project uses YAML for configuration. The Work Package proposes TOML. No project documentation specifies a required format — this is inferred from the existing configuration files (convention evidence, two locations found).

**Causal Reasoning**: Using a different configuration format creates inconsistency. The implementer may not notice the format difference and may assume TOML is an accepted alternative. The inconsistency adds cognitive load for future readers.

**Conclusion**: Convention violation. Medium confidence (only two configuration files verified).

### Weak Evidence and Justification

**Finding**: The Work Package's acceptance criteria may be incomplete.

**Evidence**: The acceptance criteria describe the happy path but do not mention error scenarios. No existing acceptance criteria in the repository are available for comparison (the project has no prior acceptance criteria to use as templates).

**Causal Reasoning**: Without acceptance criteria for error scenarios, the implementer may not implement error handling. However, the project's convention is that error handling follows existing patterns regardless of acceptance criteria — so the implementer may still handle errors correctly.

**Conclusion**: Potential issue but insufficient evidence to determine impact. Note as Medium finding with recommendation to add error scenario acceptance criteria, but acknowledge that the impact is uncertain.
