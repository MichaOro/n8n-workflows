---
name: review-output-contract
description: Canonical contract for how engineering review results are communicated. Defines report structure, section ordering, mandatory and optional sections, presentation standards for findings, evidence, scores, confidence, and implementation guidance. Reusable across all review types (Work Package, Epic, ADR, Design, Audit, Architecture).
---

# Review Output Contract

## Purpose

This document defines the official output contract for engineering reviews. It specifies **what** every review report must contain, **how** it must be structured, and **how** findings, evidence, scores, and recommendations must be presented.

Work Package Reviews output a deterministic JSON object containing:
- **Metadata**: Review provenance and unique identifiers (`reviewId`, `reviewType`, `schemaVersion`)
- **Summary**: Machine-readable decision, score, confidence, finding counts, and routing-ready fields (`implementationReady`, `blockingFindings`, `totalFindings`) for workflow automation
- **Findings**: Structured array of all review findings with severity, dimension, evidence, recommendations, and a lifecycle `status` field
- **Report**: A structured report object (`{ format, version, content }`) whose `content` is a complete standalone markdown document suitable for archival and human reading

The JSON object is the **canonical source of truth**. The markdown inside `report.content` is a rendering derived from the JSON — it must never introduce information that does not exist elsewhere in the structured data. See "Source of Truth" below.

This contract ensures that every review output is:

- **Deterministic**: Same JSON structure for all decision outcomes (PASS, FAIL, CONDITIONAL PASS)
- **Machine-Readable**: Summary object designed for n8n Switch nodes, GitHub comments, dashboards, and metrics
- **Parseable**: No markdown outside the JSON. Single valid JSON object as complete output.
- **Complete**: All mandatory sections present in JSON and markdown report.
- **Consistent**: Two reviews from different reviewers follow the same structure.
- **Actionable**: Implementers can immediately act on findings.
- **Verifiable**: Every claim backed by traceable repository evidence.
- **Archival**: Markdown report is complete and understandable in isolation.

This document defines both JSON structure and markdown report contents. It does not define methodology, scoring, decision logic, or engineering principles.

## Contract Rules

### Rule 1: Output Format is JSON, RFC 8259 Compliant

Every review produced within the framework must return **exactly one** valid JSON object conforming to [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259). This is non-negotiable:

- Exactly one top-level JSON object. Not an array, not multiple concatenated objects.
- No markdown code fences (` ```json ... ``` `) wrapping the object.
- No explanatory text, preamble, or trailing commentary before or after the JSON.
- No comments inside the JSON (JSON does not support comments; do not add them).
- The output must be parseable by `JSON.parse()` (or any standards-compliant parser) with zero preprocessing.

If the model's response cannot be passed directly to a JSON parser without stripping surrounding text, the output violates this contract.

### Rule 2: All reviews must follow this contract

Every review produced within the framework must adhere to this output contract. Variation is limited to metadata fields and review-type-specific summary fields. Structure, mandatory fields, and field types are required.

### Rule 3: Deterministic Field Presence

All four top-level fields (`metadata`, `summary`, `findings`, `report`) must always exist, regardless of decision outcome. The `findings` array may be empty, but the field itself must be present. The `report` field is always an object (`{ format, version, content }`), never a bare string.

### Rule 4: Every finding must be self-contained

Each finding must include all five links of the Engineering Justification Chain: evidence, principle, problem, impact, and recommendation. An implementer should never need to cross-reference other findings or the JSON summary to understand a single finding.

### Rule 5: Presentation must not replace content

Well-formatted output does not compensate for missing analysis. This contract defines how content is presented and structured, not what content is required. Methodology documents (repository analysis, simulation, scoring) define what must be produced.

### Rule 6: Findings Array Ordering

Findings in the JSON array are ordered by severity: Critical first, then High, Medium, Low. Within each severity level, order by dimension or logical grouping.

### Rule 7: JSON is the Source of Truth

The structured JSON (`metadata`, `summary`, `findings`) is authoritative. `report.content` is a generated rendering of that same data for human/archival consumption. See "Source of Truth" below for the full rule set.

