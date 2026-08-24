---
name: review-pr
description: 동료가 올린 PR(확인 요청)을 검토하는 도구 — "이 PR 좀 봐줘" / "이 PR 봐줘" / "동료가 보낸 링크인데 확인해줘" / "리뷰해줘" / "이거 승인해도 돼요?" 할 때, 또는 GitHub PR 주소·번호를 받았을 때 사용합니다. 대화에 PR 링크가 등장하면 gh 명령을 직접 치지 말고 먼저 이 스킬을 실행하세요. 무엇이 어떻게 바뀌는지 쉬운 말로 요약하고, 겹치는 부분이 있으면 어느 파일인지 찾아주고, 문제없으면 승인·merge까지, 의견이 있으면 PR에 코멘트로 남겨드려요.
---

# 확인 요청 검토하기 (review-pr)

`pr` 스킬이 **올리는 쪽**이라면, 이 스킬은 **받는 쪽**이다. 동료가 "이거 좀 봐주세요"
하고 보낸 PR(확인 요청 — "내 작업을 main에 합쳐도 될까요"라고 올리는 요청)을 읽고,
승인하거나 의견을 남긴다.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md`를 읽는다 — 특히 §0(개발자 없는 팀:
확인도 merge도 팀이 직접 한다), 어휘(§1), 카드(§2), 에러 번역표(§3), 원격 명령(§3.6),
`gh` 확인 순서(§4), 안전 기준선(§5).

**이 스킬의 약속**: 코드 diff(바뀐 줄 원문)를 사용자에게 쏟아내지 않는다. 비개발자가
판단할 수 있는 것은 "무엇이 달라지는가"이지 "어느 줄이 바뀌었는가"가 아니다.

## Usage

```
/jay-skills:review-pr https://github.example.com/team/repo/pull/12
/jay-skills:review-pr 12
/jay-skills:review-pr          # 이 저장소에서 지금 열려 있는 요청 목록부터
```

## Step 1: 조용히 확인

`gh` 명령은 서버에 접속한다 — §3.6대로 **실행 전에 미리 알린다**: "확인 요청 내용을
불러올게요 — 몇 초 걸릴 수 있어요."

```bash
command -v gh >/dev/null && echo gh-ok || echo gh-missing
HOST=$(git remote get-url origin 2>/dev/null \
       | sed -E 's#^[a-z]+://##; s#^[^@/]*@##; s#[:/].*$##')
