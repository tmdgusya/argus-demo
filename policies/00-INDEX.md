# Argus Demo 정책서 인덱스

수강생 실습은 아래 순서대로 진행합니다.
기본 운영은 `강사 -> PM 에이전트` 단일 지시 방식입니다.
`backend/dba/frontend`에 직접 붙여넣는 방식은 예외 상황에서만 사용합니다.

## A. 기본 운영(권장)

1. [01-LECTURE-ORCHESTRATOR.md](./01-LECTURE-ORCHESTRATOR.md)
2. [06-LINEAR-TICKET-BACKLOG.md](./06-LINEAR-TICKET-BACKLOG.md)
3. [07-PM-OPERATIONS.md](./07-PM-OPERATIONS.md)
4. [05-HANDOFF-TEMPLATES.md](./05-HANDOFF-TEMPLATES.md)

### 복붙 실행 순서 (강사용)

1. PM 에이전트에게 시작 지시 붙여넣기
- 원본: `07-PM-OPERATIONS.md`의 `2. 강사가 PM에게 보내는 시작 지시문 (복붙)`
- 대상: Hermes PM(Project Manager) 에이전트 채팅

2. Linear 티켓 생성 시 본문 붙여넣기
- 원본: `06-LINEAR-TICKET-BACKLOG.md`의 각 티켓 `제목`, `설명`, `완료 기준(AC)`
- 대상: Linear 이슈 생성 화면

3. 주기 점검 지시 붙여넣기
- 원본: `07-PM-OPERATIONS.md`의 `3. 강사가 PM에게 보내는 주기 점검 지시문 (복붙)`
- 대상: Hermes PM(Project Manager) 에이전트 채팅

4. 종료 점검 지시 붙여넣기
- 원본: `07-PM-OPERATIONS.md`의 `4. 강사가 PM에게 보내는 종료 점검 지시문 (복붙)`
- 대상: Hermes PM(Project Manager) 에이전트 채팅

## B. 직접 지시(옵션)

1. [02-BACKEND-AGENT-POLICY.md](./02-BACKEND-AGENT-POLICY.md)
2. [03-DBA-AGENT-POLICY.md](./03-DBA-AGENT-POLICY.md)
3. [04-FRONTEND-AGENT-POLICY.md](./04-FRONTEND-AGENT-POLICY.md)

## 목표

- `backend`가 API 스펙 문서를 작성해 `dba`에 전달
- `dba`가 Docker 기반 DB를 생성하고 접속 정보를 문서화
- `frontend`가 `DESIGN.md`와 API 문서를 기준으로 화면 완성
- 모든 진행상황은 `.agent/` 문서로 기록

## 공통 원칙

- 말로 전달하지 말고 문서로 전달합니다.
- 각 단계 종료 시 반드시 Telegram 그룹에 완료 보고를 보냅니다.
- 블로커가 생기면 즉시 `dep:blocked` 상태로 문서에 기록합니다.