## Source of Truth

This section is authoritative on the relationship between the structured JSON and the markdown report. It resolves any apparent ambiguity elsewhere in this document.

### The structured JSON is canonical

`metadata`, `summary`, and `findings` are the review's source of truth. Every fact the review asserts — the decision, the score, the confidence, every finding's evidence and recommendation — must exist in these three fields first. Nothing is true because the markdown says so; the markdown is true because the JSON says so.

### The markdown report is a rendering, not a second source

`report.content` is generated *from* `metadata`, `summary`, and `findings`. It exists for archival and human reading — someone browsing a repository of past reviews should be able to open the markdown and understand the review without needing the surrounding JSON. But it is a presentation of the same facts, not an independent one.

Concretely:

1. **No orphan facts in markdown.** If the markdown's Executive Summary says "the score is 92," `summary.score` must be `92`. If a Detailed Finding describes evidence at `src/foo.ts:12`, that exact evidence must appear in the corresponding `findings[].evidence`. The markdown must never contain a claim, number, file reference, or recommendation that cannot be traced back to `metadata`, `summary`, or `findings`.
2. **No missing facts in JSON.** Conversely, every finding discussed in the markdown's Detailed Findings section must have a corresponding entry in the `findings` array with the same `id`. A reviewer must not describe a finding in prose without also emitting its structured form.
3. **Counts must match.** `summary.critical`/`high`/`medium`/`low`/`totalFindings`/`blockingFindings` must match what is actually present in the `findings` array and what the markdown's Dimension Summary / Key Findings sections describe. If these disagree, the output is invalid regardless of which one is "more correct" — the review must be redone so they agree.
4. **The decision must be reproducible from the findings.** `summary.decision` is not an independent editorial judgment layered on top of the markdown narrative; it is the output of applying `references/decision-matrix.md` to the findings and scoring. If the markdown's Decision Rationale describes reasoning that would produce a different decision than `summary.decision`, the output is inconsistent and must be corrected before publication (see Phase 10, Consistency Validation, and the Quality Gates in `references/quality-gates.md`).

### Practical implication for generation order

Because the JSON is canonical, generate it in this order: compute `findings` first (Phases 5-6), compute `summary` from `findings` and the decision/scoring methodology (Phases 7-8), then generate `report.content` by rendering `metadata` + `summary` + `findings` into the markdown structure (Phase 9). Never draft the markdown narrative first and back-fill the JSON to match prose — that inverts the source of truth and is a form of the "Score-Driven Decisions" anti-pattern described in `references/decision-matrix.md`.

## JSON Schema

Every review returns a single JSON object with this structure:

```json
{
  "metadata": {
    "skill": "work-package-review",
    "version": "2.0",
    "schemaVersion": "2.1",
    "reviewId": "string (unique per review)",
    "reviewType": "Work Package Review",
    "reviewDate": "ISO-8601 timestamp",
    "repository": "string",
    "workPackage": "string"
  },
  "summary": {
    "decision": "PASS | FAIL | CONDITIONAL PASS",
    "score": 0-100,
    "scoreCeiling": 0-100,
    "confidence": "High | Medium | Low",
    "readiness": "Implementation Ready | Requires Changes",
    "implementationReady": true,
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0,
    "totalFindings": 0,
    "blockingFindings": 0
  },
  "findings": [
    {
      "id": "F-001",
      "severity": "Critical | High | Medium | Low",
      "status": "open",
      "title": "string",
      "dimension": "string",
      "description": "string",
      "evidence": "string",
      "impact": "string",
      "recommendation": "string"
    }
  ],
  "report": {
    "format": "markdown",
    "version": "2.0",
    "content": "string (complete markdown document)"
  }
}
```

## JSON Field Specifications

### metadata

All fields required. Provides provenance and unique identification for review reproduction and traceability.