[ -n "$HOST" ] || HOST=github.com
gh auth status --hostname "$HOST" 2>&1
gh api user --hostname "$HOST" --jq .login 2>/dev/null     # 지금 내 계정
gh pr view <ref> --json title,body,author,files,additions,deletions,mergeable,mergeStateStatus,reviewDecision,url,baseRefName,headRefName 2>&1
```

- **`gh` 없음** → 설치 안내만 하고 끝낸다 (자동 설치 금지):
  > "확인 요청을 보려면 `gh`라는 GitHub 도구가 필요해요. **Mac**: `brew install gh`
  > / **Windows**: `winget install GitHub.cli`. 설치 후 다시 실행해 주세요."
- **미로그인** → `/jay-skills:github-login` → 끝나면 여기로 돌아온다.
- **연결 실패**(§3 맨 윗줄) → VPN 안내 후 **정지**. 로그인 문제로 넘기지 않는다.
- **`<ref>` 없이 실행됨** → `gh pr list` 로 열려 있는 요청을 보여주고 어느 것을 볼지
  AskUserQuestion으로 고르게 한다.
- **찾을 수 없음 / 404** → 주소를 다시 확인하도록 안내한다. 사내 서버의 요청 주소를
  github.com에서 찾으려 한 경우가 흔하다.

**본인 요청인지 확인한다.** `author` 가 `gh api user` 로 확인한 내 계정과 같으면 차단하지는
않되 반드시 고지한다:

> "이건 본인이 올린 확인 요청이에요. 승인은 동료가 해주는 게 의미가 있어요 — 내용 요약은
> 그대로 보여드릴게요."

## Step 2: 충돌 여부로 갈린다

`mergeable` 값을 먼저 본다. **충돌이 있으면 요약보다 이게 먼저다.**

- `CONFLICTING` **이 아니면** → Step 3.
- `CONFLICTING` **이면** → 링크만 돌려주지 말고 **무엇이 겹치는지 그 자리에서 보여준다.**

```bash
git -c http.lowSpeedLimit=1000 -c http.lowSpeedTime=15 fetch origin 2>&1
git merge-tree --write-tree --name-only origin/<baseRefName> origin/<headRefName> 2>&1
```

`git merge-tree` 는 **읽기 전용**이다 — 합쳐보기만 하고 내 작업 폴더와 브랜치는 하나도
건드리지 않는다. 출력을 읽는 법 두 가지를 반드시 지킨다:

1. **충돌 신호는 종료 코드다** (충돌 없으면 0, 있으면 0이 아님). 출력이 있다고 충돌인 게
   아니다.
2. **첫 줄은 파일명이 아니라 tree ID**(40자리 hex)다. **첫 줄은 버리고** 그다음 줄부터가
   겹치는 파일 목록이다. 첫 줄을 파일명으로 착각해 아래 명령에 넣지 않는다.

```bash
git log -1 --format='%an' -- <파일>      # 목록의 각 파일에 대해 (tree ID 제외)
```

**`origin/<baseRefName>` 이나 `origin/<headRefName>` 이 내 컴퓨터에 없으면**(fetch로도 안
따라온 브랜치) 이 명령은 그냥 에러를 낸다. 그때는 raw 에러를 보여주지 말고, Step 1에서
이미 받아둔 `files` 목록으로 **바뀐 파일만** 말하고 끝낸다 — 어디가 겹치는지는 모른다고
정직하게 말한다:

> "겹치는 부분이 있다고 GitHub이 알려주는데, 어느 파일인지까지는 제 컴퓨터에서 확인할 수
> 없었어요. 이 요청이 건드리는 파일은 `{목록}` 이에요. 올린 분이 아래 절차로 정리하시면
> 됩니다."

그리고 이렇게 말하고 **여기서 끝낸다** (내가 남의 브랜치를 고치지 않는다):

> "지금은 그대로 합칠 수 없는 상태예요. 겹치는 파일: `{목록}`.
> 이 부분을 최근에 고친 사람은 **{이름}** 님이에요 — 같이 보시는 걸 권해요.
>
> 정리는 **요청을 올린 분이** 자기 브랜치에서 하는 게 가장 안전해요. 그분께 이렇게
> 전해 주세요:
> `/jay-skills:pull 로 main 최신 내용을 받아온 다음, /jay-skills:fix-conflict 로 겹치는
> 부분을 정리하고 /jay-skills:push 하시면 이 확인 요청이 자동으로 갱신돼요.`"

## Step 3: 비개발자용 요약

`title` · `body` · `files` 로 **"무엇이 달라지나"**를 쓴다. 바뀐 줄 원문은 보여주지 않고,
파일의 **수와 성격**만 말한다.

```
### 이 요청은 무엇인가요
{제목을 쉬운 말로 한 줄}

### 무엇이 달라지나요
- {요청 본문에서 읽은 변화 1 — 화면·문구·규칙 중심}
- {변화 2}

### 어디가 바뀌었나요
- 화면 파일 {n}개, 설정 파일 {m}개 (총 {k}개 파일)
- {특별히 눈여겨볼 파일이 있으면 그 성격 한 줄 — 예: "로그인 관련 파일이 하나 포함돼 있어요"}

