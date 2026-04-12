# Argus — 팀 구조 정의

> Hermes 에이전트 팀으로 Argus를 개발하기 위한 프로필 정의.
> 커리큘럼 v2의 팀 빌딩 섹션(Part 2~4)에서 실제로 사용할 구성.

---

## 1. 팀 개요

| 역할 | Profile명 | 책임 영역 | 코딩 도구 |
|------|-----------|-----------|-----------|
| PM (강사/학생) | - | 티켓 등록, 리뷰, 승인 | Linear + Discord |
| Backend Engineer | `backend` | REST API, 데이터 수집 로직, 서비스 구동 | Codex |
| Frontend Engineer | `frontend` | 대시보드 UI, 차트, 브라우저 테스트 | Codex |
| Database Engineer | `dba` | argus.db 스키마, 데이터 검증, 수집 정확도 | Codex |

### PM (사람)

- Linear에 티켓 생성하고 라벨 부여
- Discord에서 각 에이전트의 작업 현황 관찰
- PR 리뷰 → merge → 다음 티켓 의존성 해제
- `.agent/context.md`, `.agent/conventions.md` 작성

---

## 2. Argus 데이터 플로우

```
Hermes Profiles (데이터 원본)
  │  state.db, memory-decay/memories.db, gateway_state.json, auth.json, logs/* ...
  │  (read-only, WAL-safe concurrent access)
  │
  ▼
[DBA] argus.db 스키마 정의
  │  profiles, sessions, tool_usage, credential_status, gateway_events
  │  수집 검증 쿼리, 데이터 품질 테스트
  │
  ▼
[Backend] Collector + REST API
  │  Hermes 데이터 소스 읽기 → argus.db에 적재
  │  REST API 엔드포인트로 프론트엔드에 데이터 제공
  │  argus.db를 SQLite로 직접 서빙
  │
  ▼
[Frontend] Web Dashboard
     Profile 선택 → 대시보드 렌더링
     크로스 프로필 뷰, 실시간 업데이트
```

### 의존성 순서

```
DBA (스키마 + 검증) → Backend (API + 수집) → Frontend (UI)
```

PM은 이 순서대로 Linear 티켓을 생성하고, 선행 작업이 merge된 후 다음 티켓의 `dep:blocked`를 `dep:ready`로 전환.

---

## 3. Profile별 상세 정의

### 3.1 Backend Engineer (`backend`)

**역할**: Hermes 데이터를 수집하고 REST API로 제공하는 서비스를 구축.

**책임 영역**:
- Hermes 프로필 디렉토리 자동 감지 로직
- `state.db`, `memories.db`, JSON 파일 읽기 (read-only)
- 주기적 폴링으로 argus.db에 데이터 적재
- REST API 엔드포인트 설계 및 구현
- `.agent/api-spec.md` 작성/유지

**기술 스택**:
- Python 3.11+
- SQLite (argus.db, WAL 모드)
- Alembic (DB migration — SQLite 지원)
- FastAPI (REST API)
- uvicorn (ASGI 서버)
- SQLAlchemy (ORM)

**읽어야 할 `.agent/` 파일**:
- `.agent/schema-changelog.md` — DBA가 정의한 argus.db 스키마
- `.agent/data-sources.md` — 수집 대상 Hermes 데이터 소스 정리 (PM 또는 DBA 작성)
- `.agent/conventions.md` — 팀 컨벤션

**작성해야 할 `.agent/` 파일**:
- `.agent/api-spec.md` — API 엔드포인트 스펙 (프론트엔드가 참조)
- `.agent/deployment.md` — 서비스 실행 포트/URL 정보

---

### 3.2 Frontend Engineer (`frontend`)

**역할**: Argus 대시보드 Web UI를 구축.

**책임 영역**:
- Profile 선택기 (여러 Hermes 프로필 중 선택)
- Per-Profile 대시보드:
  - Status (게이트웨이 상태, 활성 세션, 모델 정보)
  - Activity (메시지 스트림, 현재 툴 호출)
  - Cost (토큰 사용량, 추정 비용, 캐시 적중률)
  - Tools (사용 분포, 호출 빈도)
- Cross-Profile Overview:
  - 전체 비용, 프로필별 토큰 소비 비교
  - 활동 히트맵
- 반응형 디자인

**기술 스택**:
- HTML + CSS + Vanilla JS (또는 React — PM이 결정)
- Chart.js (또는 비슷한 차트 라이브러리)
- Fetch API로 backend REST 호출

