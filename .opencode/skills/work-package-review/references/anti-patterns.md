---
name: anti-patterns
description: Canonical engineering standard for recognizing, understanding, and preventing poor engineering reviews. Defines every anti-pattern, failure mode, review smell, cognitive bias, and common mistake that degrades review quality. Does not define methodology, principles, scoring, or decision rules.
---

# Anti-Patterns

## Purpose

This document defines the recurring engineering failure modes that reduce review quality. It is the single source of truth for every anti-pattern, reviewer mistake, cognitive bias, and review quality degradation mechanism within the framework.

A review anti-pattern is not a simple mistake. It is a recurring failure mode with a recognizable signature, root cause, and remediation. Treating anti-patterns as engineering failure modes rather than personal errors allows reviewers to detect them before they damage review quality, recover from them when they occur, and prevent them in future reviews.

This document teaches engineering judgment about the review process itself. It answers:

- Which reviewer behaviors reduce review quality?
- Which engineering mistakes appear repeatedly?
- Which shortcuts produce unreliable reviews?
- Which cognitive biases affect engineering judgment?
- How can anti-patterns be recognized before they cause damage?
- How can they be prevented at the framework level?
- Which anti-patterns invalidate a review entirely?

## How to Use This Document

Before every review, scan the detection cues to check whether any anti-pattern is active. After every review, scan the prevention guidance to identify framework improvements.

Anti-patterns are organized by root cause category, not by severity or frequency. A reviewer who recognizes a pattern in one category should check the entire category — anti-patterns within the same category share root causes and frequently co-occur.

## Anti-Pattern Classification

Every anti-pattern is classified by:

| Dimension | Meaning |
|---|---|
| Severity | How much damage this anti-pattern causes when active |
| Recoverability | Whether a review can be salvaged or must be restarted |
| Frequency | How often this anti-pattern occurs in typical reviews |

### Severity

| Level | Meaning |
|---|---|
| Critical | The review is invalid. Findings cannot be trusted. Must restart. |
| High | Significant quality degradation. Findings may be unreliable. Requires correction before output. |
| Medium | Noticeable quality reduction. Some findings may be affected. Should be corrected. |
| Low | Minor quality impact. Awareness is sufficient. |

### Recoverability

| Level | Meaning |
|---|---|
| Recoverable | The reviewer can recognize and correct the anti-pattern without restarting |
| Salvageable | The review can be completed but requires partial re-execution of affected phases |
| Irrecoverable | The review must be restarted from the beginning |

---

## 1. Knowledge Failures

Knowledge failures occur when the reviewer lacks sufficient understanding of what they are reviewing. These are the most common and most damaging anti-patterns because every subsequent judgment depends on the reviewer's understanding.

### 1.1 Repository Blindness

**Also known as**: Reviewing Without Repository Access, Insufficient Repository Knowledge

**Why it happens**: The reviewer evaluates the Work Package without reading the repository. They may assume the repository matches their expectations, rely on prior knowledge of similar projects, or skip repository analysis due to time pressure. LLMs are particularly susceptible because they can generate plausible-sounding reviews from general training data without accessing the actual codebase.

**How to recognize it**:
- Findings reference architecture or patterns that do not exist in the repository
- Recommendations introduce patterns the project has explicitly rejected
- No repository file paths appear in any finding
- Evidence citations are generic ("the codebase" rather than a specific path)
- The reviewer expresses high confidence after examining fewer than three source files

**Why it is dangerous**: A reviewer who has not read the repository cannot detect the most impactful issues: architectural inconsistencies, missed reuse opportunities, and incorrect assumptions about existing code. The review may appear thorough while missing every issue that matters.

**Engineering consequences**:
- Work Packages pass that contradict the repository
- Existing code is duplicated because the reviewer did not find it
- Architectural drift accelerates because inconsistencies go undetected
- The implementer discovers gaps during implementation — the review failed its purpose

**How to recover**: The review cannot be salvaged without repository access. The reviewer must execute the full repository analysis methodology defined in `references/repository-analysis.md` before any finding can be trusted. If repository access is unavailable, the review must be marked as incomplete with Low confidence.

**How to prevent it**:
- Framework: Repository analysis is mandatory before simulation. The review checklist in `references/review-checklist.md` verifies repository analysis completion.
- Reviewer: Before writing any finding, verify that you have read relevant files from at least three locations in the repository.

### 1.2 Premature Conclusions

**Also known as**: Premature Judgment, Jumping to Conclusions

**Why it happens**: The reviewer forms conclusions about the Work Package's quality while reading it, before examining the repository. A Work Package that looks well-formatted or matches the reviewer's prior experience creates an initial impression of readiness that the reviewer then validates rather than challenges. Reading the Work Package first (as the methodology requires) is necessary but creates the risk of premature judgments that repository analysis confirms rather than tests.

**How to recognize it**:
- Findings address peripheral or surface-level issues while missing the central problem
- The review focuses on writing quality, formatting, or terminology rather than implementation readiness
- The reviewer passes a Work Package that contains hidden assumptions or unstated dependencies
- The reviewer's hypotheses from Phase 1 are all confirmed — none are refuted
- The review mentions only positive observations without identifying gaps

**Why it is dangerous**: Premature conclusions produce reviews that validate the Work Package author's assumptions rather than challenging them. The review becomes a rubber stamp. The most dangerous Work Packages — those that look good but contain fundamental flaws — are the ones most likely to receive a premature pass.

