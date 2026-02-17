# WORKFLOW.template.md - human-readable workflow description

My usual workflow is:

1. Prepare the `TODO.md` with the task
2. Agent prepares the test cases for it in the `TESTS.md`
3. Review the tests and coverage, probably multiple turns
4. After conformation the Agent implements tests
5. After review of the tests the Agent begins the multi-turn `PLAN.md` preparation
6. After review of the `PLAN.md` by the user, Agent begins the implementation
   - After each subtask finishing the Agent calls for the subagent to append the report into the `REPORT.md` 
7. After the `PLAN.md` completed, tests are fine and changes are reviewed by Codex and user Agent prepares the `SUMMARY.md`

```tree
TODO
├── index.md # the tree of the TODO folder up to a first level of subtasks
├── {YYMMDD:date}-{todo_taskname}-{status:Literal[tbd|in_progress|finished|blocked|cancelled]}
│   ├── TODO.md     # being created at the moment the task appears
│   ├── TESTS.md    # created and discussed before test implementation
│   ├── PLAN.md     # created and discussed before implementation and after test preparation
│   ├── REPORT.md   # being appended with new data on the end of each todo or plan paragraph
│   ├── SUMMARY.md  # summary of useful information, being prepared after todo is finished
│   └── subtasks
│       ├── {YYMMDD:date}-{todo_subtaskname}-{status:Literal[tbd|in_progress|finished|blocked|cancelled]}
│       │   ├── subtasks
│       │   │   ├── ... # this won't go into the `index.md`
│       │   │   ...
│       │   ├── TODO.md
│       │   ├── TESTS.md
│       │   ├── PLAN.md
│       │   ├── REPORT.md
│       │   └── SUMMARY.md
│       ├── {YYMMDD:date}-{todo_subtaskname}-{status:Literal[tbd|in_progress|finished|blocked|cancelled]}
│       ...
├── {YYMMDD:date}-{todo_taskname}-{status:Literal[tbd|in_progress|finished|blocked|cancelled]}
│   ├── ...
... ...
```

## TODO format
```xml-without-root
<meta>
    Created : YY-MM-DD HH:MM
    Modifier: YY-MM-DD HH:MM
    Model: {agent_name:Literal[Codex|ClaudeCode|OpenCode|...]} : {modelname}
    User : {github_name}
</meta>
<context>
    Background information, required or possibly useful for the task
</context>
<goal>
    <short_term>
        Purpose of this task
    </short_term>
    <long_term>
        Optional: long-term purpose of this task
    </long_term>
</goal>
<task>
    <formulation>
        The formal statement of the problem
    </formulation>
    <todo_paragraph0_name>
        Optional: The paragraph 0
    </todo_paragraph0_name>
    <todo_paragraph1_name>
        Optional: The paragraph 1
    </todo_paragraph1_name>
    ...
    <todo_paragraphN_name>
        Optional: The paragraph N
    </todo_paragraphN_name>
</task>
```

## TESTS format
```xml-without-root
<meta>
    Created : YY-MM-DD HH:MM
    Modifier: YY-MM-DD HH:MM
    Model: {agent_name:Literal[Codex|ClaudeCode|OpenCode|...]} : {modelname}
    User : {github_name:Optional}
</meta>
<test_index>
    - [ ] todo list with a links to the [test](line_N)
</test_index>
<test_group_1>
    <group_description>
        These tests covers specific part of the planned changes. Description of which.
    </group_description>
    <test0>
        <case>
            What's the case are we testing
        </case>
        <description>
            How are we testing it
        </description>
    </test0>
    ...
    <testN>
        <case>
            What's the case are we testing
        </case>
        <description>
            How are we testing it
        </description>
    </testN>
</test_group_1>
...
<test_group_N>
    ...
</test_group_N>
```
NB: the todo list should be updated after test implementation. 
If required, the test large groups could be moved lower in the hierarchy into the `subtasks` subfolder near `TODO.md` and `TESTS.md`.
**NB2: before implementation the `TESTS.md` should be reviewed by the user.**

## PLAN format
```xml-without-root
<meta>
    Created : YY-MM-DD HH:MM
    Modifier: YY-MM-DD HH:MM
    Model: {agent_name:Literal[Codex|ClaudeCode|OpenCode|...]} : {modelname}
    User : {github_name:Optional}
</meta>
<plan>
    - [ ] The concise todo list with a link to the [subtask0 field](line_N)
    - [ ] ...
    - [ ] The concise todo list with a link to the [subtaskN field](line_N)
</plan>
<subtask_name0>
    The first subtask description
</subtask_name0>
...
<subtask_nameN>
    The Nth subtask description
</subtask_nameN>
```
NB: the todo list should be updated after subtask completion. 
If required, the large subtasks could be moved lower in the hierarchy into the `subtasks` subfolder near `TODO.md`, `TESTS.md` and `PLAN.md`.
**NB2: before implementation the `PLAN.md` should be reviewed by the user.**

## REPORT format
```xml-without-root
<meta>
    Created : YY-MM-DD HH:MM
    Modifier: YY-MM-DD HH:MM
    Model: {agent_name:Literal[Codex|ClaudeCode|OpenCode|...]} : {modelname}
    User : {github_name:Optional}
</meta>
<subtask_name0>
    What had been done, what was tried, what are the results, what failed and why
</subtask_name0>
...
<subtask_nameN>
    What had been done, what was tried, what are the results, what failed and why
</subtask_nameN>
```
NB: each `<subtask_name>...</subtask_name>` field should be appended right after finishing this subtask

## SUMMARY format
```xml-without-root
<meta>
    Created : YY-MM-DD HH:MM
    Modifier: YY-MM-DD HH:MM
    Model: {agent_name:Literal[Codex|ClaudeCode|OpenCode|...]} : {modelname}
    User : {github_name:Optional}
</meta>
<status_cause>
    Task was finished / cancelled / is blocked due to ...
</status_cause>
<report_summary>
    <short_version>
        Concise report
    </short_version>
    <fails>
        What was tried and failed and why
    </fails>
    <findings>
        <useful>
            Useful tips for the future
        </useful>
        <important>
            Important findings that probably should be included into the `AGENTS.md` / `CLAUDE.md`
        </important>
    </findings>
</report_summary>
```
NB: created after implementation finish and conformation from both tests and user that everything works fine.
If in the prototyping stage and comprehensive testing is impossible - requires Codex review, and then user's review + conformation. 
