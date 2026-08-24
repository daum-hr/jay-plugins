---
name: handoff
description: 오늘 작업을 여기서 멈추고 나중에(또는 동료가) 이어서 할 수 있게 지금 상황을 .claude/handoff.md 파일로 남깁니다. "오늘 여기까지 할게요" / "정리해줘" / "인수인계 메모 남겨줘" / "내일 이어서 할 수 있게 저장해줘" / 대화를 지우기(/clear) 전에 사용. 파일로 남기는 게 핵심이에요 — 채팅으로만 정리하고 끝내지 않습니다. 커밋이나 push는 하지 않아요.
---

# Handoff Skill

비개발자를 위한 도구입니다. Saves a short note about what you've been doing so you can come back later (or hand off to a teammate) without losing track.

**이 스킬은 파일 하나(`.claude/handoff.md`)만 쓴다.** 커밋도 push도 하지 않는다. 커밋 안 된
변경이 있으면 "커밋 안 된 변경 {n}개 있음"으로 **기록만** 하고, 커밋할지는 사용자가 따로
정한다 (`references/git-collab.md` §5).

채팅으로 정리해서 말하는 것으로 끝내지 않는다 — **파일로 남기는 것이 이 스킬의 전부다.**
요청이 "오늘 여기까지 할게요" / "정리해줘" 처럼 들어와도 마찬가지다.

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

> 이 메모는 위 커밋까지의 상황이에요. 그 뒤에 새로 커밋된 작업이 있다면,
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

커밋 안 된 변경이 있으면 파일의 "진행 중인 일"에 한 줄로 적는다:
`- 커밋 안 된 변경 {n}개 있음 ({파일 목록})`. **여기서 커밋하지 않는다.**

### Step 4: Tell the User
Output, in plain language:

> "정리해서 `.claude/handoff.md` 파일로 남겼어요.
> 이제 `/clear` 하셔도 괜찮습니다. 다음에 돌아오시면
> 'handoff.md 읽고 이어서 해줘' 라고 말씀하세요.
> 다만 그 사이에 새로 커밋한 작업이 있었다면, 이 메모는
> 참고자료로만 쓰고 최신 상황부터 봐 달라고 하시면 돼요."

커밋 안 된 변경이 있었으면 그 사실도 한 줄로 덧붙인다: "커밋 안 된 변경이 {n}개 있어요 —
커밋해 둘까요? (`/jay-skills:commit`)" **묻기만 하고 실행하지 않는다.**

## Rules
- **Never commit or push.** 파일 하나만 쓰고 끝낸다. 사용자가 따로 "커밋해줘"라고
  말하면 그때 `/jay-skills:commit`.
- Always overwrite `.claude/handoff.md` (one file, not date-versioned).
- Keep it under 60 lines — it's a note, not a report.
- Plain language. No jargon.
- Skip empty sections — if there are no open questions, don't include the heading.
- Never include passwords, tokens, or sensitive personal info.