**Engineering consequences**:
- Work Packages with hidden assumptions pass review
- Implementation failures are discovered during coding, not before
- The review creates false confidence that delays problem detection

**How to recover**: Set aside the initial conclusion and force-test at least one hypothesis that would refute it. Search for evidence that contradicts the initial impression. If such evidence exists, the initial conclusion was premature.

**How to prevent it**:
- Framework: The review lifecycle separates Context Acquisition from Repository Intelligence explicitly. The reviewer forms hypotheses, not conclusions, during Phase 1.
- Reviewer: After repository analysis, ask: "What would I need to find in the repository to disprove my initial assessment?" Then search for it.

### 1.3 Shallow Analysis

**Also known as**: Underanalysis, Insufficient Depth

**Why it happens**: The reviewer reads enough to form a general impression but not enough to identify specific gaps. Repository analysis is time-consuming, and the reviewer may feel pressure to produce output quickly. "I've seen similar code before" creates a false sense of familiarity that justifies stopping early.

**How to recognize it**:
- The reviewer examined fewer than three files from the impact zone
- Conventions are inferred from a single example
- Dependency mapping covers only direct, first-level dependencies
- The architecture summary describes what the project calls itself, not what the code reveals
- Every dimension evaluation references the same small set of files

**Why it is dangerous**: Shallow analysis misses the majority of findings. The most impactful issues — architectural drift, missed reuse opportunities, silent deviation risks — require understanding how code connects across modules. Surface-level reading reveals surface-level issues.

**Engineering consequences**:
- Missed findings compound during implementation
- The implementer discovers gaps that the review should have caught
- Trust in the review process erodes when issues surface after passing

**How to recover**: Identify the thinnest area of analysis and deepen it. A reliable heuristic: if any dimension evaluation references fewer than three repository files, the analysis for that dimension is insufficient.

**How to prevent it**:
- Framework: Depth guidance in `references/repository-analysis.md` maps analysis depth to Work Package complexity.
- Reviewer: Before concluding analysis, verify you have read at least one file from each architectural layer in the impact zone and verified every convention across three independent locations.

---

## 2. Judgment Failures

Judgment failures occur when the reviewer makes incorrect engineering assessments. These are distinct from knowledge failures: the reviewer may understand the repository but apply incorrect criteria.

### 2.1 Confusing Requirements with Implementation

**Why it happens**: Reviewers who are also experienced engineers may instinctively evaluate implementation details rather than requirements. They see what the Work Package proposes to build and judge the implementation approach, not the specification completeness. This is particularly common when the reviewer has strong opinions about technical approaches.

**How to recognize it**:
- Findings criticize how the Work Package proposes to implement something rather than whether the requirements are complete
- The review recommends specific implementation approaches that are not required by the Work Package's scope
- The review rejects a Work Package because the described approach differs from what the reviewer would do
- Findings prescribe code structure, library choice, or algorithm selection rather than specification gaps

**Why it is dangerous**: Implementation details belong in the code, not the Work Package. A Work Package should describe what to build, not how to build it (unless specific implementation constraints are required for consistency). Judging implementation choices in a Work Package review creates two problems: it forces the reviewer to evaluate code that does not yet exist, and it constrains the implementer's engineering judgment.

**Engineering consequences**:
- Work Packages are rejected for implementation style rather than specification gaps
- Implementers receive contradictory guidance when they implement differently than the review assumed
- The review evaluates the wrong level of abstraction

**How to recover**: For each finding, ask: "Is this about completeness of specification or about implementation approach?" If it is about implementation approach, it belongs in code review, not Work Package review.

**How to prevent it**:
- Framework: The review scope explicitly excludes implementation details.
- Reviewer: Distinguish between "the Work Package does not specify X" and "I would implement X differently."

### 2.2 Severity Inflation

**Also known as**: Over-classification

**Why it happens**: The reviewer assigns Critical or High severity to findings that are minor inconsistencies or style preferences. This often stems from a desire to be thorough — if everything is important, nothing will be missed. It can also stem from the reviewer overestimating their own certainty about a finding's impact.

**How to recognize it**:
- Most findings are High or Critical
- Low and Medium findings are absent despite many minor issues
- Severity is justified with generic language ("this is important") rather than specific impact analysis
- The review contains more High findings than Medium and Low combined

**Why it is dangerous**: Severity inflation erodes the severity model. When every finding is Critical, implementers cannot distinguish what actually requires attention. The severity model exists to communicate priority — inflation destroys this communication. It also makes it harder to fail a Work Package appropriately, because a review with fifteen High findings appears to describe a different level of readiness than a review with two.

**Engineering consequences**:
- Implementers learn to ignore severity labels
- Critical findings lose their power to stop the line
- Work Packages with many High findings are treated the same as Work Packages with a few Medium findings

**How to recover**: Re-evaluate every High and Critical finding against the severity criteria in `references/implementation-readiness.md`. For each, ask: "Does this finding cause incorrect behavior, data loss, or unrecoverable damage?" If not, it is not Critical.

**How to prevent it**:
- Framework: Severity assignment rules require reviewers to choose the lower severity when uncertain.
- Reviewer: Before assigning severity, enumerate the concrete impact. If you cannot describe the specific consequence, the severity is likely inflated.

### 2.3 Severity Deflation

**Why it happens**: The reviewer assigns Medium or Low severity to findings that are genuinely Critical or High. This often stems from a desire to avoid conflict, a belief that the implementer will "figure it out," or uncertainty about whether the finding is correct. LLMs may also deflate severity because they are trained to be agreeable and avoid strong negative classifications.

