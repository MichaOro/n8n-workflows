---
name: work-package-review-migration
description: Migration guide for work-package-review output format changes. Covers Version 2.0 to 2.1 (report string → structured object, additive metadata/summary/findings fields) and the earlier Version 1.x to 2.0 migration (markdown-only to JSON output).
---

# Work Package Review — Migration Guide

This document covers two migrations:

1. **Version 2.0 → 2.1** (current) — an additive refinement, with one breaking change to the `report` field. Most consumers need only this section.
2. **Version 1.x → 2.0** (historical) — the original move from plain markdown to JSON. Kept below for reference if you are migrating a workflow that never made it past Version 1.x.

---

# Part 1: Version 2.0 → 2.1 Migration Guide

## Summary of Changes

| Aspect | Version 2.0 | Version 2.1 |
|---|---|---|
| **`report` field** | Bare string containing markdown | Object: `{ format, version, content }` — markdown is now at `.content` |
| **`metadata` fields** | `skill`, `version`, `reviewDate`, `repository`, `workPackage` | Adds `schemaVersion`, `reviewId`, `reviewType` |
| **`summary` fields** | `decision`, `score`, `scoreCeiling`, `confidence`, `readiness`, `critical`, `high`, `medium`, `low` | Adds `implementationReady`, `totalFindings`, `blockingFindings` |
| **`findings[]` fields** | `id`, `severity`, `title`, `dimension`, `description`, `evidence`, `impact`, `recommendation` | Adds `status` (always `"open"` at emission time) |
| **Breaking Change** | — | Only `report` — everything else is additive |

## The One Breaking Change: `report`

**Version 2.0**:
```json
{
  "report": "## Work Package Review: WP-6 ...\n\n### Metadata\n..."
}
```

**Version 2.1**:
```json
{
  "report": {
    "format": "markdown",
    "version": "2.0",
    "content": "## Work Package Review: WP-6 ...\n\n### Metadata\n..."
  }
}
```

**Fix required in any 2.0-era workflow**:
```javascript
// Before (2.0)
const markdown = review.report;

// After (2.1)
const markdown = review.report.content;
```

That is the entire required change. Every other field path from 2.0 (`summary.decision`, `summary.score`, `findings[].evidence`, `metadata.workPackage`, etc.) is untouched.

## The Additive Changes (No Action Required, but Available)

### metadata.schemaVersion, metadata.reviewId, metadata.reviewType

```json
{
  "metadata": {
    "skill": "work-package-review",
    "version": "2.0",
    "schemaVersion": "2.1",
    "reviewId": "WP-6-20260725T143000Z",
    "reviewType": "Work Package Review",
    "reviewDate": "2026-07-25T14:30:00Z",
    "repository": "...",
    "workPackage": "..."
  }
}
```

- `schemaVersion` lets a workflow branch on the JSON contract version independent of `version` (the methodology version, unchanged at `"2.0"`). Use this to detect the `report` shape change: `metadata.schemaVersion === "2.1"` means `report` is an object.
- `reviewId` gives every review a stable, unique key — useful for trend-analysis pipelines and for de-duplicating re-runs.
- `reviewType` is fixed to `"Work Package Review"` for this skill; present for systems that aggregate multiple review types.

### summary.implementationReady, summary.totalFindings, summary.blockingFindings

```json
{
  "summary": {
    "decision": "PASS",
    "implementationReady": true,
    "totalFindings": 0,
    "blockingFindings": 0
  }
}
```

- `implementationReady` — boolean equivalent of `decision === "PASS"`. Use for IF-node routing instead of a string switch, if you prefer.
- `totalFindings` — `critical + high + medium + low`, so you don't have to sum them or count the array.
- `blockingFindings` — `critical + high`, a single number for a "does this block implementation" quality gate.

None of these require workflow changes to adopt — they're new fields alongside the ones you already read. Ignore them and your 2.0 logic still works (aside from the `report` fix above).

### findings[].status

```json
{
  "findings": [
    { "id": "F-001", "severity": "Critical", "status": "open", "...": "..." }
  ]
}
```

Always `"open"` when emitted by this skill (it has no way to observe remediation). If you're persisting findings across reviews (a tracker, a dashboard), this is the field to write `"accepted"` / `"fixed"` / `"rejected"` / `"verified"` into as your own system tracks lifecycle — see `OUTPUT-FORMAT.md` § "Finding Lifecycle."

## Updated Consumption Snippet (2.1)

