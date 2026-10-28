---
name: business-memory
description: Restore and maintain durable business knowledge at the start and finish of a development workflow. Use an explicit start or finish phase to verify relevant memory against key code paths and history, then assess task-specific changes and update business_docs when warranted. Also supports explicitly requested business-memory analysis or maintenance.
---

# Business Memory

Restore relevant business context before development and preserve verified business knowledge afterward. The calling workflow invokes this skill twice and supplies the phase; this skill does not detect lifecycle transitions or run the development workflow itself.

## Invocation Contract

For lifecycle calls, accept:

- `phase`: required, `start` or `finish`; never infer it from conversation or Git status.
- `project_root`: target project containing the business code. Resolve its Git root separately for projects inside larger repositories.
- `run_id`: required, stable and unique for this workflow run; reuse across both calls and retries.
- `requirement`: business goal, expected behavior and known constraints; required at start and carried forward in state.
- `state_dir`: optional workflow-owned storage for this run. Default: `<project_root>/.business-memory/runs/<run_id>/`.
- At finish: implementation summary, available verification results, and any authoritative task commit range or patch supplied by the workflow.

Require explicit phase, target project and run identity before lifecycle writes. Report missing inputs to the caller rather than guessing. Reuse existing workflow state storage through an equivalent contract instead of duplicating state. Further development after a completed run starts a new run with a fresh baseline; a finish retry reconciles the same completed task.

Read [Persistence Contract](references/persistence.md) when capturing or recovering state, assessing task changes, or writing business events.

## Knowledge Boundaries

| Information | Location | Purpose |
| --- | --- | --- |
| Run state and baseline evidence | Workflow storage or `.business-memory/runs/<run_id>/` | Cross-session continuity and task attribution |
| Current business knowledge | `business_docs/` Markdown | Rules, modules, flows, key paths, dependencies and pitfalls |
| Durable business change events | `business_docs/changes/events.jsonl` | Append-only history of independently meaningful business changes |

Scope memory to the requested business area. Exclude generic coding style, task plans, chat transcripts, routine run logs and obvious local implementation details. Keep runtime state out of navigation and do not stage or commit it. Honor existing ignore conventions; if the default state directory is not ignored, report that fact without silently changing project configuration.

## Progressive Reading

1. Start with `business_docs/navigation.md` when present. Reuse an established equivalent project-owned business knowledge location.
2. Select docs by business concepts, module, flow, API, state or entry point. If navigation is missing or incomplete, search filenames and content first.
3. Read `README.md` for onboarding and project-owned templates only when writing. Do not load the entire knowledge base or event log by default.
4. Treat memory and commit messages as leads. Verify important claims against relevant source and, when needed, commit diffs.
5. Distinguish verified behavior, historical descriptions, inferences and unresolved questions.

## Start Phase

Run before broad implementation analysis. Retries for the same run resume existing state without replacing the initial baseline.

1. Resolve the requirement and project scope. Capture the initial Git identity and worktree state before development edits, using the persistence contract.
2. Load relevant memory. If none exists, inspect only this task's business area; do not initialize whole-project documentation.
3. Inspect selected code nodes to confirm entry points, core rules, state transitions and important dependencies. This bounded source inspection is part of start and precedes broader development analysis.
4. Review relevant history: prioritize versions referenced by selected docs and subsequent commits affecting the business area. Without recorded versions, inspect a bounded set of relevant path history. Do not equate the repository's latest commit with the previous workflow or business change. Follow relevant renames when needed.
5. Save a compact business summary, selected docs, verified code references, relevant commit IDs, stale-memory findings and unresolved questions in run state. Store baseline evidence separately from this summary.
6. Return restored context, key paths, business constraints and risks, verification gaps, and the state location.

Start writes runtime state only. Preserve durable-memory gaps for finish. When scope expands, extend context without replacing the original baseline; disclose paths whose initial content was not captured.

## Finish Phase

Run after implementation and appropriate workflow verification, before handoff. Retries resume the assessment and reuse event identities.

1. Recover state for the same project and `run_id`; check repository identity. Apply the persistence contract's fallback if state is missing or incomplete.
2. Establish the task's actual increment, including committed changes, staged/unstaged edits and new or deleted files. Exclude unrelated initial or concurrent changes and runtime artifacts. A current `git diff` alone is insufficient.
3. Compare requirement, attributed increment, final code, available verification and relevant memory. Report material intent/implementation conflicts; do not present an unexplained mismatch as the intended business rule.
4. Apply the update decision below. Update verified current knowledge and append attributable business events where warranted. Inconclusive parts must not prevent independent, supported updates.
5. Create only needed files and maintain navigation. Update automatically when warranted, respecting an explicit user instruction not to write docs; in that case report the proposed updates.
6. Perform completion checks. Save the assessment, evidence, changed paths, stable event identities and unresolved issues in run state for retries.
7. Report `updated`, `no_update_needed`, or `insufficient_evidence` and the reason. If supported updates were made but gaps remain, report `updated` and those gaps. Pending required writes must be reported explicitly, not described as complete.

