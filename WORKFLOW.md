# WORKFLOW.md - Canonical Task Workflow
This file is the **single source of truth** for the repo’s agent workflow.  
`AGENTS.md` / `CLAUDE.md` should include a concise version of this workflow and link back here for full details + schemas.

## Goals
- Make agent work predictable: **spec -> tests -> plan -> implement -> verify -> summarize**
- Ensure review/approval happens at explicit **stop points**
- Keep a durable audit trail per task (plans, decisions, failures, verification)

---

## Directory & Naming Conventions

### Task root
All tasks live under:
- `TODO/` (execution units and their artifacts)
- (optional) `ISSUES/` (problem statements / requests / RFC-like docs)

### Task folder naming
```
TODO/{id}-{task_slug}-{status}/
```

**Recommended `{id}`**: `YYMMDD-SEQ` (e.g., `260217-01`)

**`task_slug`**: lowercase, words separated by underscores.

### Allowed statuses
`Literal[tbd | in_progress | finished | blocked | cancelled]`

### Status meaning & transitions (hard rules)

| Status       | Meaning | Allowed transitions | When to set |
|--------------|---------|---------------------|-------------|
| `tbd`        | Task exists; no execution has started | -> `in_progress`, `cancelled`, `blocked` | Immediately when folder is created |
| `in_progress`| Execution has started (tests and/or code changes in motion) | -> `finished`, `blocked`, `cancelled` | **When implementing tests begins** |
| `blocked`    | Work cannot proceed without external input/decision | -> `in_progress`, `cancelled` | When a blocking dependency appears |
| `cancelled`  | Task stopped intentionally | (terminal) | When user/agent decides to stop |
| `finished`   | Done + verified + summarized | (terminal) | After verification + `SUMMARY.md` completed |

**Terminal rule**: `finished/` and `cancelled/` folders are historical records and must not be deleted.

---

## Required Files Per Task Folder

Each task folder contains:

- `TODO.md` - what/why + acceptance criteria (created first)
- `TESTS.md` - **test design/spec**, reviewed before test code is written
- `PLAN.md` - implementation plan, reviewed before implementation begins
- `REPORT.md` - append-only progress log (updated as work proceeds)
- `SUMMARY.md` - final outcome + verification steps + key findings
- `subtasks/` - optional nested tasks (same structure)

> Note: `TESTS.md` is a *test plan/specification*. Actual test code lives in the repository’s normal test suite.

---

## TODO/index.md
`TODO/index.md` is a lightweight catalog of tasks.

**Update rules**
- Update `index.md` when:
  - a new task folder is created
  - a task changes status
  - a task is finished/cancelled/blocked
- Include only:
  - top-level tasks
  - first-level subtasks (direct children of a task’s `subtasks/`)
- Do not include deeper nesting.

**Suggested format (example)**
- `- [status] [id-task] (last_modified) - one-line description`

---

## Execution Workflow (Stop Points Included)

### 0) Task creation (user + agent)
1. Create task folder in `TODO/...-tbd/`
2. Create `TODO.md` with context + goals + **acceptance criteria**
3. (Optional) Link task to an issue in `ISSUES/` (recommended)

### 1) Test design phase (agent drafts, user approves)
1. Agent drafts `TESTS.md` (test cases, coverage, approach)
2. **STOP POINT A:** Agent must ask the user to review/approve `TESTS.md`
3. Iterate until approved

### 2) Test implementation phase (agent executes, user reviews if desired)
4. Rename/move folder status to `in_progress` when test implementation begins
5. Agent implements test code in the repo test suite
6. Agent updates the `TESTS.md` checklist to reflect what’s implemented
7. Optional review of test code/results by user

### 3) Planning phase (agent drafts, user approves)
1. Agent drafts `PLAN.md` as a checklist of small, verifiable steps
2. **STOP POINT B:** Agent must ask the user to review/approve `PLAN.md`
3. Iterate until approved

### 4) Implementation phase (agent executes; report as you go)
1. Agent implements changes following `PLAN.md`
2. After completing each plan item:
   - update `PLAN.md` checklist state
   - append a corresponding entry to `REPORT.md`
3. **Subagent reporting (optional):**
   - after each completed plan item, call a lightweight subagent to append to `REPORT.md`
   - if subagent is unavailable, append manually and note the fallback in `REPORT.md`

### 5) Verification & completion
1. Run relevant checks (tests + lint/format/type-check where applicable)
2. Ensure acceptance criteria in `TODO.md` are satisfied
3. Agent drafts `SUMMARY.md` (outcome, how to verify, key findings)
4. **STOP POINT C (recommended):** user confirms results
5. Rename/move task folder status to `finished` (or `blocked/cancelled` with rationale)

---

## Definition of Done (DoD)
A task can be marked `finished` only when:
- All required files exist: `TODO.md`, `TESTS.md`, `PLAN.md`, `REPORT.md`, `SUMMARY.md`
- Acceptance criteria are satisfied (or explicitly deferred with rationale)
- Verification commands have been run (or inability to run is documented)
- `SUMMARY.md` contains:
  - what changed and why
  - how to verify (commands)
  - risks/limitations
  - key findings worth promoting into `AGENTS.md` / `CLAUDE.md`

