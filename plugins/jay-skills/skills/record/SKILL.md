---
name: record
description: 중요한 작업·결정을 나중에 찾아볼 수 있게 docs/memories/ 에 기록으로 남깁니다. "기록으로 남겨줘" / "오늘 한 일 정리해서 기록해줘" / "이 결정 적어둬" 할 때 사용. 큰 변경이면 /jay-skills:commit 이 알아서 함께 불러줍니다.
---

# Record Skill

비개발자를 위한 도구입니다. Saves a short note about meaningful work so you (or a teammate) can find it later. Think of it like a project diary entry.

## Usage
```
/jay-skills:record                    # Auto-summarize recent work
/jay-skills:record "title in Korean"  # Use a custom title
```

This is also called automatically by `/jay-skills:commit` when you make a big change.

## When to Record (Done Automatically)
- A new page or feature was added
- A real problem was fixed (not a typo)
- An important decision was made (e.g., "we'll show announcements on the homepage")
- 5+ files were changed at once

## Process

### Step 1: Gather Context
```bash
git log -3 --oneline
git diff --name-only HEAD~1
```

Also pull from the conversation: what the user wanted, what got built, why.

### Step 2: Pick a Filename
```
docs/memories/YYYY-MM-DD_brief-title.md
```

Examples:
- `2026-04-27_onboarding-checklist-added.md`
- `2026-04-27_notice-list-sorting-rule.md`
- `2026-04-27_homepage-news-section-fix.md`

### Step 2.5: 겹치는 기록 확인 (Check for Overlap)

Before writing, search for earlier notes on the same topic:
```bash
grep -il "{핵심 키워드 1-2개}" docs/memories/*.md 2>/dev/null
```

**`docs/memories/` 폴더가 아직 없거나 비어 있으면** 이 명령은 아무것도 출력하지 않는다
(`2>/dev/null` 이 에러를 가린다). 그럴 때는 **겹침 없음으로 보고 그대로 Step 3으로**
간다 — 폴더가 있는지 따로 확인하거나 다른 명령으로 바꿔 시도할 필요가 없다.

- 겹치는 파일이 없거나, 있어도 모순되지 않으면 → 그대로 Step 3.
- 이전 기록과 **모순**되면 (예: 예전 노트 "매니저 승인 필요" ↔ 이번 변경 "자동 승인"):
  1. 이전 파일의 제목(H1) 바로 아래에 배너 **한 줄만** 추가:
     `> **대체됨 (SUPERSEDED by {새 파일명})** — {무엇이 바뀌었는지 한 줄}`
  2. 새 파일에 `**대체 (Supersedes)**: {이전 파일명}` 한 줄 표기.
  3. 이전 파일의 본문은 절대 고치지 않기 — 배너 한 줄이 전부예요.

### Step 3: Write a Plain-Language Note

Use this template (skip any section that doesn't apply):

```markdown
# {제목 / Title}

**날짜 (Date)**: YYYY-MM-DD
**종류 (Type)**: 새 기능 (feature) | 수정 (fix) | 결정 (decision)

## 무엇을 했나요 (Summary)
{1-2 sentences in plain language — what changed and why}

## 주요 변경 (Key Changes)
- {Change 1, in plain words}
- {Change 2}

## 다음에 기억할 것 (Notes for Next Time)
{Anything that'll help future-you or a teammate — links, gotchas, decisions made}
```

### Step 4: Save
Write the file (필요하면 `docs/memories/` 폴더를 함께 만든다). Briefly tell the user:

> "기록 저장했어요: `docs/memories/{filename}`"

## Rules
- Plain language, no jargon. If a technical term sneaks in, explain it in parentheses.
- Keep it short — usually under 30 lines.
- Don't ask the user to fill in fields. Infer everything from the conversation and git.
- If you genuinely can't tell what happened, ask one simple question: "이 변경을 한 줄로 어떻게 설명할까요?"