```javascript
const review = $input.first().json;

// Unchanged from 2.0
const decision = review.summary.decision;
const score = review.summary.score;
const findings = review.findings;

// Changed from 2.0 — report is now an object
const markdown = review.report.content;

// New in 2.1 — optional, but convenient
const isReady = review.summary.implementationReady;
const totalIssues = review.summary.totalFindings;
const reviewId = review.metadata.reviewId;
```

## Handling Both 2.0 and 2.1 During Transition

```javascript
const output = $input.first().json;
const markdown = output.metadata?.schemaVersion === "2.1"
  ? output.report.content   // 2.1: report is an object
  : output.report;           // 2.0: report is a bare string

// summary/findings/metadata field paths are identical across 2.0 and 2.1,
// aside from the new additive fields — no branching needed for those.
```

## Examples (Version 2.1)

- `examples-output-pass-no-findings.json` — PASS decision, empty `findings` array
- `examples-output-pass-low-findings.json` — PASS decision with only Low-severity, non-blocking findings
- `examples-output-fail-high-finding.json` — FAIL decision driven by a single High-severity finding with no independent remediation path, no Critical finding
- `examples-output-fail-multiple-findings.json` — FAIL decision with a Critical finding (score forced to 0) plus High and Medium findings
- `examples-output-conditional-pass.json` — CONDITIONAL PASS decision (High + Medium findings, no Critical, all with documented remediation) including Implementation Guidance

Validate any 2.1 parsing code against all five — they cover the empty-findings, low-findings, single-high-finding, multi-severity, and conditional-pass shapes.

---

# Part 2: Version 1.x → 2.0 Migration Guide (Historical)

This section is kept for reference. If your workflow is already on 2.0, skip to Part 1 above for the 2.0 → 2.1 change — you do not need anything in this section.

## Summary of Changes

| Aspect | Version 1.x | Version 2.0 |
|---|---|---|
| **Output Format** | Plain markdown only | JSON with embedded markdown |
| **Top-level Structure** | Markdown text | JSON object with 4 fields |
| **Decision Extraction** | Parse markdown text | Access `summary.decision` |
| **Score Extraction** | Parse markdown regex | Access `summary.score` |
| **Findings Access** | Parse markdown sections | Access `findings` array |
| **Machine Readability** | No | Yes |
| **Markdown Report** | N/A | Included in `report` field |
| **Breaking Change** | N/A | Yes — requires workflow updates |

## Version 1.x Output Format (Deprecated)

```markdown
## Work Package Review: WP-6 — Cross-Platform sync-from-git Workflow

### Metadata
- **Reviewer**: opencode work-package-review skill
- **Review Type**: Work Package Review
...

### Decision
- **Readiness Score**: 45
- **Simulation Ceiling**: 69
- **Decision**: FAIL
- **Confidence**: Medium
...

[more markdown sections]
```

**Problem with Version 1.x**: n8n workflows had to parse markdown text to extract decisions and findings. Markdown parsing is brittle, error-prone, and difficult to maintain.

## Version 2.0 Output Format (Superseded by 2.1 — see Part 1)

```json
{
  "metadata": {
    "skill": "work-package-review",
    "version": "2.0",
    "reviewDate": "2026-07-25T14:30:00Z",
    "repository": "C:\\Users\\micha\\repos\\n8n-workflows",
    "workPackage": "WP-6 — Cross-Platform sync-from-git Workflow"
  },
  "summary": {
    "decision": "FAIL",
    "score": 45,
    "scoreCeiling": 69,
    "confidence": "Medium",
    "readiness": "Requires Changes",
    "critical": 1,
    "high": 1,
    "medium": 1,
    "low": 0
  },
  "findings": [
    {
      "id": "F-001",
      "severity": "Critical",
      "title": "Primary design relies on unavailable child_process module",
      "dimension": "Architectural Fit",
      "description": "...",
      "evidence": "...",
      "impact": "...",
      "recommendation": "..."
    }
  ],
  "report": "## Work Package Review: WP-6 — Cross-Platform sync-from-git Workflow\n\n..."
}
```

Note: this is the 2.0 shape shown for historical context. As of 2.1, `report` is `{ format, version, content }` (see Part 1) and `metadata`/`summary`/`findings[]` have the additional fields described there.

**Advantages of Version 2.0 over 1.x**:
- Machine-readable `summary` object for decisions, scores, and statistics
- Structured `findings` array for metrics, dashboards, and automated issue creation
- Markdown report preserved (originally in the `report` field directly; as of 2.1, in `report.content`) for archival and human reading
- Deterministic JSON structure for reliable automation

