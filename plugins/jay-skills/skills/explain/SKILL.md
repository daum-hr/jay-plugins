---
name: explain
description: Get a plain-language explanation of any code, file, error message, or change. Useful when something looks confusing or scary.
---

# Explain Skill

비개발자를 위한 도구입니다. Takes anything confusing — a file, a code snippet, a red error message, a recent change — and explains it in plain language so you know whether to worry, fix it yourself, or stop and ask a teammate.

## Usage
```
/jay-skills:explain {file path}
/jay-skills:explain "{error message or question}"
/jay-skills:explain                        # Explains the most recent change
```

Examples:
- `/jay-skills:explain src/app/notice/page.tsx`
- `/jay-skills:explain "TypeError: Cannot read properties of undefined"`
- `/jay-skills:explain` (after a commit, to understand what just happened)

## Process

### Step 1: Figure Out What the User Gave You
- A path → read the file
- An error message (often has words like `Error`, `failed`, `undefined`) → analyze it
- A free-form question → answer it directly
- Empty / no input → look at recent git changes (`git diff HEAD~1`)

### Step 2: Read the Relevant Context
Use Read, Grep, or `git diff` to gather what you need. Don't dump huge file contents on the user — read silently and summarize.

### Step 3: Produce a 3-Section Response

Always use this structure:

```markdown
## 이게 뭔가요 (What this is)
{1-2 sentences in plain Korean/English. No jargon — or if you must use a technical word, explain it like "이 파일은 '템플릿(template)' 즉 화면의 뼈대를 만드는 코드예요."}

## 왜 중요한가요 (Why it matters)
{What would happen if it weren't there, or what business purpose it serves.
Example: "이 파일이 없으면 공지사항 페이지가 열리지 않아요."}

## 안심하고 바꿀 수 있는 것 (What you can safely change)
- {Safe edit 1 — usually text, labels, colors, dates}
- {Safe edit 2}

또는 (or):

이 파일은 혼자 고치지 마시고 팀과 함께 보는 게 좋아요. 이유: {brief reason}
```

### Step 4 (Only If Input Is an Error): Add "How to Fix It"

If the user pasted an error message, add a fourth section:

```markdown
## 해결 방법 (How to fix it)
1. {Concrete step 1 — e.g., "페이지를 새로고침 해보세요"}
2. {Step 2 — e.g., "그래도 안 되면 마지막에 바꾼 파일에서 X를 확인해 보세요"}
3. {Step 3 — 여기까지 해도 안 되면 멈추고 팀에 물어볼 시점}
```

## Rules

- **Never use jargon without explaining it.** If a word like "props", "API", "라우팅" appears, add a short parenthetical: "props (속성 값들)".
- Keep each section to 2-4 sentences. This is a quick explanation, not a tutorial.
- If something is genuinely risky to change (auth, database, env files), say so clearly: "이 파일은 잘못 고치면 전체가 멈출 수 있어요. 혼자 바꾸지 마시고 팀과 상의해 주세요." (개발자가 없는 팀을 가정한다 — "개발자에게 맡기세요"는 답이 되지 않는다.)
- Korean is the preferred output language. Use English for actual code/file names only.
- Do not just paste back the file content. The user can already see it — they need an interpretation.
- If you genuinely don't know what the input is, ask one short question: "이게 어떤 파일/메시지인지 한 줄로 알려주실 수 있나요?"