**How to recognize it**:
- A finding describes an issue that would cause incorrect behavior but is classified as Medium
- The review has no Critical findings despite describing a fundamental architectural contradiction
- The review's decision (PASS) contradicts the severity of described issues
- The reviewer notes "this is serious" in the finding text but assigns Low or Medium severity

**Why it is dangerous**: Severity deflation is more dangerous than inflation because it causes implementers to proceed with insufficient caution. A Critical finding that is classified as High may be addressed during implementation; a Critical finding classified as Medium is likely ignored.

**Engineering consequences**:
- Work Packages pass that contain implementation-blocking issues
- The implementer proceeds with incorrect assumptions
- The review creates false confidence that the Work Package is safe to build

**How to recover**: Re-evaluate every finding that describes an issue affecting correctness, security, or data integrity. If the finding describes a scenario where incorrect behavior would result, it is at least High.

**How to prevent it**:
- Framework: Decision rules in `references/implementation-readiness.md` map severity to readiness. A review with undetected Critical findings will produce an incorrect PASS.
- Reviewer: When uncertain about severity, evaluate impact as if the finding is correct. If the impact is significant, the severity should reflect that.

### 2.4 False Confidence

**Also known as**: Over-Confidence Without Simulation

**Why it happens**: The reviewer assigns High confidence because the Work Package looks well-written, matches their domain expertise, or is authored by a trusted contributor. Confidence is based on presentation quality and prior experience rather than on the depth of simulation and evidence collection.

**How to recognize it**:
- Confidence is High but the reviewer examined fewer than three source files
- Confidence is High but the reviewer did not execute simulation
- Confidence is High but the review contains assumptions documented as untested
- The reviewer states "I have reviewed similar Work Packages before" as a confidence justification

**Why it is dangerous**: False confidence causes the implementer to trust a review that has not earned that trust. A High confidence review signals to the implementer that all significant issues have been found. If the review is actually shallow, the implementer proceeds with unwarranted certainty.

**Engineering consequences**:
- Implementation begins with insufficient information
- Issues discovered during implementation are blamed on the Work Package rather than on the inadequate review
- The review process loses credibility when confident reviews produce incorrect outcomes

**How to recover**: Recalculate confidence using the criteria in `references/implementation-readiness.md`. If simulation was not executed or repository analysis was incomplete, confidence cannot exceed Medium.

**How to prevent it**:
- Framework: Confidence criteria in `references/implementation-readiness.md` tie confidence directly to analysis depth and evidence strength.
- Reviewer: Confidence is earned through thorough investigation, not through domain familiarity.

### 2.5 Score-Driven Reviews

See the Score-Driven Decisions anti-pattern in `references/decision-matrix.md`. The decision methodology there defines how to prevent score-driven decision making: classify severity before calculating the score, calculate the score before determining the decision.

### 2.6 Checklist-Driven Reviews

**Why it happens**: The reviewer treats the review as a checklist verification exercise rather than an engineering judgment. They verify that each dimension has been evaluated, that evidence exists, and that the format is correct, without exercising independent engineering judgment about whether the Work Package is actually implementable.

**How to recognize it**:
- Every dimension evaluation produces the same boilerplate ("no issues found")
- Findings follow a template that would apply to any Work Package
- The review mentions methodology compliance but not engineering reasoning
- The reviewer cannot explain why the Work Package passed or failed without reading from the checklist

**Why it is dangerous**: Checklists verify methodology compliance; they do not substitute for engineering judgment. A checklist-compliant review can still be wrong if the reviewer did not apply judgment to the specific Work Package. The most dangerous Work Packages — those that look complete but contain subtle flaws — pass checklist reviews because they meet the surface criteria.

**Engineering consequences**:
- Work Packages pass that contain issues the checklist does not catch
- The review process becomes procedural rather than judgment-based
- Reviewers stop thinking independently and follow templates

**How to recover**: Put the checklist aside and ask: "Does this Work Package pass or fail based on my engineering judgment?" If the answer differs from the checklist result, investigate why. The checklist may need updating, or the reviewer may have identified a gap the checklist does not cover.

**How to prevent it**:
- Framework: The checklist in `references/review-checklist.md` references methodology documents rather than attempting to define all criteria inline. It is a verification tool, not a review methodology.
- Reviewer: Use the checklist after forming conclusions, not before. It catches omissions; it does not create insights.

---

## 3. Bias Failures

Bias failures occur when cognitive biases distort the reviewer's judgment. These are the most difficult anti-patterns to self-detect because biases operate below conscious awareness.

### 3.1 Confirmation Bias

**Why it happens**: The human brain naturally seeks evidence that confirms existing beliefs and discounts evidence that contradicts them. In a review context, the reviewer's initial impression of the Work Package (positive or negative) creates a filter: the reviewer notices evidence that supports the impression and overlooks evidence that challenges it. LLMs exhibit similar patterns through instruction-following — if the review prompt frames the task as "check if this Work Package is ready," the model searches for confirming evidence.

**How to recognize it**:
- All findings are of the same type (all positive or all negative)
- The reviewer found evidence that confirms the Work Package is ready but did not search for gaps
- Hypotheses from Phase 1 were validated but none were refuted
- The review mentions what the Work Package includes but does not mention what it omits
- Search queries target confirming patterns rather than revealing contradictions

