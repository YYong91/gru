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

### Step 2: Write missing tests and run full test suite

작업 내용에 맞는 테스트가 없으면 먼저 작성한다. 그 다음 전체 테스트를 실행한다.

**테스트 실행 (프로젝트별):**

- **Backend**: `cd {project_path}/backend && pytest` (또는 `uv run pytest backend/tests/`)
- **Frontend**: `cd {project_path}/frontend && npm run test:run`

**실패 시 처리 규칙:**
- 실패한 테스트 원인을 파악하고 코드를 수정한다
- 수정 후 다시 전체 테스트를 실행한다
- 통과할 때까지 반복한다 (무한 루프 방지: 최대 3회 수정 시도 후에도 실패하면 보고서에 기록하고 계속 진행)
- 테스트가 통과하면 수정사항을 커밋한다:
  ```bash
  git add -p  # 변경된 파일만 선택적으로 스테이징
  git commit -m "test: {작업명} 테스트 추가 및 버그 수정"
  ```

### Step 3: Push to remote

전체 테스트 통과 후, push되지 않은 커밋을 모두 push한다.

```bash
git push origin HEAD
```

실패하면 에러를 보고서에 기록하고 계속 진행한다 (push 실패해도 보고서는 작성).

### Step 4: Summarize work done

Based on conversation history, generate a concise summary autonomously. Do NOT ask the user.
테스트 결과(몇 개 통과, 새로 작성한 테스트 수)도 요약에 포함한다.
`리뷰 필요` is always `yes` — Gru will decide on merge/deploy.

### Step 5: Write report

Create `~/.claude/org/reports/{YYYY-MM-DD}-{project}-{task-slug}.md`:

```markdown
## 완료 보고서

**작업:** {task_description}
**프로젝트:** {project}
**완료일시:** {YYYY-MM-DD HH:MM}
**결과:** {summary}
**테스트:** backend {N}개 통과 / frontend {N}개 통과 (신규 {N}개 추가)
**PR:** {pr_link or "없음"}
**남은 것:** {remaining or "없음"}
**리뷰 필요:** yes
```

### Step 6: Remove from active.md

Read active.md. Remove the matching `### [project] task` section.
Write back.

### Step 7: Send Telegram notification

Load credentials from `~/.claude/org/.env` (source the file or read TELEGRAM_BOT_TOKEN and TELEGRAM_CHAT_ID).

Run via Bash:
```bash
source ~/.claude/org/.env
MESSAGE="✅ 미니언 완료: [{project}] {task_description}
결과: {one-line summary}
테스트: {test_summary}
리뷰 필요: yes — /gru:review 로 확인하세요"
curl -s -X POST "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage" \
  -d chat_id="${TELEGRAM_CHAT_ID}" \
  --data-urlencode "text=${MESSAGE}"
```

### Step 8: Confirm

Show: "✅ 보고서 저장됨: reports/{filename}"
Show: "Gru에게 알림 전송됨. `/gru:review` 로 확인 가능합니다."
