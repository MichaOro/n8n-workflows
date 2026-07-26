---
name: work-package-review-output-format
description: Documentation for the JSON output format of work-package-review skill. Explains the structure, fields, and how to consume the output in n8n workflows and automation systems.
---

# Work Package Review — Output Format Documentation

## Overview

The work-package-review skill returns deterministic JSON output designed for both human reading (via the included markdown report) and machine processing (via the structured JSON object).

**Schema version**: 2.1 (refined 2026-07-25; see `MIGRATION.md` for the 2.0 → 2.1 changelog). **Methodology version**: 2.0 (unchanged — this refinement only touched the output contract, not the review methodology).

The JSON is the **canonical source of truth**. The markdown inside `report.content` is generated from `metadata`, `summary`, and `findings` — it is a rendering for human/archival consumption, not an independent source of facts. See `references/review-output-contract.md` § "Source of Truth" for the full rule set.

## Output Structure

Every review returns a single valid JSON object with four top-level fields:

```json
{
  "metadata": { ... },
  "summary": { ... },
  "findings": [ ... ],
  "report": { "format": "markdown", "version": "2.0", "content": "..." }
}
```

## Field Reference

### metadata

Contains provenance information and unique identifiers for review reproduction and traceability.

```json
{
  "metadata": {
    "skill": "work-package-review",
    "version": "2.0",
    "schemaVersion": "2.1",
    "reviewId": "WP-6-20260725T143000Z",
    "reviewType": "Work Package Review",
    "reviewDate": "2026-07-25T14:30:00Z",
    "repository": "C:\\Users\\micha\\repos\\n8n-workflows",
    "workPackage": "WP-6 — Cross-Platform sync-from-git Workflow"
  }
}
```

| Field | Type | Description |
|---|---|---|
| skill | string | Always `"work-package-review"` |
| version | string | Review **methodology** version. Currently `"2.0"`. Changes only when the review phases/scoring/decision rules change. |
| schemaVersion | string | JSON **output contract** version. Currently `"2.1"`. Changes whenever this JSON structure changes, independent of methodology. |
| reviewId | string | Unique identifier for this specific review execution (e.g. re-reviewing WP-6 after revision produces a different `reviewId`). Treat as an opaque unique string — do not parse it. |
| reviewType | string | The review type performed. Fixed value `"Work Package Review"` for this skill. Lets systems aggregating multiple review types (Epic, ADR, etc.) distinguish them without inspecting `skill`. |
| reviewDate | string | ISO 8601 timestamp when the review was completed. |
| repository | string | Repository path or identifier being reviewed. |
| workPackage | string | Work Package ID or title being reviewed. |

### summary

Machine-readable summary designed for workflow automation (n8n Switch/IF nodes, GitHub comments, dashboards, metrics, quality gates). Every field here is **derived** from `findings` and the decision methodology — never an independently-asserted value.

```json
{
  "summary": {
    "decision": "PASS",
    "score": 92,
    "scoreCeiling": 100,
    "confidence": "High",
    "readiness": "Implementation Ready",
    "implementationReady": true,
    "critical": 0,
    "high": 0,
    "medium": 1,
    "low": 0,
    "totalFindings": 1,
    "blockingFindings": 0
  }
}
```

| Field | Type | Values | Description |
|---|---|---|---|
| decision | string | "PASS" / "FAIL" / "CONDITIONAL PASS" | Review decision. Use this in Switch nodes and decision logic. |
| score | integer | 0-100 | Readiness score. Integer only, no decimals. |
| scoreCeiling | integer | 0-100 | Simulation ceiling. Constrains the score. Indicates the maximum confidence possible given discovered gaps. |
| confidence | string | "High" / "Medium" / "Low" | Confidence level. High = additional investigation unlikely to change decision. Low = more investigation recommended. |
| readiness | string | "Implementation Ready" / "Requires Changes" | Human-readable label. Maps from decision: PASS → "Implementation Ready", FAIL/CONDITIONAL PASS → "Requires Changes". |
| implementationReady | boolean | `true` / `false` | Boolean equivalent of `readiness`, for IF-node/boolean routing instead of string comparison. `true` iff `decision === "PASS"`. Note: `false` for CONDITIONAL PASS too — that decision means *portions* may proceed, which a single boolean cannot express. Read `report.content`'s Implementation Guidance section for the partial-readiness detail. |
| critical | integer | ≥0 | Count of Critical-severity findings. Typically indicates blocking issues. |
| high | integer | ≥0 | Count of High-severity findings. Typically indicates significant issues requiring resolution. |
| medium | integer | ≥0 | Count of Medium-severity findings. Typically indicates improvements needed before implementation. |
| low | integer | ≥0 | Count of Low-severity findings. Typically indicates suggestions or minor issues. |
| totalFindings | integer | ≥0 | `critical + high + medium + low`, equal to `findings.length`. Saves the workflow from summing the counts itself. |
| blockingFindings | integer | ≥0 | `critical + high`. The subset of findings severe enough to block or constrain implementation. Use this for a single "is there blocking work" quality-gate check without inspecting individual findings. |

