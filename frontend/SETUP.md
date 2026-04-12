# Frontend Profile 설정

## 1. Telegram 봇 생성

1. Telegram에서 @BotFather 검색 후 대화 시작
2. `/newbot` 전송
3. 표시 이름 입력: `Argus Frontend`
4. 사용자 이름 입력: `argus_frontend_bot` (고유해야 함, `bot`으로 끝나야 함)
5. BotFather가 준 **API 토큰** 복사

## 2. 그룹 채팅 설정

1. Telegram에서 새 그룹 생성 (예: `Argus Frontend`)
2. 생성한 봇을 그룹에 초대
3. 봇을 그룹 **관리자**로 승격

## 3. 프로파일 생성

```bash
hermes profile create frontend --clone
```

## 4. SOUL.md 교체

```bash
cp ~/projects/argus/frontend/SOUL.md ~/.hermes/profiles/frontend/SOUL.md
```

## 5. Telegram 토큰 설정

```bash
sed -i '/^TELEGRAM_BOT_TOKEN=/d' ~/.hermes/profiles/frontend/.env
echo 'TELEGRAM_BOT_TOKEN=<your-bot-token>' >> ~/.hermes/profiles/frontend/.env
```

## 6. 허용 사용자 설정

```bash
sed -i '/^TELEGRAM_ALLOWED_USERS=/d' ~/.hermes/profiles/frontend/.env
echo 'TELEGRAM_ALLOWED_USERS=<your-telegram-user-id>' >> ~/.hermes/profiles/frontend/.env
```

## 7. 홈 채널 설정 (선택)

```bash
sed -i '/^TELEGRAM_HOME_CHANNEL=/d' ~/.hermes/profiles/frontend/.env
echo 'TELEGRAM_HOME_CHANNEL=<your-group-chat-id>' >> ~/.hermes/profiles/frontend/.env
```

## 8. (선택) 워크플로우 스킬 확인

```bash
hermes -p frontend skills list | grep -E 'github-pr-workflow|linear'
```

> 참고: 이 단계는 setup 필수가 아닙니다.
> Codex는 `codex` CLI 스킬이 아니라 Hermes에 연동하는 provider/모델 의미입니다.

## 9. 확인

```bash
hermes profile show frontend
hermes -p frontend skills list
frontend chat
```

## 10. Gateway 실행

```bash
frontend gateway run
```
