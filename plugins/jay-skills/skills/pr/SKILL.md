---
name: pr
description: 지금 브랜치의 작업을 팀 최종본(main)에 merge해도 될지 동료에게 확인 요청(PR — Pull Request)으로 올립니다. 제목·설명을 한국어로 자동 작성해서 보여주고, 확인 후 GitHub에 올려요. 아직 push 안 한 작업이 있으면 push까지 함께 처리합니다. 이미 올라간 PR이 있으면 상태를 보여주고, 동료 확인이 끝났으면 merge까지 도와줍니다. "확인 요청해줘" / "PR 올려줘" / "이제 합쳐줘" 할 때 사용. 동료가 보낸 PR을 *검토*하는 건 review-pr 입니다.
---

# 확인 요청 올리기 (pr)

PR(Pull Request)은 **"내 작업을 팀 최종본(main)에 합쳐도 될까요"라고 동료에게 올리는
확인 요청**이다 — 이 한 줄로 설명하고 시작한다. 그다음부터는 그냥 **PR**이라고 부른다.

이 스킬은 **올리는 쪽**이다. 동료가 보낸 PR을 검토하는 것은 `/jay-skills:review-pr`.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md`를 읽는다 — 특히 §0(팀 전제:
개발자 없이 동료끼리 확인하고 **merge도 팀이 직접 한다**), 어휘(§1), 카드(§2),
에러 번역표(§3), 원격 명령(§3.6), `gh` 확인 순서(§4), 안전 기준선(§5).

## Step 1: 조용히 확인

`gh` 명령은 서버에 접속한다 — `git-collab.md` §3.6대로 **실행 전에 미리 알린다**:
"팀 저장소 연결을 확인할게요 — 몇 초 걸릴 수 있어요."

```bash
command -v gh >/dev/null && echo gh-ok || echo gh-missing
HOST=$(git remote get-url origin 2>/dev/null \
       | sed -E 's#^[a-z]+://##; s#^[^@/]*@##; s#[:/].*$##')
