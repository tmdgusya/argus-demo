# Linear 티켓 백로그 (수강생 복붙용)

이 문서는 강의 실습용 Argus 작업을 Linear에 등록하기 위한 복붙 템플릿입니다.

사용 방법:
1. 각 섹션의 `제목`, `설명`, `완료 기준(AC)`을 Linear 이슈 생성 화면에 그대로 붙여넣습니다.
2. 라벨(`area:*`, `dep:blocked`)을 동일하게 적용합니다.
3. 의존성이 있는 티켓은 선행 티켓 완료 전 `dep:blocked`를 유지합니다.

---

## 공통 라벨

- `area:backend`
- `area:frontend`
- `area:database`
- `dep:blocked`

---

## T1. SETUP-01 Hermes 3프로파일/Telegram 그룹 검증

### 제목
```text
SETUP-01 Hermes 3프로파일/Telegram 그룹 검증
```

### 설명
```text
목표:
- backend/frontend/dba Hermes 프로파일과 Telegram 연동을 실습 가능한 상태로 준비한다.

작업:
- 각 프로파일 gateway 실행 확인
- 그룹챗에서 각 봇 멘션 응답 확인
- 각 봇 /sethome 설정
- .agent/STATUS.md 초기화

참고 문서:
- SETUP.md
- backend/SETUP.md
- frontend/SETUP.md
- dba/SETUP.md
```

### 완료 기준(AC)
```text
- backend/frontend/dba 봇이 그룹 멘션에 각각 응답한다.
- 각 프로파일 .env에 TELEGRAM_HOME_CHANNEL이 설정되었다.
- .agent/STATUS.md에 시작 상태가 기록되었다.
```

### 라벨
```text
area:backend
```

### 의존성
```text
없음
```

---

## T2. BE-01 API v1 명세 작성 (.agent/API_SPEC.md)

### 제목
```text
BE-01 API v1 명세 작성 (.agent/API_SPEC.md)
```

### 설명
```text
목표:
- 프론트엔드와 DBA가 동시에 참고 가능한 API 기준 문서를 만든다.

작업:
- 인증 방식 명시
- 핵심 엔드포인트 정의 (요청/응답/에러)
- JSON 예시 포함
- 버전 표기(v1)

산출물:
- .agent/API_SPEC.md
```

### 완료 기준(AC)
```text
- 인증, 엔드포인트, 요청/응답, 에러 코드가 문서에 있다.
- 각 엔드포인트에 예시 JSON이 있다.
- 프론트/DBA가 추가 질문 없이 다음 작업을 시작할 수 있다.
```

### 라벨
```text
area:backend
```

### 의존성
```text
T1
```

---

## T3. BE-02 DB 요청 문서 작성 (.agent/DB_REQUEST.md)

### 제목
```text
BE-02 DB 요청 문서 작성 (.agent/DB_REQUEST.md)
```

### 설명
```text
목표:
- DBA가 Docker DB와 스키마를 설계할 수 있도록 요구사항을 전달한다.

작업:
- 필요한 엔티티/컬럼/제약 정의
- 조회/생성/수정/삭제 쿼리 패턴 정의
- 트랜잭션/정합성/인덱스 요구 명시

산출물:
- .agent/DB_REQUEST.md
```

### 완료 기준(AC)
```text
- 엔티티별 필수 컬럼/제약이 명시되었다.
- 예상 쿼리 패턴과 성능 요구가 기록되었다.
- DBA가 바로 구현 가능한 수준으로 작성되었다.
```

### 라벨
```text
area:backend
```

### 의존성
```text
T2
```

---

## T4. DB-01 Docker DB 기동 구성

### 제목
```text
DB-01 Docker DB 기동 구성
```

### 설명
```text
목표:
- 수강생이 동일한 명령으로 로컬 DB를 실행할 수 있게 한다.

작업:
- docker-compose.yml 준비 (또는 동등한 실행 스크립트)
- 실행/중지/로그 확인 명령 정리
- 기본 접속 확인

산출물:
- DB 실행 구성 파일
- 실행 명령 문서
```

### 완료 기준(AC)
```text
- docker compose up -d 로 DB가 기동된다.
- 접속 확인 명령이 성공한다.
- 재실행 절차(down/up)가 문서화되었다.
```

### 라벨
```text
area:database
```

