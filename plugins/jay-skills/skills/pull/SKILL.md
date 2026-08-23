---
name: pull
description: 동료들이 팀 저장소에 올린 최신 작업을 내 컴퓨터로 받아옵니다. "최신으로 받아줘" / "동료 작업 가져와줘" / 올리기가 "먼저 받아오세요(fetch first)"라며 거부됐을 때 사용. 받아오다가 충돌(같은 곳을 두 사람이 고침)이 나면 fix-conflict로 자연스럽게 이어집니다. 받아오기 전에 상황을 먼저 보여주고 진행해요.
---

# 최신 작업 받아오기 (pull)

비개발자를 위한 "동료 작업 가져오기". 받아오기 = 목록 확인(fetch) + 합치기(merge)인데,
사용자에게는 하나의 행동으로 보여준다.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md`를 읽는다 — 어휘(§1), 카드(§2),
에러 번역표(§3), 안전 기준선(§5).

## Process

### Step 1: 조용히 확인

```bash
git rev-parse --git-dir 2>&1
ls .git/MERGE_HEAD .git/rebase-merge .git/rebase-apply 2>/dev/null
git status --porcelain 2>&1
git rev-parse --abbrev-ref HEAD 2>&1
git remote get-url origin 2>&1
git fetch 2>&1
git log --oneline HEAD..@{u} 2>/dev/null   # 동료가 올린 것
git log --oneline @{u}..HEAD 2>/dev/null   # 내가 아직 안 올린 것
```

- 이미 합치는 중이면(`MERGE_HEAD` 등) 새로 받아오지 않는다 → `/jay-skills:fix-conflict`.
- 팀 저장소 미연결 → "받아올 곳이 아직 연결 안 돼 있어요" → `/jay-skills:welcome`.
- `fetch`가 인증 문제로 실패 → `/jay-skills:github-login`.

### Step 2: 받아오기 *전에* 상황을 먼저 말한다

숫자를 풀어서:

> "동료가 팀 저장소에 올린 새 저장이 {m}건 있어요. 내 컴퓨터에는 아직 안 올린 저장이
> {n}건 있고요. 받아오면 두 흐름이 하나로 합쳐집니다."

받아올 게 없으면 여기서 끝낸다: "이미 최신이에요. 받아올 게 없습니다." (마무리 카드)

### Step 3: 저장 안 된 변경이 있으면 고르게 한다

AskUserQuestion:

> "저장(commit) 안 된 변경이 {n}개 있어요. 이대로 받아오면 섞일 수 있어서 먼저 정리할게요.
> ① `/jay-skills:commit`으로 저장하기 (권장 — 기록이 남아서 가장 안전해요)
> ② 보관함에 잠깐 치워두기 (받아온 뒤 자동으로 다시 꺼내드려요)
> ③ 그만두기"

② → `git stash push -m "받아오기 전 임시 보관"`. 받아온 뒤 `git stash pop`으로 되돌리고,
그때 충돌이 나면 `/jay-skills:fix-conflict`로 넘기되 라벨을 "치워뒀던 내 작업" /
"방금 받아온 내용"으로 바꿔 쓰라고 알려준다.

**자동으로 저장하거나 자동으로 치워두지 않는다.**

### Step 4: 받아오기

```bash
git pull --no-rebase
```

`--no-rebase`를 항상 명시한다 — 사용자 컴퓨터의 설정이 재정렬(rebase) 방식으로 되어
있어도 이 도구는 언제나 합치기(merge) 방식으로 간다 (`git-collab.md` §5의 이유).

- **깨끗하게 합쳐짐** → Step 5.
- **충돌 발생** → 겁주지 말고 이렇게 알린다:

  > "충돌이 났어요. 고장이 아니라 '같은 곳을 두 사람이 다르게 고쳤으니 어느 쪽을
  > 남길지 정해달라'는 요청이에요. 하나씩 같이 골라볼게요."

  그리고 `/jay-skills:fix-conflict`의 절차를 그대로 이어서 진행한다 (그 SKILL.md를 읽고
  따른다 — 여기에 복사하지 않는다).

### Step 5: 마무리

```
### 지금 상태
- 동료 작업 {m}건이 내 컴퓨터에 합쳐졌어요. (치워뒀던 작업도 다시 꺼내뒀습니다.)
- 아직 팀에 안 올린 내 저장: {n}건

### 다음에 할 수 있는 것
- 내 작업을 올리려면: `/jay-skills:push`
- 무엇이 바뀌었는지 보려면: `/jay-skills:explain`
- 잘 모르겠으면: 여기서 멈추고 팀에 물어보기 — 지금 상태 그대로 두면 아무것도 망가지지 않아요
```

## 실패 갈래

- **이 갈래가 팀 저장소에 아직 없음** (`no tracking information`): "이 갈래는 아직 팀
  저장소에 없어서 받아올 게 없어요." main의 최신 내용을 이 갈래로 가져올지
  AskUserQuestion으로 묻는다(기본값 아니오). 예 → `git merge origin/main`, 충돌 시
  `/jay-skills:fix-conflict`.
- **인증 실패** → `/jay-skills:github-login`.

## 절대 하지 않는 것

- `git pull --rebase` / 어떤 형태의 rebase도.
- 저장 안 된 변경 위에 `checkout` / `restore` 덮어쓰기.
- 사용자에게 묻지 않고 저장하거나 보관함에 치우기.
- 충돌 조각을 임의로 고르기 (잠금 파일은 예외 — `git-collab.md` §6).
