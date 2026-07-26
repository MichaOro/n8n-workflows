---
name: implementation-readiness
description: Canonical methodology for determining whether a Work Package is ready for implementation. Defines implementation simulation, gap analysis, and severity model. Scoring methodology is defined in references/review-scoring.md. Decision methodology is defined in references/decision-matrix.md. Applies the principles defined in references/review-principles.md.
---

# Implementation Readiness

## Purpose

This document defines how implementation readiness is determined. It covers the evaluation pipeline from simulation through scoring: running implementation simulations, analyzing gaps, classifying severity, and calculating scores. Decision methodology is defined in `references/decision-matrix.md`.

Implementation readiness answers one question: can a competent engineer build the Work Package correctly using only the Work Package, the repository, and project documentation — without requiring additional clarification?

This document is the single source of truth for all readiness-related methodology within the Work Package Review framework.

## Readiness Criteria

A Work Package is implementation-ready when all of the following conditions are met:

1. **The intent is simulable.** A mental dry-run of the implementation completes without stopping due to missing, ambiguous, or contradictory information.

2. **The Work Package is internally consistent.** No requirement, constraint, or assumption contradicts another within the Work Package itself.

3. **The Work Package is externally consistent.** Every file path, type name, module reference, and architectural assumption has been validated against the repository.

4. **Acceptance criteria are testable.** Every criterion can be verified through automated tests, manual testing, or observable behavior. No criterion requires subjective judgment.

5. **Error paths are specified.** What happens when each operation fails is documented or can be inferred from existing patterns with certainty.

6. **Integration surfaces are enumerated.** Every touch point between new and existing code is identified. No dependency is discovered during simulation that was not in the Work Package.

7. **Risk is acceptable.** The identified risks (integration, data, security, performance, regression, migration) are within acceptable bounds given the change's value.

If any condition is not met, the Work Package is not ready in its current form.

## Principles Applied

This methodology implements the following principles from `references/review-principles.md`:

- **Explicit over Implicit**: Every readiness condition must be verifiable from the Work Package and repository. Requirements, contracts, and assumptions must be explicit.
- **Evidence Based**: Every finding during simulation and gap analysis is backed by specific evidence — file paths, code excerpts, documented conventions.
- **Deterministic Reviews**: The severity model, scoring (per `references/review-scoring.md`), and decision methodology (per `references/decision-matrix.md`) are designed so that different reviewers applying them to the same inputs reach the same conclusion.
- **Engineering over Opinion**: Severity is determined by impact on correctness and maintainability, not by the reviewer's confidence or preference.

## When Readiness Is Determined

Readiness is evaluated during the review lifecycle after repository intelligence and engineering validation are complete. The readiness determination occurs in two phases:

1. **Discovery** (Implementation Simulation + Gap Analysis): Run the implementation dry-run and analyze the four-layer gap model. These phases produce raw gaps and discrepancies.

2. **Judgment** (Severity + Scoring + Decision): Classify the severity of each gap, calculate the readiness score per `references/review-scoring.md`, and apply the decision methodology per `references/decision-matrix.md` to produce a PASS / FAIL / CONDITIONAL PASS determination.

Readiness is never evaluated before repository analysis is complete. Repository analysis establishes the ground truth; readiness evaluates against that ground truth.

## Implementation Simulation

Implementation simulation is the core readiness discovery activity. It is a mental dry-run of the implementation executed by walking through each step an engineer would take, using the Work Package as the sole source of truth.

### Simulation Methodology

For each logical unit of work in the Work Package, execute the following steps:

1. **Name the unit.** Define what is being built. Example: "Add a `ShippingAddress` type to the order model."
2. **Identify the starting point.** Which existing file would the implementer open first?
3. **Trace what they would read.** What existing code must they understand before writing new code?
4. **Write the first lines.** What would the type signature, function signature, or class definition look like?
5. **Write the connections.** How would this new code connect to existing code: imports, constructor injection, event handlers, route registration?
6. **Handle errors.** What happens when each step fails? Does the Work Package say?
7. **Handle edge cases.** What about empty states, duplicate data, concurrent access? Does the Work Package say?
8. **Verify correctness.** How would the implementer know this works? Does the Work Package specify acceptance criteria with enough precision to write a test?

### Simulation Depth

| Depth | What You Simulate | When |
|---|---|---|
| Shallow | File creation, type definitions, function signatures | New files, clearly scoped additions |
| Medium | Control flow, error paths, integration points | Most implementation work |
| Deep | State transitions, concurrent access, data migration, rollback | Critical paths, data operations, distributed systems |

Apply medium depth by default. Use deep simulation for any Work Package that touches data persistence, cross-service communication, or security boundaries. Use shallow only when the Work Package describes a purely additive change in a well-known area with no side effects.

### Recording Simulation Gaps

Each time the simulation stops because information is missing, ambiguous, or contradictory, record:

- **Where**: which file, function, or step was being implemented.
- **What is missing**: the exact information gap.
- **Engineer response**: would they guess, search the codebase, or ask for clarification?
- **Consequence**: would a wrong guess cause incorrect behavior, or just delay?

A Work Package with zero simulation gaps passes the core readiness test. Every gap is a finding.

### Simulation Ceiling

The worst gap found during simulation determines the ceiling for the readiness score:

| Worst Gap | Score Ceiling |
|---|---|
| No gaps | 100 |
| Minor gaps (implementer can proceed with brief clarification) | 89 |
| Significant gaps (implementer must guess or ask) | 69 |
| Critical gaps (implementer would produce wrong code) | 39 |

