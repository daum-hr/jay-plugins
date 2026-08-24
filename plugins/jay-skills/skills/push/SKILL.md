---
name: push
description: 내 컴퓨터에 커밋(/jay-skills:commit)해 둔 작업을 팀 저장소(GitHub)에 push해서 동료가 볼 수 있게 합니다. main에 직접 push하지 않고 안전한 작업 브랜치를 자동으로 만들어 올려요. "push 해줘" / "팀에 공유해줘" / "올려줘" 할 때 사용. push 전 커밋 안 된 변경이 있으면 먼저 /jay-skills:commit을 권합니다.
---

# 팀에 push하기 (push)

비개발자를 위한 공유 흐름. 안전 축은 하나다 — **내 컴퓨터 ↔ 팀 저장소**.

push(내 커밋을 팀 저장소로 올려 동료가 볼 수 있게 하는 것)는 **커밋된 것만** 보낸다.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md`를 읽는다 — 어휘(§1), 마무리
카드(§2), 에러 번역표(§3), 안전 기준선(§5), 묻지 않고 정하는 것(§6).

## Process

### Step 1: 조용히 확인

```bash
git rev-parse --git-dir 2>&1
git status --porcelain 2>&1
git rev-parse --abbrev-ref HEAD 2>&1
git remote get-url origin 2>&1
git rev-parse --abbrev-ref '@{u}' 2>&1
git log --oneline @{u}..HEAD 2>/dev/null
```

팀 저장소를 건드리는 `fetch`는 `git-collab.md` §3.6대로 **미리 알리고 시간 제한을 걸어**
실행한다:

```bash
git -c http.lowSpeedLimit=1000 -c http.lowSpeedTime=15 fetch 2>&1
```

git 저장소가 아니면 `/jay-skills:status`의 안내로 넘긴다.

**`fetch`가 실패하면 `git-collab.md` §3 번역표로 먼저 분류한다** — 맨 윗줄부터 본다.

- **네트워크(연결 실패)** → 그 자리에서 VPN 안내 후 **정지**. `github-login`으로 보내지
  않는다. 사내 저장소인데 회사망에 연결돼 있지 않으면 로그인을 몇 번 다시 해도 안 된다.
- **인증** → Step 5의 인증 경로(`/jay-skills:github-login`)로.
- 그 밖 → §3 표에서 맞는 행을 따른다.

### Step 2: 커밋 안 된 변경이 있으면 먼저 묻는다

AskUserQuestion:

> "커밋 안 된 변경이 {n}개 있어요. push는 *커밋된 것*만 보냅니다.
> ① 먼저 커밋하고 같이 push하기 (권장) ② 커밋된 것까지만 push하기 ③ 그만두기"

① → `/jay-skills:commit`을 먼저 돌린다. **자동으로 커밋하지 않는다.**

### Step 3: main 위라면 안전한 브랜치를 만든다

지금 브랜치(내 작업만 따로 담아두는 공간)가 `main`(또는 `master`)이면:

접두어와 이름은 커밋 내용에서 자동으로 고른다(기술 결정 — 묻지 않는다). 새 기능이면
`feature/`, 고치는 작업이면 `fix/`, 정리면 `chore/`. 이름은 **영문 소문자와 하이픈만**
쓴다 (`git-collab.md` §6 — 브랜치 이름이 GitHub 주소에 그대로 들어가기 때문이다).
AskUserQuestion으로 **이름만** 확인한다:

> "main은 팀 모두가 함께 쓰는 최종본이라 바로 push하지 않아요 — 실수가 팀 전체에 그대로
> 퍼지거든요. 대신 작업 브랜치
> `feature/{자동 제안}` 을 만들어서 push할게요. 지금 커밋해 둔 작업은 그대로 따라갑니다.
> (브랜치 이름은 주소에 들어가서 영문으로 지어요.)
> ① 이 이름으로 진행 ② 이름 바꿀래요 ③ 그만두기"

```bash
git switch -c {브랜치이름}
```

지금 위치에서 브랜치만 새로 만드는 것이라 **아무것도 이동하거나 사라지지 않는다**.

### Step 4: 팀 저장소 주소가 없으면

AskUserQuestion:

> "팀 저장소 주소가 아직 연결 안 돼 있어요.
> ① 주소를 알고 있어요 (입력할게요) ② 없어요 / 모르겠어요"

② → "지금까지 작업은 내 컴퓨터에 전부 안전하게 기록돼 있어요. 저장소를 만든 동료에게
주소를 물어봐 주세요." 로 종료. ① → 받은 주소로 `git remote add origin "{URL}"`.
**주소 검증은 하지 않는다** — 비공개 저장소는 확인 명령이 실패해서 멀쩡한 주소를
틀렸다고 오해하게 만든다.

### Step 5: push하기

```bash
git push -u origin {현재 브랜치}
```

`-u`는 항상 붙인다(기술 결정). 처음 한 번만 설명한다: "다음부터는 그냥 '올려줘'만
하시면 되도록 짝을 지어뒀어요."

**실패하면 원문을 해석해서 경로를 고른다** (`git-collab.md` §3 — 맨 윗줄부터):

| 실패 | 대응 |
|---|---|
| **연결 실패** (`Couldn't connect` · `Failed to connect` · `port 443` · `Connection timed out` · `Connection refused` · `i/o timeout` · `dial tcp` · `could not resolve host`) | **인증 문제가 아니다.** "`{호스트}` 서버에 연결 자체가 안 되고 있어요. 사내 저장소라면 회사 VPN을 켜신 뒤 다시 해주세요." → **여기서 정지.** github-login으로 보내지 않는다 |
| 인증 실패 / 401 / 비밀번호 거부 | `/jay-skills:github-login` → 끝나면 push 재시도 |
| 403 (권한 없음) | `/jay-skills:github-login`의 403 진단으로 |
| `GH006` / protected branch | main에 push하려 한 것 → Step 3으로 돌아가 작업 브랜치 생성 |
| `! [rejected]` / `fetch first` | "동료가 먼저 push한 게 있어요. pull한 다음 push해야 합니다." → `/jay-skills:pull` 실행 → 성공하면 push 재시도. **`--force`는 어떤 경우에도 제안하지 않는다.** |
| `Repository not found` | 현재 주소를 보여주고 팀에서 받은 주소와 대조하도록 안내 |

