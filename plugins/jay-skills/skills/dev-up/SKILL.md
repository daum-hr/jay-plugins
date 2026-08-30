---
name: dev-up
description: 프로젝트를 브라우저에서 실제로 볼 수 있게 개발환경을 처음부터 끝까지 준비합니다 — README와 package.json이 선언한 대로 Node 버전 맞추기, 라이브러리 설치, 준비 스크립트 실행, 개발 서버 켜기, 접속 확인까지. "개발환경 준비해줘" / "서버 켜줘" / "README대로 세팅해줘" / "localhost가 안 열려요" / "포트가 사용중이래요" / "화면 보고 싶어요" 할 때, 또는 EADDRINUSE·EBADENGINE 같은 에러를 만났을 때 사용. 뭔가 설치하거나 다른 프로그램을 꺼야 할 때는 무엇인지 보여주고 물어본 뒤에만 진행해요. 여러 번 실행해도 안전하고, 이미 떠 있으면 확인만 합니다. /jay-skills:welcome 이 끝나면 자연스럽게 이어집니다.
---

# 개발환경 켜기 (dev-up)

코드를 받아온 뒤 **브라우저에서 화면이 보이는 상태**까지 데려다준다. **이 프로젝트가
무엇을 필요로 하는지는 프로젝트가 스스로 적어 둔 것에서 읽는다** — README의 개발환경
안내, `package.json`, `.nvmrc`. 없는 절차를 지어내지 않는다.

먼저 `${CLAUDE_PLUGIN_ROOT}/references/git-collab.md` — 에러 분류 순서(§3), 원격 명령
예고(§3.6), 같은 이름 스킬(§3.5). 프로젝트가 자체 `/dev-up`을 갖고 있으면 그쪽을 쓴다.

**모든 Bash 명령은 `export PATH="$HOME/.volta/bin:$PATH"` 로 시작한다.** Volta(폴더마다
알맞은 Node 버전을 자동으로 골라주는 도구)를 방금 깔았으면 새 터미널을 열기 전까진 경로가
안 잡히는데, 이 줄이 **같은 대화에서 그대로 이어가게** 해준다. Volta가 없어도 무해하다.

## 언제 쓰나

- 저장소를 막 받아왔고 화면을 처음 띄울 때 (`/jay-skills:welcome` 다음)
- 매일 작업 시작할 때 서버를 켜려고 / `localhost`가 안 열릴 때
- `EADDRINUSE`(포트가 이미 사용 중) · `EBADENGINE`(Node 버전 안 맞음) 에러를 만났을 때

**여러 번 실행해도 안전하다** — 이미 떠 있으면 확인만 하고 끝낸다.

## Process

### Step 1: 이 프로젝트가 뭘 선언하는지 읽기

읽기만 한다. 아무것도 바꾸지 않는다.

```bash
uname                                    # Darwin 인가
cat package.json 2>/dev/null
grep -hE '^PORT=' .env .env.example 2>/dev/null    # .env 는 이 줄만 본다
cat .nvmrc 2>/dev/null
```

README에서 개발환경 안내를 찾는다 (`개발환경`/`시작하기`/`Getting Started`/`설치`).
**있으면 그 문서가 우선이다** — 거기 적힌 명령·포트·"정상인 메시지" 목록을 그대로 따른다.

- **Node 버전** — `package.json`의 `volta.node` → `engines.node` → `.nvmrc`
- **실행할 명령** — `scripts.setup` / `scripts.dev` / `scripts.start`
- **포트** — `.env`의 `PORT` → `.env.example` → README

`package.json`도 README 개발환경 안내도 없으면 멈춘다: "이 폴더에는 아직 실행할 앱이
없어요. 받아오는 중이라면 `/jay-skills:welcome` 부터, 이미 받았는데 이 메시지가 나오면
저장소를 만든 동료에게 실행 방법을 물어봐 주세요."

**macOS가 아니면** 자동 진행은 여기까지다. 방법만 주고 멈춘다:

