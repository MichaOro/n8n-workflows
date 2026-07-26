---
name: repository-reuse
description: Canonical engineering standard for repository reuse. Defines how to discover, evaluate, and decide on reuse opportunities. Covers extension vs creation, abstraction evaluation, duplication analysis, reuse confidence, and reuse trade-offs. Does not define repository analysis, engineering principles, or review workflow.
---

# Repository Reuse

## Purpose

This document defines how reviewers identify, evaluate, and decide on reuse opportunities within an existing repository. It is the single source of truth for all reuse methodology within the framework.

Reuse is not "always reuse." Good engineering requires balancing reuse against maintainability, simplicity, coupling, abstraction cost, and future evolution. This document teaches reviewers how to make sound reuse decisions rather than blindly maximizing reuse.

Repository analysis (`references/repository-analysis.md`) discovers what exists. This document determines how existing assets should be used. These responsibilities are separated: analysis answers "what is here?"; reuse answers "how should it be used?"

## How Reuse Fits Into the Review

Reuse evaluation occurs during Dimension Evaluation (review lifecycle Phase 4 in `references/review-process.md`), specifically within the **Reuse Efficiency** dimension. It depends on the Reuse Inventory produced during Repository Intelligence (Phase 2) via the methodology in `references/repository-analysis.md`.

The reviewer evaluates every type, function, component, and pattern the Work Package proposes to create against the inventory of existing assets, then makes a reuse decision.

---

## Evaluation Framework

For every proposed new creation in the Work Package, evaluate through five lenses:

| Lens | Question |
|---|---|
| Existence | Does an equivalent already exist in the repository? |
| Fitness | Is the existing asset suitable for the new requirement? |
| Extension | Can the existing asset be extended without degrading it? |
| Coupling | Does reuse create inappropriate dependencies? |
| Confidence | How certain is the reuse judgment? |

A finding is warranted when the answer to any lens indicates a gap between what the Work Package proposes and what the repository supports.

### Reuse Decision Matrix

| Exists? | Fit? | Extensible? | Decision |
|---|---|---|---|
| No | — | — | Create new. Verify no missed discovery. |
| Yes | Yes | — | Reference existing. No change needed. |
| Yes | Partial | Yes | Extend existing. |
| Yes | Partial | No | Create new. Document why extension is infeasible. |
| Yes | No | — | Create new. Document why existing is unsuitable. |
| Multiple | Varies | Varies | Flag fragmentation. Project has multiple implementations of the same concept. |

---

## 1. Existence: Does It Already Exist?

### What to Verify

For every type, function, component, utility, or service the Work Package proposes to create, verify whether the repository already contains an equivalent. Equivalence is determined by behavior, not by name.

### Evidence Standards

Evidence quality and strength follow the hierarchy defined in `references/evidence-and-justification.md`. For reuse-specific evidence:

| Evidence | Strength | What It Tells You |
|---|---|---|
| Same name, same behavior | Strong | Direct reuse candidate |
| Different name, same behavior | Medium | Terminology mismatch in Work Package |
| Same name, different behavior | Medium | Work Package may reference wrong code |
| Similar pattern, different domain | Weak | Architectural reuse opportunity |

### Common Mistakes

- **Name-only search**: Searching for the Work Package's terminology without considering synonyms, alternative naming conventions, or the project's established vocabulary.
- **Surface-level equivalence**: Confirming a type exists but not verifying its behavior matches the Work Package's requirements. A `validateEmail` function exists, but does it handle the same edge cases?
- **Missing multiple implementations**: Finding one match and stopping. The project may have multiple implementations of the same concept, which is itself a finding about fragmentation.

---

## 2. Fitness: Is It Suitable?

### Evaluation Criteria

An existing asset is suitable for reuse when it satisfies all of the following:

| Criterion | Question |
|---|---|
| Behavioral match | Does the existing code do what the Work Package requires? |
| Interface compatibility | Can the Work Package use this code without modifying its own interface? |
| Quality baseline | Does the existing code meet the project's quality standards? |
| Maintenance status | Is the existing code actively maintained, deprecated, or abandoned? |
| Dependency compatibility | Does using this code introduce acceptable dependencies? |

### When Existing Code Is Not Suitable

An existing asset is unsuitable when:

- **Behavioral mismatch**: The code does something different than what the Work Package requires. Verifying equivalence requires reading the implementation, not just the type signature.
- **Quality below threshold**: The existing code has no tests, is tightly coupled to unrelated concerns, or uses patterns the project has moved away from. Reusing low-quality code propagates technical debt.
- **Deprecated or abandoned**: The existing code has no recent commits, no callers, or explicit deprecation markers. Reusing it revives a code path the project intended to phase out.
- **Over-fitted**: The existing code is too specific to its original use case. Extending it to handle the new requirement would be more expensive than creating a clean implementation.

### Common Mistakes

- **Treating every existing implementation as a valid reuse candidate**: Some code exists but should not be reused. Verify fitness before recommending reuse.
- **Assuming test coverage implies quality**: Tests verify behavior; they do not guarantee good design. A tested but over-fitted utility may still be unsuitable.
- **Ignoring maintenance status**: Code that has not been touched in two years may be stable — or abandoned. Check commit history and caller count.

---

## 3. Extension: Can It Be Extended?

### Extension Strategies

When an existing asset partially matches the Work Package's requirements, evaluate extension strategies in preference order:

| Strategy | Description | When to Use |
|---|---|---|
| Compose | Use the existing asset as-is by composing it with new code | The existing behavior is correct and complete; the new requirement adds orthogonal functionality |
| Parameterize | Add a parameter to make the existing asset configurable for the new use case | The new requirement is a variation of the same operation |
| Specialize | Create a more specific version that builds on the general one | The general case is well-defined; the new requirement is a constrained subset |
| Extend interface | Add a new method, property, or handler to the existing interface | The extension is a natural addition to the existing abstraction |
| Wrap | Create a thin wrapper that adapts the existing interface to the new requirement | The existing interface does not match but the behavior is correct |

### When Extension Is the Wrong Choice

Extension is the wrong choice when:

- **Abstraction leak**: The extension requires the caller to understand implementation details of the extended code. A parameter that changes control flow is an abstraction leak.
- **Complexity spike**: The extension makes the existing code significantly harder to understand. A function that started as 10 lines and would become 60 lines is better replaced.
- **Violated responsibility**: The extension adds a concern the original module should not own. Adding logging to a data access function is reasonable; adding billing logic is not.
- **Cascading changes**: Extending one module requires changing all its consumers. The extension propagates coupling rather than containing it.

### Common Mistakes

- **Choosing extension when composition suffices**: Adding a parameter to a shared function that only one caller uses. Composition keeps the shared function simple and isolates the special case.
- **Choosing creation when extension suffices**: Creating a new implementation because the existing one is unfamiliar. Extension preserves the single implementation; creation adds a second.
- **Over-parameterizing**: Adding five optional parameters to handle future use cases that may never materialize. Prefer composition for speculative requirements.

---

## 4. Abstraction Evaluation

Reuse often requires abstraction — extracting shared logic, introducing interfaces, or creating utility modules. Not every abstraction is justified.

### When Abstraction Is Justified

| Justification | Example |
|---|---|
| Three or more independent locations use the same logic | Validation logic repeated across three controllers |
| The abstraction represents a stable domain concept | Date formatting, currency conversion, user identification |
| The abstraction hides a volatile implementation | Database access behind a repository interface |
| The abstraction enables testing | Dependency injection interface for a payment gateway |

### When Abstraction Is Premature