**읽어야 할 `.agent/` 파일**:
- `.agent/api-spec.md` — backend가 정의한 API 엔드포인트
- `.agent/deployment.md` — backend URL/포트

---

### 3.3 Database Engineer (`dba`)

**역할**: argus.db의 스키마를 설계하고, Hermes 데이터 수집의 정확성을 검증.

**책임 영역**:
- argus.db 스키마 설계 (테이블, 인덱스, 제약조건)
- Hermes 원본 데이터 소스(state.db 등)의 실제 스키마 분석
- 데이터 수집이 정확한지 검증하는 테스트 쿼리 작성
- `.agent/schema-changelog.md` 작성/유지
- Hermes DB 읽기 안전성 가이드 (WAL 모드, 동시 읽기 주의사항)

**기술 스택**:
- SQLite (스키마 설계, 인덱스 최적화)
- Alembic (migration 버전 관리 — SQLite 지원)
- SQLAlchemy (ORM 모델 정의)
- Python 3.11+ (스키마 검증 스크립트)
- sqlite3 CLI (수동 검증)

**읽어야 할 `.agent/` 파일**:
- `.agent/data-sources.md` — Hermes 데이터 소스 정의
- `.agent/conventions.md` — 팀 컨벤션

**작성해야 할 `.agent/` 파일**:
- `.agent/schema-changelog.md` — argus.db 스키마 변경 이력
- `.agent/data-sources.md` — Hermes 데이터 소스 매핑 (원본 컬럼 → argus.db 컬럼)

**주의사항**:
- Hermes의 `state.db`는 WAL 모드이므로 Argus가 읽는 동안 Hermes가 쓰기 가능
- `auth.json`은 파일 기반 락(`auth.lock`)이 있으므로 동시 읽기 시 주의
- Default profile(`~/.hermes/`)과 named profiles(`~/.hermes/profiles/<name>/`)의 경로 차이 처리

---

## 4. 협업 규칙

### 4.1 `.agent/` 공유 파일 구조

```
.agent/
├── context.md          ← PM 작성: 프로젝트 전체 컨텍스트
├── conventions.md      ← PM 작성: 팀 컨벤션 (코딩 스타일, 커밋 규칙 등)
├── schema-changelog.md ← DBA 작성: argus.db 스키마 정의 및 변경 이력
├── api-spec.md         ← Backend 작성: REST API 엔드포인트 스펙
├── deployment.md       ← Backend 작성: 서비스 포트/URL 정보
└── data-sources.md     ← DBA 작성: Hermes 데이터 소스 매핑
```

### 4.2 정보 흐름

```
DBA ──작성──→ schema-changelog.md, data-sources.md
                  │
Backend ──읽기──→ schema-changelog.md, data-sources.md
Backend ──작성──→ api-spec.md, deployment.md
                  │
Frontend ──읽기──→ api-spec.md, deployment.md
```

### 4.3 Git Worktree

```
~/projects/argus/              ← main (공유)
~/projects/argus-backend/      ← backend 브랜치 (backend 에이전트 전용)
~/projects/argus-frontend/     ← frontend 브랜치 (frontend 에이전트 전용)
~/projects/argus-dba/          ← dba 브랜치 (dba 에이전트 전용)
```

### 4.4 Linear 라벨 체계

| 카테고리 | 라벨 | 설명 |
|----------|------|------|
| Area (필수, 상호배타) | `area:database` | DBA 작업 |
| | `area:backend` | Backend 작업 |
| | `area:frontend` | Frontend 작업 |
| 의존성 (필요시) | `dep:blocked` | 선행 작업 대기 중 |
| | `dep:ready` | 작업 가능 |

### 4.5 Cron Polling

| Profile | 주기 | 대상 |
|---------|------|------|
| `backend` | 5분 | Linear `area:backend` + `state:todo` 또는 `state:backlog` |
| `frontend` | 5분 | Linear `area:frontend` + `state:todo` 또는 `state:backlog` |
| `dba` | 5분 | Linear `area:database` + `state:todo` 또는 `state:backlog` |

---

## 5. Linear 워크플로우

### 5.1 PM의 작업 흐름

1. Linear에 티켓 생성 (제목 + 설명 + `area:*` 라벨)
2. 선행 의존이 있으면 `dep:blocked` 라벨 부여
3. Discord에서 에이전트 작업 현황 관찰
4. PR 생성 알림 수신 → 코드 리뷰
5. 승인 → merge
6. 종속 티켓의 `dep:blocked` → `dep:ready` 전환