**These identities always hold** (see `references/review-output-contract.md` § "Source of Truth"):
```
totalFindings    === critical + high + medium + low === findings.length
blockingFindings === critical + high
implementationReady === (decision === "PASS")
readiness === "Implementation Ready"  <=>  implementationReady === true
```

### findings

Array of all findings discovered during review. May be empty if no issues were found — the field itself is always present.

```json
{
  "findings": [
    {
      "id": "F-001",
      "severity": "Critical",
      "status": "open",
      "title": "Primary design relies on unavailable child_process module",
      "dimension": "Architectural Fit",
      "description": "The Work Package proposes using child_process.execSync() in n8n Code nodes...",
      "evidence": "sync-from-git.json:17 — Current executeCommand node runs git commands...",
      "impact": "Implementer will follow the WP's primary design, write code expecting child_process...",
      "recommendation": "Replace the primary target design with the alternative approach..."
    }
  ]
}
```

Each finding object contains:

| Field | Type | Description |
|---|---|---|
| id | string | Unique identifier: `F-NNN` (zero-padded). Assigned sequentially by severity. |
| severity | string | "Critical" / "High" / "Medium" / "Low". Indicates impact severity. |
| status | string | Lifecycle status. This skill always emits `"open"` — it has no way to observe subsequent remediation. Reserved values for downstream tracking systems that persist findings across reviews: `"accepted"`, `"fixed"`, `"rejected"`, `"verified"`. See "Finding Lifecycle" below. |
| title | string | One-line title (max 80 chars). Concise summary of the issue. |
| dimension | string | Evaluation dimension. Examples: "Specification Completeness", "Architectural Fit", "Data Contract Completeness", etc. Use for categorizing issues. |
| description | string | Detailed explanation of the finding. Why it's a problem. Context and scope. |
| evidence | string | Repository evidence with file paths and line numbers. Specific enough to verify without opening files. |
| impact | string | Concrete consequence if not addressed. What will actually happen (not speculation). |
| recommendation | string | Specific, actionable recommendation. Detailed enough for implementer to act on without clarification. |

**Ordering**: Findings are ordered by severity (Critical first, then High, Medium, Low). Within each severity, they are ordered by dimension or logical grouping.

#### Finding Lifecycle

`status` starts as `"open"` on every finding this skill emits. It exists so a system that persists findings across multiple reviews (a dashboard, a tracker, a re-review pipeline) has a field to write state transitions into, without needing to invent one:

```
open ──> accepted ──> fixed ──> verified
  │                       │
  └──────> rejected <─────┘
```

This skill does not perform these transitions itself — a fresh review of a revised Work Package produces a fresh set of findings with fresh `id`s and `status: "open"`, it does not update the status of findings from a prior review. Tracking a finding's lifecycle across reviews (e.g., "F-001 from the 2026-07-25 review was fixed by 2026-08-01") is the responsibility of whatever system persists a copy of the finding — match findings across reviews by `title` + `dimension` + `evidence` similarity, since `id` is only unique within a single review's `findings` array, not across reviews.

### report

A structured object wrapping the markdown document. Suitable for archival, human reading, and standalone understanding.

```json
{
  "report": {
    "format": "markdown",
    "version": "2.0",
    "content": "## Work Package Review: WP-6 — Cross-Platform sync-from-git Workflow\n\n### Metadata\n..."
  }
}
```

| Field | Type | Description |
|---|---|---|
| format | string | Rendering format of `content`. Currently only `"markdown"`. Reserved for future formats (e.g. `"html"`) — branch on this field rather than assuming markdown. |
| version | string | Version of the markdown report's structure (section ordering, mandatory sections). Currently `"2.0"`, independent of `metadata.schemaVersion`. |
| content | string | The complete standalone markdown document. |

**Why an object instead of a bare string**: wrapping the markdown in `{ format, version, content }` lets the contract add new report formats later without changing what shape the `report` field has — code that reads `report.content` today keeps working even if a future version adds `report.format: "html"` alongside it.

`report.content` contains:

1. **Header** — Review title and subject
2. **Metadata** — Reviewer, Review ID, date, repository, simulation ceiling
3. **Executive Summary** — 1-3 paragraphs with decision, readiness assessment, key issue
4. **Decision Block** — Score, ceiling, decision, confidence, finding counts
5. **Review Context** — What was examined, artifacts analyzed
6. **Key Findings** (optional) — Prioritized list of Critical/High findings
7. **Detailed Findings** — Complete findings with all justification chain links
8. **Dimension Summary** (optional) — Table of findings per evaluation dimension
9. **Decision Rationale** — Why the decision was reached given the findings
10. **Implementation Guidance** (CONDITIONAL PASS only) — Blocked work, unblocked work, recommended order
11. **Appendix** (optional) — Supplementary details

`report.content` is a complete, standalone document. It should be understandable and actionable without reading the rest of the JSON object — but every fact in it must be traceable to `metadata`, `summary`, or `findings`. It is generated from that data, not an independent source (see "Source of Truth" in `references/review-output-contract.md`).

## Consumption Patterns

### Pattern 1: Decision in n8n Switch Node

Use the `decision` field to route workflows:

```javascript
// n8n Switch node
switch ($input.first().json.summary.decision) {
  case "PASS":
    // Route to "proceed with implementation"
    break;
  case "CONDITIONAL PASS":
    // Route to "send for clarification"
    break;
  case "FAIL":
    // Route to "request revision"
    break;
}
```

Or, for a simple two-way IF node instead of a three-way Switch, use the boolean directly:

```javascript
// n8n IF node condition
$input.first().json.summary.implementationReady === true
// true  -> proceed
// false -> covers both FAIL and CONDITIONAL PASS; inspect `decision` downstream if you need to split them
```

### Pattern 2: GitHub Comment

Use the `summary` object to automatically comment on GitHub PRs:

```javascript
const { summary, metadata } = $input.first().json;
const comment = `## Work Package Review Result

Decision: **${summary.decision}** (Score: ${summary.score}/${summary.scoreCeiling})

Findings: ${summary.critical} critical, ${summary.high} high, ${summary.medium} medium, ${summary.low} low

Confidence: ${summary.confidence}

