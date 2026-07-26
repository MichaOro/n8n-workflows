---
name: repository-analysis
description: Canonical methodology for repository analysis during implementation-readiness reviews. Defines how to discover project topography, reconstruct architecture, identify conventions, catalog reuse opportunities, and map dependencies. Applies the principles defined in references/review-principles.md.
---

# Repository Analysis

## Purpose

Repository analysis is the practice of building a structural understanding of a codebase before evaluating a Work Package. It establishes the ground truth against which every Work Package statement is measured.

A reviewer who does not understand the repository cannot detect architectural inconsistencies, missed reuse opportunities, or incorrect assumptions. Repository analysis is the foundation of every reliable review.

This document defines how repository analysis is performed. It is the single source of truth for all repository analysis methodology within the Work Package Review framework.

## Principles Applied

Repository analysis implements the following principles from `references/review-principles.md`:

- **Repository First**: The repository is the ground truth. Every Work Package claim is validated against what exists in the codebase.
- **Evidence Based**: Analysis findings are supported by file paths, code excerpts, and documented conventions. No finding is asserted without evidence.
- **Challenge Assumptions**: The reviewer actively reconstructs architecture and discovers conventions rather than assuming they match the Work Package or documentation.
- **Verifiability**: Every analysis output includes enough context for the implementer to verify independently.

## When Repository Analysis Applies

Repository analysis is performed during every review. It is never optional.

- **Every review**: Standard analysis depth is required regardless of Work Package size.
- **Trivial changes**: A one-line configuration change in a well-known area may justify light analysis, but the phase is still executed — only the depth varies.
- **Large or unfamiliar repositories**: Analysis depth increases proportionally to the reviewer's unfamiliarity with the codebase.

Repository analysis must be completed before Implementation Simulation begins. The simulation depends on the repository understanding built during analysis.

## Analysis Depth

Analysis depth is determined by the reviewer's existing familiarity with the repository and the complexity of the Work Package.

| Depth | Coverage | When |
|---|---|---|
| Light | Project topography, architectural pattern identification, direct matching of Work Package references | Work Package is a trivial change in a well-known area of a familiar repository. Reviewer has recent, thorough knowledge of all affected modules. |
| Standard | Full five-area investigation: topography, architecture, conventions, reuse, dependencies | Default for all reviews. Applies when the reviewer has general familiarity but needs to confirm specifics. |
| Deep | Extended investigation including historical patterns, migration artifacts, dead code paths, and convention exceptions | Work Package crosses subsystem boundaries, modifies core infrastructure, or introduces new patterns. Repository is large or unfamiliar. |

When uncertain, choose the deeper level. Insufficient analysis depth is the most common source of undetected findings.

## The Five Investigation Areas

Repository analysis covers five areas. Each area is an investigation, not a sequential phase. The reviewer may investigate areas in any order, revisit areas as new questions arise, and iterate between areas as understanding deepens.

### 1. Project Topography

**Purpose**: Understand the fundamental characteristics of the project before examining implementation details.

**Why this exists**: Every project makes foundational choices about language, build system, structure, and configuration. These choices constrain what the Work Package can and cannot do. Understanding them first prevents the reviewer from making recommendations that violate project-level constraints.

**Investigation**:

Read the top-level directory structure and root configuration files to identify:

- **Language and runtime**: package.json, Cargo.toml, pyproject.toml, go.mod, build.gradle, or equivalent.
- **Build system**: How the project is built, tested, and linted. The build configuration reveals which tools are standard and which are absent by choice.
- **Project structure**: Monorepo with packages, single-module project, or multi-repo composition. Package boundaries reveal ownership and coupling.
- **Configuration conventions**: How environment variables, feature flags, secrets, and application settings are managed. Look for .env files, config modules, feature flag systems, and secrets infrastructure.
- **Testing infrastructure**: Test framework, test runner configuration, coverage requirements, and test organization conventions.

**Techniques**:

- Read the root directory listing first. The top-level entries reveal the project's organizational model.
- Read each root configuration file in full. Build configuration, linter configuration, and type-checker configuration encode engineering decisions.
- For monorepos, read each package's configuration file to understand package boundaries and inter-package dependencies.

**Output**: A summary of language, build system, project structure, configuration approach, and testing infrastructure.

**Common mistakes**:

- Inferring project structure from directory names alone without reading configuration files. A directory named `packages/` suggests a monorepo but the build configuration confirms it.
- Skipping configuration files because they seem boilerplate. Configuration files encode the project's actual tooling choices, which differ from defaults more often than expected.
- Treating the build system as irrelevant to the review. Build system constraints determine what dependencies, language features, and tooling are available.

