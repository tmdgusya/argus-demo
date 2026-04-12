# Argus — 전체 셋업 가이드

> 이 문서는 Argus 프로젝트를 Hermes 에이전트 팀으로 개발하기 위해
> 사전에 준비해야 할 모든 인프라 셋업 순서를 정리합니다.
> 강의 실습에서는 이 순서대로 진행합니다.

---

## 0. 전제 조건

- Hermes Agent 설치 완료 (`hermes --version` 동작)
- Git 계정 설정 완료 (`gh auth status` 동작)
- Python 3.11+ / Node.js 20+ 설치
- Telegram 앱 또는 Telegram Web 접근 가능

---

## 1. Telegram Bot 3개 생성

> **핵심**: 프로파일마다 **별도 Telegram Bot Token**이 필요합니다.
> backend/frontend/dba가 같은 토큰을 재사용하면 어떤 프로파일이 어떤 대화를 처리하는지 섞입니다.

1. Telegram에서 `@BotFather` 검색 후 대화 시작
2. `/newbot`을 3번 실행해 봇 3개 생성

| Profile | Bot username |
|---------|--------------|
| `backend` | `@argus_backend_bot` |
| `frontend` | `@argus_frontend_bot` |
| `dba` | `@argus_dba_bot` |

3. 각 봇의 토큰을 안전한 곳에 보관

> root `SETUP.md`에서는 전체 흐름만 정리합니다.
> 개별 BotFather 입력/그룹 설정은 `backend/SETUP.md`, `frontend/SETUP.md`, `dba/SETUP.md`에서 각각 진행합니다.

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

## 3. 프로파일별 Telegram Bot Token 주입

각 프로파일의 `.env`에 자신의 Telegram bot token을 설정합니다.

```bash
echo 'TELEGRAM_BOT_TOKEN=<backend-bot-token>' >> ~/.hermes/profiles/backend/.env
echo 'TELEGRAM_BOT_TOKEN=<frontend-bot-token>' >> ~/.hermes/profiles/frontend/.env
echo 'TELEGRAM_BOT_TOKEN=<dba-bot-token>' >> ~/.hermes/profiles/dba/.env
```

> **주의**: 각 프로파일의 `.env`에는 **자기 token만** 있어야 합니다.
> `--clone`으로 복사된 기존 `TELEGRAM_BOT_TOKEN`은 반드시 새 토큰으로 교체하세요.

---

## 4. 프로파일별 Telegram 기본 설정 확인

개별 profile `SETUP.md`에서 다음 값을 설정한 뒤, root 단계에서는 전체 팀 기준으로 다시 확인합니다.

- `TELEGRAM_ALLOWED_USERS`
- `TELEGRAM_HOME_CHANNEL` (선택)

```bash
grep -E 'TELEGRAM_ALLOWED_USERS|TELEGRAM_HOME_CHANNEL' ~/.hermes/profiles/backend/.env
grep -E 'TELEGRAM_ALLOWED_USERS|TELEGRAM_HOME_CHANNEL' ~/.hermes/profiles/frontend/.env
grep -E 'TELEGRAM_ALLOWED_USERS|TELEGRAM_HOME_CHANNEL' ~/.hermes/profiles/dba/.env
```

> 그룹 홈 채널을 쓸 경우 `TELEGRAM_HOME_CHANNEL`에는 그룹 chat ID를 넣습니다.

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
- PR 생성 및 Telegram 그룹 보고 지시

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

> **강의 기준**: Codex는 `codex` CLI 스킬이 아니라 Hermes에 연동하는 provider/모델 의미입니다.
> 따라서 셋업 단계에서는 `openai-codex` 로그인 및 모델 설정만 확인하면 충분합니다.

---

## 7. (선택) 워크플로우 스킬 확인

```bash
hermes -p backend skills list | grep -E 'github-pr-workflow|linear'
hermes -p frontend skills list | grep -E 'github-pr-workflow|linear'
hermes -p dba skills list | grep -E 'github-pr-workflow|linear'
```

