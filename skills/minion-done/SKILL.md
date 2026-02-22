---
name: minion-done
description: Use when a Minion session has completed its assigned task. Writes a completion report and clears the task from active.md.
---

# Minion Done

작업 완료 보고서를 작성하고 active 목록에서 제거합니다.

## Process

### Step 1: Find current task

Read `~/.claude/org/active.md`. Find task whose `project_path` matches current working directory.
If not found: "현재 디렉토리에 배정된 작업이 없습니다. `/gru:assign` 으로 작업을 배정받으세요."

### Step 2: Summarize work done

Based on conversation history, generate a concise summary autonomously. Do NOT ask the user.
`리뷰 필요` is always `yes` — Gru will decide on merge/deploy.

### Step 3: Write report

Create `~/.claude/org/reports/{YYYY-MM-DD}-{project}-{task-slug}.md`:

```markdown
## 완료 보고서

**작업:** {task_description}
**프로젝트:** {project}
**완료일시:** {YYYY-MM-DD HH:MM}
**결과:** {summary}
**PR:** {pr_link or "없음"}
**남은 것:** {remaining or "없음"}
**리뷰 필요:** yes
```

### Step 4: Remove from active.md

Read active.md. Remove the matching `### [project] task` section.
Write back.

### Step 5: Send Telegram notification

Load credentials from `~/.claude/org/.env` (source the file or read TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID).

Run via Bash:
```bash
source ~/.claude/org/.env
MESSAGE="✅ 미니언 완료: [{project}] {task_description}
결과: {one-line summary}
리뷰 필요: yes — /gru:review 로 확인하세요"
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d chat_id="${TELEGRAM_CHAT_ID}" \
  --data-urlencode "text=${MESSAGE}"
```

### Step 6: Confirm

Show: "✅ 보고서 저장됨: reports/{filename}"
Show: "Gru에게 알림 전송됨. `/gru:review` 로 확인 가능합니다."