Use available workflow checks and targeted source verification. Do not initiate deployment, run a complete test workflow or commit changes. Distinguish implementation, verification, merge, deployment and business activation; never infer one from another.

## Update Decision

Update when all three hold:

1. Knowledge has lasting business value: a rule, visible behavior, API meaning, state transition, data interpretation, key flow, dependency or recurring business pitfall.
2. Inspected implementation or explicit evidence supports the claim, with limitations stated.
3. Existing memory is missing, stale or inconsistent with this knowledge.

| Finding | Action |
| --- | --- |
| Business rule or state transition changes | Update current knowledge; append a meaningful attributable event if missing from history |
| Bugfix reveals a durable constraint or recurring pitfall | Preserve constraint and cause; record the meaningful behavior change if applicable |
| Behavior unchanged, documented key entry point moved | Repair affected paths; no behavior-change event required |
| Existing docs accurately cover final behavior | Skip duplicate prose; record an event only if it adds missing meaningful business history |
| Formatting or local cleanup leaves memory accurate | Skip |
| Requested behavior unimplemented or unverified | Do not present it as an established fact; report the gap |
| Current facts verified, task attribution uncertain | Repair current knowledge with version/scope limits; omit unsupported task events |

Do not create events simply because a run finished or docs changed. Discovering an existing rule can warrant a module/flow update without an event.

## Documentation Contract

Prefer this structure, creating only needed files:

```text
business_docs/
├── navigation.md
├── modules/
├── flows/
├── changes/
│   ├── events.jsonl
│   └── change_YYYY-MM-DD_topic.md   # Optional detailed explanation
└── graph/                         # Optional relationship diagrams
```

Preserve existing README, templates, filenames and Markdown change history. Do not force migration to JSONL.

### Current Knowledge

Update related docs rather than duplicating them. Include relevant items instead of filling irrelevant headings:

- **Module:** purpose and boundary, entry points, key file responsibilities, concepts/enums, business rules, input/output and API meaning, dependencies, pitfalls and related flows.
- **Flow:** trigger, entry point, steps and key symbols, API/input/output semantics, state changes, error handling and side effects. Include UI effects when applicable; support services, jobs and message consumers equally.
- **Evidence:** verification timestamp, applicable code version or working-tree scope, paths and symbols, verification limitations, and flags or activation conditions.

Prefer stable paths and symbols over bare line numbers. Scope behavior to inspected code; do not label local or undeployed changes as production behavior.

### Change History

Append JSONL events using the persistence contract's schema and retry rules. For complex changes, link a Markdown explanation covering summary, before/after behavior, impact, reason, code evidence, risks and related docs. Include rollback information only when known and useful. Keep the full explanation in one place.

Module/flow docs evolve with current implementation. Independent events and detailed change explanations retain their history; do not overwrite an old event to describe a later change. Clarifications to a detailed document for the same event are allowed.

### Navigation and Graphs

Keep `navigation.md` short: named links and one-line scopes grouped into modules, key flows, changes and optional relationships. Link the event log once with guidance to filter by run, event, business terms or code paths; do not copy every event into navigation. Link detailed explanations individually.

Maintain navigation when adding, moving or removing docs, or changing their discoverable scope. Use lightweight Markdown relationships and small Mermaid diagrams only when useful.

## Completion Checks and Output

For finish writes, check:

- Changed Markdown links resolve, new knowledge is reachable from navigation, and current code references exist. Label historical paths with their historical version instead of requiring them to exist now.
- Changed claims match inspected evidence, with implementation and activation limits explicit.
- JSONL lines conform to the schema, event IDs are unique, correction references resolve and retries have not duplicated events. Serialize shared-log writes without losing concurrent entries.
- Runtime artifacts are absent from durable memory.

At start report context, key paths, related history, docs/code checked, gaps and state location.

At finish report decision and reason, docs/events changed, navigation changes, inspected evidence, available verification used and unresolved gaps. Say when no durable docs were changed. Keep output concise and usable by the caller.

For an explicitly requested read-only analysis or memory review outside the lifecycle, inspect relevant docs and evidence and recommend corrections without writes. Explicit maintenance requests may update memory within scope without inventing a lifecycle run or historical event.
