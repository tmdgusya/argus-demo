# DBA Profile 설정

## 1. Telegram 봇 생성

1. Telegram에서 @BotFather 검색 후 대화 시작
2. `/newbot` 전송
3. 표시 이름 입력: `Argus DBA`
4. 사용자 이름 입력: `argus_dba_bot` (고유해야 함, `bot`으로 끝나야 함)
5. BotFather가 준 **API 토큰** 복사

## 2. 그룹 채팅 설정

1. Telegram에서 새 그룹 생성 (예: `Argus DBA`)
2. 생성한 봇을 그룹에 초대
3. 봇을 그룹 **관리자**로 승격

## 3. 프로파일 생성

```bash
hermes profile create dba --clone
```

## 4. SOUL.md 교체

```bash
cp ~/projects/argus/dba/SOUL.md ~/.hermes/profiles/dba/SOUL.md
```

## 5. Telegram 토큰 설정

```bash
sed -i '/^TELEGRAM_BOT_TOKEN=/d' ~/.hermes/profiles/dba/.env
echo 'TELEGRAM_BOT_TOKEN=<your-bot-token>' >> ~/.hermes/profiles/dba/.env
```

## 6. 허용 사용자 설정

```bash
sed -i '/^TELEGRAM_ALLOWED_USERS=/d' ~/.hermes/profiles/dba/.env
echo 'TELEGRAM_ALLOWED_USERS=<your-telegram-user-id>' >> ~/.hermes/profiles/dba/.env
```

## 7. 홈 채널 설정 (선택)

```bash
sed -i '/^TELEGRAM_HOME_CHANNEL=/d' ~/.hermes/profiles/dba/.env
echo 'TELEGRAM_HOME_CHANNEL=<your-group-chat-id>' >> ~/.hermes/profiles/dba/.env
```

## 8. Discord 토큰 제거 (있는 경우)

```bash
sed -i '/^DISCORD_BOT_TOKEN=/d' ~/.hermes/profiles/dba/.env
sed -i '/^DISCORD_ALLOWED_USERS=/d' ~/.hermes/profiles/dba/.env
```

## 9. 스킬 설치

```bash
hermes -p dba skills install codex
hermes -p dba skills install github-pr-workflow
hermes -p dba skills install linear
```

## 10. 확인

```bash
hermes profile show dba
hermes -p dba skills list
dba chat
```

## 11. Gateway 실행

```bash
dba gateway run
```
