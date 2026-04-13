# LINEAR SYNC

- Updated: 2026-04-13 22:36 KST
- Team: Roachdevteam (`ROA`)
- Team ID: `159aef2b-b7f3-470b-920e-fd1b39407428`
- Project: Argus
- Project ID: `07047b2f-b6bd-4a45-a2c8-3e1078b7f34b`
- Source of truth: `SETUP.md` section 10, `policies/06-LINEAR-TICKET-BACKLOG.md`

## Registered labels

| Label | Color | Label ID | Purpose |
|---|---|---|---|
| `area:backend` | `#5B8DEF` | `5b17e245-05bd-462f-b6cd-5f17b8447f10` | backend 에이전트 작업 라우팅 |
| `area:frontend` | `#4BC6B9` | `9d10a7c4-47f9-477d-978d-9722f5343401` | frontend 에이전트 작업 라우팅 |
| `area:database` | `#F2C94C` | `b8eb0ef9-d611-4c92-acf6-304f71ec17b1` | dba 에이전트 작업 라우팅 |
| `dep:blocked` | `#EB5757` | `4246ffa9-c129-4a61-b588-7b496704d7b2` | 선행 의존성 대기 |
| `dep:ready` | `#27AE60` | `da2b23c8-a14a-4c9c-a4d2-62a05fa2d911` | 즉시 작업 가능 |

## Workflow states

| State | Type | State ID |
|---|---|---|
| `Backlog` | `backlog` | `5c682885-e40c-4235-91bc-0a3ff052b2f0` |
| `Todo` | `unstarted` | `f2199d90-e051-42f6-8a6f-019922816244` |
| `In Progress` | `started` | `3036d277-5f9f-4d0f-a1e2-832f94ad790e` |
| `Done` | `completed` | `37712061-37b7-44d6-af1f-d37c9b1db7db` |

## Ticket backlog

| ID | Linear | Title | Area | Depends on | Labels | Status |
|---|---|---|---|---|---|---|
| T1 | ROA-6 | @backend SETUP-01 Hermes 3프로파일/Telegram 그룹 검증 | backend | - | area:backend | Todo |
| T2 | ROA-7 | @backend BE-01 API v1 명세 작성 (.agent/API_SPEC.md) | backend | T1(ROA-6) | area:backend, dep:blocked | Todo |
| T3 | ROA-8 | @backend BE-02 DB 요청 문서 작성 (.agent/DB_REQUEST.md) | backend | T2(ROA-7) | area:backend, dep:blocked | Todo |
| T4 | ROA-9 | @dba DB-01 Docker DB 기동 구성 | database | T3(ROA-8) | area:database, dep:blocked | Todo |
| T5 | ROA-10 | @dba DB-02 스키마/마이그레이션 문서화 (.agent/DB_SPEC.md) | database | T4(ROA-9) | area:database, dep:blocked | Todo |
| T6 | ROA-11 | @dba DB-03 Backend 핸드오프 문서 작성 (.agent/DB_HANDOFF_TO_BACKEND.md) | database | T5(ROA-10) | area:database, dep:blocked | Todo |
| T7 | ROA-12 | @backend BE-03 API 구현 1차 (DB 연동) | backend | T6(ROA-11) | area:backend, dep:blocked | Todo |
| T8 | ROA-13 | @frontend FE-01 DESIGN 기반 화면 골격 구현 | frontend | T1(ROA-6) | area:frontend, dep:blocked | Todo |
| T9 | ROA-14 | @frontend FE-02 API 연동 명세 작성 (.agent/FRONTEND_INTEGRATION.md) | frontend | T2(ROA-7) | area:frontend, dep:blocked | Todo |
| T10 | ROA-15 | @frontend FE-03 API 실제 연동 + 로딩/에러/빈상태 처리 | frontend | T7(ROA-12), T9(ROA-14) | area:frontend, dep:blocked | Todo |
| T11 | ROA-16 | @backend INT-01 통합 검증 및 데모 시나리오 작성 | backend | T10(ROA-15) | area:backend, dep:blocked | Todo |

## Dependency map

```
T1(ROA-6) ──→ T2(ROA-7) ──→ T3(ROA-8) ──→ T4(ROA-9) ──→ T5(ROA-10) ──→ T6(ROA-11) ──→ T7(ROA-12) ─┐
    │                  │                                                                                   │
    └──→ T8(ROA-13)    └──→ T9(ROA-14) ─────────────────────────────────────────────────────────────────┤──→ T10(ROA-15) ──→ T11(ROA-16)
```

## Agent assignment

| Profile | Tickets | First action |
|---|---|---|
| backend | T1, T2, T3, T7, T11 | T1 부터 시작 (의존성 없음) |
| dba | T4, T5, T6 | T3 완료 후 T4 시작 |
| frontend | T8, T9, T10 | T1 완료 후 T8, T2 완료 후 T9 병렬 시작 가능 |