| Field | Type | Description |
|---|---|---|
| skill | string | Fixed value: `"work-package-review"` |
| version | string | Methodology/skill version. Currently `"2.0"`. Increments only when the review methodology itself changes (phases, scoring, decision rules). |
| schemaVersion | string | JSON output contract version. Currently `"2.1"`. Increments whenever the JSON structure changes (fields added, removed, or retyped), independent of methodology changes. |
| reviewId | string | Unique identifier for this specific review execution. Must be unique across reviews of the same Work Package (e.g., re-reviews after revision). Recommended format: `{workPackageId}-{reviewDate-compact}`, e.g. `"WP-6-20260725T143000Z"`. Consumers should treat this as an opaque unique string, not parse it. |
| reviewType | string | The type of review performed. Fixed value `"Work Package Review"` for this skill. Present so downstream systems that aggregate multiple review types (Epic Review, ADR Review, etc.) can distinguish them without inspecting `skill`. |
| reviewDate | string | ISO 8601 timestamp (e.g., `"2026-07-25T14:30:00Z"`) |
| repository | string | Repository path or identifier being reviewed |
| workPackage | string | Work Package ID or title being reviewed |

### summary

Machine-readable summary for workflow automation (n8n Switch/IF nodes, GitHub comments, dashboards, metrics, quality gates). All fields required. Every field here is a **derived value** — it must be computable purely from `findings` and the decision methodology, never asserted independently of them (see "Source of Truth").

| Field | Type | Values | Description |
|---|---|---|---|
| decision | string enum | "PASS" \| "FAIL" \| "CONDITIONAL PASS" | Review decision. Derived per decision matrix in `references/decision-matrix.md`. |
| score | integer | 0-100 | Readiness score. Integer only, no decimals. Calculated per `references/review-scoring.md`. |
| scoreCeiling | integer | 0-100 | Simulation ceiling. Determined in Phase 4 (Implementation Readiness Assessment). Constrains the score. |
| confidence | string enum | "High" \| "Medium" \| "Low" | Reviewer confidence level. Determines whether additional investigation would change the decision. |
| readiness | string enum | "Implementation Ready" \| "Requires Changes" | Human-readable readiness label: "Implementation Ready" for PASS, "Requires Changes" for FAIL/CONDITIONAL PASS. |
| implementationReady | boolean | `true` \| `false` | Boolean equivalent of `readiness`, for workflows that route on booleans (n8n IF nodes) rather than string comparison. `true` if and only if `decision === "PASS"`. Always `false` for FAIL and CONDITIONAL PASS — CONDITIONAL PASS means *portions* may proceed, not that the Work Package as a whole is unconditionally ready; consumers needing partial-readiness detail must read `report.content`'s Implementation Guidance section. |
| critical | integer | ≥0 | Count of Critical-severity findings. |
| high | integer | ≥0 | Count of High-severity findings. |
| medium | integer | ≥0 | Count of Medium-severity findings. |
| low | integer | ≥0 | Count of Low-severity findings. |
| totalFindings | integer | ≥0 | `critical + high + medium + low`. Equal to `findings.length`. Provided so consumers do not need to sum the counts or count the array themselves. |
| blockingFindings | integer | ≥0 | `critical + high`. Count of findings severe enough to block or constrain implementation per the decision matrix (Critical always blocks; High blocks unless it carries clear remediation guidance). Provided for quality gates that need a single "is there blocking work" number without inspecting individual findings. |

**Derivation rules** (must hold for every review, always):
- `totalFindings === critical + high + medium + low === findings.length`
- `blockingFindings === critical + high`
- `implementationReady === (decision === "PASS")`
- `readiness === "Implementation Ready"` if and only if `implementationReady === true`

### findings[]

Array of all findings discovered during review. May be empty if no findings. All fields required for each finding.

