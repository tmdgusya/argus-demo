# Argus — 전체 셋업 가이드

> 이 문서는 Argus 프로젝트를 Hermes 에이전트 팀으로 개발하기 위해
> 사전에 준비해야 할 모든 인프라 셋업 순서를 정리합니다.
> 강의 실습에서는 이 순서대로 진행합니다.

---

## 0. 전제 조건

- Hermes Agent 설치 완료 (`hermes --version` 동작)
- Git 계정 설정 완료 (`gh auth status` 동작)
- Python 3.11+ / Node.js 20+ 설치
- 웹 브라우저 (Discord Developer Portal 접근용)

---

## 1. Discord Bot 3개 생성

> **핵심**: Discord는 bot token 하나당 gateway 커넥션 하나만 유지 가능.
> 두 프로파일이 같은 토큰으로 gateway를 띄우면 충돌(Conflict) 발생.
> 따라서 프로파일마다 **별도 Discord Bot**이 필요합니다.

### 1-1. Discord Developer Portal

1. https://discord.com/developers/applications 접속
2. **New Application** 버튼 클릭 × 3개 생성

| Application 이름 | 용도 |
|-----------------|------|
| `argus-backend` | backend 프로파일 전용 |
| `argus-frontend` | frontend 프로파일 전용 |
| `argus-dba` | dba 프로파일 전용 |

### 1-2. Bot 토큰 확보

각 Application에서:
1. 좌측 메뉴 **Bot** 클릭
2. **Reset Token** → 토큰 복사 (각각 안전한 곳에 보관)

> 토큰은 Base64 문자열 형태 (Developer Portal에서만 확인 가능)

### 1-3. Bot 초대 권한 설정

각 Application에서:
1. 좌측 메뉴 **OAuth2** → **URL Generator**
2. Scopes: `bot`
3. Bot Permissions:
   - `Send Messages`
   - `Read Message History`
   - `Add Reactions`
   - `Create Public Threads` (선택)
   - `Send Messages in Threads` (선택)
4. 생성된 URL을 브라우저에서 열어 **Argus 서버**에 각각 초대

### 1-4. 봇 채널 분리 (권장)

Discord 서버에 채널 구성:

```
#argus
├── #argus-backend      ← argus-backend 봇이 활동
├── #argus-frontend     ← argus-frontend 봇이 활동
├── #argus-dba          ← argus-dba 봇이 활동
└── #argus-general      ← 전체 공지 / PM이 관찰
```

---

## 2. Hermes Profile 3개 생성

### 2-1. 프로파일 생성 (기존 설정 복제)

```bash
hermes profile create backend --clone
hermes profile create frontend --clone
hermes profile create dba --clone
```

`--clone`은 현재 default 프로파일의 `config.yaml`, `.env`, `SOUL.md`를 복사합니다.
기존에 설정된 LINEAR_API_KEY, GitHub credential 등이 그대로 상속됩니다.

### 2-2. 생성된 프로파일 구조 확인

```bash
ls ~/.hermes/profiles/backend/
# config.yaml, .env, SOUL.md, skills/, memory/, sessions/, ...
```

### 2-3. 프로파일별 alias 확인

프로파일 생성시 자동으로 셸 alias가 등록됩니다 (source ~/.bashrc 또는 재로그인 필요):

```bash
backend chat    # backend 프로파일로 대화 시작
frontend chat   # frontend 프로파일로 대화 시작
dba chat        # dba 프로파일로 대화 시작
```

---

## 3. 프로파일별 Discord Bot Token 주입

각 프로파일의 `.env`에 자신의 bot token을 설정합니다.

```bash
# 각 프로파일의 .env에 DISCORD_BOT_TOKEN 추가
# 방법 1: 직접 파일 편집
echo 'DISCORD_BOT_TOKEN=<argus-backend-bot-token>' >> ~/.hermes/profiles/backend/.env
echo 'DISCORD_BOT_TOKEN=<argus-frontend-bot-token>' >> ~/.hermes/profiles/frontend/.env
echo 'DISCORD_BOT_TOKEN=<argus-dba-bot-token>' >> ~/.hermes/profiles/dba/.env

# 방법 2: hermes config 명령 (지원되는 경우)
hermes -p backend config set discord.bot_token <argus-backend-bot-token>
hermes -p frontend config set discord.bot_token <argus-frontend-bot-token>
hermes -p dba config set discord.bot_token <argus-dba-bot-token>
```

