---
name: review-principles
description: Canonical source of engineering philosophy and review principles for the Work Package Review framework. Defines every principle, its rationale, application guidance, and resolution rules when principles conflict.
---

# Review Principles

## Purpose

This document defines the engineering philosophy underpinning every review performed within the Work Package Review framework.

A review is an engineering judgment, not a mechanical checklist. Principles exist to make that judgment consistent, defensible, and reproducible across reviewers, teams, and codebases. They answer the question "why?" when methodology documents answer "how?"

Every other document in this framework implements these principles. No other document should define them.

## How to Read This Document

Principles are organized into five groups:

| Group | Scope |
|---|---|
| Foundation | Principles that underpin every review. Violating these invalidates the review entirely. |
| Evidence | How conclusions are formed and justified. |
| Design | Principles for evaluating the Work Package's technical decisions. |
| Maintainability | Principles for evaluating long-term cost and consistency. |
| Decision | How trade-offs are resolved and judgments reached. |

Groups are not hierarchical. A Foundation principle does not automatically override a Design principle. Priority is determined by the specific conflict, not by group membership. See Principle Conflicts.

## Foundation Principles

### Repository First

**Statement**: Never evaluate a Work Package in isolation. The repository is the authoritative source of truth. Every claim in the Work Package must be validated against what actually exists in the codebase.

**Why this exists**: A Work Package describes an intention. The repository describes reality. Intentions that conflict with reality produce incorrect code, regardless of how well the intention is documented. The repository contains the project's actual architecture, conventions, constraints, and trade-offs — not what the Work Package author assumed they were.

**Application**: Before accepting any Work Package statement as true, verify it against the repository. File paths must exist. Type names must match. Architectural patterns must be confirmed by reading code, not inferred from naming.

**What it prevents**: Reviewing in abstraction leads to approving Work Packages that contradict the codebase, recommend patterns the project has rejected, or miss existing implementations that should be extended rather than replaced.

**Mandatory**: Yes. A review that does not reference the repository is not a review.

### Evidence Based

**Statement**: Every finding must cite observable evidence from the repository or the Work Package. Subjective statements, impressions, and unsupported assertions have no place in a review.

**Why this exists**: Evidence is what makes a review falsifiable. An implementer reading a finding should be able to verify the claim by reading the same file at the same line. Without evidence, a finding is an opinion. With evidence, a finding is a fact the implementer can act on.

**Application**: Every finding must include file paths, line numbers, and relevant excerpts per the evidence standards in `references/evidence-and-justification.md`. The relationship between the evidence and the conclusion must be explicit. Evidence must be from the repository or the Work Package itself — not inferred, assumed, or recalled from memory.

**What it prevents**: Unfounded assertions, reviewer bias, and findings that cannot be verified. Evidence forces the reviewer to demonstrate rather than claim.

**Mandatory**: Yes. Findings without evidence are incomplete.

### Single Source of Truth

**Statement**: Every engineering concept within the framework is owned by exactly one document. Other documents may reference it but must not duplicate it.

**Why this exists**: Duplicated knowledge diverges over time. When the same concept is defined in two places, updates to one create inconsistencies with the other. A reviewer relying on outdated definitions produces unreliable findings. Single ownership forces updates to propagate correctly and eliminates ambiguity about which definition is authoritative.

**Application**: When a finding references an engineering principle, the principle definition lives in this document. When a methodology step requires a guideline, the guideline lives in the methodology document. Cross-references are preferred over inline definitions. If a concept appears useful in a second context, reference it rather than redefining it.

**What it prevents**: Contradictory definitions, stale knowledge, and confusion about which version of a rule applies.

**Mandatory**: Yes. Framework evolution depends on strict ownership.

### Deterministic Reviews

**Statement**: Two reviewers applying the same methodology to the same Work Package and repository should arrive at similar conclusions. Review outcomes are a property of the methodology and the evidence, not of the individual reviewer.

**Why this exists**: A review that depends on the specific reviewer is not an engineering gate — it is a personality test. The framework exists because individual judgment varies. Standardized principles, methodology, and evidence requirements make outcomes reproducible. Disagreement between reviewers exposes a gap in the methodology, not a difference in taste.

**Application**: Principles must be specific enough to apply consistently. The methodology for achieving and measuring consistency is defined in `references/review-consistency.md`. When two reviewers disagree, the disagreement should be traced to a specific principle, evidence gap, or methodological ambiguity — then the framework is updated to eliminate the ambiguity.

**What it prevents**: Reviews that pass or fail based on which reviewer was assigned. Inconsistent outcomes across the same team.

**Mandatory**: Yes. If two reviewers following this framework reach different conclusions on the same inputs, the framework has failed.

## Evidence Principles

