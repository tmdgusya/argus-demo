# Backend Profile 설정

## 1. Telegram 봇 생성

1. Telegram에서 @BotFather 검색 후 대화 시작
2. `/newbot` 전송
3. 표시 이름 입력: `Argus Backend`
4. 사용자 이름 입력: `argus_backend_bot` (고유해야 함, `bot`으로 끝나야 함)
5. BotFather가 준 **API 토큰** 복사 (예: `123456789:ABCdefGHIjklMNOpqrSTUvwxYZ`)

## 2. 그룹 채팅 설정

1. Telegram에서 새 그룹 생성 (예: `Argus Backend`)
2. 생성한 봇을 그룹에 초대
3. 봇을 그룹 **관리자**로 승격 (모든 메시지를 읽으려면 필요)

> **관리자 승격이 필요한 이유**: Telegram 봇은 기본적으로 privacy mode가 켜져 있어서
> `/` 명령어만 볼 수 있습니다. 관리자로 승격하면 모든 메시지를 읽을 수 있습니다.

## 3. 프로파일 생성

```bash
hermes profile create backend --clone
```

`--clone`은 default 프로파일의 config.yaml, .env, SOUL.md를 복사합니다.

## 4. SOUL.md 교체

```bash
cp ~/projects/argus/backend/SOUL.md ~/.hermes/profiles/backend/SOUL.md
```

## 5. Telegram 토큰 설정

기존 Telegram 토큰이 복사되어 있으므로 **반드시 새 봇 토큰으로 교체**해야 합니다:

```bash
# 기존 Telegram 토큰 줄 삭제
sed -i '/^TELEGRAM_BOT_TOKEN=/d' ~/.hermes/profiles/backend/.env

# 새 봇 토큰 추가 (Step 1에서 복사한 토큰)
echo 'TELEGRAM_BOT_TOKEN=<your-bot-token>' >> ~/.hermes/profiles/backend/.env
```

## 6. 허용 사용자 설정

```bash
# TELEGRAM_ALLOWED_USERS 줄이 있으면 삭제
sed -i '/^TELEGRAM_ALLOWED_USERS=/d' ~/.hermes/profiles/backend/.env

# 본인의 Telegram user ID 추가
echo 'TELEGRAM_ALLOWED_USERS=<your-telegram-user-id>' >> ~/.hermes/profiles/backend/.env
```

> Telegram user ID는 @userinfobot 에게 메시지를 보내면 알 수 있습니다.

## 7. 홈 채널 설정 (선택)

cron 작업 결과를 받을 채널:

```bash
# 그룹 채팅 ID 추가 (그룹 채팅 ID는 음수)
sed -i '/^TELEGRAM_HOME_CHANNEL=/d' ~/.hermes/profiles/backend/.env
echo 'TELEGRAM_HOME_CHANNEL=<your-group-chat-id>' >> ~/.hermes/profiles/backend/.env
```

> 그룹 채팅 ID는 @userinfobot을 그룹에 초대하면 확인할 수 있습니다.

## 8. (선택) 워크플로우 스킬 확인

```bash
hermes -p backend skills list | grep -E 'github-pr-workflow|linear'
```

> 참고:
> - 이 단계는 setup 필수가 아닙니다.
> - Codex는 `codex` CLI 스킬이 아니라 Hermes에 연동하는 provider/모델 의미입니다.
> - E2E 테스트 하네스는 backend 개발 과정에서 구축하는 산출물이지, 셋업 선행조건이 아닙니다.

## 9. 확인

```bash
hermes profile show backend
hermes -p backend skills list
backend chat
```

## 10. Gateway 실행

별도 터미널에서 foreground로 실행:

```bash
backend gateway run
```

systemd 서비스로 백그라운드 실행 (선택):

```bash
backend gateway install
backend gateway start
backend gateway status
backend gateway logs
backend gateway stop
```