> "자동 준비는 지금 Mac에서만 동작해요. Windows에서는 ① `winget install Volta.Volta` 후
> 터미널 새로 열기 ② 저장소 폴더에서 `npm install` → `npm run setup` → `npm run dev`
> ③ 포트가 쓰이는 중이면 `netstat -ano | findstr :{포트}` 로 번호를 찾아
> `taskkill /PID {번호}` — **무엇을 끄는지 확인하고** 하세요. 막히면 화면을 보여주세요."

읽은 것을 **한 번에 정리해 알린다** (질문이 아니라 안내):

> "이 프로젝트는 Node {버전}이 필요하고, `npm install` → `npm run setup` →
> `npm run dev`(포트 {P}) 순서로 준비돼요. 지금부터 순서대로 진행할게요."

### Step 2: Node 준비

```bash
export PATH="$HOME/.volta/bin:$PATH"
node --version 2>&1
command -v volta >/dev/null && volta --version || echo volta-missing
```

**버전이 맞으면** 한 줄 확인하고 넘어간다.

**안 맞거나 Node가 없을 때** — Volta가 있고 `package.json`에 `volta` 항목이 있으면,
프로젝트 폴더에서 `node --version`을 한 번 더 부르는 것만으로 Volta가 알맞은 버전을
자동으로 내려받는다(시간이 걸릴 수 있다고 미리 알린다). `volta` 항목이 없으면
`volta install node@{필요한 버전}`.

**Volta 자체가 없으면 AskUserQuestion으로 반드시 묻는다:**

> "이 프로젝트는 Node {버전}이 필요한데 지금 컴퓨터의 것과 달라요. Volta(폴더마다 알맞은
> Node 버전을 자동으로 골라주는 도구)를 설치해서 맞춰 드릴까요?
> ① 설치하고 계속하기 ② 방법만 알려주세요"

① 이면 brew를 찾는다. **비대화형 실행에서는 brew가 경로에 없을 수 있어 절대경로도 본다:**

```bash
BREW=$(command -v brew || echo /opt/homebrew/bin/brew)
[ -x "$BREW" ] || BREW=/usr/local/bin/brew
[ -x "$BREW" ] && "$BREW" install volta && volta setup
```

brew가 없으면 **한 번 더 묻는다** — 인터넷에서 받은 설치 스크립트 실행은 별도 동의가
필요하다: "brew(Mac 프로그램 설치 도구)가 없네요. Volta 공식 설치 스크립트를 받아 실행할
까요? ① 진행 ② 방법만". `volta setup` 뒤에는 알린다: "**새로 여는 터미널부터** 자동으로
잡히고, 지금 이 대화에서는 제가 경로를 직접 지정해 그대로 이어갈게요."

내려받기가 오래 걸리다 실패하면 **네트워크부터 의심한다**(§3 맨 윗줄) — VPN 안내 후 정지.
"Volta가 고장났다"고 말하지 않는다.

### Step 2.5: nvm이 Volta를 가리고 있을 때 (해당될 때만)

예전에 nvm(다른 Node 버전 관리 도구)을 깔았던 컴퓨터는 nvm이 먼저 잡혀 방금 맞춘 버전이
안 보일 수 있다. **파일을 읽어서만 판단한다** — 대화형 셸을 새로 띄우지 않는다(설정에
따라 멈춰 있을 수 있다):

```bash
[ -d "$HOME/.nvm" ] && grep -n 'NVM_DIR\|nvm.sh' "$HOME/.zshrc" 2>/dev/null
grep -n 'VOLTA_HOME\|\.volta/bin' "$HOME/.zshrc" 2>/dev/null
```

nvm 줄이 있고 **그보다 뒤에** Volta 줄이 없으면 AskUserQuestion:

> "터미널이 Node를 찾을 때 예전에 깔린 nvm이 먼저 잡혀서, 직접 `npm run dev` 하실 때 옛
> 버전이 쓰일 수 있어요. `~/.zshrc` **맨 끝에** 두 줄을 더하면 정리됩니다. 기존 내용은
> 하나도 건드리지 않아요. ① 추가해 주세요 ② 아니요 (이 대화에서는 제가 알아서 처리해요)"