**Why it is dangerous**: Confirmation bias produces reviews that validate the current state rather than finding gaps. The review becomes a rubber stamp for the Work Package author's assumptions. The most impactful findings — the ones that prevent implementation failures — are the ones that contradict expectations.

**Engineering consequences**:
- Work Packages with hidden assumptions pass review
- Implementation failures surprise both the author and the reviewer
- The review process fails to detect its own systematic blind spots

**How to recover**: Actively search for disconfirming evidence. For each claim in the Work Package, ask: "What evidence would prove this claim wrong?" Then search for that evidence. If nothing in the repository contradicts the claim, the claim is validated. If the search reveals contradictions, the Work Package has a finding.

**How to prevent it**:
- Framework: The review lifecycle requires the reviewer to form hypotheses before examining the repository and explicitly test them.
- Reviewer: After repository analysis, explicitly list what you expected to find but did not. These absences are often the most important findings.

### 3.2 Affinity Bias

**Also known as**: Deference to Authority

**Why it happens**: The reviewer defers to the Work Package author's seniority, expertise, or reputation rather than evaluating the Work Package on its merits. A Work Package from a senior engineer or domain expert receives less scrutiny because the reviewer assumes it is correct. LLMs may exhibit this by being more agreeable when the Work Package is well-formatted or uses authoritative language.

**How to recognize it**:
- Findings from junior-authored Work Packages outnumber findings from senior-authored Work Packages with similar quality
- The reviewer notes "the author is an expert in this area" as justification for accepting a claim without evidence
- The reviewer does not challenge architectural decisions from a senior contributor that they would challenge from a junior contributor
- The review treats well-formatted Work Packages as more credible than poorly-formatted ones

**Why it is dangerous**: Engineering quality is independent of authorship. A senior engineer can make incorrect assumptions about parts of the codebase they have not recently worked on. Deference to authority replaces evidence-based evaluation with status-based evaluation.

**Engineering consequences**:
- Incorrect assumptions from senior contributors go undetected
- The codebase accumulates architectural drift based on unchallenged decisions
- The review process reinforces hierarchy rather than engineering quality

**How to recover**: Re-evaluate the Work Package as if it were authored by an unknown contributor. For every claim that was accepted without evidence, apply the standard validation process. If the evidence supports the claim, the original assessment was correct but for the wrong reason.

**How to prevent it**:
- Framework: The Challenge Assumptions principle in `references/review-principles.md` requires treating every Work Package statement as a hypothesis regardless of authorship.
- Reviewer: Apply the same scrutiny to every finding regardless of who wrote the Work Package.

### 3.3 Anchoring Bias

**Why it happens**: The first piece of information the reviewer encounters — the Work Package's stated approach, the first file read during analysis, the initial hypothesis — becomes an anchor that influences all subsequent judgments. Subsequent information is interpreted relative to the anchor rather than independently. If the first file read confirms the Work Package is correct, all subsequent files are read through that lens.

**How to recognize it**:
- The first dimension evaluated dominates the review
- Later findings are less detailed or less confident than earlier ones
- The reviewer forms a strong opinion after reading one file and subsequent files do not change it
- The review's conclusion matches the initial hypothesis regardless of evidence depth

**Why it is dangerous**: Anchoring narrows the reviewer's evaluation window. Important evidence that contradicts the anchor is discounted or missed entirely. The review becomes a one-dimensional assessment of the initial impression rather than a multi-dimensional evaluation.

**Engineering consequences**:
- The review misses findings in dimensions evaluated later
- One strong initial impression (positive or negative) determines the outcome
- The evaluator's attention is distributed unevenly across dimensions

**How to recover**: After completing all dimension evaluations, revisit the first dimension evaluated and re-evaluate it independently of the other dimensions. If the re-evaluation differs from the initial evaluation, anchoring affected the judgment.

**How to prevent it**:
- Framework: The review lifecycle evaluates dimensions independently. Each dimension has its own evaluation criteria and finding set.
- Reviewer: Randomize the order of dimension evaluation. Do not always start with the same dimension.

### 3.4 Certainty Bias

**Why it happens**: The reviewer expresses absolute certainty about engineering judgments that involve inherent trade-offs and uncertainty. This may stem from overconfidence in their own understanding, a desire to appear authoritative, or a lack of awareness of what they do not know. LLMs are particularly susceptible because they generate confident-sounding text without internal uncertainty.

**How to recognize it**:
- Findings use language like "definitely," "certainly," "always," "never"
- The reviewer does not acknowledge alternative approaches or trade-offs
- Recommendations are stated as the only valid approach
- The reviewer expresses High confidence despite shallow analysis
- The review does not mention any limitations or assumptions

**Why it is dangerous**: Certainty bias creates false precision. Engineering decisions involve trade-offs; absolute certainty about a complex judgment indicates the reviewer is unaware of their own assumptions. When the review is wrong, the certainty makes it harder for the implementer to disagree — they assume the reviewer's certainty is justified.

**Engineering consequences**:
- Implementers follow incorrect recommendations because they trust the reviewer's certainty
- The review creates a false sense of correctness
- When certain recommendations fail, trust in the review process erodes

**How to recover**: For every finding stated with certainty, add uncertainty qualifications. "This will cause performance problems" becomes "This may cause performance problems under high load because..." Acknowledge conditions and alternatives.

**How to prevent it**:
- Framework: Decision Philosophy in `references/review-principles.md` states: "Certainty is rare. Engineering decisions involve trade-offs."
- Reviewer: Use calibrated language. "This approach creates a risk of X" is more accurate than "this approach is wrong."

