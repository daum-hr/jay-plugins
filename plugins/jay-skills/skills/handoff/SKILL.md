---
name: handoff
description: Save your current session's state to .claude/handoff.md before clearing the conversation. Use this when you've worked for a while and want to take a break.
---

# Handoff Skill

비개발자를 위한 도구입니다. Saves a short note about what you've been doing so you can come back later (or hand off to a teammate) without losing track.

## Usage
```
/jay-skills:handoff
```

## When to Use
- You've been working for a while and want to take a break.
- The conversation is getting long and you want to `/clear` it.
- You're stopping for the day with something half-done.
- You need to hand off to a teammate.

## Process

### Step 1: Gather Context (Silent)
Look at what happened in this session:
```bash
git status --short
git log -5 --oneline
git diff --stat
```

Also check the conversation history for:
- What the user was trying to do
- What got finished
- What's still in progress
- Any questions the user asked that are still open

### Step 2: Draft a Friendly Summary

Write to `.claude/handoff.md` (overwrite if it exists):

```markdown
# 세션 인수인계 (Session Handoff)

**날짜**: YYYY-MM-DD
**브랜치**: {current branch}
**기준 커밋**: {git log -1 --format="%h (%ad)" --date=short 출력}

> 이 메모는 위 커밋까지의 상황이에요. 그 뒤에 새로 저장(커밋)된 작업이 있다면,
> 이 메모는 참고자료로만 보고, 최신 커밋 기록부터 확인해 주세요.

## 무엇을 하려고 했나요
{1 paragraph in plain language — what the user was working on}

## 끝낸 일
- {Completed item 1}
- {Completed item 2}

## 진행 중인 일
- {What's half-done, and where it stands}

## 다음에 해야 할 일
- [ ] {Next step 1}
- [ ] {Next step 2}

## 막혀서 물어봐야 할 것
- {Open questions, or things you're stuck on and need to ask a teammate about}

## 메모
{Any extra context for next time — file names, links, decisions}
```

### Step 3: Save
Write the file. No need to ask for confirmation — keep it simple.

### Step 4: Tell the User
Output, in plain language:

> "정리해서 `.claude/handoff.md`에 저장했어요.
> 이제 `/clear` 하셔도 괜찮습니다. 다음에 돌아오시면
> 'handoff.md 읽고 이어서 해줘' 라고 말씀하세요.
> 다만 그 사이에 새로 저장(커밋)한 작업이 있었다면, 이 메모는
> 참고자료로만 쓰고 최신 상황부터 봐 달라고 하시면 돼요."

## Rules
- Always overwrite `.claude/handoff.md` (one file, not date-versioned).
- Keep it under 60 lines — it's a note, not a report.
- Plain language. No jargon.
- Skip empty sections — if there are no open questions, don't include the heading.
- Never include passwords, tokens, or sensitive personal info.