## Migration Steps

### Step 1: Identify Workflows Using work-package-review

Search for workflows that call the work-package-review skill:

```javascript
// In your n8n workflows, look for nodes that reference 'work-package-review'
// or nodes that parse markdown output containing "Readiness Score"
```

### Step 2: Update Decision Logic

**Old Pattern (Markdown Parsing)**:
```javascript
// Old: Parse the markdown output
const reviewOutput = $input.first().json;
const markdown = JSON.stringify(reviewOutput);

// Fragile regex parsing
const decisionMatch = markdown.match(/Decision:\s*(\w+)/);
const decision = decisionMatch ? decisionMatch[1] : "UNKNOWN";

// Use decision in Switch node
if (decision === "PASS") {
  // ...
}
```

**New Pattern (JSON Access)**:
```javascript
// New: Access the JSON structure directly
const review = $input.first().json;
const decision = review.summary.decision; // "PASS" | "FAIL" | "CONDITIONAL PASS"

// Use decision in Switch node
if (decision === "PASS") {
  // ...
}
```

### Step 3: Update Score Extraction

**Old Pattern**:
```javascript
// Old: Parse markdown
const scoreMatch = markdown.match(/Readiness Score[:\s]+(\d+)/);
const score = scoreMatch ? parseInt(scoreMatch[1]) : 0;
```

**New Pattern**:
```javascript
// New: Direct JSON access
const score = review.summary.score; // already an integer 0-100
```

### Step 4: Update Finding Collection

**Old Pattern**:
```javascript
// Old: Parse markdown sections
const findingsSection = markdown.match(/### Detailed Findings\n([\s\S]*?)(?=###|$)/);
// Manual parsing of finding text...
```

**New Pattern**:
```javascript
// New: Structured array access
const findings = review.findings; // Array of finding objects
const criticalFindings = findings.filter(f => f.severity === "Critical");
const highFindings = findings.filter(f => f.severity === "High");

// Each finding has: id, severity, title, dimension, description, evidence, impact, recommendation
for (const finding of findings) {
  console.log(`${finding.id} [${finding.severity}] ${finding.title}`);
  console.log(`Evidence: ${finding.evidence}`);
  console.log(`Impact: ${finding.impact}`);
  console.log(`Recommendation: ${finding.recommendation}`);
}
```

### Step 5: Update GitHub Comment Generation

**Old Pattern**:
```javascript
// Old: Use the markdown as-is
const comment = `## Review Result\n\n${reviewOutput}`;
```

**New Pattern**:
```javascript
// New: Build comment from JSON fields
const { summary, findings, metadata } = review;
const comment = `## Work Package Review: ${metadata.workPackage}

**Decision**: ${summary.decision} (Score: ${summary.score}/${summary.scoreCeiling})
**Confidence**: ${summary.confidence}
**Findings**: ${summary.critical} critical, ${summary.high} high, ${summary.medium} medium, ${summary.low} low

${findings.length === 0 ? "No issues found!" : "See review details below."}

[Full review report](link-to-report)`;
```

### Step 6: Update Dashboard Aggregation

**Old Pattern**:
```javascript
// Old: Store the entire markdown
return {
  review: reviewOutput,
  timestamp: new Date().toISOString()
};
```

**New Pattern**:
```javascript
// New: Store structured data
const { summary, metadata } = review;
return {
  timestamp: metadata.reviewDate,
  workPackage: metadata.workPackage,
  decision: summary.decision,
  score: summary.score,
  ceiling: summary.scoreCeiling,
  confidence: summary.confidence,
  issues: {
    critical: summary.critical,
    high: summary.high,
    medium: summary.medium,
    low: summary.low
  }
};
```

## Gradual Migration Strategy

If you have multiple workflows using work-package-review, migrate them gradually:

1. **Week 1**: Migrate review extraction workflows (those that call the skill)
2. **Week 2**: Migrate automation workflows (Switch nodes, GitHub comments)
3. **Week 3**: Migrate dashboards and metrics aggregation
4. **Week 4**: Verify all old parsing code has been removed

## Handling Both Versions (Temporary)

If you need to support both versions during transition:

```javascript
const output = $input.first().json;

// Detect version
const isNewFormat = output.metadata && output.metadata.version === "2.0";

let decision;
let score;