---

## 4. Scope Failures

Scope failures occur when the reviewer evaluates the wrong scope — examining too much, too little, or the wrong aspects of the Work Package.

### 4.1 Scope Creep

**Also known as**: Scope Drift, Codebase Audit

**Why it happens**: The reviewer finds pre-existing code quality issues during repository analysis and includes them as findings. These issues may be genuine problems in the codebase, but they are unrelated to the Work Package under review. The reviewer feels that ignoring them would be irresponsible, so the review expands from Work Package evaluation to full codebase audit.

**How to recognize it**:
- Findings about pre-existing code quality issues that the Work Package does not affect
- Recommendations to refactor unrelated subsystems
- The review becomes a code audit of the entire repository
- Finding count is disproportionately high for the Work Package's scope

**Why it is dangerous**: Scope creep dilutes the review's focus. The implementer cannot distinguish between findings that block the Work Package and findings about pre-existing issues. The Work Package may fail review because of repository problems that predate it and are outside its scope to fix.

**Engineering consequences**:
- Work Packages are rejected or delayed for pre-existing issues
- The review creates more work than the Work Package itself
- Implementers lose trust in the review process when they are held accountable for unrelated issues

**How to recover**: Separate findings into two categories: Work Package-specific and pre-existing. Pre-existing issues should be noted separately (in an appendix or informational section) and should not affect the readiness determination. Only Work Package-specific findings influence the score.

**How to prevent it**:
- Framework: The review scope defines what is evaluated. The Work Package is the subject; the repository is the context.
- Reviewer: For each finding, ask: "Does the Work Package introduce this issue, or does it pre-exist?" Only findings about issues the Work Package introduces or affects should influence readiness.

### 4.2 Ignoring Context

**Why it happens**: The reviewer evaluates the Work Package in isolation without considering the broader system context — how it interacts with other subsystems, what conventions govern the affected area, or what historical decisions led to the current architecture.

**How to recognize it**:
- Findings are technically correct but miss the system-level impact
- The review evaluates each Work Package section independently without considering cross-section interactions
- The reviewer accepts an approach that is reasonable in isolation but inconsistent with the surrounding system
- Integration surface evaluation is absent or minimal

**Why it is dangerous**: Every Work Package exists within a system. A decision that is correct in isolation may be incorrect when evaluated against the system's architecture, conventions, and dependencies. Ignoring context produces reviews that pass Work Packages that cause integration failures or architectural drift.

**Engineering consequences**:
- Work Packages that fit in isolation cause integration failures at deployment
- Architectural drift accumulates from individually reasonable decisions
- The review misses cross-cutting concerns that only emerge at the system level

**How to recover**: For every finding, verify that it considers the system context. If a finding recommends a pattern, verify that the pattern is consistent with the repository's conventions. If a finding identifies a specification gap, verify that it accounts for the affected module's consumers.

**How to prevent it**:
- Framework: Repository intelligence, dependency mapping, and the six evaluation dimensions in `references/review-process.md` each require system-level analysis.
- Reviewer: After completing dimension evaluations, cross-check findings for system-level interactions. A finding that is valid in isolation may need different treatment when the system context is considered.

### 4.3 Architecture Drift

**Why it happens**: The reviewer evaluates the Work Package against the intended architecture (documentation, README, architecture diagrams) rather than the actual architecture reconstructed from the code. When the intended and actual architectures differ, the review validates against a fiction.

**How to recognize it**:
- The reviewer references architecture documentation rather than code
- Architecture analysis is based on directory structure rather than dependency direction
- The reviewer assumes the architecture matches the project's stated pattern without verification
- The review does not mention any divergence between documented and actual architecture

**Why it is dangerous**: Every codebase has an architecture, but it may not match the documentation. Architecture documentation describes intent at the time of writing; the codebase evolves. A review against the documented architecture will approve changes that contradict the actual architecture, accelerating drift.

**Engineering consequences**:
- The gap between documented and actual architecture widens
- New code follows the documented architecture while existing code follows the evolved architecture — creating inconsistency
- Architectural migration becomes harder as the gap grows

**How to recover**: Reconstruct the actual architecture from the code following the methodology in `references/repository-analysis.md`. If the reconstructed architecture differs from the documented architecture, note the divergence. Evaluate the Work Package against the actual architecture.

**How to prevent it**:
- Framework: Repository analysis mandates architecture reconstruction from code, not from documentation.
- Reviewer: Architecture exists in the code, not in the README. Read the code.

---

## 5. Process Failures

Process failures occur when the reviewer bypasses or incorrectly applies the review methodology.

### 5.1 Simulation as Document Review

**Also known as**: Passive Reading

**Why it happens**: The reviewer treats simulation as a static reading exercise rather than a dynamic mental dry-run. They read the Work Package and assess whether it looks complete, rather than mentally executing each step an implementer would take. LLMs are particularly susceptible because they process text as a reader, not as an implementer.

**How to recognize it**:
- The reviewer describes what the Work Package says rather than what the implementer would do
- Simulation gaps are identified as "this section is incomplete" rather than "I could not write this function because X is missing"
- The review focuses on document quality rather than implementability
- The reviewer cannot describe the implementation sequence for any single requirement
- All gaps are identified as "missing information" rather than "stuck at step X"

