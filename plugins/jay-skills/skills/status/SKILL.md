---
name: status
description: 지금 git 저장소가 무슨 상태인지 쉬운 한국어로 알려주는 진단 도구. "지금 어떤 상태예요?" / "뭔가 꼬인 것 같아요" / "제 작업 어디 갔어요?" / "이 에러가 뭐예요?" / git 관련해서 어디서부터 시작할지 모를 때 먼저 실행. 읽기만 하고 아무것도 바꾸지 않아요. 결과에 따라 알맞은 스킬(push, pull, fix-conflict, undo, github-login, review-pr)로 안내합니다.
---

# 지금 상태 보기 (status)

비개발자를 위한 git 진단 도구. **읽기만 한다 — 어떤 파일도, 어떤 이력도 바꾸지 않는다.**
사용자는 원인이 아니라 증상을 말한다("뭔가 이상해요"). 이 스킬이 증상을 상태로
번역하고, 알맞은 스킬로 넘긴다.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md`를 읽는다 — 어휘표(§1), 마무리
카드(§2), 에러 번역표(§3)를 그대로 쓴다.

## 언제 쓰나

- "지금 뭐가 어떻게 된 거예요?" / "꼬인 것 같아요" / "제 작업 어디 갔어요?"
- 빨간 에러 메시지를 봤는데 무슨 말인지 모를 때
- 다른 git 스킬을 쓰기 전에 상황부터 확인하고 싶을 때

## Process

### Step 1: 조용히 상태 수집 (전부 읽기 전용)

```bash
git rev-parse --git-dir 2>&1
git status --porcelain=v2 --branch 2>&1
git rev-parse --abbrev-ref HEAD 2>&1
ls .git/MERGE_HEAD .git/rebase-merge .git/rebase-apply 2>/dev/null
git remote get-url origin 2>&1
git stash list 2>&1
git log --oneline -5 2>&1
command -v gh >/dev/null && gh auth status 2>&1     # 참고용 (판정은 --hostname 으로)
```

**네트워크는 건드리지 않는다** — `git fetch`도 하지 않는다. 그래서 "팀 저장소의 최신
상황"은 알 수 없고, 그 사실을 정직하게 말한다: "팀 저장소에 새로 올라온 게 있는지까지
보려면 `/jay-skills:pull`을 실행하세요."

사용자가 에러 메시지를 함께 붙여넣었다면 `git-collab.md` §3 번역표로 해석한다.

### Step 2: 특수 상태 먼저 판정

아래에 해당하면 그 안내부터 하고 Step 3으로 간다.

**(a) git 저장소가 아님** (`not a git repository`)

> "이 폴더는 아직 git으로 관리되고 있지 않아요. git은 작업 이력을 남기고 되돌릴 수 있게
> 해주는 도구인데, 이 폴더엔 그 기록장이 없는 상태예요.
> - 템플릿 zip을 풀어서 시작하신 거라면 → `/jay-skills:welcome`을 먼저 실행하세요.
> - 원래 GitHub에 있는 프로젝트인데 **웹에서 'Download ZIP'으로 받으셨다면** 이력이
>   통째로 빠진 사본이라 push가 안 돼요. 저장소를 만든 동료에게 clone(이력까지 통째로
>   복사해 오는 것) 주소를 물어보시는 게 가장 빠릅니다."

**(b) merge 중 / rebase 중** (`.git/MERGE_HEAD` 또는 `rebase-*` 존재)

→ 상세 설명 없이 `/jay-skills:fix-conflict`로 넘긴다. 그 스킬이 중단·정리를 모두 담당한다.

**(c) 기록 열람 모드** (detached HEAD)

> "지금 '기록 열람 모드'예요 — 예전 시점을 들여다보는 상태라, 여기서 커밋해도 어느
> 브랜치에도 붙지 않고 나중에 찾기 어려워집니다."

커밋 안 된 변경이 있으면 구조 옵션을 AskUserQuestion으로 제시한다: ① 지금 작업을 새
브랜치에 안전하게 담기(`git switch -c rescue/{날짜}`) ② 원래 브랜치로 switch해서
돌아가기(변경은 함께 따라감) ③ 그대로 두고 팀에 물어보기. 깨끗하면 ①은 빼고 제시한다.

### Step 3: 상태를 쉬운 한국어로 보고

숫자는 전부 풀어서 말한다 — `ahead 2, behind 3` 같은 표현을 쓰지 않는다.

```
### 지금 상태
- 지금 브랜치: `feature/notice-list` (main이 아니라 안전한 작업 공간이에요)
- 커밋 안 된 변경: 3개 파일
- 내 컴퓨터에만 있는 커밋: 2건 (아직 push 안 함)
- 팀 저장소: 연결됨 (github.com/회사/저장소)
- stash(잠깐 치워둔 작업): 1개 — "표 정렬 작업 중"
```

각 줄은 있는 것만 쓴다. 없는 항목은 아예 빼거나 "없음"으로 한 줄.

### Step 4: 다음 행동 안내 (라우팅)

수집한 상태에 맞는 것만 고른다:

| 관찰된 상태 | 안내 |
|---|---|
| 커밋 안 된 변경 있음 | `/jay-skills:commit` — 지금까지 작업을 커밋 |
| 내 컴퓨터에만 있는 커밋 있음 | `/jay-skills:push` — 팀 저장소로 push |
| 팀 저장소 미연결 | `/jay-skills:welcome` 또는 `/jay-skills:push`(주소 물어봄) |
| `gh` 미설치·미로그인 + 올릴 게 있음 | `/jay-skills:github-login` |
| 충돌 · merge 중 | `/jay-skills:fix-conflict` |
| 뭔가 잘못 커밋한 것 같다 | `/jay-skills:undo` |
| 내가 push한 작업을 확인받고 싶다 | `/jay-skills:pr` |
| 동료가 보낸 PR을 봐야 한다 | `/jay-skills:review-pr` |
| 에러 메시지가 git과 무관 | `/jay-skills:explain` |

`git-collab.md` §2 마무리 카드로 끝낸다.

## Rules

- **아무것도 바꾸지 않는다.** 파일 수정, 커밋, 네트워크 접근 전부 금지. 이 스킬은
  "안심하고 눌러보는 버튼"이어야 한다.
- raw git 출력을 그대로 보여주지 않는다 — 사용자가 원문을 요청한 경우만 예외.
- 상태가 깨끗하면 그것도 명확히 말한다: "다 정리돼 있어요. 지금 할 일은 없습니다."
- 상태를 단정하기 어려우면 추측하지 말고 그대로 말한 뒤, 멈추고 팀에 물어보기를 권한다.
