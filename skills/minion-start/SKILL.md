---
name: minion-start
description: Use when a Minion session is ready to begin its assigned task. Claims the task, outlines the plan, and begins autonomous execution.
---

# Minion Start

배정된 작업을 시작합니다. 작업을 점유하고 자율 실행 모드로 진입합니다.

## Process

### Step 1: Find assigned task

The command may be invoked as `/minion:start` or `/minion:start {N}` where N is a session number.

Read `~/.claude/org/active.md` and find the task to claim:

**If session number N is provided** (`/minion:start 1`, `/minion:start 2`, etc.):
- Find the task with `session: {N}` in active.md
- If not found: "세션 {N}에 배정된 작업이 없습니다."
- If found but `status: in_progress`: "세션 {N}은 이미 진행 중입니다. `/minion:done` 으로 완료 처리 후 재시작하세요."

**If no session number provided** (backward compatibility):
- Find task where `project_path` matches current working directory AND `status: assigned`
- If none found: "배정된 작업이 없습니다. `/gru:assign` 으로 작업을 배정받으세요."
- If `status: in_progress` task found: "이미 진행 중인 작업이 있습니다. `/minion:done` 으로 완료 처리 후 새 작업을 시작하세요."

### Step 2: Claim the task

In `active.md`, update the matching section:
- Change `status: assigned` → `status: in_progress`
- Add `started_at: {YYYY-MM-DD HH:MM}`

### Step 3: Analyze and outline plan

Based on the task description, briefly output:

```
## 🚀 작업 시작: {task_title}

### 실행 계획
1. {step 1}
2. {step 2}
...

시작합니다.
```

Keep the plan concise (3-5 steps max). Do NOT ask for confirmation.

### Step 4: Execute autonomously

Begin implementation immediately following the plan.

**자율 실행 규칙:**
- 사용자에게 질문하지 않는다
- 막히면 합리적인 선택을 하고 계속 진행한다
- 테스트가 실패하면 직접 수정한다
- 모든 구현 결정을 스스로 판단한다
- 완료되면 `/minion:done` 을 실행한다