[ -n "$HOST" ] || HOST=github.com
gh auth status --hostname "$HOST" 2>&1
git rev-parse --abbrev-ref HEAD 2>&1
git status --porcelain 2>&1
git log --oneline @{u}..HEAD 2>/dev/null
gh pr view --json url,state,mergeable,mergeStateStatus,reviewDecision 2>/dev/null
```

로그인 판정은 `--hostname` 을 붙인 결과로만 한다 (`git-collab.md` §4). 오래 걸린 끝에
실패했다면 인증부터 의심하지 말고 §3 **맨 윗줄(연결 실패)**부터 분류하고, 네트워크면
VPN 안내 후 **정지**한다.

- **`gh` 없음** → 설치 안내만 하고 끝낸다 (자동 설치 금지):
  > "확인 요청을 올리려면 `gh`라는 GitHub 도구가 필요해요. **Mac**: `brew install gh`
  > / **Windows**: `winget install GitHub.cli`. 설치 후 `/jay-skills:pr`을 다시
  > 실행해 주세요. 지금까지 작업은 아무것도 바뀌지 않았어요."
- **미로그인** → `/jay-skills:github-login` → 끝나면 여기로 돌아온다.
- **이미 열려 있는 PR이 있음** → 새로 만들지 않고 **Step 5(상태 보기)**로 간다.
- **지금 main 위** → PR은 작업 브랜치에서 올린다 → `/jay-skills:push`의 브랜치 생성
  흐름을 먼저 진행한다.

## Step 2: push 안 한 작업이 있으면 먼저 push한다

커밋 안 된 변경이 있으면 `/jay-skills:commit`을, push 안 한 커밋이 있으면
`/jay-skills:push`를 먼저 진행한다. 사용자에게는 한 흐름으로 보이게 이어서 처리하되,
각 스킬의 확인 질문은 그대로 살린다 (건너뛰지 않는다).

## Step 3: 제목과 설명 초안을 보여주고 확인받는다

커밋 메시지들과 대화 내용에서 초안을 만든다. 한국어로 쓴다. 읽는 사람이 **비개발자
동료**라는 것을 전제로, 코드 용어가 아니라 무엇이 달라지는지를 쓴다.

- **제목**: 한 줄. 예: `공지사항 목록 화면 추가`
- **설명**: 세 부분
  - **무엇을 했나요** — 두세 줄, 쉬운 말로
  - **왜 필요한가요** — 한두 줄
  - **봐주셨으면 하는 것** — 동료가 확인해줬으면 하는 지점 (화면 위치, 눌러볼 버튼 등)

프로젝트에 체크리스트 문서가 있으면 그 내용을 "봐주셨으면 하는 것"에 활용한다.

초안 전체를 보여주고 AskUserQuestion:

> "이대로 팀에 올릴까요?
> ① 이대로 올리기 ② 제목·설명을 고칠래요 ③ 그만두기"

merge될 대상 브랜치(base)는 저장소 기본값을 그대로 쓴다 — 기술 결정이라 묻지 않는다.

## Step 4: 올리기

```bash
gh pr create --title "{제목}" --body "{설명}"
```

실패하면:

| 실패 | 대응 |
|---|---|
| 403 / 404 | `/jay-skills:github-login`의 권한 진단으로 |
| `no commits between` | "팀 최종본과 차이가 없어서 요청할 내용이 없어요." → `/jay-skills:status` |
| 브랜치가 팀 저장소에 없음 | `/jay-skills:push`를 먼저 |

**올린 직후 겹치는 부분이 있는지 그 자리에서 말한다.** 방금 만든 요청의 상태를 한 번 더
읽어(`gh pr view --json mergeable,mergeStateStatus`) 사용자가 링크를 공유하기 *전에*
알린다 — 겹친 채로 공유하면 동료가 열어보고 나서야 알게 된다.

- 겹치지 않으면: "지금 기준으로 겹치는 부분 없이 합칠 수 있는 상태예요."
- `CONFLICTING` 이면: "그 사이 main이 바뀌어서 겹치는 부분이 생겼어요. `/jay-skills:pull`
  로 최신을 받아온 뒤 `/jay-skills:fix-conflict` 로 정리하면 이 요청이 자동으로 갱신돼요."
  → 공유를 권하기 전에 이 정리를 먼저 권한다.

올린 뒤에는 **동료에게 알려야 한다**는 것을 명시한다 — GitHub에 올렸다고 자동으로
누가 보는 게 아니다:

> "올렸어요 → {URL}. 이 주소를 팀 채팅방에 공유해서 봐달라고 알려 주세요.
> 동료분도 Claude를 쓰신다면 이렇게 전해 주시면 편해요:
> `/jay-skills:review-pr {URL}` — 무엇이 달라지는지 요약해서 보여주고 승인까지 도와줍니다."

## Step 5: 이미 올라간 PR의 상태 보기

`gh pr view` 결과를 쉬운 말로 옮긴다. 세 가지를 말한다.

1. **동료 확인 상태** — `reviewDecision`: `APPROVED`면 "동료가 확인해 줬어요",
   `CHANGES_REQUESTED`면 "고쳐달라는 의견이 있어요", 비어 있으면 "아직 아무도 안 봤어요".
2. **merge할 수 있는 상태인지** — `mergeable`: `CONFLICTING`이면 "그 사이 팀 최종본이
   바뀌어서 겹치는 부분이 생겼어요" → `/jay-skills:pull` 후 `/jay-skills:fix-conflict`로
   안내하고 **여기서 멈춘다**.
3. **다음에 할 수 있는 것** — 계속 고치기 / merge(Step 6).

## Step 6: merge (요청받았을 때만)

§0대로 **merge도 팀이 직접 한다.** 다만 팀 최종본을 바꾸는 일이라 게이트를 둔다.

**아래 중 하나라도 아니면 merge하지 않는다**:

- 겹치는 부분 없음 (`mergeable` != `CONFLICTING`)
- 자동 검사가 있다면 실패 상태가 아님 (`mergeStateStatus`가 `BLOCKED`/`BEHIND`면 멈춤 —
  `BEHIND`는 `/jay-skills:pull` 후 다시)
- 사용자가 명시로 "합쳐줘"라고 함

**여기 열거되지 않은 `mergeStateStatus` 값**(`HAS_HOOKS` · `UNSTABLE` · `DRAFT` 등)을
만나면 **말없이 통과시키지 않는다.** 막는 값이 아니므로 진행하되, 상태를 한 줄로 알린다:

> "GitHub이 알려준 상태는 `{값}` 이에요 — merge를 막는 상태는 아니어서 그대로 진행합니다."

모르는 값을 조용히 지나가면, 나중에 문제가 생겼을 때 사용자는 아무 신호도 못 받은 것이
된다.

확인 상태를 **사실대로 말하고** 묻는다. 아무도 안 봤으면 그 사실을 숨기지 않는다:

> "지금 merge하면 이 작업이 팀 최종본(main)에 들어가고, 다음에 동료들이 pull할 때 함께
> 갑니다. {동료 확인 상태 한 줄}.
> ① merge하기 ② 조금 더 기다리기 (동료 확인 후에) ③ 그만두기"

② 를 권장으로 둔다 — 아무도 안 본 상태라면 특히.

```bash
gh pr merge --squash
```

`--squash`를 쓴다(기술 결정 — 묻지 않는다). 여러 개의 커밋이 팀 최종본에는 **한 줄로**
들어가서 나중에 이력을 읽기 쉽다.

merge한 뒤:

> "merge됐어요. 내 컴퓨터의 main은 아직 예전 상태라, `/jay-skills:pull`로 한 번
> 받아오시면 정리됩니다."

**되돌리고 싶으면** `/jay-skills:undo`의 "이미 push한 커밋 되돌리기"로 갈 수 있다는 것도
함께 알린다 — merge가 영구 결정이 아니라는 안심.

## Step 7: 마무리 카드

```
### 지금 상태
- PR #{번호} 가 올라가 있어요 → {URL} (또는: main에 merge됐어요)
- 동료 확인: {상태 한 줄}

### 다음에 할 수 있는 것
- 팀 채팅방에 {URL}을 공유해서 봐달라고 알리기 (동료는 `/jay-skills:review-pr {URL}`)
- 더 고칠 게 있으면: 같은 브랜치에서 작업 → `/jay-skills:commit` → `/jay-skills:push`
  (이 PR에 자동 반영돼요)
- merge한 뒤라면: `/jay-skills:pull`로 내 컴퓨터 정리
- 잘 모르겠으면: 여기서 멈추고 팀에 물어보기 — 지금 상태 그대로 두면 아무것도 망가지지 않아요
```

## 절대 하지 않는 것

- **검사·확인을 건너뛰고 merge** (`gh pr merge --admin`) — 잠가둔 데는 이유가 있다.
- 겹치는 부분(충돌)이 있는데 merge.
- 사용자가 명시로 요청하지 않았는데 merge. Step 4에서 올린 직후 자동으로 merge하지 않는다.
- 다른 사람의 PR을 닫거나 merge하거나 고치기 (검토는 `/jay-skills:review-pr`).
- PR을 새로 고치겠다고 force push.
- merge한 뒤 브랜치 삭제 — 남아 있어도 아무 문제 없다. 정리는 사용자가 따로 요청할 때만.
- `gh` 자동 설치.