> 참고:
> - 이 단계는 setup 필수가 아닙니다.
> - `github-pr-workflow`, `linear`는 반복 작업을 줄이는 보조 스킬입니다.
> - E2E 테스트 하네스는 backend 개발 과정에서 구축하는 산출물이지, 사전 셋업 항목이 아닙니다.

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

## 11. (선택) Telegram 그룹챗 연결

> 이 단계는 `backend/SETUP.md`, `frontend/SETUP.md`, `dba/SETUP.md`를 각각 끝낸 뒤 진행합니다.
> Telegram 그룹챗에서는 default 개인 봇이 아니라 **프로파일 전용 봇 3개**를 직접 초대해야 합니다.

초대할 봇:
- `@argus_backend_bot`
- `@argus_frontend_bot`
- `@argus_dba_bot`

중요:
- 개인 DM에서 응답하던 기본 봇(예: `@roach_robot`)이 있다고 해서 위 3개 프로파일 봇이 자동으로 그룹에서 동작하는 것은 아닙니다.
- Telegram 그룹에서는 **실제로 그룹에 들어가 있는 봇 username**과 **현재 띄운 gateway 프로파일**이 일치해야 합니다.

권장 순서:
1. Telegram 그룹을 하나 만든다
2. 위 3개 봇을 그룹에 초대한다
3. 각 봇에 대해 개별 profile gateway를 띄운다
4. 그룹에서 각 봇을 정확한 username으로 멘션해 응답을 확인한다

```bash
# 각각 별도 터미널에서 실행
backend gateway    # → @argus_backend_bot 담당
frontend gateway   # → @argus_frontend_bot 담당
dba gateway        # → @argus_dba_bot 담당
```

빠른 검증 예시:
- `@argus_backend_bot 안녕하세요`
- `@argus_frontend_bot 안녕하세요`
- `@argus_dba_bot 안녕하세요`

응답이 없으면 먼저 확인할 것:
- 그룹에 초대한 봇 username이 정말 `@argus_backend_bot`, `@argus_frontend_bot`, `@argus_dba_bot`인지
- 해당 프로파일 gateway가 실제로 떠 있는지 (`backend gateway`, `frontend gateway`, `dba gateway`)
- 개별 profile `SETUP.md`에서 안내한 Telegram privacy/admin 설정을 적용했는지

---

## 12. Gateway 기동

모든 설정이 완료되면 각 프로파일의 gateway를 기동합니다:

```bash
# 각각 별도 터미널에서 실행
backend gateway    # → @argus_backend_bot 담당
frontend gateway   # → @argus_frontend_bot 담당
dba gateway        # → @argus_dba_bot 담당
```

정상 기동 확인:
- 각 gateway 프로세스가 정상 실행 중인지 확인
- Telegram 그룹에서 각 봇을 멘션했을 때 응답하는지 테스트

---

## 13. 검증 체크리스트

| # | 항목 | 확인 방법 |
|---|------|----------|
| 1 | Telegram Bot 3개 생성 | BotFather에서 3개 봇 생성 완료 |
| 2 | Profile 3개 생성 | `hermes profile list`에 backend, frontend, dba 표시 |
| 3 | Bot Token 분리 주입 | 각 `.env`에 다른 `TELEGRAM_BOT_TOKEN` |
| 4 | Telegram 기본 설정 확인 | 각 프로파일에 `TELEGRAM_ALLOWED_USERS`가 설정됨 |
| 5 | SOUL.md 작성 완료 | `backend chat`으로 역할 인지 확인 |
| 6 | 모델 설정 완료 | `hermes -p backend profile show` 확인 |
| 7 | (선택) workflow skill 확인 | `hermes -p backend skills list`에 github-pr-workflow / linear 표시 |
| 8 | Linear 라벨 생성 | Linear 프로젝트에 area:*, dep:* 라벨 존재 |
| 9 | `.agent/` 디렉토리 | `context.md`, `conventions.md` 파일 존재 |
| 10 | Telegram 그룹챗 연결 (선택) | `@argus_backend_bot`, `@argus_frontend_bot`, `@argus_dba_bot` 멘션 시 각 프로파일이 응답 |
| 11 | Gateway 기동 | 3개 gateway 프로세스가 정상 실행 |
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
