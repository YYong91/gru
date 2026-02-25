---
name: gru-assign
description: Use when Gru wants to assign backlog tasks to active sessions. Picks top items and writes to active.md.
---

# Gru Assign

백로그 작업을 Active Minion으로 등록합니다.

## Process

### Step 1: Read backlog

Read `~/.claude/org/backlog.md`. Find all unchecked items (`- [ ]`).
If empty: "백로그가 비어있습니다. `/gru:plan` 으로 작업을 추가하세요."

### Step 2: Show options and get selection

Display unchecked items numbered. Ask user which to assign (can pick multiple).
Use AskUserQuestion with the list.

### Step 3: Determine project_path for each selected task

- If project is a specific app (e.g. `podo-budget`): use `~/Developer/{project}`
- If project is `전체공통` or spans multiple projects: ask "어느 디렉토리에서 작업할 예정인가요? (예: ~/Developer/podo-budget)"

### Step 4: Determine next session numbers

Read `~/.claude/org/active.md` and scan for all existing `session:` fields (e.g., `- session: 3`).
Find the highest session number. New tasks get sequential numbers starting from highest+1.
If no existing sessions, start from 1.

### Step 5: For each selected task, append to `~/.claude/org/active.md`

```
### [{project}] {description}
- session: {N}
- assigned: {YYYY-MM-DD HH:MM}
- project_path: {project_path}
- priority: {priority}
- status: assigned
- 작업 내용: {description}
- 세션 시작: `claude --dangerously-skip-permissions` ({project_path} 에서) → `/minion:start {N}`
```

### Step 6: Mark as assigned in backlog

In `backlog.md`, change `- [ ]` to `- [x]` for assigned items.

### Step 7: Confirm

Show a table of assigned tasks with their session numbers:
```
세션 | 프로젝트 | 작업
-----|---------|----
1    | ...     | ...
2    | ...     | ...
```

Then say:
"✅ {N}개 작업이 배정되었습니다. 각 세션 번호에 맞게 실행하세요:
- 세션 1: `{project_path}` 에서 `claude` → `/minion:start 1`
- 세션 2: `{project_path}` 에서 `claude` → `/minion:start 2`
..."