---

## Document Formats (Schemas)

### Shared meta (for all docs)
Use ISO 8601 timestamps where possible.

```xml
<meta>
  <created>YYYY-MM-DDTHH:MM:SSZ</created>
  <modified>YYYY-MM-DDTHH:MM:SSZ</modified>
  <agent>
    <name>Codex|ClaudeCode|OpenCode|Other</name>
    <model>model-identifier</model>
  </agent>
  <user>github_name_or_handle</user>
  <issue_ref>optional path or URL to ISSUES entry</issue_ref>
</meta>
```

---

## TODO.md format
```xml
<todo>
  <meta>...</meta>

  <context>
    Background information required or useful for the task.
  </context>

  <goal>
    <short_term>Purpose of this task.</short_term>
    <long_term>Optional: long-term purpose.</long_term>
  </goal>

  <acceptance_criteria>
    <item>Observable, testable outcome #1.</item>
    <item>Observable, testable outcome #2.</item>
    ...
    <item>Observable, testable outcome #N.</item>
  </acceptance_criteria>

  <non_goals>
    <item>Optional: explicitly out of scope.</item>
  </non_goals>

  <task>
    <problem_statement>
      A formal statement of what must change.
    </problem_statement>

    <!-- Optional structured paragraphs (stable IDs recommended) -->
    <section id="p0" name="paragraph0_name">...</section>
    <section id="p1" name="paragraph1_name">...</section>
  </task>
</todo>
```

---

## TESTS.md format (test design/spec, not code)
```xml
<tests>
  <meta>...</meta>

  <test_index>
    <item id="t1" checked="false">Short name of test case/group</item>
    ...
    <item id="tN" checked="false">...</item>
  </test_index>

  <test_group id="g1">
    <group_description>
      What part of the change this group covers.
    </group_description>

    <test_case id="t1">
      <case>What scenario are we testing?</case>
      <description>How we will test it (unit/integration/e2e, mocks, fixtures).</description>
      <expected>What is the expected result?</expected>
    </test_case>
	...
    <test_case id="tN">
      <case>What scenario are we testing?</case>
      <description>How we will test it (unit/integration/e2e, mocks, fixtures).</description>
      <expected>What is the expected result?</expected>
    </test_case>
  </test_group>
</tests>
```

**Rules**
- `TESTS.md` must be user-reviewed before writing test code (**STOP POINT A**).
- After implementing tests, update `test_index` to mark which are implemented (e.g., `checked="true"`).

If a test group becomes large, you may move it into a subtask under `subtasks/` and keep `TESTS.md` at the parent level as an index.

---

## PLAN.md format
```xml
<plan_doc>
  <meta>...</meta>

  <plan>
    <item id="p1" checked="false">Concise step 1</item>
    <item id="p2" checked="false">Concise step 2</item>
  </plan>

  <subtask id="p1">
    Description of step 1, including files to change, approach, and edge cases.
  </subtask>

  <subtask id="p2">
    Description of step 2...
  </subtask>
</plan_doc>
```

**Rules**
- `PLAN.md` must be user-reviewed before implementation (**STOP POINT B**).
- Update `checked="true"` as each plan item is completed.
- Large subtasks may be split into `subtasks/` folders.

---

## REPORT.md format (append-only log)
```xml
<report>
  <meta>...</meta>

  <entry ref_id="p1">
    <what_changed>What was done (concrete changes).</what_changed>
    <result>What happened (tests, behavior).</result>
    <notes>Anything learned, decisions made.</notes>
    <failures>What failed and why (if any).</failures>
    <timestamp>YYYY-MM-DDTHH:MM:SSZ</timestamp>
  </entry>

  <entry ref_id="p2">
    ...
  </entry>
</report>
```

**Rules**
- Append an `<entry>` immediately after finishing each `PLAN.md` item.
- If using a subagent: note it; if fallback to manual: note it.

---

## SUMMARY.md format (final outcome + verification)
```xml
<summary>
  <meta>...</meta>

  <status_cause>
    finished | cancelled | blocked - explain why and what remains (if anything).
  </status_cause>

  <report_summary>
    <short_version>Concise summary of what changed and why.</short_version>

    <verification>
      <command>Exact command(s) to verify (tests/lint/run).</command>
      <notes>Environment assumptions or limitations.</notes>
    </verification>

    <fails>
      What was tried and failed and why (if relevant).
    </fails>

    <findings>
      <useful>Tips for future work.</useful>
      <important>
        Findings that should be promoted into AGENTS.md / CLAUDE.md.
      </important>
    </findings>
  </report_summary>
</summary>
```

**Rules**
- Write `SUMMARY.md` only after implementation is complete and verification is done (or limitations documented).
- For prototyping situations where full testing is impossible:
  - require explicit user confirmation before marking finished
  - document the verification limitations in `<verification>`.

---

## Notes for AGENTS.md / CLAUDE.md extraction
When generating `AGENTS.md` / `CLAUDE.md` from this file, include:
- the lifecycle steps + stop points
- statuses + transition rules
- required files list
- “Definition of Done”
and link to `WORKFLOW.md` for full schemas.