- **Two locations, one caller**: Two places use similar logic but one is a test. The production code is the only real consumer.
- **Speculative generality**: "We might need this later" without a concrete requirement. Unused abstractions impose a reading cost on every future developer.
- **One-line extraction**: A two-line expression extracted into a named function. The name adds more code than the expression itself.
- **Technology abstraction**: Wrapping a library "in case we change it later" when the team has never changed a library of that type. The wrapper duplicates the library's interface without adding value.

### Abstraction Cost

Every abstraction has costs that must be justified by reuse benefit:

| Cost | Description |
|---|---|
| Discovery | Future developers must find and understand the abstraction |
| Indirection | Callers must trace through the abstraction to understand behavior |
| Evolution | Changing the abstraction affects all consumers |
| Testing | The abstraction must be tested independently |
| Configuration | The abstraction may require its own configuration, dependency injection wiring, or registration |

### Common Mistakes

- **Abstracting before repetition**: Creating a reusable abstraction the first time a pattern appears. Abstraction before three independent occurrences is speculative.
- **Leaky abstraction**: An interface that exposes implementation details. Every consumer must understand the implementation to use the interface correctly.
- **God abstraction**: A "utils" module that accumulates unrelated functions. High cohesion within modules matters for abstractions too.
- **Framework abstraction**: Wrapping a framework or library in a custom abstraction without adding value. The wrapper becomes a maintenance burden that must stay in sync with the wrapped library.

---

## 5. Duplication Analysis

Not all duplication is bad. Some duplication is preferable to the wrong abstraction.

### When Duplication Is Acceptable

| Condition | Rationale |
|---|---|
| The duplicated code is stable and unlikely to change | No maintenance burden from keeping copies in sync |
| The duplicated code serves different architectural layers | Coupling the layers through shared code would violate separation of responsibilities |
| The duplication is temporary during a migration | The old and new implementations coexist during transition; one will be removed |
| The alternatives (abstraction, parameterization) would increase complexity more than duplication | The abstraction cost exceeds the duplication cost |

### When Duplication Is Harmful

- **Bug propagation**: A bug fix in one copy must be applied to all copies. Each missed copy is a production incident waiting to happen.
- **Divergent evolution**: Over time, the copies develop different behavior for the same concept. Future developers cannot tell whether the difference is intentional or an oversight.
- **Onboarding friction**: New team members learn the codebase and find the same logic in multiple places, unsure which one is authoritative.

### Duplication Assessment

Ask three questions before treating duplication as a finding:

1. **Would the copies ever need to change independently?** If yes, duplication is correct — they are not the same concept.
2. **Would sharing code introduce inappropriate coupling?** If yes, duplication is the lesser evil.
3. **Would the abstraction cost exceed the duplication cost over the system's expected lifetime?** If yes, accept the duplication.

### Common Mistakes

- **Treating all duplication as harmful**: Duplication in different architectural layers (e.g., a validation rule defined once in the API layer and once in the domain layer) may be correct separation.
- **Ignoring coupling cost of reuse**: Sharing code to eliminate duplication creates a dependency between previously independent modules. That dependency may cost more than the duplication it eliminates.
- **Applying the wrong abstraction to eliminate duplication**: Extracting shared code into a module that has no natural home. The module becomes a "miscellaneous" dumping ground because the only shared property of its contents is that they were duplicated.

---

## 6. Reuse Confidence

Every reuse decision carries uncertainty. Confidence expresses how certain the reviewer is that the reuse assessment is correct.

| Confidence | Meaning | When |
|---|---|---|
| High | The existing code is a behavioral match, actively maintained, and properly tested. Reuse is the correct decision. | Direct behavioral match verified by reading the implementation. Code has callers, tests, and recent commits. |
| Medium | The existing code appears to match but some uncertainty remains. Reuse is likely correct but should be verified during implementation. | Behavioral match inferred from naming and structure but not fully verified. Code exists but has no callers or tests. |
| Low | The existing code is the closest match found but significant uncertainty exists. Reuse is provisional and must be verified. | Discovery was challenging. The code found may or may not match the requirement. The reviewer would not bet on reuse without further investigation. |

