# Persistence Contract

Read for lifecycle state, task attribution and business event writes. Runtime JSON stores run state; JSONL stores durable business events. Current module and flow explanations remain Markdown.

## Run State

Prefer workflow-owned storage. Otherwise use:

```text
<project_root>/.business-memory/runs/<run_id>/
├── state.json
└── baseline/
    ├── staged.patch
    ├── unstaged.patch
    └── files/              # Initial content of relevant pre-existing new files
```

An equivalent workflow store is sufficient. Keep state local and out of automatic commits. Resolve run identity within the target project. Preserve the original baseline on retries; record failed or incomplete captures explicitly.

`state.json` or equivalent fields:

| Field | Meaning |
| --- | --- |
| `schema_version` | `1` |
| `run_id`, `project_root`, `git_root` | Run and target identities |
| `requirement` | Goal and constraints; retain revisions when scope changes |
| `started_at` | ISO 8601 timestamp with explicit offset |
| `baseline` | Initial HEAD, branch/detached state, repository status, scoped patch/snapshot references, captured paths and gaps |
| `context` | Selected docs, code symbols, business summary, history references, stale claims and questions |
| `assessment` | Finish decision, evidence, attributed scope, available checks, updated paths, pending writes and remaining gaps |
| `events` | Stable semantic event keys mapped to event IDs and write status, including correction identities |

Use optional fields only where applicable. Keep large patches outside the context summary. Do not copy credentials or unrelated sensitive files into snapshots.

## Capturing the Baseline

Before development edits:

1. Record repository identity, initial HEAD, branch/detached state and repository-wide path status including pre-existing untracked paths. This lightweight inventory helps recognize later scope expansion.
2. Save staged and unstaged diffs for relevant paths separately, covering additions, deletions, renames and binary changes where relevant. Preserve content or sufficient snapshots for relevant pre-existing untracked files; status or hashes alone cannot reconstruct content.
3. Record captured paths and evidence. Reconstruct relevant tracked files from initial HEAD and saved patches; save extra snapshots when that is insufficient.
4. Save discovered context without changing this initial baseline.

Useful read-only commands include `git status --porcelain=v1 --untracked-files=all`, `git rev-parse HEAD`, `git diff --binary`, `git diff --cached --binary`, and targeted `git log` / `git show`. Handle an unborn repository explicitly: there may be no initial HEAD. Without Git, use scoped initial snapshots and report reduced history support.

Do not stash, reset, stage or commit to capture a baseline. If scope expands to a pre-existing modified file whose original content was not saved, disclose that gap; the path inventory cannot reconstruct its initial content.

## Task Increment

Use baseline, workflow scope and development evidence together:

- Compare initial worktree content against final content, including original staged, unstaged and relevant untracked changes; do not only compare two HEADs.
- Include commits during the run, final staged/unstaged edits and new/deleted files. A clean final worktree can still contain task changes.
- Separate unrelated initial and concurrent changes, including within the same file. A path in both snapshots does not establish task ownership.
- Inspect actual task commits and final source as needed. Do not assume every intervening commit belongs to the run or substitute the latest commit for task scope.
- Account for branch changes, rebases and rewritten history using saved evidence. Limit claims if attribution remains ambiguous.
- Exclude run artifacts and documentation written by this finish call from the business-code increment. Inspect docs separately for consistency.

A worktree comparison establishes net changes, not authorship. Use workflow evidence to substantiate task attribution, especially with concurrent work.

### Missing or Incomplete State

First use an explicit task range or patch supplied by the workflow. Inspect before/after context and account for remaining worktree changes whose provenance is known.

If task scope cannot be recovered, verified current facts may still repair module/flow docs with version and verification limits. Do not append unsupported task events, invent before behavior or automatically select the latest commit. Report `insufficient_evidence` if no supported update can be made. Never replace a missing initial baseline with the current state.

## JSONL Business Events

Default: `business_docs/changes/events.jsonl`; preserve an established equivalent location. Use UTF-8, one complete JSON object per line, no Markdown fences or multiline objects, and terminate appended records with newlines.

Required fields:

Use strings for IDs, timestamps, summaries, before/after assertions and reasons; `schema_version` is an integer. `code_refs` and `doc_refs` are arrays. Each code reference uses string paths/symbols, a string or null revision, and a string scope when present. A null revision requires scope. Keep unknown reasons explicit rather than fabricating intent.

| Field | Meaning |
| --- | --- |
| `schema_version` | `1` |
| `event_id` | Stable unique ID, persisted before append |
| `run_id` | Workflow run responsible for the record |
| `recorded_at` | Write time, ISO 8601 with explicit offset |
| `type` | `business_change` or `correction` |
| `summary` | Concise business meaning |
| `before`, `after` | Supported behavior; for corrections, inaccurate and corrected assertions |
| `reason` | Known business rationale or bug cause; explicitly state when unknown |
| `code_refs` | Array of objects with `path`, relevant `symbol`, `revision` and optional `scope` |
| `doc_refs` | Array of paths relative to `business_docs/` for related knowledge or detailed explanations |

Use `revision` for the actual inspected commit. For uncommitted code use `revision: null` and a `scope` identifying the working tree, run and base commit when available; do not imply the base commit contains those edits. Before references may use historical revisions.

Optional fields: `effective_at` when independently known, `verification` for checks and limitations, and `corrects_event_id` (required for corrections). Keep write time separate from business activation. Omit unknown activation time; never infer it or deployment from commits or checks.

Example event:

```jsonl
{"schema_version":1,"event_id":"run-42:refund-window","run_id":"run-42","recorded_at":"2026-10-28T21:30:00+08:00","type":"business_change","summary":"Extend the refund application window","before":"Applications accepted within 7 days","after":"Applications accepted within 15 days","reason":"Support the agreed refund policy","code_refs":[{"path":"src/refunds/policy.ts","symbol":"canApplyForRefund","revision":null,"scope":"working tree for run-42; not deployed"}],"doc_refs":["modules/refunds.md"],"verification":{"checks":["Refund policy unit tests passed"],"limitations":["Deployment and activation not verified"]}}
```

### Append, Retry and Correct

1. Identify independent events from verified changes. Persist a semantic key and stable ID for each before writing. A run may have several events; `run_id` alone is insufficient as an event ID.
2. Look up relevant existing events by ID, run, business terms or references. Reuse an existing event instead of duplicating it. Recover identities from the log if an append succeeded but the state update failed.
3. Serialize shared-log writes through workflow locking or a single writer. Recheck identity after acquiring write access; append without overwriting concurrent records. If serialization cannot be ensured, defer append and report it pending; supported Markdown updates may proceed.
4. Validate records before append and the resulting log afterward. Stream large logs for structural checks rather than loading all event prose into context. Do not silently discard or rewrite malformed existing records; report them and defer append until safe.
5. Save write results to state. After interruption between docs and event writes, reconcile actual contents rather than skipping work because an assessment was saved.
6. Correct an inaccurate event by appending a `correction` referencing it, with a stable correction ID across retries. Keep the original intact. A later legitimate business change is a new `business_change`, not a correction.

Preserve Markdown change history without bulk conversion or duplicate historical events. Link detailed Markdown explanations from JSONL. Pure documentation repairs do not require events.