[View full review report](#)`;
```

### Pattern 3: Dashboard Display

Use summary + findings for dashboard visualization. `totalFindings` and `blockingFindings` mean the dashboard doesn't need to compute them:

```javascript
const review = $input.first().json;
return {
  status: review.summary.decision,
  score: review.summary.score,
  ceiling: review.summary.scoreCeiling,
  totalFindings: review.summary.totalFindings,
  blockingFindings: review.summary.blockingFindings,
  issues: {
    critical: review.summary.critical,
    high: review.summary.high,
    medium: review.summary.medium,
    low: review.summary.low
  },
  topIssues: review.findings.slice(0, 3).map(f => ({
    title: f.title,
    severity: f.severity,
    dimension: f.dimension,
    status: f.status
  }))
};
```

### Pattern 4: Automated Issue Creation

Create GitHub Issues from findings, tagging each with its lifecycle `status`:

```javascript
const review = $input.first().json;
const criticalFindings = review.findings.filter(f => f.severity === "Critical");

for (const finding of criticalFindings) {
  // Create GitHub issue with finding details
  const issue = {
    title: `[REVIEW] ${finding.title}`,
    body: `### ${finding.dimension}\n\n${finding.description}\n\n**Impact**: ${finding.impact}\n\n**Recommendation**: ${finding.recommendation}\n\n_Review: ${review.metadata.reviewId}_`,
    labels: ["review-finding", finding.severity.toLowerCase(), `status:${finding.status}`]
  };
}
```

When the issue is later closed, the tracking system should update its own copy of the finding's `status` (e.g. to `"fixed"`) — this skill does not do that automatically; see "Finding Lifecycle" above.

### Pattern 5: Trend Analysis

Track review metrics over time, keyed by the unique `reviewId`:

```javascript
const review = $input.first().json;
return {
  reviewId: review.metadata.reviewId,
  timestamp: review.metadata.reviewDate,
  workPackage: review.metadata.workPackage,
  decision: review.summary.decision,
  implementationReady: review.summary.implementationReady,
  score: review.summary.score,
  findingCount: review.summary.totalFindings,
  blockingCount: review.summary.blockingFindings,
  criticalCount: review.summary.critical
};
```

## Migration from Version 2.0 (report string → object)

If you built a workflow against the 2.0 schema (released 2026-07-25 earlier the same day as this 2.1 refinement), there is exactly one breaking change to apply:

```javascript
// 2.0: report was a bare string
const markdown = review.report;

// 2.1: report is an object; the markdown is now at .content
const markdown = review.report.content;
```

Every other 2.0 field path (`summary.decision`, `summary.score`, `findings[].evidence`, etc.) is unchanged. The new fields (`metadata.schemaVersion`, `metadata.reviewId`, `metadata.reviewType`, `summary.implementationReady`, `summary.totalFindings`, `summary.blockingFindings`, `findings[].status`) are additive — ignoring them costs nothing. See `MIGRATION.md` for the full 2.0 → 2.1 changelog.

## Migration from Version 1.x

### What Changed

**Version 1.x** (Old):
- Output was plain markdown only
- No machine-readable structure
- Workflows had to parse markdown to extract decisions
- No structured findings

**Version 2.1** (Current):
- Output is JSON with embedded markdown
- `summary` object for automation, including derived fields (`implementationReady`, `totalFindings`, `blockingFindings`)
- `findings` array for metrics and dashboards, each with a lifecycle `status`
- `report` is a structured object (`{ format, version, content }`); the markdown is at `report.content`
- `metadata` includes a unique `reviewId` and both a methodology `version` and a `schemaVersion`

### Migration Steps

1. **Update n8n workflows** that depend on the old markdown output:
   - Replace markdown parsing with JSON field access
   - Use `$.summary.decision` instead of parsing the markdown "PASS"/"FAIL" text
   - Use `$.summary.score` instead of extracting from markdown "Score: 92"

2. **Example: Old pattern** (parsing markdown):
   ```javascript
   const markdown = inputData.review;
   const passMatch = markdown.match(/Decision:\s*(\w+)/);
   const decision = passMatch ? passMatch[1] : "UNKNOWN";
   ```

3. **Example: New pattern** (JSON):
   ```javascript
   const decision = inputData.summary.decision; // "PASS" | "FAIL" | "CONDITIONAL PASS"
   const markdown = inputData.report.content;   // note: report is an object as of 2.1, not a string
   ```

### Backward Compatibility

Version 2.x is a breaking change relative to 1.x. Workflows consuming Version 1.x output will not work with Version 2.x without modification. No automatic conversion is provided.

If you need to maintain support for both 1.x and 2.x during a transition period:

```javascript
const output = $input.first().json;
const isNewFormat = output.metadata && output.metadata.version === "2.0";

let decision;
if (isNewFormat) {
  decision = output.summary.decision;
} else {
  // Old markdown parsing logic
  decision = parseDecisionFromMarkdown(output);
}
```

If you're already on 2.0 and only need to handle the 2.0 → 2.1 `report` shape change, check `metadata.schemaVersion` instead:

```javascript
const output = $input.first().json;
const markdown = output.metadata?.schemaVersion === "2.1"
  ? output.report.content   // 2.1: object
  : output.report;          // 2.0: bare string
```

## Examples

See the examples directory in the skill folder. Names indicate the scenario each demonstrates:

- `examples-output-pass-no-findings.json` — PASS decision, empty `findings` array
- `examples-output-pass-low-findings.json` — PASS decision with only Low-severity, non-blocking findings
- `examples-output-fail-high-finding.json` — FAIL decision driven by a single High-severity finding with no independent remediation path (no Critical finding present)
- `examples-output-fail-multiple-findings.json` — FAIL decision with a Critical finding (score forced to 0) plus High and Medium findings
- `examples-output-conditional-pass.json` — CONDITIONAL PASS decision (High + Medium findings, no Critical, all with clear remediation guidance) including the Implementation Guidance section

Each example is schema-valid: `summary.totalFindings`/`blockingFindings` match the `findings` array exactly, `report.content` describes only facts present in the structured data, and every finding has `status: "open"`.

## Determinism & Stability

The JSON schema is deterministic:

- All four top-level fields always exist (never omitted)
- `findings` array may be empty, but the field is always present
- `report` is always an object with `format`, `version`, `content` — never a bare string
- Field types are consistent: `decision` is always a string enum, `score` is always an integer, etc.
- Field ordering is stable (no random ordering of findings)
- Findings are always ordered by severity (Critical, High, Medium, Low)
- Derived `summary` fields (`totalFindings`, `blockingFindings`, `implementationReady`) always match their formula — see the identities listed under "summary" above

This stability makes it safe to write downstream automation that expects the structure to remain consistent across reviews.

## Documentation References

- `references/review-output-contract.md` — Full output contract specification
- `prompt.md` — Execution orchestration prompt
- `SKILL.md` — Skill overview and methodology phases