### Challenge Assumptions

**Statement**: Treat every statement in the Work Package as a hypothesis that must be validated against the repository. Do not assume correctness because the Work Package looks professional, comes from a senior contributor, or matches prior work.

**Why this exists**: Work Package authors have incomplete knowledge of the codebase. They may be unaware of existing implementations, architectural decisions, or conventions that contradict their proposal. The reviewer's role is to find these gaps before they reach implementation.

**Application**: For every architectural claim, dependency decision, and design choice in the Work Package, ask: "What evidence from the repository supports this?" Claim validation follows the methodology in `references/evidence-and-justification.md`. If the evidence does not exist or contradicts the claim, the claim is a finding. Actively search for disconfirming evidence — find reasons the Work Package might be wrong, not reasons it might be right.

**What it prevents**: Confirmation bias, silent acceptance of incorrect assumptions, and reviews that validate rather than challenge.

**Mandatory**: Yes. The reviewer is not the Work Package author's validator. The reviewer is the codebase's advocate.

### Verifiability

**Statement**: Every finding, recommendation, and conclusion in a review must be independently verifiable by reading the repository or the Work Package.

**Why this exists**: Trust in a review comes from transparency. If an implementer cannot verify a finding by following the evidence trail, the finding is not actionable. Verifiability also protects the reviewer — an evidenced finding cannot be dismissed as personal opinion.

**Application**: Evidence citations must follow the traceability requirements in `references/evidence-and-justification.md`. When citing conventions, provide at least three independent examples.

**What it prevents**: Unverifiable claims, appeals to authority, and findings that require the implementer to take the reviewer's word.

## Design Principles

### Simplicity

**Statement**: Prefer the simplest solution that satisfies the requirements. Complexity is a cost that must be justified by a proportional benefit.

**Why this exists**: Every abstraction, configuration knob, dependency, and indirection layer has a maintenance cost. Unnecessary complexity accumulates as the codebase evolves, making future changes harder, slower, and more error-prone. The simplest correct solution minimizes long-term cost.

**Application**: When the Work Package proposes multiple approaches, evaluate whether the simplest one satisfies all requirements. When it introduces a new pattern, evaluate whether an existing simpler pattern would suffice. When it adds configuration, evaluate whether each knob has a demonstrated use case. Flag complexity without a clear justification.

**What it prevents**: Over-engineering, unnecessary abstractions, speculative generality, and configuration bloat.

**Mandatory**: No. Simplicity is a strong preference, but some problems require complex solutions. The principle demands justification for complexity, not prohibition of it.

### Explicit over Implicit

**Statement**: The Work Package should state requirements, constraints, contracts, and assumptions explicitly rather than relying on the implementer to infer them.

**Why this exists**: Implicit knowledge is the primary source of implementation偏差. When the implementer must infer the data model, error handling strategy, or integration approach, their inference may differ from the Work Package author's intent. Explicit specification eliminates this gap. An explicit requirement can be reviewed, tested, and verified. An implicit one cannot.

**Application**: If a requirement, constraint, or behavior is important enough to include in the Work Package, it is important enough to specify explicitly. Acceptable inference is limited to patterns that are:
- Documented as project conventions
- Verified across three or more existing implementations
- Unambiguous in their application to the new code

**What it prevents**: Silent deviation between intent and implementation. Rework caused by mismatched assumptions. Untestable acceptance criteria.

**Mandatory**: Yes for all requirements that affect correctness, integration, or observable behavior. No for internal implementation choices that are purely stylistic.

### Separation of Responsibilities

**Statement**: Each module, component, or layer in the system should have a clearly defined responsibility that does not overlap with other units.

**Why this exists**: Overlapping responsibilities create coupling. When two modules handle the same concern, changes to one may break the other, and developers cannot predict which module implements a given behavior. Clean separation makes the system predictable, testable, and maintainable.

**Application**: Evaluate whether the Work Package's proposed changes respect existing boundaries. Does it add data access logic to a presentation component? Does it place cross-cutting concern handling in a domain module? Does it create a new module whose responsibility overlaps with an existing one?

**What it prevents**: Blurred boundaries, god modules, and the gradual erosion of architectural integrity.

### High Cohesion, Low Coupling

**Statement**: Related behavior should be kept together (high cohesion). Unrelated behavior should be kept apart with minimal dependencies between modules (low coupling).

**Why this exists**: High cohesion makes code easier to find, understand, and change — all related logic lives in one place. Low coupling makes code easier to test, replace, and reason about — changes to one module do not cascade to others. These are the two most predictive structural properties of long-term maintainability.

