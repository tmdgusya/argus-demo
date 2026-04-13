# 에이전트 간 핸드오프 템플릿

아래 템플릿을 복사해서 `.agent/` 문서로 사용하세요.

## 1. Backend -> DBA (`.agent/DB_REQUEST.md`)

```md
# DB Request from Backend

## 목적
- 어떤 API/기능을 위해 DB가 필요한지

## 필요한 엔티티
- Entity:
- 필수 컬럼:
- 제약조건:

## 쿼리 패턴
- 조회:
- 생성:
- 수정:
- 삭제:

## 성능 요구
- 예상 트래픽:
- 인덱스 필요 조건:

## 완료 기준
- DBA가 제공해야 할 산출물:
```

## 2. DBA -> Backend (`.agent/DB_HANDOFF_TO_BACKEND.md`)

```md
# DB Handoff to Backend

## 실행 정보
- 실행 명령: `docker compose up -d`
- 중지 명령: `docker compose down`

## 접속 정보
- HOST:
- PORT:
- DB_NAME:
- USER:

## 마이그레이션/시드
- 마이그레이션 명령:
- 시드 명령:

## 구현 시 주의사항
- 트랜잭션 정책:
- FK/인덱스 관련 주의:
```

## 3. Backend -> Frontend (`.agent/API_SPEC.md` 핵심 섹션)

```md
# API Spec v1

## 인증
- 방식:
- 토큰 헤더:

## Endpoint
### GET /...
- Request:
- Response:
- Error:

### POST /...
- Request:
- Response:
- Error:
```

## 4. Frontend 완료 보고 템플릿

```md
# Frontend Delivery Report

## 구현 화면
- 화면 1:
- 화면 2:

## API 연동 현황
- 완료:
- 미완료:

## 사용자 상태 처리
- 로딩:
- 에러:
- 빈 상태:

## 데모 방법
- 실행 명령:
- 확인 URL:
```