① 이면 **끝에 덧붙이기만 한다.** 위치가 핵심이다 — nvm 줄보다 뒤여야 효력이 있다:

```bash
printf '\n# Volta — nvm 로드보다 뒤에 있어야 한다\nexport VOLTA_HOME="$HOME/.volta"\nexport PATH="$VOLTA_HOME/bin:$PATH"\n' >> "$HOME/.zshrc"
```

②를 골라도 흐름은 그대로 간다 — 이 스킬의 명령은 이미 경로를 직접 지정한다.

### Step 3: 라이브러리 설치

오래 걸릴 수 있다고 **먼저 알리고** 실행한다.

```bash
export PATH="$HOME/.volta/bin:$PATH"
npm install 2>&1 | tail -30
```

`npm warn`(노란 줄)은 실패가 아니다. 멈추는 것은 `npm error` 다. 설치가
`package-lock.json`을 바꿨으면 **그 사실을 알린다** — 나중에 `/jay-skills:status`에서 보고
놀라지 않도록: "잠금 파일(어떤 버전을 썼는지 적어두는 파일)이 바뀌었어요. 저장하려면
`/jay-skills:commit` 하시면 됩니다."

### Step 4: 설정 파일과 준비 스크립트

**설정 파일이 먼저다** — 준비 스크립트가 그 값을 읽을 수 있다.

`.env`가 없고 `.env.example`이 있으면 AskUserQuestion: "설정 파일(`.env`)이 없어요. 예시
파일을 복사해 만들어 둘까요? ① 만들어 주세요 ② 아니요". ① 이면 `cp .env.example .env`.
그다음 **어떤 항목이 있는지 이름만 알린다 — 값은 절대 화면에 내지 않는다.** 진짜 비밀값이
필요한데 README에 없으면 멈춘다: "이 값은 저장소를 만든 동료에게 받으셔야 해요."

`scripts.setup`이 있으면(또는 README가 시키면) **무엇을 하는지 README에 적힌 대로 먼저
말하고** 실행한다. README가 "여러 번 실행해도 안전"이라 밝혀 뒀으면 그대로, 그런 말이
없으면 한 번 묻고 진행한다.

```bash
export PATH="$HOME/.volta/bin:$PATH"
npm run setup 2>&1 | tail -20
```

### Step 5: 포트 확인 — 누가 쓰고 있나

포트는 앱이 쓰는 **번호가 붙은 창구**다. 한 번호는 한 프로그램만 쓸 수 있어서, 다른 게
먼저 쓰고 있으면 우리 서버가 못 뜬다.

```bash
lsof -nP -iTCP:{P} -sTCP:LISTEN 2>/dev/null
```

비어 있으면 Step 6으로. 쓰는 것이 있으면 **정체부터 확인한다** (같은 프로그램이 여러 줄로
나오므로 번호를 중복 제거한다):

```bash
lsof -a -p {PID} -d cwd -Fn 2>/dev/null    # 어느 폴더에서 도는지
ps -o user=,command= -p {PID}
```

**(a) 우리 프로젝트 서버가 이미 떠 있음** (도는 폴더가 지금 폴더와 같음)

```bash
curl -s -o /dev/null -w '%{http_code}' "http://localhost:{P}"
```

응답이 오면(404·500이어도 살아 있는 것) **아무것도 하지 않고** 끝낸다: "이미 떠 있어요 —
http://localhost:{P}". 응답이 없으면(멈춘 상태) 다시 켤지 묻는다.

**(b) 다른 프로그램이 쓰고 있음** (내 계정, 다른 폴더) — **반드시 묻는다. 정체를 보여주지
않고 끄는 일은 없다.**