> **주의**: 각 프로파일의 `.env`에는 **자기 token만** 있어야 합니다.
> `--clone`으로 복사된 기존 `DISCORD_BOT_TOKEN`은 반드시 새 토큰으로 교체하세요.

---

## 4. 프로파일별 Discord 채널 설정

각 프로파일의 `config.yaml`에서 Discord 채널을 지정합니다.

```bash
# 방법 1: 직접 편집
hermes -p backend config edit
# 방법 2: config set
hermes -p backend config set discord.allowed_channels "<channel-id>"
```

설정 예시 (config.yaml):

```yaml
discord:
  require_mention: false          # 전용 채널이면 mention 불필요
  allowed_channels: "1234567890"  # #argus-backend 채널 ID
  auto_thread: true
  reactions: true
```

> 채널 ID 확인 방법: Discord 설정 → 고급 → "개발자 모드" 켜기 → 채널 우클릭 → "ID 복사"

---

## 5. 프로파일별 SOUL.md 작성

각 프로파일에 맞는 SOUL.md를 작성합니다. 파일 위치:

```
~/.hermes/profiles/backend/SOUL.md
~/.hermes/profiles/frontend/SOUL.md
~/.hermes/profiles/dba/SOUL.md
```

내용은 커리큘럼 v2 (02-curriculum-v2.md) S09 섹션의 예시를 참고.
핵심 구성 요소:
- 역할 정의 (Backend Engineer / Frontend Engineer / Database Engineer)
- 기술 스택 지시
- Linear 티켓 처리 규칙 (`@mention` 인식, `area:*` 라벨, `dep:blocked` 스킵)
- `.agent/` 공유 파일 읽기/쓰기 규칙
- PR 생성 및 Discord 보고 지시

---

## 6. 프로파일별 모델 설정

```bash
hermes -p backend setup     # 대화형 설정 마법사
hermes -p frontend setup
hermes -p dba setup
```

또는 직접 지정:
```bash
hermes -p backend model     # 모델 선택 프롬프트
```

> **강의 기준**: Codex를 코딩 도구로 사용하므로 각 프로파일에 `codex` 스킬이 필요합니다.

---

## 7. 프로파일별 스킬 설치

```bash
# 코딩 도구 (필수)
hermes -p backend skills install codex
hermes -p frontend skills install codex
hermes -p dba skills install codex

# GitHub PR 워크플로우
hermes -p backend skills install github-pr-workflow
hermes -p frontend skills install github-pr-workflow
hermes -p dba skills install github-pr-workflow

# Linear 연동 (티켓 폴링용)
hermes -p backend skills install linear
hermes -p frontend skills install linear
hermes -p dba skills install linear
```

---

## 8. Linear 라벨 생성

Argus 프로젝트(ROA 팀)에 티켓 라우팅용 라벨을 생성합니다.

```bash
# area 라벨 (각 프로파일의 담당 영역)
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { labelCreate(input: { name: \"area:backend\", color: \"#5B8DEF\", teamId: \"159aef2b-b7f3-470b-920e-fd1b39407428\" }) { label { id name } success } }"}' | python3 -m json.tool

curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { labelCreate(input: { name: \"area:frontend\", color: \"#4BC6B9\", teamId: \"159aef2b-b7f3-470b-920e-fd1b39407428\" }) { label { id name } success } }"}' | python3 -m json.tool

curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { labelCreate(input: { name: \"area:database\", color: \"#F2C94C\", teamId: \"159aef2b-b7f3-470b-920e-fd1b39407428\" }) { label { id name } success } }"}' | python3 -m json.tool

# 의존성 라벨
curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { labelCreate(input: { name: \"dep:blocked\", color: \"#EB5757\", teamId: \"159aef2b-b7f3-470b-920e-fd1b39407428\" }) { label { id name } success } }"}' | python3 -m json.tool

curl -s -X POST https://api.linear.app/graphql \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"mutation { labelCreate(input: { name: \"dep:ready\", color: \"#27AE60\", teamId: \"159aef2b-b7f3-470b-920e-fd1b39407428\" }) { label { id name } success } }"}' | python3 -m json.tool
```