**Why it is dangerous**: Passive reading reveals surface-level issues — incomplete sentences, missing sections, formatting problems. It does not reveal implementation gaps — missing data contracts, ambiguous integration points, unstated error handling assumptions. The most impactful gaps are only discoverable through active simulation.

**Engineering consequences**:
- Work Packages that read well but hide serious implementation gaps pass review
- The implementer discovers simulation-detectable gaps during coding
- The review creates a false sense of completeness

**How to recover**: Execute the simulation methodology in `references/implementation-readiness.md` for each logical unit of work. Name the unit, trace reads, write connections, handle errors. If you cannot describe the exact code an implementer would write, the simulation is incomplete.

**How to prevent it**:
- Framework: Implementation simulation is a mandatory phase with a defined methodology.
- Reviewer: Before declaring a gap, write (mentally or on paper) the actual code the implementer would need to produce. If you cannot, the gap is real.

### 5.2 Happy-Path-Only Simulation

**Why it happens**: The reviewer simulates the success path — what happens when everything works correctly — but does not simulate error paths, edge cases, or failure modes. Happy-path simulation is easier and faster because it follows a linear sequence. Error path simulation requires branching logic and domain knowledge about what can fail.

**How to recognize it**:
- Simulation gaps are all about missing requirements or specifications
- No simulation gaps describe error handling, edge cases, or failure behavior
- The review evaluates what the Work Package says but does not evaluate what happens when things go wrong
- Error and Observability dimension has "no issues found" despite being a complex operation

**Why it is dangerous**: Happy-path-only simulation misses the majority of implementation gaps. Error handling, edge cases, and failure modes account for more implementation complexity than happy-path logic. A Work Package that specifies the happy path completely but ignores error paths will cause more implementation failures than one with vague happy-path requirements.

**Engineering consequences**:
- Implementers have no guidance for error scenarios and must guess
- Production incidents from unhandled error paths
- The review passes Work Packages that specify the easy parts and ignore the hard parts

**How to recover**: For each operation in the Work Package, simulate: what happens when this operation fails? What are the edge cases for each input? What happens under concurrent access, empty state, or data corruption?

**How to prevent it**:
- Framework: Implementation simulation steps explicitly include error handling and edge case simulation.
- Reviewer: For every requirement, spend as much simulation time on error paths as on the happy path.

### 5.3 Stopping at Repository Comparison

**Why it happens**: The reviewer performs the first comparison in gap analysis (Repository → Work Package) and stops, believing they have completed the analysis. This comparison catches factual errors — wrong file paths, incorrect type names, outdated assumptions. It does not catch the more dangerous silent deviation risks that are revealed by tracing Expected Implementation → Implementation Reality.

**How to recognize it**:
- Gap analysis covers what the Work Package says about the repository but does not project what the implementer would build
- No gaps are classified as "silent deviation risk"
- The review focuses on factual accuracy rather than implementation completeness
- The reviewer cannot describe what the implementer would produce based on the Work Package alone

**Why it is dangerous**: Factual errors are the least dangerous type of gap. They are obvious and easily corrected. The most dangerous gaps are those where the Work Package is factually correct (the file paths exist) but the implementer would build something different than the author intended. These silent deviations cause implementation failures that are not visible until integration.

**Engineering consequences**:
- Implementers produce code that satisfies the Work Package as written but differs from the author's intent
- Implementation failures are discovered during integration or testing
- Rework costs are higher because the gap was not caught at review time

**How to recover**: Execute the Expected Implementation → Implementation Reality comparison. For each gap found during simulation, project what the implementer would build. Then check whether that projection contradicts the Work Package's intent or the repository's patterns.

**How to prevent it**:
- Framework: Gap analysis has four explicit comparison layers. The methodology requires all four.
- Reviewer: After Repository → Work Package comparison, ask: "Given what the Work Package says, what will the implementer actually build?"

### 5.4 Accepting Silent Deviation

**Why it happens**: The reviewer finds a gap during simulation and does not record it as a finding because "the engineer will figure it out." This rationalization treats every implementation gap as surmountable — the engineer will find the right file, infer the correct data model, or choose the appropriate error handling strategy.

**How to recognize it**:
- Simulation gaps are noted mentally but not recorded
- The reviewer says "the implementer can figure this out" about a specific gap
- The review has fewer findings than simulation gaps
- Gaps about ambiguous requirements are dismissed rather than documented

**Why it is dangerous**: Every gap filled independently by the implementer is a potential deviation from the Work Package's intent. The implementer's guess may be wrong, may conflict with other parts of the system, or may match the Work Package author's intent only by coincidence. Every accepted silent deviation increases implementation risk.

**Engineering consequences**:
- Implementations deviate from the Work Package's intent
- The deviations are undocumented and therefore untestable
- Rework is required when the deviations are discovered during integration

**How to recover**: Record every simulation gap as a finding. If the gap is genuinely minor and the implementer's best guess is obvious, classify it as Low or Medium. But record it. An undocumented gap is an accepted risk.

**How to prevent it**:
- Framework: Recording gaps is a mandatory step in simulation methodology.
- Reviewer: Every gap is a finding until validated otherwise. Document or discard — do not leave gaps unrecorded.

### 5.5 Review Integrity Failure

**Why it happens**: The reviewer bypasses the methodology entirely — skipping repository analysis, simulation, or evidence collection — and produces output based on general knowledge or intuition. This may happen when the reviewer is confident in their domain expertise, under time pressure, or using an LLM that generates plausible reviews without genuine analysis.