> "포트 {P}를 다른 프로그램이 쓰고 있어요.
> - 번호(PID): {PID} / 프로그램: `{명령}` / 도는 폴더: `{폴더}`
>
> ① 그 프로그램을 끄고 우리 서버 띄우기 — **저장 안 한 작업이 있다면 먼저 확인하세요**
> ② 우리 서버를 다른 번호({P+1})로 띄우기
> ③ 그만두기"

> 프로젝트에 따라 같은 이름의 스킬이 자동으로 끄기도 한다. 이 플러그인은 여러 사람·여러
> 컴퓨터에서 쓰이므로 **묻는 쪽이 기본이다** — 일부러 다르게 두었다.

**(c) 시스템이나 다른 사용자의 프로그램** (소유자가 root이거나 내 계정이 아님, 또는
`launchd`·`ControlCenter`·`rapportd`·편집기 같은 것) — **끄는 선택지를 아예 내지 않는다.**
②(다른 번호)나 ③만 낸다. 다른 번호는 프로젝트가 `PORT` 설정을 읽을 때만 된다. 번호가
코드에 고정돼 있으면 설명하고 멈춘다.

**끄기로 동의를 받았을 때만:**

```bash
kill -TERM {PID}
sleep 3; lsof -nP -iTCP:{P} -sTCP:LISTEN 2>/dev/null    # 아직 살아 있나
```

정중한 요청으로 안 꺼지면 **그 사실을 말한 뒤** `kill -9 {PID}`. 다시 물을 필요는 없지만
(이미 끄기로 동의했다) 말은 반드시 한다. 마지막에 포트가 실제로 비었는지 확인하고 Step 6.

### Step 6: 서버 켜기 + 접속 확인

대화가 끝나도 떠 있도록 백그라운드로 띄운다. 기록은 저장소 **바깥**에 남긴다 — 안에 두면
커밋할 파일 목록이 지저분해진다.

```bash
export PATH="$HOME/.volta/bin:$PATH"
LOG="${TMPDIR:-/tmp}/dev-up-$(basename "$PWD").log"
nohup npm run dev >> "$LOG" 2>&1 &
echo "$! $LOG"
```

기록 위치를 한 번 알린다: "문제가 생기면 이 파일을 보여주세요."

2초 간격으로 최대 60초 확인한다:

```bash
curl -s -o /dev/null -w '%{http_code}' "http://localhost:{P}"
```

동시에 기록에서 두 가지를 본다. ① README가 "이게 보이면 정상"이라 적어 둔 문구
② **번호가 바뀌었는지** — 개발 서버는 번호가 차 있으면 조용히 옆 번호로 옮겨간다.
기록의 `localhost:` 뒤 숫자를 찾아 **실제 번호**로 확인한다. 선언된 번호만 믿으면 남의
서버를 우리 것으로 착각할 수 있다.

- **200~399** → 준비됨. **404** → 서버는 살아 있다(첫 화면이 없는 앱일 수 있음) — 떴다고
  보고하고 주소를 준다. **500** → 떴지만 안에서 에러 — 기록 마지막을 **원문 그대로** 보여
  주고 `/jay-skills:explain`.
- **응답 없음** → 프로세스가 죽었으면 기록 마지막 원문 + 아래 번역표로 분류, 살아 있으면
  "느리게 뜨는 중일 수 있어요" 후 30초 더 기다렸다가 기록을 보여준다.

성공하면 **말하고 나서** 브라우저를 연다: "브라우저를 열어드릴게요 —
http://localhost:{실제 번호}" → `open "http://localhost:{실제 번호}"`

## 실패했을 때 (번역표)

**위에서 아래로 내려가며 처음 맞는 줄을 쓴다.** 네트워크 문제를 설정 문제로 잘못
분류하면 사용자는 되지도 않을 일만 반복하게 된다.