---

## 9. Argus 레포 초기화

```bash
cd ~/projects/argus

# .agent/ 공유 디렉토리 생성
mkdir -p .agent

# 팀 컨벤션/컨텍스트 파일 생성 (PM이 작성)
touch .agent/context.md
touch .agent/conventions.md

# Git 초기화 (이미 done이라면 skip)
git init
git add -A
git commit -m "chore: argus 프로젝트 초기 구조"
```

---

## 10. Cron Polling 설정

각 프로파일에서 Linear를 주기적으로 폴링하도록 cron을 설정합니다.

```bash
# backend 예시: 5분마다 area:backend 티켓 확인
hermes -p backend cron create \
  --schedule "*/5 * * * *" \
  --prompt "Linear에서 @backend가 언급되고 area:backend 라벨이 달린 Todo/Backlog 티켓을 확인하고, dep:blocked가 없으면 작업을 시작하세요. 작업 시작 전 티켓을 In Progress로 변경하고, 완료 후 PR 링크를 티켓에 코멘트로 남기세요."

# frontend, dba도 동일하게 설정
```

> **참고**: cron 세부 설정은 커리큘럼 v2 S19에서 다룹니다.

---

## 11. Discord Gateway 기동

모든 설정이 완료되면 각 프로파일의 gateway를 기동합니다:

```bash
# 각각 별도 터미널에서 실행
backend gateway    # → argus-backend 봇이 #argus-backend 채널에서 대기
frontend gateway   # → argus-frontend 봇이 #argus-frontend 채널에서 대기
dba gateway        # → argus-dba 봇이 #argus-dba 채널에서 대기
```

정상 기동 확인:
- Discord 서버의 각 채널에 봇이 온라인 상태로 표시되는지 확인
- 채널에 메시지를 보내 봇이 응답하는지 테스트

---

## 12. 검증 체크리스트

| # | 항목 | 확인 방법 |
|---|------|----------|
| 1 | Discord Bot 3개 생성 | Developer Portal에 3개의 Application 존재 |
| 2 | Bot 3개 서버 초대 | 서버 멤버 목록에 3개 봇 표시 |
| 3 | Profile 3개 생성 | `hermes profile list`에 backend, frontend, dba 표시 |
| 4 | Bot Token 분리 주입 | 각 `.env`에 다른 `DISCORD_BOT_TOKEN` |
| 5 | Discord 채널 매핑 | 각 프로파일이 다른 채널에서 응답 |
| 6 | SOUL.md 작성 완료 | `backend chat`으로 역할 인지 확인 |
| 7 | 모델 설정 완료 | `hermes -p backend profile show` 확인 |
| 8 | 스킬 설치 완료 | `hermes -p backend skills list`에 codex 표시 |
| 9 | Linear 라벨 생성 | Linear 프로젝트에 area:*, dep:* 라벨 존재 |
| 10 | `.agent/` 디렉토리 | `context.md`, `conventions.md` 파일 존재 |
| 11 | Gateway 기동 | 3개 봇이 각자 채널에서 온라인 |
| 12 | Cron 동작 | 티켓 생성 후 자동으로 에이전트가 인식 |

---

## 참고: 프로파일 관리 명령어 모음

```bash
# 프로파일 목록
hermes profile list

# 프로파일 상세 보기
hermes profile show backend

# 프로파일 전환 (기본 프로파일 변경)
hermes profile use backend

# 특정 프로파일로 명령 실행
hermes -p backend chat          # 대화
hermes -p backend skills list   # 스킬 목록
hermes -p backend config show   # 설정 보기
hermes -p backend status        # 상태 확인

# 프로파일 삭제 (주의)
hermes profile delete backend

# 프로파일 내보내기/가져오기
hermes profile export backend -o backend.tar.gz
hermes profile import backend.tar.gz --name backend

# alias 관리
hermes profile alias backend          # alias 스크립트 경로 확인
hermes profile alias backend --remove  # alias 삭제
hermes profile alias backend --name argus-be  # 커스텀 alias 이름
```