| Field | Type | Description |
|---|---|---|
| id | string | Unique identifier: `F-NNN` (three-digit zero-padded). Assigned sequentially by severity: Critical first, then High, Medium, Low. |
| severity | string enum | "Critical" \| "High" \| "Medium" \| "Low". Determined per severity model in `references/implementation-readiness.md`. |
| status | string enum | Lifecycle status of the finding. At creation time, every finding is emitted with `"open"`. Reserved future values (not produced by this skill, but consumers may write them back into a persisted copy of the finding for tracking): `"accepted"` (acknowledged, will be fixed), `"fixed"` (remediation applied), `"rejected"` (reviewed and dismissed with justification), `"verified"` (fix confirmed in a subsequent review). The skill itself always emits `"open"` — it has no mechanism to observe remediation. Status transitions are the responsibility of the consuming workflow/system, not this skill. |
| title | string | One-line title, max 80 characters. Concise statement of the issue. |
| dimension | string | Evaluation dimension. Examples: "Specification Completeness", "Data Contract Completeness", "Integration Surface Completeness", "Error and Observability Completeness", "Architectural Fit", "Reuse Efficiency". Must match a defined dimension from review-process.md. |
| description | string | Detailed description of the finding. Explains the issue thoroughly. Must be self-contained. |
| evidence | string | Repository evidence backing the finding. Must include file paths and line numbers (e.g., `src/utils/validators.ts:45-47`). Code excerpts included when file is readable. |
| impact | string | Concrete consequence if the finding is not addressed. Describes what will actually happen (not just risk speculation). |
| recommendation | string | Specific, actionable recommendation. Must be detailed enough for implementer to act on without additional clarification. Not generic advice like "improve error handling" but specific steps. |

**Ordering rule**: Findings in the array are ordered by severity (Critical, High, Medium, Low). Within each severity level, order by dimension or logical grouping.

### report

A structured object wrapping the markdown document, rather than a bare string. This makes the format extensible: future versions may add alternative renderings (e.g. `"format": "html"`) without changing the shape of the `report` field or breaking existing consumers that read `report.content`.

| Field | Type | Description |
|---|---|---|
| format | string enum | Rendering format of `content`. Currently only `"markdown"` is produced. Reserved for future formats (e.g. `"html"`, `"plaintext"`) — consumers should branch on this field rather than assuming markdown. |
| version | string | Version of the markdown report *structure* (section ordering and mandatory sections), independent of `metadata.schemaVersion`. Currently `"2.0"`, matching the structure defined in "Markdown Report Structure" below. |
| content | string | The complete standalone markdown document. Must be understandable and actionable without reading the rest of the JSON object. Follows the structure defined in "Markdown Report Structure" section below. |

**Why a structured object instead of a bare string**: A bare string field cannot express what format it is in or what structural version it follows without out-of-band knowledge. Wrapping it in `{ format, version, content }` lets the contract evolve (new formats, new report structures) while `report` remains an object with a stable shape — old consumers reading `report.content` continue to work unmodified when `format`/`version` change.

### Markdown Report Structure

The `report.content` field contains a complete standalone markdown document. The markdown must be suitable for archival and understandable without reading the rest of the JSON. The markdown sections appear in this order:

| # | Section | Mandatory | Condition |
|---|---|---|---|
| 1 | Header | Yes | Always |
| 2 | Metadata | Yes | Always |
| 3 | Executive Summary | Yes | Always |
| 4 | Decision Block | Yes | Always |
| 5 | Review Context | Yes | Always |
| 6 | Key Findings | No | Only when meaningful findings exist |
| 7 | Detailed Findings | Yes | Always |
| 8 | Dimension Summary | No | Multi-dimension reviews |
| 9 | Decision Rationale | Yes | Always |
| 10 | Implementation Guidance | No | CONDITIONAL PASS only |
| 11 | Appendix | No | Supplementary material |

## Markdown Report Contents

The following sections describe the structure and content of the markdown report that appears in the `report.content` field. The report is a complete, standalone document suitable for archival. It follows the section ordering below. Every fact stated in this markdown must also be present in `metadata`, `summary`, or `findings` — the markdown is a rendering, not an independent source of information (see "Source of Truth").

### 1. Header

The report title identifies the review type and subject.

```
## [Review Type] Review: [Subject Title or ID]
```

**Mandatory**: Yes. Every report must have a header that identifies the review type (e.g., "Work Package Review", "Epic Review", "ADR Review") and the subject being reviewed.

### 2. Metadata

Metadata provides the provenance necessary to reproduce the review.

