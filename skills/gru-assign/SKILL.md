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

### Step 4: For each selected task, append to `~/.claude/org/active.md`

```
### [{project}] {description}
- assigned: {YYYY-MM-DD HH:MM}
- project_path: {project_path}
- priority: {priority}
- status: assigned
```

### Step 5: Mark as assigned in backlog

In `backlog.md`, change `- [ ]` to `- [x]` for assigned items.

### Step 6: Confirm

Show assigned tasks and say:
"✅ {N}개 작업이 배정되었습니다. 해당 디렉토리에서 `claude --dangerously-skip-permissions` 로 세션을 열고 `/minion:start` 를 실행하세요."