### Step 6: main 정리 제안 (push 성공 + Step 3에서 브랜치를 만든 경우만)

이 시점에는 그 커밋들이 **새 브랜치에도 있고 팀 저장소에도 올라가 있다** — 그래서 내
컴퓨터의 main을 팀 저장소와 같은 상태로 되돌려도 잃을 것이 없다.

AskUserQuestion:

> "실수로 main에 커밋돼 있던 작업은 이제 `{브랜치}`에도, 팀 저장소에도 안전하게 들어가
> 있어요. 내 컴퓨터의 main을 팀 저장소와 같은 상태로 정리해 둘까요? 나중에 헷갈리지
> 않게 하는 정리 작업이에요.
> ① 정리하기 (권장) ② 그대로 두기"

① → `git switch main && git reset --keep origin/main && git switch {브랜치}`.
`--keep`은 커밋 안 된 변경이 있으면 **스스로 거부한다** — 거부하면 그대로 두고
사용자에게 알린다("정리는 건너뛰었어요. 작업은 전부 그대로 있습니다"). 실패해도
문제없다 — main은 push하지 않으므로.

### Step 7: 마무리

`git-collab.md` §2 카드. 예:

```
### 지금 상태
- `feature/notice-list` 브랜치가 팀 저장소에 올라갔어요. 내 컴퓨터와 같은 내용입니다.
- 커밋 안 된 변경: 없음

### 다음에 할 수 있는 것
- 동료에게 확인 요청(PR)을 올리려면: `/jay-skills:pr`
- 계속 작업하고 또 push하려면: 작업 → `/jay-skills:commit` → `/jay-skills:push`
- 잘 모르겠으면: 여기서 멈추고 팀에 물어보기 — 지금 상태 그대로 두면 아무것도 망가지지 않아요
```

## 절대 하지 않는 것

- main에 직접 push하기.
- `--force` / `--force-with-lease` — 거부당했을 때도, 사용자가 요청해도. 요청받으면
  이유를 설명하고 `/jay-skills:pull` 경로를 제안한다.
- 사용자 확인 없이 커밋하기 (`git-collab.md` §5).
- 팀 저장소의 브랜치 삭제, 이미 push한 이력 고치기(amend·rebase).
- raw git 출력 그대로 보여주기.