### 확인 상태
- 동료 확인: {reviewDecision 을 쉬운 말로}
- 합칠 수 있는 상태: {mergeable / mergeStateStatus 를 쉬운 말로}
```

`reviewDecision` 이 비어 있으면 "아직 아무도 안 봤어요"라고 **사실대로** 말한다.

`.env` · 인증(`auth`) · `migration` 이 이름에 들어간 파일이 포함돼 있으면 한 줄 덧붙인다:
"잘못 합치면 서비스가 멈출 수 있는 파일이 들어 있어요 — 올린 분과 한 번 더 확인하시는
게 좋아요."

## Step 4: 무엇을 할지 고르기

AskUserQuestion (문구는 제출 전에 다시 읽는다 — `git-collab.md` §7):

> "이 요청을 어떻게 할까요?
> ① 승인만 하기 (합치는 건 올린 분이 하도록)
> ② 승인하고 바로 merge하기 (main에 들어갑니다)
> ③ 의견 남기기 (고쳤으면 하는 점을 적어 보냅니다)
> ④ 나중에 볼래요"

- **① 승인만**
  ```bash
  gh pr review <ref> --approve --body "{확인한 내용 한두 줄}"
  ```
- **② 승인하고 merge** — `pr` 스킬 Step 6과 **같은 게이트**를 통과해야 한다:
  - `mergeable` 이 `CONFLICTING` 이면 **하지 않는다** (Step 2로).
  - `mergeStateStatus` 가 `BLOCKED` 또는 `BEHIND` 면 **멈춘다**. `BEHIND` 는 올린 분이
    main 최신을 받아와야 한다는 뜻이니 그렇게 전한다.
  - 그 밖의 값(`HAS_HOOKS` · `UNSTABLE` 등 여기 없는 값)은 **진행하되 상태를 한 줄로
    알린다**: "GitHub이 알려준 상태는 `{값}` 이에요 — 막는 상태는 아니어서 진행합니다."
  - 아무도 안 본 상태면 그 사실을 말하고 ①을 권한다.
  ```bash
  gh pr review <ref> --approve --body "{확인한 내용}"
  gh pr merge <ref> --squash
  ```
  merge 후: "합쳐졌어요. 내 컴퓨터의 main은 아직 예전 상태라 `/jay-skills:pull`로 한 번
  받아오시면 정리됩니다." 되돌리고 싶으면 `/jay-skills:undo`로 갈 수 있다는 것도 알린다.
- **③ 의견 남기기** — 무엇이 마음에 걸리는지 먼저 묻고, 그 말을 그대로 살려서 쓴다.
  고쳐달라는 요청이면 `--request-changes`, 단순 질문·제안이면 코멘트로 남긴다.
  ```bash
  gh pr review <ref> --request-changes --body "{사용자 말로 쓴 의견}"
  gh pr comment <ref> --body "{질문·제안}"
  ```
  올린 뒤 알린다: "의견을 남겼어요. 올린 분이 고치면 이 요청이 자동으로 갱신돼요."
- **④ 나중에** — 아무것도 하지 않고 링크만 남긴다.

## Step 5: 마무리 카드

`git-collab.md` §2 형식. 예:

```
### 지금 상태
- 확인 요청 #12 를 승인했어요 → {URL}
- 아직 main에 merge되지는 않았습니다 (merge는 올린 분이 하거나, 다시 불러서 ②로)

### 다음에 할 수 있는 것
- 올린 분에게 "승인했다"고 알려주기 (GitHub이 자동으로 알려주지 않을 수 있어요)
- 합쳐진 뒤 내 컴퓨터를 최신으로: `/jay-skills:pull`
```

## 절대 하지 않는 것

- **남의 요청을 닫기**(`gh pr close`) — 어떤 경우에도.
- **검사를 건너뛰고 merge**(`gh pr merge --admin`) — 잠가둔 데는 이유가 있다.
- 충돌이 있는 채로 merge.
- 남의 브랜치에 커밋하거나 push해서 대신 고쳐주기 — 정리는 올린 사람이 자기 브랜치에서
  한다.
- 사용자가 고르지 않았는데 승인·merge·의견 남기기.
- 코드 diff 원문을 그대로 쏟아내기.
- `gh` 자동 설치.