### Factors That Reduce Confidence

- No test coverage for the existing code
- No callers for the existing code
- The existing code is in a different domain or layer than the Work Package
- The behavioral match was inferred from documentation or naming, not from reading code
- The existing code has recent churn or bug fixes
- The existing code uses deprecated APIs or patterns the project is migrating away from

### When Confidence Affects the Finding

- **High confidence, missed reuse opportunity**: Clear finding. The Work Package should have referenced the existing code.
- **Medium confidence**: Flag as a finding but note the uncertainty. The implementer should verify during implementation.
- **Low confidence**: Note as a reference rather than a finding. The implementer should investigate further but should not be blocked.

---

## 7. Architectural Reuse

Reuse extends beyond individual functions and types. Architectural patterns, structural conventions, and design approaches can also be reused.

### What Architectural Reuse Includes

| Type | Example |
|---|---|
| Design patterns | Repository pattern, service layer, middleware chain, event-driven architecture |
| Communication patterns | Request-response, publish-subscribe, command-query separation |
| Error handling strategy | Result types, exception hierarchy, error response format |
| State management | Unidirectional data flow, observable state, immutable updates |
| Testing approach | Test organization, fixture strategy, mock patterns |

### Evaluating Architectural Reuse

For each architectural pattern the Work Package proposes, evaluate:

1. **Does this pattern exist elsewhere in the repository?** If yes, the Work Package should use the same pattern.
2. **If the pattern exists, is the Work Package following it correctly?** The proposed approach should match the established pattern, not just share its name.
3. **If the pattern does not exist, is the Work Package introducing a new pattern?** A new pattern requires justification — why the existing patterns are insufficient.

### Architectural Reuse vs. Code Reuse

A Work Package may correctly implement an existing architectural pattern without reusing any specific code. This is still reuse: the structural knowledge, naming conventions, and behavioral expectations are reused even if every line of code is new.

Architectural reuse is as important as code reuse. A Work Package that introduces a new architectural pattern without justification should produce a finding even if no code is duplicated.

---

## 8. Reuse Trade-Offs

Reuse decisions involve competing engineering values. The framework's principles provide resolution guidance.

### Reuse First vs. Simplicity

Adding reuse may introduce indirection, configuration, or abstraction that makes the simple case harder to understand.

**Resolution**: Reuse First outranks Simplicity when the reuse eliminates genuine duplication (three or more independent occurrences). Simplicity wins when the reuse is speculative — abstracting before repetition exists.

### Reuse First vs. Separation of Responsibilities

Reusing an existing utility may create a dependency between modules that should be independent.

**Resolution**: Separation of Responsibilities wins. The correct resolution is to extract shared logic into a neutral module that both can depend on, rather than creating an inappropriate dependency. Document this as a finding if the Work Package proposes the wrong dependency direction.

### Reuse First vs. Incremental Change

Reusing existing code may require refactoring it first, increasing the change surface.

**Resolution**: Evaluate the refactoring cost. If refactoring the existing code is cheaper than duplicating it (including long-term maintenance), reuse wins. If the refactoring is a large, risky change and the duplication is small and stable, duplication may be acceptable.

### Reuse First vs. Long-term Maintainability

An existing implementation may work but use patterns the project is migrating away from. Reusing it perpetuates the old patterns.

**Resolution**: Long-term Maintainability wins. Creating a new implementation using current patterns is better than extending legacy patterns. Document why the existing code was not reused.

---

## 9. Repository Leverage

Repository leverage measures how much of the Work Package's requirements can be satisfied by existing code rather than new code. It is not a formal metric but a diagnostic tool.

### Estimating Leverage

For each major component of the Work Package:

| Leverage Level | Meaning | Implication |
|---|---|---|
| Full | Requirement satisfied by existing code unchanged | No new code needed for this component |
| High | Existing code covers most requirements; minor extension needed | Low risk, low cost |
| Medium | Existing code covers some requirements; significant new code needed | Moderate risk; verify reuse fitness carefully |
| Low | Existing code covers little; mostly new implementation | High risk; the Work Package is effectively greenfield |

### When to Flag Low Leverage

Low leverage is not automatically a finding. Some domains legitimately require new code. Flag low leverage when:

- The Work Package's domain overlaps significantly with existing code but the Work Package does not reference it
- The Work Package proposes a new implementation pattern when existing patterns would suffice
- The Work Package creates new types, functions, or components that duplicate existing capabilities

---

## 10. Reuse in the Review Lifecycle

### During Repository Analysis (Phase 2)

References `references/repository-analysis.md` for the Reuse Inventory methodology. The inventory identifies what exists. This document evaluates what to do with what exists.

### During Dimension Evaluation (Phase 4)

Within the **Reuse Efficiency** dimension, apply this document's framework to evaluate the Work Package's reuse decisions. The evaluation questions are:

- For every type, function, or component the Work Package proposes to create: what existing code matches, and how should it be used?
- For every extension proposed: is extension the correct strategy, or should the Work Package compose, create, or wrap instead?
- For every abstraction proposed: is the abstraction justified by three or more independent occurrences?
- For every duplicated capability: is the duplication harmful or acceptable?
- For every architectural pattern proposed: does it match the repository's established patterns?

### During Implementation Readiness (Phase 3)

Reuse evaluation feeds into the implementation simulation. If the Work Package proposes creating code that duplicates existing capabilities, the simulation reveals whether the implementer would discover the existing code or create a duplicate.

---

## 11. Examples

### Missed Reuse Opportunity

**Work Package proposes**: Create a new `validateEmail` function in the notification service module.

**Repository**: `src/utils/validators.ts:47` exports `isValidEmail(email: string): boolean` with the same behavior, used by three modules.

**Evaluation**: Exists (yes). Fit (yes — behavioral match verified by reading implementation). Extension (not needed — direct behavioral match).

**Finding**: The Work Package creates unnecessary duplication. The existing `isValidEmail` should be referenced or, if the requirement differs, extended.

### Correct Creation Decision

**Work Package proposes**: Create a new `OrderInvoice` domain type with specific behavior for order invoicing.

**Repository**: `CustomerInvoice` exists in the billing domain but handles different behavior (payment reconciliation, tax calculation, credit notes).

**Evaluation**: Exists (named different but domain-adjacent). Fit (no — the domain concepts are different despite sharing the "invoice" concept). Attempting to extend `CustomerInvoice` would violate Separation of Responsibilities by mixing order processing with billing.

**Finding**: No finding. The new type is justified. The Work Package should document that `CustomerInvoice` was considered but not suitable because of different domain responsibilities.

### Harmful Abstraction

**Work Package proposes**: Extract a shared `NotificationSender` abstraction because two notification types (email and SMS) both need to send notifications.

**Repository**: Only two notification types exist. Both are simple — email is 40 lines, SMS is 35 lines.

**Evaluation**: Abstraction violates the rule of three — only two independent occurrences. The abstraction would add interface files, dependency injection registration, and configuration. The duplication cost (sending logic duplicated between two small files, unlikely to change independently) is lower than the abstraction cost.

**Finding**: Abstraction is premature. The Work Package should keep the implementations separate until a third notification type emerges. Flag as a Medium finding with the rationale documented.

### Architectural Reuse Missed

**Work Package proposes**: Use a direct HTTP call to a downstream service.

**Repository**: All other service-to-service communication uses a message queue with retry logic and dead-letter routing.

**Evaluation**: The architectural pattern exists and is established. The Work Package introduces a different pattern (synchronous HTTP) where the established pattern (async messaging) would work.

**Finding**: The Work Package introduces a new communication pattern without justification. Either use the established message queue pattern or document why synchronous HTTP is necessary.
