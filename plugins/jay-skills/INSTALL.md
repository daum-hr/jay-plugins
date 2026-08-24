# jay-skills 설치 안내

바이브코딩을 도와주는 도구 모음이에요. 설치는 **명령 두 줄**이면 끝납니다.

## 설치 (권장 방법)

Claude Code를 열고, 채팅창에 아래를 **한 줄씩** 입력하세요.

### 1. 도구 목록이 있는 곳을 등록합니다

```
/plugin marketplace add daum-hr/jay-plugins
```

한 번만 하면 됩니다. "등록됐다"는 메시지가 나오면 성공이에요.

### 2. 설치합니다

```
/plugin
```

목록 화면이 열립니다. **Discover** 탭에서 `jay-skills` 를 화살표로 골라 Enter를 누르고,
설치 범위를 물어보면 **User** (내 컴퓨터 전체)를 고르세요.

> 화면을 거치지 않고 바로 설치하려면 이렇게 쳐도 됩니다:
> `/plugin install jay-skills@jay-plugins`

### 3. Claude Code를 다시 켭니다

완전히 종료했다가 다시 실행하면 준비 끝이에요.

## 잘 됐는지 확인

새 대화에서 `/` 를 입력했을 때 `jay-skills:` 로 시작하는 항목들(`welcome`, `push`,
`pull` 등)이 보이면 설치된 것입니다.

처음이라면 `/jay-skills:welcome` 을 실행해 보세요 — 필요한 준비가 됐는지 확인하고,
빠진 것만 채워줍니다.

## 무엇이 들어 있나요

| 도구 | 언제 쓰나요 |
|---|---|
| `/jay-skills:welcome` | 맨 처음 한 번 — 준비 상태 확인 |
| `/jay-skills:status` | 지금 무슨 상태인지 모를 때 (제일 먼저 눌러보는 버튼) |
| `/jay-skills:commit` | 작업을 내 컴퓨터에 커밋(기록) |
| `/jay-skills:push` | 커밋한 작업을 팀 저장소(GitHub)에 push |
| `/jay-skills:pull` | 동료가 올린 최신 작업 pull로 받아오기 |
| `/jay-skills:fix-conflict` | 충돌(같은 곳을 두 사람이 고침) 정리 |
| `/jay-skills:pr` | 내 작업을 동료에게 확인 요청(PR)으로 올리기 · 확인되면 merge |
| `/jay-skills:review-pr` | 동료가 보낸 확인 요청(PR) 검토하기 · 승인 · 의견 남기기 |
| `/jay-skills:github-login` | GitHub 연결이 안 될 때 |
| `/jay-skills:undo` | 실수 되돌리기 |
| `/jay-skills:explain` | 어려운 파일·에러 메시지를 쉬운 말로 |
| `/jay-skills:handoff` | 잠깐 쉬기 전에 지금 상황 메모해두기 |
| `/jay-skills:record` | 중요한 작업·결정을 기록으로 남기기 |

리포트 방식도 함께 바뀝니다 — 결과를 알려드릴 때 **한 줄 결론 → 쉬운 설명 → 자세한
내용은 물어보실 때만** 순서로 말합니다. 이 방식은 설치한 뒤로 **모든 프로젝트**에
적용돼요. 원래 방식으로 돌아가고 싶으시면 `/config` 에서 Output style을 바꾸시면 됩니다.

> 회사에서 받은 프로젝트 템플릿을 함께 쓰고 계시다면, 그 프로젝트가 정해 둔 리포트
> 방식보다 이 도구 모음의 방식이 우선 적용됩니다. 두 방식은 거의 같지만, 프로젝트 쪽에만
> 있는 개인정보 관련 문구 규칙은 프로젝트의 다른 설정 파일이 계속 담당하니 그 부분은
> 그대로 지켜집니다.

## 새 버전 받기 (업데이트)

새 버전이 나왔다고 안내받으시면, **두 줄을 순서대로** 치고 Claude Code를 다시 켜세요.

```
/plugin marketplace update jay-plugins
/plugin update jay-skills@jay-plugins
```

`0.1.0 에서 0.2.0 으로 업데이트했습니다` 처럼 버전이 바뀌었다는 메시지가 나오면 끝이에요.
그다음 Claude Code를 완전히 껐다 켜야 적용됩니다.

**두 줄 다 필요합니다.** 첫 줄은 "새 버전이 나왔는지 목록만 새로고침"하는 것이고, 실제로
내 컴퓨터의 도구를 바꾸는 건 둘째 줄이에요. 첫 줄만 치면 겉으로는 성공 메시지가 나오지만
도구는 예전 버전 그대로입니다.

**이름을 줄여 쓰면 안 됩니다.** `jay-skills` 만 쓰면 "찾을 수 없다"고 나와요.
`jay-skills@jay-plugins` 처럼 `@` 뒤까지 그대로 붙여 주세요.

> 설치할 때 "이 프로젝트에서만"을 고르셨다면 둘째 줄 뒤에 ` --scope project` 를 붙이세요.
> 그냥 설치하셨으면 붙이지 않아도 됩니다.

(자동으로는 갱신되지 않습니다 — 안전을 위해 자동 갱신이 꺼져 있어요. 그래서 새 버전
안내를 받으셨을 때만 위 명령을 쓰시면 됩니다.)

## 잘 안 될 때

- **`/plugin marketplace add` 가 실패해요** → GitHub에 연결이 안 되는 환경일 수 있어요.
  아래 "zip으로 설치하기"를 써주세요.
- **`/` 를 눌러도 `jay-skills:` 항목이 안 보여요** → Claude Code를 **완전히 종료**했다가
  다시 켰는지 확인해 주세요. 창만 닫으면 반영되지 않을 수 있어요.
- 그래도 안 되면 jay에게 문의해 주세요.

---

## zip으로 설치하기 (보조 방법)

회사망에서 GitHub 접근이 막혀 있거나, 위 방법이 안 될 때만 쓰세요.

### 1. 압축 풀기

**Mac** — 터미널에서:

```bash
mkdir -p ~/.claude/skills
unzip ~/Downloads/jay-skills-v0.1.0.zip -d ~/.claude/skills/
```

**Windows** — 파일 탐색기 주소창에 `%USERPROFILE%\.claude\skills` 를 입력해 그 폴더로
간 다음, zip 파일을 우클릭 → "압축 풀기"로 그 안에 풉니다. (`skills` 폴더가 없으면
새로 만드세요.)

풀고 나면 `~/.claude/skills/jay-skills/` 안에 `.claude-plugin` 폴더가 바로 보여야 맞습니다.

### 2. Claude Code 다시 켜기

그게 전부예요. 확인 방법은 위와 같습니다.

### zip 방식의 업데이트

기존 폴더를 지우고(`~/.claude/skills/jay-skills`) 새 zip을 같은 자리에 푼 다음, Claude
Code를 다시 켜세요.

### 이 프로젝트에서만 쓰고 싶다면

`~/.claude/skills/` 대신 그 프로젝트 폴더 안의 `.claude/skills/` 에 풀면 그 프로젝트에서만
동작합니다. 처음 열 때 폴더를 신뢰할지 묻는 창이 한 번 뜹니다.