If the simulation cannot complete, the Work Package is not ready regardless of how well-written it appears. The simulation ceiling is the primary constraint on the readiness score — no score may exceed the ceiling.

## Gap Analysis

Gap analysis is the four-layer comparison that reveals inconsistencies between what exists, what is proposed, what the engineer would build, and what would actually ship.

### The Four-Layer Model

```
Repository (what exists)
    │
    ▼
Work Package (what is proposed)
    │
    ▼
Expected Implementation (what the engineer would build based on the Work Package)
    │
    ▼
Implementation Reality (what would actually ship after accounting for gaps)
```

### How to Execute

Execute in order:

1. **Repository → Work Package**: Verify every file path, type name, module reference, and assumption in the Work Package against the actual repository. Identify factual errors and outdated assumptions.

2. **Work Package → Expected Implementation**: Use the simulation output. For each gap found during simulation, project what a reasonable engineer would build. This projection becomes the Expected Implementation layer.

3. **Expected Implementation → Implementation Reality**: For each case where the engineer would fill a gap independently, determine whether their likely implementation would differ from the Work Package's original intent. This delta is silent deviation risk.

4. **Repository → Implementation Reality**: Check whether the engineer's projected implementation would contradict existing repository patterns, conventions, or architecture. This reveals architectural drift.

### Comparison Table

| Comparison | Question | What It Reveals |
|---|---|---|
| Repository → Work Package | Does the Work Package accurately reflect existing code? | Factual errors, outdated assumptions |
| Work Package → Expected Implementation | Can I simulate the implementation from the Work Package alone? | Missing detail, ambiguity |
| Expected Implementation → Implementation Reality | What would differ from the Work Package's intent if the engineer filled gaps independently? | Silent deviation risk |
| Repository → Implementation Reality | Would the resulting code contradict existing patterns? | Architectural drift, inconsistency |

### Output

Gap analysis produces a list of discrepancies organized by comparison type. Each discrepancy maps to one or more findings that are classified by severity in the next step.

### Common Mistakes

- Stopping after Repository → Work Package comparison. This only catches factual errors, not the more dangerous silent deviation risks.
- Failing to trace Expected Implementation → Implementation Reality. This is the step that catches the most impactful gaps — the ones where the engineer unknowingly deviates from intent.
- Treating gaps as independent rather than chained. A single missing specification in the Work Package can produce a cascade through all four layers.

## Severity Model

Every finding is classified by severity. Severity determines how the finding impacts the readiness determination.

### Severity Levels

| Severity | Criteria | Required Response |
|---|---|---|
| Critical | Causes incorrect behavior, data loss, security vulnerability, or unrecoverable architectural damage. Implementation based on this finding would produce a system that does not satisfy requirements or violates non-negotiable constraints. | Work Package must be revised and re-reviewed. |
| High | Significantly increases implementation risk, creates substantial maintenance burden, or contradicts established conventions without justification. The implementer could produce working code, but it would be inconsistent, fragile, or costly to maintain. | Work Package should be revised before implementation. Conditional PASS possible with clear guidance. |
| Medium | Creates minor inconsistency, misses a reuse opportunity, or omits information that would slow implementation but not prevent it. The implementer could work around this without architectural damage. | Should be addressed but does not block. May be tracked as a follow-up. |
| Low | Suggestion for improvement. Does not affect implementation correctness or risk. | Optional. Implementation can proceed without addressing. |

### Severity Assignment Rules

1. A finding's severity is determined by its impact on implementation correctness and long-term maintainability, not by the reviewer's confidence in the finding.
2. When uncertain between two severity levels, choose the lower severity. Over-classification erodes credibility.
3. If the same root cause produces multiple symptoms, file one finding at the severity of the worst symptom.
4. A finding that could become Critical under specific conditions but is not guaranteed to manifest should be classified as High with the conditions documented.

## Readiness Score

The readiness score is calculated following the methodology in `references/review-scoring.md`. The simulation ceiling (see above) serves as the primary constraint on the score.

## Decision Model

The decision model — including PASS / FAIL / CONDITIONAL PASS rules, conditional pass guidance, and confidence assessment — is defined in `references/decision-matrix.md`. The decision model consumes the severity-weighted finding set (classified above), the readiness score (calculated per `references/review-scoring.md`), and the simulation ceiling.

## Common Mistakes

### Simulation as Document Review

Treating simulation as a static reading exercise rather than a dynamic mental dry-run. The most important gaps are discovered during active simulation, not during passive reading.

### Happy-Path-Only Simulation

Simulating only the success path and ignoring error handling, edge cases, and failure modes. Happy-path-only simulation misses the majority of implementation gaps.

### Shallow Depth for Complex Changes

Applying shallow simulation depth to Work Packages that touch data persistence, cross-service communication, or security boundaries. These always require deep simulation.

### Stopping at Repository Comparison

Performing only the first comparison in gap analysis (Repository → Work Package) and omitting the Expected Implementation and Implementation Reality layers. These later comparisons catch the most dangerous gaps.

### Accepting Silent Deviation

Finding a gap during simulation and accepting it with "the engineer will figure it out." Every gap filled independently by the engineer is a potential deviation from the Work Package's intent. It must be recorded and evaluated.

### Over-Confidence Without Simulation

Assigning High confidence when the simulation was shallow, incomplete, or skipped entirely. Confidence assessment follows the methodology in `references/decision-matrix.md` and must be proportional to simulation depth.