| 화면에 나온 말 | 실제 의미 | 어떻게 |
|---|---|---|
| `ETIMEDOUT` / `ENOTFOUND` / `network` / 오래 멈췄다 실패 | 인터넷·회사망이 안 닿는 것. 설정 문제가 아니다 | VPN 안내 후 **정지** |
| `EBADENGINE` / `Unsupported engine` / `Required: {"node":...}` | 지금 이 창에서 잡히는 Node가 프로젝트와 다르다 | 같은 명령 안에서 `node --version`으로 확인시키고 **Step 2로** |
| `EADDRINUSE` / `address already in use` | 포트를 다른 게 이미 쓰고 있다 | **Step 5**로 |
| `Cannot find module` / `MODULE_NOT_FOUND` | 라이브러리 설치가 안 됐거나 끊겼다 | **Step 3** 다시 |
| `.env` 관련 / 설정값이 `undefined` / 인증 키 없음 | 설정 파일이 없거나 값이 비었다 | **Step 4**로 |
| `no such table` / `database` / `SQLITE` | 데이터 준비가 안 됐다 | **Step 4**의 준비 스크립트 다시 |
| `EACCES` / `permission denied` | 권한 문제 — 이 스킬이 풀 수 없다 | 멈추고 팀에 물어보기 |
| `npm warn ...` (노란 줄) | 실패가 아니라 안내다 | 그대로 진행 |

여기 없는 말이 나오면 **원문을 줄이지 말고 그대로 보여준 뒤** 쉬운 말로 풀이하고
`/jay-skills:explain` 을 안내한다.

## 끝난 뒤 알리는 것

한 줄 결론부터: "준비 완료 — http://localhost:{번호} 가 브라우저에 열렸어요."
그다음 짧게 — Node 버전 / 라이브러리 설치 / 준비 스크립트 / 서버 실행, 각각 "됐어요"
또는 "이미 돼 있었어요". **결정이 있었을 때만** 덧붙인다: 무엇을 껐는지(동의받고),
`~/.zshrc`에 두 줄을 더했는지, `.env`를 만들었는지, 포트가 옮겨졌는지.

마지막으로: "서버는 이 대화가 끝나도 계속 떠 있어요. 끄고 싶으면 '서버 꺼줘'라고 하세요."
**"서버 꺼줘"** 를 들으면 `lsof`로 그 포트의 번호를 찾아 **정체를 보여주고 확인받은 뒤**
`kill -TERM`. Step 5의 (b)(c) 규칙이 그대로 적용된다.

git 상태를 바꾸지 않는 흐름이므로 마무리 카드(§2)는 붙이지 않는다. 다음에 할 수 있는 것만
한 줄: 화면 확인 → 작업 → `/jay-skills:commit`.

## Rules

- **설치·시스템 변경은 무엇인지 보여주고 물어본 뒤에만 한다.** Volta 설치, 인터넷 설치
  스크립트 실행, `~/.zshrc` 수정, `.env` 생성 — 전부 AskUserQuestion.
- **프로세스는 정체(번호·프로그램·폴더)를 보여주고 동의를 받기 전에는 절대 끄지 않는다.**
  시스템이나 다른 사용자의 프로세스는 **동의가 있어도 끄지 않는다** — 다른 번호로 비켜간다.
- 셸 설정 파일은 **끝에 덧붙이기만** 한다. 기존 줄을 고치거나 지우지 않는다.
- **`.env`의 값을 화면에 내지 않는다.** 항목 이름까지만.
- **프로젝트가 선언한 명령만 실행한다** (README·`package.json`). 없는 절차를 지어내지 않는다.
- `sudo` 쓰지 않는다. `npm install -g` 하지 않는다. localhost 밖으로 나가지 않는다.
- 모든 명령은 `export PATH="$HOME/.volta/bin:$PATH"` 로 시작한다.
- **여러 번 실행해도 안전해야 한다.** 이미 떠 있으면 확인만 하고 끝낸다.
- **git 상태를 바꾸지 않는다.** 설치가 잠금 파일을 바꾸면 그 사실만 알린다.
- 에러 원문은 줄이거나 바꿔 쓰지 않는다. 그대로 보여준 뒤 쉬운 말을 덧붙인다.