```
### Metadata
- **Reviewer**: [reviewer name or tool identifier]
- **Review ID**: [metadata.reviewId]
- **Review Type**: [Work Package Review | Epic Review | ADR Review | ...]
- **Subject**: [file path, ID, or link to the reviewed artifact]
- **Repository**: [repo path, branch, commit hash]
- **Date**: [review date ISO 8601]
```

**Mandatory**: Yes. The Review ID line must echo `metadata.reviewId` exactly — it is the same identifier, not a separately-generated one.

**Additional fields**: Review types may extend the metadata block with review-type-specific fields. For example, a Work Package Review may add `Simulation Ceiling` and `Finding Count`. Extended fields must be documented in the review type's methodology document. Any additional field in the markdown metadata block must correspond to a field that exists in the JSON `metadata` or be derivable from `summary`/`findings` — see "Source of Truth".

### 3. Executive Summary

A concise statement of the review outcome and the most important takeaway.

```
### Executive Summary

[One to three paragraphs communicating the key result. Must state the decision
(PASS / FAIL / CONDITIONAL PASS), the overall readiness assessment, and the
most critical issue if any. Should be understandable to someone who reads
only this section.]
```

**Mandatory**: Yes.

**Guidelines**:
- Must be comprehensible without reading the rest of the report.
- Must state the decision explicitly in the first sentence.
- Should identify the single most important finding or risk.
- Should be no more than three paragraphs.

### 4. Decision Block

The decision block provides a compact, scannable summary of the outcome.

```
### Decision
- **Readiness Score**: [0-100]
- **Decision**: [PASS | FAIL | CONDITIONAL PASS]
- **Confidence**: [High | Medium | Low]
```

**Mandatory**: Yes.

**Additional fields**: Review types may add fields to the decision block. Common additions include:
- `Simulation Ceiling`: The ceiling determined by simulation (Work Package Review).
- `Finding Count`: Count of findings by severity, e.g., "2 Critical, 3 High, 1 Medium, 0 Low".

**Multi-review-type note**: Different review types may use different score labels. The score field name should reflect the review type's scoring model (e.g., "Readiness Score" for Work Package Review, "Quality Score" for Code Review). Consistency within a review type is mandatory.

### 5. Review Context

Summarizes the context required to understand the review findings.

```
### Review Context

[Brief summary of the analysis that produced this review: the artifact's scope,
the key areas examined, and the most relevant existing context. References
specific files, modules, or architectural boundaries as applicable.]
```

**Mandatory**: Yes.

**Guidelines**:
- Summarize what was examined, not how. Methodology details belong in methodology documents.
- Reference the specific artifacts, files, or systems analyzed.
- Should be brief — three to six sentences.

### 6. Key Findings (Optional)

A prioritized list of the most significant findings. Appears only when the review has findings that warrant immediate attention.

```
### Key Findings

1. **Critical**: [Finding title] — [One-sentence summary of the issue]
2. **High**: [Finding title] — [One-sentence summary of the issue]
```

**Guidelines**:
- Include only Critical and High findings.
- No more than five items. If there are more than five significant findings, group them thematically.
- Each item is a single sentence. Full detail appears in Detailed Findings.
- Omit this section entirely if there are no Critical or High findings, or if the reviewer judges the findings are best understood in full detail without prioritization.

### 7. Detailed Findings

Every finding is presented in full. Findings must be ordered by severity (Critical first, then High, Medium, Low) and within each severity by dimension or logical grouping.

```
#### [F-NNN] [Severity] Finding Title
- **Dimension**: [Dimension name]
- **Evidence**: [Excerpt from the reviewed artifact or repository with file path and line number]
- **Engineering Principle**: [Which principle from references/review-principles.md is violated]
- **Problem**: [Why this is a problem for implementation or correctness]
- **Impact**: [What will happen if this is not addressed]
- **Recommendation**: [Exactly what needs to change]
```

**Mandatory**: Yes.

**Finding identifier format**: `F-NNN` where NNN is a zero-padded three-digit number (F-001, F-002, ..., F-999). Identifiers are assigned sequentially by severity: Critical findings first, then High, Medium, Low.