### 의존성
```text
T3
```

---

## T5. DB-02 스키마/마이그레이션 문서화 (.agent/DB_SPEC.md)

### 제목
```text
DB-02 스키마/마이그레이션 문서화 (.agent/DB_SPEC.md)
```

### 설명
```text
목표:
- DB 구조와 변경 절차를 수강생이 그대로 따라할 수 있게 한다.

작업:
- 테이블/컬럼/타입/PK/FK/인덱스 정리
- 마이그레이션 순서 정리
- 시드 정책 정리

산출물:
- .agent/DB_SPEC.md
```

### 완료 기준(AC)
```text
- 핵심 테이블 정의가 문서로 정리되었다.
- 마이그레이션 순서가 단계별로 명확하다.
- 시드 데이터 정책이 포함되었다.
```

### 라벨
```text
area:database
```

### 의존성
```text
T4
```

---

## T6. DB-03 Backend 핸드오프 문서 작성 (.agent/DB_HANDOFF_TO_BACKEND.md)

### 제목
```text
DB-03 Backend 핸드오프 문서 작성 (.agent/DB_HANDOFF_TO_BACKEND.md)
```

### 설명
```text
목표:
- 백엔드가 DB 연결/구현을 바로 진행할 수 있도록 핸드오프 문서를 제공한다.

작업:
- 접속 정보(host/port/db/user) 정리
- 마이그레이션/시드 실행 명령 정리
- 구현 주의사항(트랜잭션/FK/인덱스) 정리

산출물:
- .agent/DB_HANDOFF_TO_BACKEND.md
```

### 완료 기준(AC)
```text
- 백엔드가 추가 확인 없이 DB 연결 가능하다.
- 실행 명령이 재현 가능하다.
- 주의사항이 명확히 기록되었다.
```

### 라벨
```text
area:database
```

### 의존성
```text
T5
```

---

## T7. BE-03 API 구현 1차 (DB 연동)

### 제목
```text
BE-03 API 구현 1차 (DB 연동)
```

### 설명
```text
목표:
- API_SPEC 기준으로 핵심 API를 구현하고 DB와 연동한다.

작업:
- 핵심 엔드포인트 구현
- DB 연동 로직 구현
- 에러 응답 형식 일치 확인

산출물:
- 동작하는 API 서버
- 변경 내용 문서/코멘트
```

### 완료 기준(AC)
```text
- 핵심 API가 DB와 연동되어 동작한다.
- API_SPEC 응답 형식과 실제 구현이 일치한다.
- 실패 케이스 에러 응답이 일관된다.
```

### 라벨
```text
area:backend
```

### 의존성
```text
T6
```

---

## T8. FE-01 DESIGN 기반 화면 골격 구현

### 제목
```text
FE-01 DESIGN 기반 화면 골격 구현
```

### 설명
```text
목표:
- DESIGN.md를 기준으로 핵심 화면 구조를 먼저 완성한다.

작업:
- 주요 페이지 레이아웃 구현
- 공통 컴포넌트 구조 정리
- 라우팅/기본 상태 화면 구성

산출물:
- 초기 화면 골격
```

### 완료 기준(AC)
```text
- 핵심 화면이 DESIGN.md 방향과 일치한다.
- 주요 사용자 동선 화면이 준비되었다.
- API 연동 전 목업 상태로 화면 확인 가능하다.
```

### 라벨
```text
area:frontend
```

### 의존성
```text
T1
```

---

## T9. FE-02 API 연동 명세 작성 (.agent/FRONTEND_INTEGRATION.md)

### 제목
```text
FE-02 API 연동 명세 작성 (.agent/FRONTEND_INTEGRATION.md)
```

### 설명
```text
목표:
- API 호출 위치/파라미터/응답 매핑을 프론트 기준으로 고정한다.

작업:
- 화면별 API 호출 지점 정의
- 요청 파라미터/응답 매핑 정의
- 에러/로딩/빈 상태 처리 규칙 정의

산출물:
- .agent/FRONTEND_INTEGRATION.md
```

### 완료 기준(AC)
```text
- 화면별 API 매핑이 문서화되었다.
- 실패/로딩/빈 상태 정책이 포함되었다.
- 백엔드 명세와 충돌 여부가 확인되었다.
```

### 라벨
```text
area:frontend
```

### 의존성
```text
T2
```