**How to recognize it**:
- Findings produced without executing repository analysis per `references/repository-analysis.md`
- No evidence citations in any finding
- Engineering recommendations that contradict project patterns
- The reviewer accepted "will be handled during implementation" for a Critical finding
- The review mentions only positive observations without identifying any gaps
- The review output is generic enough to apply to any Work Package

**Why it is dangerous**: A review that bypasses the methodology is not a review. It is an opinion. It cannot detect the issues it was designed to catch. It creates false confidence that the Work Package has been evaluated when it has not. Every integrity failure undermines the review process's credibility.

**Engineering consequences**:
- The review process loses credibility
- Work Packages pass without genuine evaluation
- Implementation failures are attributed to the review process, not to the individual reviewer
- The framework's methodology is blamed for failures that occurred because the methodology was not followed

**How to recover**: The review should not be trusted. Decision confidence must be set to Low and the review flagged as incomplete. If possible, restart the review following the full methodology.

**How to prevent it**:
- Framework: Red Flags in this document and validation rules in `references/review-checklist.md` detect integrity failures. The review lifecycle requires sequential phase execution.
- Reviewer: Before producing output, verify that repository analysis, simulation, and evidence collection are complete. If any phase was skipped, the review is not complete.

---

## 6. Methodology Misapplication

Methodology misapplication occurs when the reviewer follows the methodology mechanically without understanding why it exists. These anti-patterns produce reviews that are technically compliant but substantively incorrect.

### 6.1 Mechanical Dimension Evaluation

**Why it happens**: The reviewer evaluates each dimension as a checklist item — "Specification Completeness: checked, Data Contract Completeness: checked" — without exercising judgment about what completeness means in this specific context. The dimension evaluations describe what was examined but do not identify gaps.

**How to recognize it**:
- Every dimension evaluation produces "no issues found" or identical language
- Dimension evaluations summarize the Work Package rather than evaluate it
- The reviewer cannot explain why a dimension passed without reading from their notes
- Dimensions with obvious gaps receive "no issues found"

**Why it is dangerous**: Mechanical dimension evaluation treats the review as a checklist exercise. Dimensions exist to ensure systematic coverage, but they do not substitute for judgment. A dimension that is evaluated mechanically provides no information — it does not mean the dimension has no issues; it means the reviewer did not look hard enough.

**Engineering consequences**:
- Dimensions with significant gaps receive passing evaluations
- The implementer has no signal about which dimensions need attention
- The dimension summary becomes noise rather than signal

**How to recover**: Revisit each dimension and identify at least one specific question the Work Package must answer for that dimension. If the Work Package answers it, the dimension passes. If not, there is a finding.

**How to prevent it**:
- Framework: Each dimension has specific evaluation criteria and common mistakes that guide judgment.
- Reviewer: For each dimension, formulate a specific question the Work Package must answer. Evaluate whether the answer is present and correct.

### 6.2 Template-Driven Output

**Why it happens**: The reviewer uses a fixed template for findings that does not vary with the finding's content. Every finding follows the same structure regardless of whether that structure fits. This is common when using automated review tools or LLMs that generate finding templates.

**How to recognize it**:
- Every finding has identical phrasing patterns regardless of content
- The finding structure obscures rather than clarifies the issue
- Recommendations are generic ("improve this section") rather than specific
- The review could be about a different Work Package without changing the finding text

**Why it is dangerous**: Template-driven output prioritizes format over content. A well-formatted finding that does not communicate the issue is worse than a poorly-formatted finding that clearly identifies the problem.

**Engineering consequences**:
- Implementers struggle to understand what the finding requires
- The review output is difficult to act on despite being well-formatted
- Format replaces substance in quality assessment

**How to recover**: For each finding, strip the template and write the core message in one sentence. If the core message is not actionable, rewrite the finding.

**How to prevent it**:
- Framework: The output contract in `references/review-output-contract.md` defines required fields but does not mandate template language. Each finding should be written for its specific content.
- Reviewer: Write findings as if there were no template. Ensure the content is actionable, then format it.

---

## 7. Review Smells

Review smells are indicators that an anti-pattern may be active. They are not anti-patterns themselves — they are detection signals.

### Common Smells

| Smell | What It May Indicate |
|---|---|
| All findings are Low severity | Insufficient depth, severity deflation, or passive reading |
| All findings are Critical or High | Severity inflation, confirmation bias, or surface-level review |
| No findings in a complex Work Package | Premature conclusions, checklist-driven review, or insufficient analysis |
| Every dimension has the same number of findings | Template-driven output, not genuine evaluation |
| Findings reference no file paths | Repository blindness, insufficient evidence collection |
| Findings reference only one file | Shallow analysis, anchoring bias |
| The review is shorter than the Work Package | Insufficient depth (possible, not certain — simple Work Packages may have short reviews) |
| The review reads like a summary rather than an evaluation | Passive reading, simulation as document review |
| Confidence is High but evidence is thin | False confidence, repository blindness |
| The reviewer mentions no limitations or assumptions | Certainty bias, insufficient self-awareness |

### When Smells Indicate Invalid Review

The conditions that make a review invalid — including missing repository analysis, missing evidence citations, contradictory recommendations, and confidence-analysis mismatch — are defined in the Integrity Gate of `references/quality-gates.md`. Review invalidation criteria are owned by that document.

---

## 8. Common Rationalizations

Rationalizations are the internal justifications reviewers use to convince themselves that an anti-pattern is acceptable. Recognizing the rationalization is often the first step to recognizing the anti-pattern.