**Application**: Evaluate whether the Work Package groups related logic in the same module and separates unrelated concerns across modules. Evaluate whether new dependencies create coupling that exceeds what the architecture allows. A dependency that crosses architectural layers in the wrong direction is a finding regardless of how small it seems.

**What it prevents**: Scattered logic, tight coupling that makes testing and refactoring costly, and architecture that degrades under incremental changes.

## Maintainability Principles

### Reuse First

**Statement**: Before proposing new code, search for existing implementations that already solve the problem. Extend or compose existing code before creating new code.

**Why this exists**: Every new type, function, and component is a maintenance liability. It must be understood, tested, documented, and kept compatible with everything else. Reusing existing code has zero marginal maintenance cost — the code already exists, is already tested, and is already understood by the team. Duplication multiplies maintenance cost without multiplying value.

**Application**: For every type, function, or component the Work Package proposes to create, verify whether the repository already contains an equivalent applying the discovery methodology in `references/repository-analysis.md` and the evaluation methodology in `references/repository-reuse.md`. When an equivalent exists, evaluate whether it can be extended to satisfy the new requirement. If it cannot, document why. Flag missed reuse opportunities even if they seem minor — small duplications compound.

**What it prevents**: Unnecessary code growth, divergent implementations of the same concept, and the gradual fragmentation of the codebase.

**Mandatory**: Yes. The reviewer must actively search for reuse candidates. Reuse opportunities that are missed by the Work Package but identified by the reviewer are always findings.

### Incremental Change

**Statement**: Prefer small, reversible changes over large, irreversible ones. A Work Package should minimize the surface area of change.

**Why this exists**: Large changes are harder to review, test, and roll back. They introduce more risk per deployment and create more opportunities for subtle integration failures. Incremental changes can be validated, deployed, and rolled back independently. When a small change fails, the cost is small. When a large change fails, the cost is proportional.

**Application**: Evaluate whether the Work Package can be decomposed into smaller independent deliverables. Flag changes that modify many unrelated modules simultaneously. Flag migrations that lack coexistence or rollback strategies. Recommend incremental delivery when the Work Package describes a single large change that could be split.

**What it prevents**: High-risk deployments, unrecoverable failures, and changes that are too large to review effectively.

**Mandatory**: No, but deviation requires justification. A Work Package that proposes a large change must explain why incremental delivery is infeasible.

### Least Surprise

**Statement**: The behavior of new code should be predictable given the existing codebase conventions, naming, and patterns.

**Why this exists**: Engineers spend more time reading code than writing it. Predictable code reduces cognitive load — an engineer reading a new function can understand what it does without reading its implementation because it follows the same patterns as every other function in the codebase. Surprising code forces the reader to stop, investigate, and verify. Each surprise is a tax on future readers.

**Application**: Evaluate whether the Work Package's proposed implementation follows the project's conventions. A new module that uses a different error handling strategy, naming convention, or structural pattern than the rest of the codebase creates surprise even if it is technically correct.

**What it prevents**: Inconsistent codebases, increased onboarding time, and reduced readability.

### Long-term Maintainability

**Statement**: Evaluate Work Package decisions by their projected cost over the system's expected lifetime, not by the cost of initial implementation.

**Why this exists**: The cheapest implementation today is often the most expensive over five years. A hardcoded value avoids the cost of a configuration system today but creates a deployment bottleneck every time the value needs to change. A direct database query in a view avoids the cost of a service layer today but makes every future schema change more expensive. Short-term cost optimization produces long-term maintenance debt.

**Application**: For each design decision in the Work Package, consider the maintenance trajectory. Will this choice make future changes easier or harder? Does it create patterns that will be repeated, accumulating cost? Does it introduce coupling that will constrain future architecture decisions? Flag decisions that optimize for delivery speed at the expense of maintainability.

**What it prevents**: Accumulated technical debt, architecture erosion, and systems that become too costly to change.

## Decision Principles

### Engineering over Opinion

**Statement**: Engineering principles, evidence, and long-term cost analysis outweigh personal preferences, team habits, and industry trends.

**Why this exists**: Every engineer has preferences — a preferred framework, a preferred pattern, a preferred style. These preferences are not engineering decisions. An engineering decision is one that can be justified with evidence, principle, and impact analysis. When a reviewer's recommendation is based on "I prefer X" rather than "X minimizes maintenance cost because...", the recommendation is invalid.

**Application**: All recommendations must be justified with reference to principles defined in this document. Recommendations that cannot be traced to a principle are opinions, not engineering judgments. When the reviewer's preference conflicts with established project conventions, the conventions win unless the Work Package includes a migration plan.

**What it prevents**: Reviews that impose personal taste, decisions driven by unfamiliarity or fashion, and inconsistency across reviewers with different preferences.

### Principle Hierarchy

When principles conflict, use the following hierarchy to resolve:

1. **Repository First** and **Evidence Based** outrank all other principles. A review that contradicts the repository or lacks evidence fails regardless of how well it satisfies other principles.
2. **Deterministic Reviews** outrank **Simplicity** and **Explicit over Implicit**. A simpler or more explicit approach that makes outcomes non-deterministic is not acceptable.
3. **Reuse First** and **Long-term Maintainability** outrank **Simplicity** when the simple solution creates long-term cost. A simple hack that would need to be rewritten in six months is not simpler than a properly structured solution.
4. **Separation of Responsibilities** and **High Cohesion, Low Coupling** outrank **Least Surprise** when the surprising solution has better structural properties. A reorganization that improves module boundaries may surprise readers initially but will become predictable once understood.

This hierarchy is not absolute. See Principle Conflicts for how to resolve cases where the hierarchy does not provide clear guidance.

### Principle Conflicts

Principle conflicts arise when two valid principles pull in opposite directions. The following method resolves them:

**Resolution Method**:

1. **Name both principles explicitly.** "Simplicity says to add a boolean parameter. Separation of Responsibilities says to create a new method." Ambiguous conflicts produce ambiguous resolutions.
2. **Assess the cost of violating each principle.** Which violation has the higher long-term cost? Be specific: "Adding a boolean parameter makes this function harder to read (Simplicity violation). Creating a new method duplicates the existing test setup (Reuse First violation)."
3. **Consult the hierarchy.** If one principle outranks the other, the higher-ranking principle governs.
4. **If the hierarchy does not resolve, evaluate marginal cost.** Which principle is violated more severely? A minor Simplicity violation (one extra parameter) may be acceptable to avoid a major Separation of Responsibilities violation (cross-layer coupling).
5. **Document the conflict and resolution in the finding.** Explain which principles conflicted, why the chosen principle prevailed, and what made this case different from a case where the opposite choice would be correct.

**Examples**:

| Conflict | Resolution |
|---|---|
| Simplicity vs. Long-term Maintainability: The simplest implementation for an audit log is to append to a file. A proper implementation uses an immutable event store. | Long-term Maintainability wins. The file-based approach creates data loss risk, query limitations, and migration cost that far exceed the initial complexity of the event store. |
| Reuse First vs. Separation of Responsibilities: An existing utility function does what the Work Package needs, but importing it would create a dependency from the new module to a module it should not depend on. | Separation of Responsibilities wins. The correct resolution is to extract the shared logic into a neutral module that both can depend on, rather than creating an inappropriate dependency. |
| Explicit over Implicit vs. Simplicity: The Work Package could specify every error code explicitly (explicit) or reference the project's standard error handling pattern (simpler). | Simplicity wins if the error pattern is a documented project convention. Explicit specification adds no value when the convention is already unambiguous. |

### Decision Philosophy

**The goal is not zero findings.** The goal is to ensure every finding is justified, actionable, and necessary.

A review that finds nothing wrong with a non-trivial Work Package is statistically unlikely and should be re-examined. A review that finds many minor issues but no significant blockers may pass — minor issues do not necessarily mean the Work Package is not ready.

**Severity is proportional to impact, not to quantity.** A single Critical finding is more significant than ten Low findings. The review should not create the impression that many small issues equate to a failed review.

**A positive outcome is a successful review.** If the Work Package passes, the reviewer has succeeded in finding no blockers. The reviewer should state this explicitly. There is no expectation that every review must produce findings.

**Certainty is rare.** Engineering decisions involve trade-offs. A reviewer who expresses absolute certainty about a complex judgment is likely unaware of their assumptions. Confidence should be proportional to evidence depth. High confidence is earned through thorough investigation, not through familiarity with the domain.

## Principle Application Summary

| Principle | Mandatory | What It Governs |
|---|---|---|
| Repository First | Yes | All claims in findings |
| Evidence Based | Yes | All findings |
| Single Source of Truth | Yes | Framework document structure |
| Deterministic Reviews | Yes | Methodology and outcome consistency |
| Challenge Assumptions | Yes | Reviewer stance |
| Verifiability | Yes | Finding structure |
| Explicit over Implicit | Yes* | Work Package requirements |
| Reuse First | Yes | Code creation decisions |
| Simplicity | No | Design preference |
| Separation of Responsibilities | No | Architecture evaluation |
| High Cohesion, Low Coupling | No | Module structure |
| Incremental Change | No | Change strategy |
| Least Surprise | No | Convention alignment |
| Long-term Maintainability | No | Design trade-offs |
| Engineering over Opinion | Yes | Recommendation justification |

*Explicit over Implicit is mandatory for requirements affecting correctness, integration, or observable behavior. It is not mandatory for internal implementation choices.