if (isNewFormat) {
  // Version 2.0: Use JSON structure
  decision = output.summary.decision;
  score = output.summary.score;
} else {
  // Version 1.x: Parse markdown
  const markdown = typeof output === "string" ? output : JSON.stringify(output);
  const decisionMatch = markdown.match(/Decision:\s*(\w+)/);
  decision = decisionMatch ? decisionMatch[1] : "UNKNOWN";
  const scoreMatch = markdown.match(/Readiness Score[:\s]+(\d+)/);
  score = scoreMatch ? parseInt(scoreMatch[1]) : 0;
}
```

**Remove this compatibility code after all workflows have migrated.**

## Rollback Plan

If you need to rollback to Version 1.x (not recommended):

1. Revert any workflow changes to markdown parsing
2. Update workflows to expect markdown output format
3. Note: Version 1.x skill will no longer be maintained

**Recommendation**: Complete the migration to Version 2.0. The JSON format is strictly superior for automation.

## Testing Your Migration

For each workflow you update, test both the happy path and error cases:

### Happy Path Tests

```javascript
// Test 1: PASS decision
// Expected: decision === "PASS", score >= 70, confidence in ["High", "Medium", "Low"]
// Expected: findings.length >= 0 (may be 0)

// Test 2: FAIL decision
// Expected: decision === "FAIL", score < 70 (usually)
// Expected: findings.length > 0, at least one Critical or High severity

// Test 3: CONDITIONAL PASS decision
// Expected: decision === "CONDITIONAL PASS"
// Expected: zero Critical findings (CONDITIONAL PASS requires none — see references/decision-matrix.md),
//           at least one High or Medium finding with documented remediation guidance
```

### Error Case Tests

```javascript
// Test 4: Malformed JSON
// Expected: workflow should handle JSON parse errors gracefully

// Test 5: Missing fields
// Expected: workflow should handle null/undefined values in optional fields

// Test 6: Empty findings
// Expected: findings array may be empty; handle gracefully in loops
```

## Examples

The example files now use the 2.1 schema (see Part 1's "Examples (Version 2.1)" section above for the current list and naming). If you're migrating a 1.x workflow, use those same files — they demonstrate the destination format (2.1) directly rather than an intermediate 2.0 shape, since 2.0 is no longer the current output format.

## FAQ

### Q: Can I use Version 1.x output with current workflows?

No. The skill now emits 2.1-schema JSON. If you call an old skill version that still emits 1.x markdown, you'll need compatibility code (see "Handling Both Versions" above).

### Q: Is the markdown report still included?

Yes. The complete markdown document is included — as of 2.1, at `report.content` (in 2.0 it was the `report` field directly; in 1.x it was the entire output). You can still archive it or display it for human reading.

### Q: What if a workflow needs both decision and markdown report?

Use both fields:
```javascript
const decision = review.summary.decision;    // JSON structure
const report = review.report.content;         // Full markdown document (2.1 shape)
```

### Q: How do I know which version I'm running?

Check the metadata:
```javascript
const output = $input.first().json;
if (output.metadata?.schemaVersion === "2.1") {
  // Current format — report is an object, extra summary/metadata/finding fields present
} else if (output.metadata?.version === "2.0") {
  // Superseded 2.0 format — report is a bare string
} else {
  // 1.x — plain markdown, no metadata object at all
}
```

### Q: What about custom parsing logic I built?

Replace it with direct JSON field access:
- Custom regex for decision → use `summary.decision`
- Custom parsing for scores → use `summary.score`, `summary.scoreCeiling`
- Custom finding extraction → use `findings` array
- Custom markdown extraction → use `report.content` (2.1) — not `report` directly

All of this is now deterministic and does not require parsing.

## Support

For migration questions or issues:

1. Consult `OUTPUT-FORMAT.md` for field reference
2. Review the examples in the `examples-output-*.json` files
3. Check `references/review-output-contract.md` for schema details, including the "Source of Truth" section

## Version Timeline

- **2026-03-15**: Version 2.0 released (1.x → 2.0 migration: markdown-only → JSON with `summary`/`findings`/`report`)
- **2026-03-15 – 2026-06-15**: Parallel running (1.x and 2.0 both available)
- **2026-06-15**: Version 1.x support ended. Version 2.0 became the only supported format.
- **2026-07-25**: Version 2.1 released (architectural refinement: `report` becomes a structured object; `metadata`/`summary`/`findings[]` gain additive fields). See Part 1 above.