**Severity label**: Must be one of: Critical, High, Medium, Low. Severity is determined per the severity model in `references/implementation-readiness.md` for Work Package Reviews, or the equivalent severity model for other review types.

**Per-finding rules**:
- **Evidence** must include file paths and line numbers. Code excerpts must be included inline when the file is accessible. When citing a convention, provide at least three example locations.
- **Engineering Principle** must reference a specific principle from `references/review-principles.md` (or the review type's equivalent principles document). Generic references to "good engineering practice" are not acceptable.
- **Problem** must explain why the finding matters in engineering terms. Not "this is wrong" but "this is wrong because..."
- **Impact** must describe a concrete consequence. Not "this could cause issues" but "this will cause incorrect behavior when..."
- **Recommendation** must be specific enough for an implementer to act on without additional clarification. Not "fix the validation" but "use `isValidEmail` from `src/utils/validators.ts:47` instead of creating a new utility."

### 8. Dimension Summary (Conditional)

A table showing the status of each evaluation dimension. Required for reviews that evaluate across multiple dimensions (e.g., Work Package Review).

```
### Dimension Summary
| Dimension | Finding Count | Status |
|---|---|---|
| Specification Completeness | N | [OK | ISSUES | BLOCKED] |
```

**Status values**:
- **OK**: No findings in this dimension.
- **ISSUES**: One or more Medium or Low findings.
- **BLOCKED**: One or more Critical or High findings.

**Mandatory**: For multi-dimension reviews. Optional for single-dimension reviews (e.g., a focused code review on a single concern).

### 9. Decision Rationale

Explains how the decision was reached, including the most important factors that influenced it.

```
### Decision Rationale

[Explanation of why the PASS/FAIL/CONDITIONAL PASS decision was reached.
References the score, ceiling, severity-weighted findings, and any
confidence-limiting factors. Should answer: "Given these findings, why this
decision?"]
```

**Mandatory**: Yes.

**Guidelines**:
- Reference the simulation ceiling and score (for scored reviews).
- Identify which findings most influenced the decision.
- Explain confidence limitations: what would need to change for confidence to increase.
- Acknowledge any assumptions made during the review.
- Do not restate findings. The rationale synthesizes, not repeats.

### 10. Implementation Guidance (Conditional)

Provides actionable guidance for the implementer. Required only for CONDITIONAL PASS decisions.

```
### Implementation Guidance

1. **Can proceed immediately**: [what the implementer can work on now]
2. **Requires clarification**: [what needs resolution before reaching that code]
3. **Recommended order**: [suggested implementation sequence]
```

**Mandatory**: Yes, for CONDITIONAL PASS. Optional for PASS (may note suggested improvements). Not applicable for FAIL.

**Guidelines**:
- Distinguish between blocked and unblocked work clearly.
- Be specific about what the implementer should do when blocked: "Contact the database team to confirm the migration plan" not "resolve the schema issue."
- Provide a recommended order when multiple items need resolution.

### 11. Appendix (Optional)

Supplementary details that are not essential for understanding the decision.

```
### Appendix: [Title]
[Optional supplementary content.]
```

**Common appendix types**: Repository analysis notes, search queries used, methodology exceptions, glossary of project-specific terms, references to external documentation.

## Evidence Presentation Standards

Evidence presentation follows the traceability requirements defined in `references/evidence-and-justification.md`. Every evidence citation must include:

1. **File path** (absolute or relative to repository root).
2. **Line number or range**.
3. **Relevant excerpt** (sufficient context to verify without opening the file).

**Acceptable**:
```
src/utils/validators.ts:45 - exports function `isValidEmail(email: string): boolean`
```

**Not acceptable**:
```
The validators file has an email function.
```

## Score Presentation

Scores are presented in the Decision Block (section 4) and discussed in the Decision Rationale (section 9).

### Score Format

Scores appear as integers in the range [0, 100]. Decimal points are not used.

**Acceptable**: `76`
**Not acceptable**: `75.8`, `about 75`, `high seventies`

### Score Field Name

The field name in the Decision Block must reflect the scoring model used:

- Work Package Review: `Readiness Score`
- Other review types: use a label that matches the scored concept

### Score and Ceiling

When the scoring model uses a ceiling (see `references/review-scoring.md`), the ceiling must be displayed alongside the score:

```
- **Readiness Score**: 46
- **Simulation Ceiling**: 69
```

## Confidence Presentation

Confidence is presented in the Decision Block (section 4) and explained in the Decision Rationale (section 9).

### Confidence Labels

Exactly three values are permitted: High, Medium, Low.

### Confidence in Rationale

The Decision Rationale must explain what would change for confidence to increase. For example:

```
Confidence is Medium because the repository analysis covered the primary impact zone
but did not fully explore the notification module's test infrastructure. Full exploration
would raise confidence to High.
```

Report consistency methodology — including internal consistency and cross-report consistency — is defined in `references/review-consistency.md`. All report consistency concepts are owned by that document.

Report completeness validation — including mandatory sections, finding structure, evidence citations, and decision presentation — is defined in the Output Contract Gate of `references/quality-gates.md`. All output validation concepts are owned by that document.

## Report Readability

### Ordering

The section order defined in this document is mandatory. It follows the implementer's information consumption pattern:

1. **Metadata**: Who, what, when, where.
2. **Executive Summary**: The headline result.
3. **Decision Block**: The compact verdict.
4. **Review Context**: What was examined.
5. **Key Findings**: What matters most.
6. **Detailed Findings**: The complete evidence.
7. **Dimension Summary**: How each area scored.
8. **Decision Rationale**: Why this conclusion.
9. **Implementation Guidance**: What to do next.
10. **Appendix**: Supporting detail.

### Brevity

- Executive Summary: three paragraphs maximum.
- Review Context: six sentences maximum.
- Decision Rationale: five paragraphs maximum.
- Finding titles: one line, 80 characters maximum.
- Finding evidence excerpts: ten lines of code maximum unless the entire function is needed.

### Language

- Use active voice: "The Work Package omits error handling" not "Error handling has been omitted by the Work Package."
- Use specific language: "src/services/order.ts:45" not "the order service file."
- Avoid subjective descriptors: "incorrect", "wrong", "bad". Use engineering terms: "contradicts the existing schema", "creates a duplicate validation path", "introduces a circular dependency."

## Review Type Extensions

Different review types may extend this contract with review-type-specific sections or fields. The following rules govern extensions:

1. **Mandatory sections cannot be removed.** All review types must include Header, Metadata, Executive Summary, Decision Block, Review Context, Detailed Findings, and Decision Rationale.
2. **Section ordering cannot be changed.** Review-type-specific sections may be inserted between existing sections but must not reorder the contract-defined sections.
3. **Additional metadata fields** are permitted in the Metadata section.
4. **Additional decision fields** are permitted in the Decision Block.
5. **Review-type-specific sections** may be added to the Appendix or between Review Context and Key Findings.
6. **Severity labels** are not part of this contract. Each review type defines its own severity model.

## Backward Compatibility & Migration

### Version 2.0 → 2.1 Refinement (Non-Breaking Additive, Except `report`)

**Version 2.0 Output Format** (Previous):
- `report` was a bare string containing markdown.
- `metadata` had five fields: `skill`, `version`, `reviewDate`, `repository`, `workPackage`.
- `summary` had eight fields: `decision`, `score`, `scoreCeiling`, `confidence`, `readiness`, `critical`, `high`, `medium`, `low`.
- `findings[]` had no lifecycle field.

**Version 2.1 Output Format** (Current):
- `report` is now an object: `{ format, version, content }`. **This is a breaking change** for any consumer that read `report` as a string — they must switch to `report.content`.
- `metadata` gains `schemaVersion`, `reviewId`, `reviewType` (additive — existing fields unchanged).
- `summary` gains `implementationReady`, `totalFindings`, `blockingFindings` (additive — existing fields unchanged).
- `findings[]` gains `status`, always emitted as `"open"` (additive — existing fields unchanged).

**Migration Impact**:
- Any workflow reading `review.report` directly as a markdown string must change to `review.report.content`.
- All other 2.0 field paths (`summary.decision`, `summary.score`, `findings[].evidence`, etc.) are unchanged and require no consumer updates.
- New fields can be adopted incrementally — a consumer that ignores `implementationReady`, `blockingFindings`, `totalFindings`, `schemaVersion`, `reviewId`, `reviewType`, and `findings[].status` continues to function exactly as it did under 2.0, aside from the `report` change above.

**Rationale**:
The `report` string was changed to an object so the contract can support additional report formats (e.g., HTML) in the future without changing the shape of the `report` field again — old consumers reading `report.content` will not need to change when a new format is introduced. The metadata and summary additions make reviews uniquely identifiable (`reviewId`) and let n8n workflows route on booleans and pre-summed counts (`implementationReady`, `blockingFindings`, `totalFindings`) instead of recomputing them from the `findings` array on every run. The `status` field on findings lays the groundwork for future finding lifecycle tracking (accepted/fixed/rejected/verified) without requiring another schema revision when that capability is built.

### Version 1.x → 2.0 Breaking Change

**Version 1.x Output Format** (Deprecated):
- Plain markdown only
- No machine-readable structure
- No JSON output
- Markdown sections as top-level output

**Version 2.0 Output Format** (Superseded by 2.1, described above):
- Single JSON object (required)
- `metadata`, `summary`, `findings`, `report` top-level fields
- `report` contained the markdown document directly as a string
- Machine-readable summary for automation
- Structured findings array for dashboards and metrics

**Migration Impact**:
- Workflows that parse the old markdown output must be updated
- n8n workflows should use the `summary` object for decisions (not markdown parsing)
- Findings are now accessible via the structured `findings` array
- The markdown report remains suitable for archival and human reading (now at `report.content` as of 2.1)

**Rationale**:
Version 1.x returned plain markdown, making it difficult for downstream automation (n8n Switch nodes, GitHub comments, dashboards) to extract and act on review results without parsing markdown. Version 2.0 separates concerns: the `summary` object contains all machine-readable information for automation, while the `report` provides a human-readable, archival-suitable markdown document.

## Contract Evolution

This output contract is versioned independently along two axes, both tracked in `metadata`:

- `metadata.version` — the review **methodology** version (phases, scoring, decision rules). Changes when `SKILL.md`'s methodology changes.
- `metadata.schemaVersion` — the **JSON contract** version (this document). Changes when fields are added, removed, retyped, or restructured, independent of methodology changes.

Decoupling these means a schema refinement (like 2.0 → 2.1) does not force a methodology re-validation, and a methodology change does not force every consumer to re-parse the JSON differently.

Changes to the output contract must:

1. Be documented in the "Backward Compatibility & Migration" section above with the breaking-vs-additive classification and rationale.
2. Document the exact migration path for workflows consuming the previous schema version, including which field paths changed and which are untouched.
3. Update all methodology documents that reference the output structure (`prompt.md`, `SKILL.md`, `OUTPUT-FORMAT.md`, `MIGRATION.md`, and all `examples-output-*.json` files).
4. Increment `schemaVersion` and, if applicable, `report.version`.

### Future Changes

Any further changes to the JSON schema or markdown structure must maintain:
- All four top-level JSON fields (`metadata`, `summary`, `findings`, `report`) must always exist.
- The `findings` array structure must remain consistent; new fields on a finding are additive, existing fields are not removed or retyped without a `schemaVersion` bump and a documented migration.
- The `report` object's `content` must remain a complete, standalone, archival-suitable rendering of `metadata` + `summary` + `findings`, per "Source of Truth" above. New `format` values may be added; `content` must not become the only place a fact is asserted.
- `summary` fields that are derived (`totalFindings`, `blockingFindings`, `implementationReady`) must remain mechanically derivable from `findings` and `decision` — never independently asserted values that could drift out of sync.
- `metadata.schemaVersion` must be incremented for any structural change to this schema.