---

## T10. FE-03 API 실제 연동 + 로딩/에러/빈상태 처리

### 제목
```text
FE-03 API 실제 연동 + 로딩/에러/빈상태 처리
```

### 설명
```text
목표:
- 프론트 화면을 실제 API와 연결하고 사용자 상태 처리를 완성한다.

작업:
- API 호출 실제 연결
- 성공/실패/로딩/빈 상태 UI 반영
- 주요 사용자 플로우 수동 검증

산출물:
- API 연동된 프론트 화면
```

### 완료 기준(AC)
```text
- 핵심 사용자 플로우가 실제 API로 동작한다.
- 로딩/에러/빈 상태가 화면에서 확인된다.
- 데모 가능한 경로가 정리되었다.
```

### 라벨
```text
area:frontend
```

### 의존성
```text
T7, T9
```

---

## T11. INT-01 통합 검증 및 데모 시나리오 작성

### 제목
```text
INT-01 통합 검증 및 데모 시나리오 작성
```

### 설명
```text
목표:
- 수강생이 처음부터 끝까지 재현 가능한 통합 데모 시나리오를 완성한다.

작업:
- 백엔드/DB/프론트 통합 동작 검증
- 실행 순서와 체크리스트 문서화
- 데모 시나리오 작성

산출물:
- 통합 검증 기록
- 데모 시나리오 문서
```

### 완료 기준(AC)
```text
- 통합 플로우가 재현 가능하다.
- 실패 시 점검 순서가 문서화되었다.
- 강의에서 바로 사용할 데모 시나리오가 준비되었다.
```

### 라벨
```text
area:backend
```

### 의존성
```text
T10
```

---

## 의존성 맵 (요약)

```text
T1 -> T2 -> T3 -> T4 -> T5 -> T6 -> T7
T1 -> T8
T2 -> T9
T7 + T9 -> T10 -> T11
```

## 운영 규칙 (복붙용)

```text
- 선행 의존성이 끝나기 전에는 dep:blocked 라벨 유지
- 티켓 완료 코멘트에는 반드시 .agent/* 문서 링크 첨부
- 완료 기준(AC) 미충족 상태에서는 Done으로 이동 금지
```

---

## PM -> 에이전트 작업 지시문 (복붙용)

아래 블록은 PM이 각 에이전트에게 티켓을 할당할 때 그대로 사용합니다.

### Backend 할당

```text
당신은 Backend 담당입니다.
이번 작업 티켓: [TICKET_KEY] [TICKET_TITLE]
참고 문서: .agent/API_SPEC.md, .agent/DB_REQUEST.md, .agent/DB_HANDOFF_TO_BACKEND.md

요청:
1) 티켓의 완료기준(AC)을 먼저 다시 적고 시작하세요.
2) 구현/문서 작업 후 .agent 문서를 갱신하세요.
3) 완료 시 아래 형식으로 보고하세요.

완료 보고 형식:
- 완료 티켓:
- 변경 파일:
- 검증 방법:
- 남은 리스크:
```

### DBA 할당

```text
당신은 DBA 담당입니다.
이번 작업 티켓: [TICKET_KEY] [TICKET_TITLE]
참고 문서: .agent/DB_REQUEST.md, .agent/DB_SPEC.md

요청:
1) Docker 기반 실행 명령을 먼저 확인하세요.
2) 스키마/마이그레이션/시드 정책을 문서화하세요.
3) 완료 시 백엔드가 즉시 쓸 수 있는 핸드오프를 남기세요.

완료 보고 형식:
- 완료 티켓:
- 실행 명령:
- 접속 정보 문서:
- 검증 결과:
```

### Frontend 할당

```text
당신은 Frontend 담당입니다.
이번 작업 티켓: [TICKET_KEY] [TICKET_TITLE]
참고 문서: DESIGN.md, .agent/API_SPEC.md, .agent/FRONTEND_INTEGRATION.md

요청:
1) DESIGN.md 기준으로 화면/컴포넌트 반영 여부를 먼저 체크하세요.
2) API 연동 시 로딩/에러/빈 상태를 반드시 포함하세요.
3) 완료 시 데모 가능한 화면 경로를 보고하세요.

완료 보고 형식:
- 완료 티켓:
- 구현 화면:
- 연동 API:
- 미완료 항목:
```