### 5.2 에이전트의 작업 흐름

1. Cron이 Linear에서 자기 `area:*` 티켓 polling
2. `dep:blocked` 티켓은 스킵
3. 작업 가능한 티켓 발견 → Linear 상태를 `In Progress`로 변경
4. `.agent/` 공유 파일에서 필요한 정보 읽기
5. Codex로 작업 수행
6. `.agent/` 공유 파일 업데이트 (api-spec, schema-changelog 등)
7. 커밋 + PR 생성 (PR에 티켓 번호 참조)
8. Linear에 코멘트로 PR 링크 남기기
9. Discord에 작업 완료 보고

---

## 6. 티켓 분할 (초안)

### Phase 1: 기반 (DBA → Backend)

| # | 제목 | Area | 의존성 |
|---|------|------|--------|
| 1 | argus.db 초기 스키마 — profiles, sessions, tool_usage 테이블 | database | - |
| 2 | argus.db 스키마 — credential_status, gateway_events 테이블 | database | - |
| 3 | Hermes 프로필 자동 감지 로직 | backend | #1 |
| 4 | state.db 데이터 수집 → argus.db 적재 | backend | #1, #3 |
| 5 | FastAPI 프로젝트 초기화 + /api/profiles 엔드포인트 | backend | #3, #4 |

### Phase 2: 핵심 기능 (Backend)

| # | 제목 | Area | 의존성 |
|---|------|------|--------|
| 6 | /api/profiles/:id/sessions 엔드포인트 | backend | #5 |
| 7 | /api/profiles/:id/activity 엔드포인트 (실시간 스트림) | backend | #5 |
| 8 | /api/profiles/:id/cost 엔드포인트 | backend | #5 |
| 9 | /api/profiles/:id/tools 엔드포인트 | backend | #5 |
| 10 | /api/overview 엔드포인트 (크로스 프로필) | backend | #6~#9 |

### Phase 3: 대시보드 (Frontend)

| # | 제목 | Area | 의존성 |
|---|------|------|--------|
| 11 | 프론트엔드 프로젝트 초기화 + Profile 선택기 | frontend | #5 |
| 12 | Per-Profile Status 카드 (게이트웨이, 활성 세션, 모델) | frontend | #6 |
| 13 | Per-Profile Cost 대시보드 (토큰, 비용, 캐시) | frontend | #8 |
| 14 | Per-Profile Activity 스트림 | frontend | #7 |
| 15 | Per-Profile Tools 사용 분포 차트 | frontend | #9 |
| 16 | Cross-Profile Overview (전체 비용, 히트맵) | frontend | #10 |

### Phase 4: 폴리싱 (전체)

| # | 제목 | Area | 의존성 |
|---|------|------|--------|
| 17 | credentials_status 엔드포인트 + DBA 검증 테스트 | database | #2, #5 |
| 18 | 반응형 디자인 적용 | frontend | #11~#16 |
| 19 | 에러 핸들링 + 로딩 상태 | frontend | #11~#16 |
| 20 | 최종 통합 테스트 + 버그 수정 | backend, frontend | #17, #18, #19 |

---

## 7. 커리큘럼 v2와의 차이점

커리큘럼 v2는 TaskTracker(PostgreSQL + Alembic + Docker Compose)를 예시로 사용.
Argus는 실제 스택이 다르므로 다음 차이를 강의에서 설명:

| 항목 | TaskTracker (커리큘럼 예시) | Argus (실제 프로젝트) |
|------|---------------------------|----------------------|
| DB | PostgreSQL + Alembic | SQLite + Alembic |
| DB 접근 | Docker Compose로 띄운 PostgreSQL | Hermes가 이미 생성한 파일을 읽기만 함 |
| Migration | Alembic으로 버전 관리 | 동일 — Alembic 사용 |
| ORM | SQLAlchemy | 동일 — SQLAlchemy 사용 |
| 데이터 원본 | 에이전트가 생성한 데이터 | Hermes Agent가 이미 저장한 데이터 |
| 인프라 | Docker Compose (postgres, backend, frontend) | SQLite 파일 기반 (Docker 불필요) |
| 프론트엔드 스택 | React + TanStack Query | PM이 결정 (Vanilla JS / React) |