| Rationalization | Reality | Anti-Pattern |
|---|---|---|
| "The engineer will figure out the missing details" | Undocumented details produce undocumented assumptions. The implementer's guesses may not match the Work Package author's intent. | Accepting Silent Deviation |
| "We can clarify during implementation" | Clarification during implementation is rework. The review exists to prevent that. | Accepting Silent Deviation |
| "The architecture is obvious from the code" | If the architecture is obvious, the Work Package should reference it explicitly. Omission is not obviousness. | Shallow Analysis |
| "This is too minor to flag" | Minor omissions compound. A single missing configuration value can cause production failures. | Severity Deflation |
| "The Work Package looks well-formatted, so it's probably complete" | Presentation quality and completeness are independent. Simulate before judging. | Premature Conclusions |
| "I reviewed a similar Work Package before, so I know the pattern" | Every Work Package must be reviewed against the actual repository at the time of review. Prior knowledge is not evidence. | Repository Blindness |
| "The test strategy can be figured out during implementation" | If the acceptance criteria cannot be tested, the requirements are not verifiable. Testing must be specified before implementation. | Accepting Silent Deviation |
| "I don't want to be too harsh" | Holding back findings to avoid conflict damages the implementer and the codebase more than a direct finding. | Severity Deflation |
| "The author is an expert in this area" | Expertise in a domain does not guarantee knowledge of every repository detail. Every claim must be validated. | Affinity Bias |
| "I've seen this pattern before, it always works" | Prior experience with a pattern does not guarantee it fits this specific context. Evaluate against the repository. | Anchoring Bias |
| "The score is about right, no need to calculate precisely" | Scores encode engineering judgment. Imprecise scores produce ambiguous decisions. | Score-Driven Reviews |

---

## 9. LLM-Specific Review Failures

This framework is designed to be used by both human reviewers and LLMs. While the methodology is the same, LLMs exhibit specific failure modes that reviewers should be aware of.

### 9.1 Plausible Hallucination

**Why it happens**: LLMs generate text based on patterns in training data, not on actual repository knowledge. When asked to review a Work Package without repository access, the model may generate plausible-sounding findings that reference non-existent files, patterns, or conventions.

**How to recognize it**:
- File paths look plausible but do not exist in the repository
- Examples reference common patterns from training data that are not in the codebase
- The review reads convincingly but cannot be verified against the repository
- The model expresses high confidence despite having no repository access

**Mitigation**: All LLM-generated reviews must be verified against the repository before release. If the LLM did not have repository access, the review is inherently incomplete.

### 9.2 Instruction Overshooting

**Why it happens**: LLMs are trained to follow instructions. When a review prompt asks the model to "find all issues," the model may find issues that are not real — treating minor style preferences as engineering problems, or identifying issues that the framework explicitly excludes from scope.

**How to recognize it**:
- Findings about issues the framework explicitly excludes
- Severity that contradicts the issue's actual impact
- The review identifies many issues but none are actionable

**Mitigation**: Review prompts should include scope boundaries. Verify LLM-generated findings against the review scope before including them.

### 9.3 Systematic Bias Amplification

**Why it happens**: LLMs amplify biases present in their training data. If the training data contains more reviews of certain types of Work Packages, the model may produce biased assessments. LLMs also tend to be agreeable and may avoid strong negative classifications.

**How to recognize it**:
- The LLM consistently produces passing reviews
- Severity is consistently lower than a human reviewer would assign
- The LLM avoids Critical classifications even for obvious issues

**Mitigation**: Cross-calibrate LLM-generated reviews with human reviews periodically. If systematic differences emerge, adjust the review prompt or methodology.

---

## Anti-Pattern Detection and Prevention

### Detection During a Review

Before producing output, scan for these signals:

1. **Duplicate patterns across findings**: If many findings follow the same structure or severity, check for template-driven output or confirmation bias.
2. **Uneven evidence distribution**: If all findings reference the same file, check for shallow analysis or anchoring bias.
3. **No refuted hypotheses**: If all hypotheses from Phase 1 were confirmed, check for confirmation bias.
4. **Confidence-evidence mismatch**: If confidence is High but evidence is thin, check for false confidence.
5. **Score-finding mismatch**: If the score says PASS but the findings describe significant issues, check for severity deflation.

### Prevention at Framework Level

1. **Methodology enforcement**: The review lifecycle phases must be executed in order. Each phase depends on the previous one.
2. **Checklist validation**: The review checklist in `references/review-checklist.md` catches omissions before output.
3. **Cross-review calibration**: Periodic calibration reviews identify systematic bias patterns across reviewers. Calibration methodology is defined in `references/review-consistency.md`.
4. **Red flag detection**: The review smells in this document provide early warning.
5. **Structured output**: The output contract in `references/review-output-contract.md` ensures findings are self-contained and verifiable.

### Prevention at Individual Level

1. **Slow down**: Anti-patterns thrive under time pressure. Allow sufficient time for each phase.
2. **Read the repository**: Repository blindness is the root cause of most anti-patterns. Read the code.
3. **Simulate actively**: Passive reading produces passive findings. Execute the simulation.
4. **Record everything**: Undocumented gaps are accepted risks. Record every simulation gap.
5. **Calibrate severity**: Use the severity model in `references/implementation-readiness.md`. When uncertain, choose the lower severity.
6. **Review your own review**: Before releasing output, verify that no anti-pattern is active. If you recognize any of the detection cues, investigate before releasing.