---

### 2. Architectural Contour

**Purpose**: Reconstruct the project's architectural pattern from the code itself.

**Why this exists**: Architecture is the set of constraints that governs how code is organized and how data flows. Every Work Package must respect these constraints. The reviewer must understand the architecture before evaluating whether the Work Package fits within it.

Architecture documentation may not exist, may be outdated, or may describe intent rather than reality. The code is the source of truth.

**Investigation**:

Read primary module or package entry points, application bootstrap files, and strategic implementation files to identify:

- **Architectural style**: Layered (controllers, services, repositories), feature-based, hexagonal / ports-and-adapters, event-driven, pipeline, plugin, or custom. The style is identified by how code is organized and how dependencies flow, not by what the project calls itself.
- **Cross-cutting concerns**: How authentication, authorization, logging, error handling, validation, and telemetry are implemented. These reveal architectural boundaries and enforcement mechanisms.
- **Data flow**: The request lifecycle from entry point to response. State management approach, caching strategy, persistence model. How data crosses architectural boundaries.
- **Module boundaries**: Where one module or package ends and another begins. How boundaries are enforced (interfaces, dependency inversion, package visibility).
- **Dependency direction**: Which layers or modules depend on which. Circular dependencies, allowed and forbidden dependency directions.
- **Extension and configuration points**: Where the system is designed to be extended (plugin systems, hooks, middleware, dependency injection containers).

**Techniques**:

- Start with the application entry point. The main function, application bootstrap, or server setup reveals the top-level wiring.
- Read module or package entry points (index files, barrel exports, public API surfaces). These reveal the public contract of each module.
- Trace one complete request or data flow from entry to response. This reveals how all architectural layers connect in practice.
- Identify dependency injection or service location infrastructure. The wiring reveals module relationships that are not visible in individual files.
- Search for interfaces, abstract classes, or traits that define architectural boundaries. The abstraction layer reveals the intended seams.
- Look for deprecated or migration paths. These reveal architectural evolution and may explain why certain patterns coexist.

**Output**: Architectural style identification, boundary file paths, data flow description, cross-cutting concern handling, and dependency direction map.

**Common mistakes**:

- Assuming directory structure reflects architecture. A directory named `services/` does not guarantee those modules follow a service pattern. Read the code.
- Assuming that because no explicit architecture documentation exists, there is no architecture. Every codebase has an architecture, even if it emerged unintentionally. Reconstruct it from the code.
- Inferring architecture from a single file. Architecture is a property of the system, not of individual files. Verify patterns across multiple files before concluding.
- Overlooking test architecture. Test organization often reveals architectural boundaries more clearly than production code because tests must explicitly set up module boundaries.

---

### 3. Convention Discovery

**Purpose**: Identify the implicit rules and patterns that govern how code is written in this project.

**Why this exists**: Conventions are the unwritten contract between all contributors. They ensure consistency without requiring explicit coordination. The reviewer must understand conventions to determine whether the Work Package follows them. Violating an undocumented convention produces code that feels wrong to project maintainers even when it is technically correct.

**Investigation**:

Read representative files from different layers and areas of the project to identify conventions. Verify each convention across at least three independent locations before treating it as project-wide.

Identify:

- **Naming conventions**: File naming (kebab-case, PascalCase, snake_case), class naming, function naming, variable casing, constant naming, type naming. Look for patterns in how names are chosen, including conventions for boolean variables, collection variables, and callback parameters.
- **Error handling patterns**: Result types vs. exceptions vs. error codes. How errors are propagated, logged, and surfaced to callers. Error response shapes for APIs. Error boundary patterns in UI code.
- **Import and module patterns**: Absolute vs. relative imports. Barrel files and re-exports. Public API surface definitions. Internal vs. external module visibility.
- **Testing conventions**: Test framework, file placement (co-located or separate directory), naming (test file names, test case names), mocking strategy (manual mocks, mock libraries, fixture data), test data management. How tests are organized by scope (unit, integration, e2e).
- **API conventions**: Request/response shapes, validation patterns (schema-based, decorator-based, middleware-based), serialization approach, pagination pattern, error response format, API versioning strategy.
- **State management**: Data flow model (unidirectional, observable, reactive). Caching strategy. State persistence and hydration. Reactivity trigger model.
- **Code organization conventions**: File length norms, function length norms, comment style, documentation expectations, dead code removal policy.

**Techniques**:

- Read files from at least three different layers or modules for each convention being investigated. A pattern observed in one file may be an individual choice; a pattern observed across three independent locations is likely a convention.
- Read recently modified files. Recent code is more likely to reflect current conventions than legacy code that has not been touched.
- Read test files. Tests reveal what the project considers important and how the project expects code to be structured.
- Read configuration files for linters, formatters, and type checkers. These encode some conventions explicitly (maximum line length, naming rules, import order).
- Read code review comments or pull request templates if available. These reveal what the team enforces during review, which is the true convention set.
- Search for TODO, FIXME, HACK, or WORKAROUND comments. These reveal where conventions are intentionally violated and why.

**Output**: A convention catalog organized by category (naming, error handling, imports, testing, API, state management, code organization). Each convention includes the pattern, three or more example file paths, and any known exceptions.

**Common mistakes**:

- Citing a convention observed in one file as project-wide. A single file may reflect an individual's style. Verify across at least three independent locations.
- Assuming linter configuration captures all conventions. Linters enforce syntax and style, not architectural or structural conventions. Read the code.
- Treating legacy code conventions as current conventions. The project may have changed conventions. Use recent files as the primary source and note legacy conventions as migration context.
- Missing negative conventions (things the project deliberately avoids). The absence of a pattern (no inheritance, no null, no default exports) is as significant as its presence.

---

### 4. Reuse Inventory

**Purpose**: Discover existing implementations that overlap with the Work Package's domain so the reviewer can evaluate whether the Work Package creates unnecessary duplication.

**Why this exists**: Every new type, function, or component is a maintenance liability. Before the Work Package creates something new, the reviewer must verify that an equivalent does not already exist. The project may contain utilities, patterns, or components that the Work Package author was unaware of. Flagging missed reuse opportunities is the primary mechanism for preventing unnecessary code growth.

**Investigation**:

Search for existing implementations in the Work Package's domain:

- **Types and interfaces**: Search for type names, interface names, and model names that match or relate to concepts in the Work Package.
- **Functions and utilities**: Search for function names and behaviors that match the operations described in the Work Package.
- **Components and patterns**: Search for UI components, services, hooks, middleware, or classes that implement patterns the Work Package proposes to create.
- **Tests**: Find existing tests for the modules the Work Package would modify. The test structure and coverage reveal the impact surface.
- **Configuration**: Find existing configuration entries that the Work Package would modify or duplicate.

**Techniques**:

- Use both semantic search (Grep) and name-based search (Glob). They find different things. Semantic search finds behavior by pattern; name-based search finds files by convention.
- Search for names that appear in the Work Package first (types, modules, file paths). These should exist in the repository. If they do not, the Work Package references non-existent code.
- Search for concepts, not just names. The project may use different terminology than the Work Package. If the Work Package says "validate" but the project uses "sanitize", name-based search alone misses the existing implementation.
- Read the actual code of each candidate. Search result snippets are insufficient for confirming behavioral equivalence.
- Check whether identified candidates are used in the codebase. An existing utility that has no callers may be deprecated or incomplete. Count callers to assess maturity.
- Check test coverage of identified candidates. Existing code with tests is more reliable than existing code without tests.

**Output**: A reuse inventory listing all existing types, functions, components, and patterns that overlap with the Work Package. Each entry includes file path, behavioral summary, caller count, test coverage status. Reuse evaluation (extend, compose, reference, or create) follows the methodology in `references/repository-reuse.md`.

**Common mistakes**:

- Relying on file name searches without also searching semantically. A utility function may have a different name than the Work Package uses, but the same behavior.
- Confirming existence without confirming behavioral equivalence. A function with the same name may have different behavior. Read the implementation.
- Stopping after finding one match. The project may have multiple implementations of the same concept, which itself is a finding.

---

### 5. Dependency Mapping

**Purpose**: Identify every repository element the Work Package would interact with, modify, or depend on.

**Why this exists**: The Work Package describes what changes, but it rarely enumerates everything the change touches. Every unlisted dependency is a risk: the implementer may miss a necessary change, or discover it during implementation and make an unplanned modification. Complete dependency mapping prevents both outcomes.

**Investigation**:

Map all dependencies the Work Package would create or modify:

- **Internal module dependencies**: Which existing modules would the new code import, call, extend, or configure? Which existing modules would be modified?
- **External library dependencies**: Would the Work Package introduce new external dependencies? Would it extend or upgrade existing ones? Check package.json or equivalent for current dependency state.
- **Database and storage dependencies**: Would database schemas, migrations, indexes, or queries change? Would storage formats, file layouts, or blob structures change?
- **API and protocol dependencies**: Would API contracts, message formats, event schemas, or protocol buffers change? Would existing endpoints be modified or new endpoints be added?
- **Configuration dependencies**: Would environment variables, feature flags, application settings, or deployment configuration change?
- **Infrastructure dependencies**: Would new services, queues, storage, or network resources be required? Would deployment topology change?

