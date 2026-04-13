# 강사용 오케스트레이션 정책서

이 문서는 강사가 전체 에이전트를 지휘할 때 쓰는 운영 기준입니다.

## 1. 시작 전 체크

1. `SETUP.md` 기준으로 `backend/frontend/dba` gateway가 모두 실행 중인지 확인
2. Telegram 그룹에서 세 봇 모두 `@멘션` 응답 확인
3. 각 봇 `/sethome` 실행 완료 확인
4. 저장소 루트에 `DESIGN.md` 존재 확인

## 2. 실습 실행 순서

1. PM 에이전트에게 Linear 백로그(T1~T11) 등록/정리 지시
2. PM 에이전트가 backend/dba/frontend에 작업 배분
3. PM 에이전트가 `.agent/` 문서와 티켓 상태를 동기화
4. 마지막에 통합 테스트/데모 시나리오 수행

## 3. 강사가 실제로 보내는 1차 지시문

아래를 PM 에이전트에게 보냅니다.

```text
너는 Argus 프로젝트의 Project Manager다.
policies/06-LINEAR-TICKET-BACKLOG.md를 기준으로 T1~T11 티켓을 정리/등록해라.
기능 에이전트에게 작업을 배분하고, 상태를 .agent 문서와 동기화해라.
- 작업 시작 전에 현재 목표/산출물을 3줄로 요약
- 5분마다 진행상황 보고
- 산출물은 반드시 .agent/ 문서로 남길 것
- 블로커 발생 시 원인/필요 입력/대안 3가지를 함께 보고
```

직접 지시가 필요한 경우에만 `02~04` 문서를 사용합니다.

## 4. 상태 문서 위치

- 전체 진행판: `.agent/STATUS.md`
- 백엔드 API: `.agent/API_SPEC.md`
- DBA 스키마/실행법: `.agent/DB_SPEC.md`
- 프론트 연동 체크리스트: `.agent/FRONTEND_INTEGRATION.md`

## 5. 게이트(통과 기준)

1. Backend Gate
- `.agent/API_SPEC.md`에 엔드포인트/요청/응답/에러코드가 명시됨

2. DBA Gate
- `docker compose up -d`로 DB가 기동됨
- `.agent/DB_SPEC.md`에 마이그레이션/시드/접속 정보가 명시됨

3. Frontend Gate
- `DESIGN.md`를 반영한 주요 화면 구현
- API 연동 완료 및 실패 상태 UI 처리

## 6. 권장 운영 명령

```bash
# 상태 문서 먼저 만들기
mkdir -p .agent
: > .agent/STATUS.md
: > .agent/API_SPEC.md
: > .agent/DB_SPEC.md
: > .agent/FRONTEND_INTEGRATION.md
```