**Techniques**:

- For each type, function, or module the Work Package mentions, find its definition and all its consumers. Use Grep to find importers and callers.
- For each proposed new dependency, search the repository for existing patterns that already fulfill the dependency's purpose. The project may already solve the problem differently.
- Check dependency direction against architectural rules. A module in the data layer should not depend on a module in the presentation layer. Verify the Work Package respects allowed dependency directions.
- For configuration changes, search for existing configuration patterns. Does the project use environment variables, config files, or a feature flag system? The Work Package should use the same mechanism.
- For infrastructure changes, check deployment configuration (Dockerfiles, CI/CD pipelines, infrastructure-as-code) to understand what currently exists.

**Output**: A complete dependency map listing every repository element the Work Package touches, organized by category (internal modules, external libraries, data storage, APIs, configuration, infrastructure). Each entry includes the file path, the nature of the change (create, modify, delete), and the impact scope.

**Common mistakes**:

- Omitting configuration changes because they seem minor. Every configuration surface change should be enumerated — an unlisted environment variable is a runtime failure waiting to happen.
- Mapping direct dependencies but missing transitive dependencies. If module A depends on module B and the Work Package modifies A, the impact on B's consumers must be traced.
- Treating dependency mapping as a static activity. Dependencies discovered during Implementation Simulation (Phase 3 of the review lifecycle) must be added to the dependency map.

## Search Strategies

Effective repository investigation requires systematic search. The following strategies cover the major search dimensions.

### Semantic Search (Grep)

Search for behavioral patterns, not just names.

- Search for function calls that perform similar operations to what the Work Package describes.
- Search for error handling patterns matching the project's conventions.
- Search for type usage patterns to understand how types flow through the system.
- Search for import statements to understand module relationships.

When searching, cast a wide net first with a broad pattern, then narrow. The first result set reveals terminology, naming conventions, and organizational patterns that inform subsequent searches.

### Name-Based Search (Glob)

Search for files and directories by naming convention.

- Search for file names matching the Work Package's domain terminology.
- Search for directory structures that reveal module organization.
- Search for test files that match production files the Work Package modifies.

Name-based search is faster than semantic search but misses behavioral overlap where terminology differs.

### Reference Search

Trace how specific code elements are used across the repository.

- Find all callers of a function to assess modification impact.
- Find all implementations of an interface to understand extension points.
- Find all imports of a module to understand coupling.
- Find all usages of a type to understand serialization and persistence boundaries.

### Search Order

For each entity the Work Package mentions:

1. **Verify existence**: Does the named file, type, or function exist? (Name-based search)
2. **Verify behavioral accuracy**: Does the existing code do what the Work Package says? (Semantic search + read)
3. **Find callers**: Who depends on this entity? (Reference search)
4. **Find related entities**: What else exists in the same domain? (Name-based + semantic search)
5. **Find patterns**: What approach does the project use for similar problems? (Semantic search)

## Navigation Strategy

The order in which files are read affects analysis efficiency and completeness.

### Standard Order

For most reviews, read in this order:

1. **Root configuration** — Understand project foundations first.
2. **Entry points** — Main application bootstrap, module entry points.
3. **Work Package-referenced files** — Files the Work Package explicitly mentions or implies.
4. **Related files by dependency** — Files that the referenced files import or depend on.
5. **Related files by domain** — Files in the same directory or module as referenced files.
6. **Test files** — Test organization and patterns for the affected modules.
7. **Cross-cutting infrastructure** — Authentication, error handling, logging, configuration infrastructure.

### Large Repository Strategy

For repositories too large to read exhaustively:

1. **Isolate the impact zone**: Use the Work Package to determine which modules or packages are affected. Focus analysis on those areas.
2. **Read boundaries first**: Read the interfaces, types, and public APIs of affected modules before reading implementations. Boundaries reveal contracts without requiring full implementation understanding.
3. **Sample across layers**: Read at least one file from each architectural layer within the impact zone. Do not read all files in one layer before sampling others.
4. **Verify conventions in the impact zone**: Conventions may vary between modules. Verify conventions within the affected area rather than assuming project-wide uniformity.
5. **Document what was not read**: If analysis is limited to a subset of the repository, document which areas were not examined. This informs the confidence assessment.

## Evaluating Repository Understanding

Before proceeding to Implementation Simulation, evaluate whether the repository understanding is sufficient.

### Sufficiency Criteria

Repository understanding is sufficient when the reviewer can answer:
- What is the architectural pattern and where are the boundaries?
- What conventions govern code in the affected modules?
- What existing code overlaps with the Work Package's domain?
- What dependencies does the Work Package create or modify?
- What test infrastructure and patterns apply to the affected modules?

### Understanding Level

| Level | Meaning | Criteria |
|---|---|---|
| Thorough | The reviewer has read all relevant code and can answer all sufficiency questions with confidence. | All five investigation areas complete within the impact zone. Conventions verified across 3+ locations. Dependency map is complete. |
| Adequate | The reviewer has read the most important code and can answer most sufficiency questions. Some assumptions remain untested. | All five investigation areas addressed. Some convention verifications based on 2 locations. Some dependencies inferred but not traced. |
| Partial | The reviewer has read key files but significant areas remain unexplored. | Some investigation areas incomplete. Dependency map has known gaps. |

### When Understanding Is Insufficient

If repository understanding is Partial or lower:
- The reviewer should expand analysis before proceeding to Implementation Simulation.
- If expansion is not possible (time constraints, access limitations), the review confidence must be set to Low and the limitations documented.

The most common mitigation for insufficient understanding is to narrow the impact zone. Focus analysis on the modules most directly affected by the Work Package rather than attempting full-repository analysis.

### Impact on Final Confidence

Repository understanding level feeds into the overall review confidence assessment defined in `references/decision-matrix.md`. Thorough understanding supports High confidence; Adequate supports Medium; Partial or lower mandates Low confidence. Document any analysis limitations so they are reflected in the final confidence assessment.

## Output Requirements

Repository analysis produces the following artifacts. These become inputs to subsequent review phases.

### Architecture Summary

A concise description of:
- Architectural style and pattern
- Key file paths for architectural boundaries
- Cross-cutting concern handling
- Data flow overview
- Dependency direction rules

### Convention Catalog

An organized catalog of conventions identified during analysis:
- One entry per convention category (naming, error handling, imports, testing, API, state management)
- Each entry includes the pattern, 3+ example file paths, and any exceptions
- Negative conventions (patterns the project avoids) are noted separately

### Reuse Inventory

A list of existing code that overlaps with the Work Package:
- Each entry: file path, behavioral summary, caller count, test coverage status
- Recommendation for each: extend, compose, reference, or flag

### Dependency Map

A complete map of all dependencies the Work Package touches:
- Organized by category: internal modules, external libraries, data storage, APIs, configuration, infrastructure
- Each entry: file path, change type (create/modify/delete), impact scope

### Analysis Coverage

A statement of what was analyzed and what was not:
- Modules and directories examined
- Depth level applied (light, standard, deep)
- Known gaps or areas not examined
- Understanding level (thorough, adequate, partial)

## Common Mistakes

### Insufficient Breadth

Reading too few files before forming conclusions. Mitigation: verify every finding across at least three independent locations. Read at least one file from each architectural layer within the impact zone.

### Over-reliance on Names

Inferring behavior from file names, directory names, or function names without reading the implementation. Mitigation: for every search result, read the actual code before drawing conclusions.

### Conflation of Convention and Coincidence

Treating a pattern observed in one or two files as a project-wide convention. Mitigation: require three or more independent locations before classifying a pattern as a convention.

### Architecture from Directory Structure

Assuming that directory layout reflects architectural intent. Mitigation: trace actual dependency direction by reading import statements, not directory listings.

### Missed Negative Conventions

Failing to notice what the project deliberately avoids. Mitigation: actively search for patterns that are common in the language ecosystem but absent in the codebase. Their absence is an engineering decision.

### Fixed Analysis Scope

Treating the initial impact zone as the final impact zone. Dependencies discovered during later phases (see `references/implementation-readiness.md` for simulation methodology) may expand the analysis scope. Mitigation: revisit the analysis as new dependencies emerge during simulation.

### Confirmation Bias

Searching for evidence that confirms the Work Package's accuracy rather than actively searching for contradictions. This is a specific manifestation of the Confirmation Bias anti-pattern defined in `references/anti-patterns.md`. In repository analysis, it produces search strategies that confirm hypotheses rather than testing them.

Mitigation: formulate hypotheses before searching, then test each hypothesis with evidence. Search for disconfirming evidence as rigorously as confirming evidence.
