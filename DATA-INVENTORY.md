# Argus — Hermes Agent Monitoring: Data Inventory

> Hermes Agent가 생성/저장하는 모든 데이터 소스와 그 스키마를 정리한 문서.
> Argus는 이 데이터들을 수집해 SQLite에 적재하고 모니터링/관리 기능을 제공한다.

---

## 1. Primary Data Source: `state.db` (SQLite)

**경로**: `~/.hermes/state.db` (현재 ~25MB, WAL 모드)
**Schema Version**: 6 (6번 마이그레이션 거침)

이게 Argus의 핵심 데이터 소스다. Hermes의 모든 세션/메시지/토큰 사용량이 여기에 있다.

### 1.1 `sessions` 테이블 (136 rows)

| Column | Type | 설명 | Argus 활용 |
|--------|------|------|------------|
| `id` | TEXT PK | `YYYYMMDD_HHMMSS_hex` 형식 | 세션 고유 키 |
| `source` | TEXT | `cli`, `telegram` 등 | 플랫폼별 통계 |
| `user_id` | TEXT | 플랫폼별 사용자 ID | 사용자 추적 |
| `model` | TEXT | `glm-5-turbo`, `claude-opus-4` 등 | 모델별 사용량/비용 |
| `model_config` | TEXT (JSON) | 모델 설정 스냅샷 | 설정 변경 이력 |
| `system_prompt` | TEXT | 시스템 프롬프트 전문 | 프롬프트 변화 추적 |
| `parent_session_id` | TEXT FK | 압축 분할/서브에이전트 체인 | 세션 계층 구조 |
| `started_at` | REAL | Unix timestamp | 시간대별 활동 분석 |
| `ended_at` | REAL | NULL이면 활성 세션 | 세션 지속시간 계산 |
| `end_reason` | TEXT | 종료 사유 | 세션 종료 패턴 분석 |
| `message_count` | INTEGER | 메시지 수 (자동 증가) | 세션 규모 지표 |
| `tool_call_count` | INTEGER | 툴 호출 수 (자동 증가) | 툴 사용 강도 |
| `input_tokens` | INTEGER | 누적 입력 토큰 | 비용 계산 |
| `output_tokens` | INTEGER | 누적 출력 토큰 | 비용 계산 |
| `cache_read_tokens` | INTEGER | 캐시 적중 토큰 | 캐시 효율성 |
| `cache_write_tokens` | INTEGER | 캐시 기록 토큰 | 캐시 효율성 |
| `reasoning_tokens` | INTEGER | 추론(사고) 토큰 | 모델 사고량 분석 |
| `billing_provider` | TEXT | `zai`, `anthropic` 등 | 프로바이더별 비용 |
| `billing_base_url` | TEXT | API 엔드포인트 URL | 엔드포인트 추적 |
| `billing_mode` | TEXT | `standard` 등 | 과금 모드 |
| `estimated_cost_usd` | REAL | 계산된 비용 | 비용 모니터링 ★ |
| `actual_cost_usd` | REAL | 프로바이더 보고 비용 | 비용 모니터링 ★ |
| `cost_status` | TEXT | `unknown` 등 | 비용 추적 정확도 |
| `cost_source` | TEXT | 비용 데이터 출처 | 데이터 품질 |
| `pricing_version` | TEXT | 가격표 버전 | 가격 변화 추적 |
| `title` | TEXT | 세션 제목 (최대 100자, UNIQUE) | 세션 검색/분류 |

**Indexes**: `idx_sessions_source`, `idx_sessions_parent`, `idx_sessions_started` (DESC), `idx_sessions_title_unique` (partial unique)

### 1.2 `messages` 테이블 (5,625 rows)

| Column | Type | 설명 | Argus 활용 |
|--------|------|------|------------|
| `id` | INTEGER PK | 자동 증가 | 메시지 고유 키 |
| `session_id` | TEXT FK → sessions | 세션 참조 | 세션별 메시지 조회 |
| `role` | TEXT | `user`, `assistant`, `tool`, `system` | 역할별 통계 |
| `content` | TEXT | 메시지 내용 (JSON 인코딩될 수 있음) | 내용 검색/분석 |
| `tool_call_id` | TEXT | 툴 결과 → 툴 호출 연결 | 툴 호출-결과 매칭 |
| `tool_calls` | TEXT (JSON) | 툴 호출 스펙 배열 (assistant 메시지) | 툴 사용 패턴 분석 ★ |
| `tool_name` | TEXT | 툴 결과의 툴 이름 | 툴별 호출 빈도 |
| `timestamp` | REAL | Unix timestamp | 시계열 분석 |
| `token_count` | INTEGER | 메시지별 토큰 수 | 메시지별 비용 |
| `finish_reason` | TEXT | `tool_calls`, `stop` 등 | 응답 종료 패턴 |
| `reasoning` | TEXT | 어시스턴트 추론 텍스트 | 사고 과정 분석 |
| `reasoning_details` | TEXT (JSON) | 구조화된 추론 | 추론 패턴 분석 |
| `codex_reasoning_items` | TEXT (JSON) | Codex 추론 항목 | 모델별 추론 비교 |

**Index**: `idx_messages_session` on `(session_id, timestamp)`

### 1.3 `messages_fts` (FTS5 가상 테이블)

`messages.content` 전체 텍스트 검색용. 자동 동기화 트리거 포함.

**Argus 활용**: 세션 내용 검색, 특정 토픽이 언급된 세션 찾기 등.

### 1.4 `schema_version` 테이블

현재 스키마 버전 추적. 마이그레이션 감지에 활용 가능.

---

## 2. Memory System: `memory-decay/memories.db` (SQLite)

**경로**: `~/.hermes/memory-decay/memories.db` (~4.3MB)
**플러그인**: `hermes-memory-decay` (포트 8100)

### 2.1 `memories` 테이블 (219 rows)

| Column | Type | 설명 | Argus 활용 |
|--------|------|------|------------|
| `id` | TEXT | 메모리 고유 ID | 메모리 추적 |
| `user_id` | TEXT | 사용자 ID | 사용자별 메모리 |
| `content` | TEXT | 메모리 내용 | 메모리 검색/분석 |
| `mtype` | TEXT | `episode`, `fact` 등 | 메모리 유형별 통계 |
| `category` | TEXT | 분류 태그 | 카테고리별 분포 |
| `importance` | REAL (0-1) | 중요도 점수 | 메모리 품질 지표 |
| `speaker` | TEXT | 발화자 (`user`, `assistant`) | 사용자 vs 에이전트 메모리 비율 |
| `created_tick` | INTEGER | 생성 틱 | 시계열 |
| `storage_score` | REAL | 저장 점수 | 디케이 상태 |
| `retrieval_score` | REAL | 검색 점수 | 검색 효율성 |
| `stability_score` | REAL | 안정성 점수 | 메모리 지속성 |
| `last_activated_tick` | INTEGER | 마지막 활성화 틱 | 사용 빈도 |
| `last_reinforced_tick` | INTEGER | 마지막 강화 틱 | 강화 패턴 |
| `retrieval_count` | INTEGER | 검색 횟수 | 메모리 활용도 ★ |

### 2.2 `activation_history` 테이블 (686 rows)

| Column | Type | 설명 | Argus 활용 |
|--------|------|------|------------|
| `memory_id` | TEXT FK | 참조 메모리 | 메모리별 활성화 이력 |
| `tick` | INTEGER | 활성화 틱 | 시계열 |
| `score` | REAL | 활성화 점수 | 메모리 관련성 변화 |
| `timestamp` | TEXT | 활성화 시간 | 시계열 분석 |

### 2.3 `associations` 테이블 (0 rows)

메모리 간 연결 관계 (source_id, target_id, weight).

### 2.4 `embedding_cache` 테이블 (221 rows)

텍스트 임베딩 캐시 (hash, model, blob). `sqlite-vec` 확장 사용.

### 2.5 `metadata` 테이블 (2 rows)

키-값 저장소.

---

## 3. Configuration Data

### 3.1 `config.yaml` (~9.4KB, 351 lines)

| Section | Key Data | Argus 활용 |
|---------|----------|------------|
| `model` | default, provider, base_url, context_length | 모델 설정 변경 추적 |
| `agent` | max_turns(60), timeout(1800), reasoning_effort | 에이전트 동작 설정 모니터링 |
| `terminal` | backend(local/docker/ssh/modal), timeout(180) | 터미널 백엔드 상태 |
| `compression` | enabled, threshold(0.6), target(0.2), summary_model | 컨텍스트 압축 통계 |
| `memory` | memory_enabled, provider, char limits | 메모리 설정 상태 |
| `display` | skin, personality, tool_progress | UI 설정 |
| `logging` | level(INFO), max_size(5MB), backups(3) | 로깅 설정 |
| `checkpoints` | enabled, max_snapshots(50) | 체크포인트 사용량 |
| `security` | redact_secrets, tirith | 보안 설정 상태 |

### 3.2 `auth.json` (~10.5KB, 135 lines)

| Data | 설명 | Argus 활용 |
|------|------|------------|
| OAuth tokens | OpenAI Codex (refresh_token, last refresh) | 토큰 만료 모니터링 |
| Credential pool | 6 프로바이더, 키당 상태/에러/요청수 | 키 상태 대시보드 ★ |
| Per-credential tracking | id, label, auth_type, priority, status, error, request_count, base_url | 크레덴셜 수명 관리 |

**현재 크레덴셜 풀**:
- zai: 3 keys (1 exhausted/401)
- anthropic: 1 (claude_code OAuth, exhausted)
- openai-codex: 1 (OAuth)
- minimax: 1, kimi-coding: 1

---

## 4. Gateway Operational Data

### 4.1 `gateway_state.json`

```json
{
  "pid": 56833,
  "kind": "hermes-gateway",
  "gateway_state": "running",
  "platforms": { "<platform>": { "state": "...", "error_code": "...", "error_message": "..." } }
}
```

**Argus 활용**: 게이트웨이 생존 모니터링, 플랫폼별 연결 상태.

### 4.2 `gateway.pid`

PID, kind, argv, start_time. 프로세스 헬스체크.

### 4.3 `channel_directory.json`

15개 플랫폼 타입 목록, 활성 채널 정보 (id, name, type, thread_id).

### 4.4 `pairing/` 디렉토리

| File | 설명 |
|------|------|
| `telegram-approved.json` | 승인된 사용자 목록 (이름, ID, 타임스탬프) |
| `telegram-pending.json` | 대기 중 승인 요청 |
| `_rate_limits.json` | 텔레그램 레이트 리밋 타임스탬프 |

---

## 5. Log Files

### 5.1 `logs/agent.log` (~180KB, 1,957 lines)

- 에이전트 생명주기: 인바운드 메시지, 응답 시간, API 호출
- 프로바이더 감지, 메모리 서버 관리, 크레덴셜 로테이션
- 모델 스위치 이벤트, 압축 모델 선택

### 5.2 `logs/errors.log` (~227KB, 2,853 lines)

- WARNING/ERROR 레벨 + 전체 스택트레이스
- 반복 패턴: 텔레그램 연결 실패(DNS/timeout), API 429 레이트 리밋, memory-decay 서버 시작 실패

### 5.3 `logs/gateway.log` (~2.4MB, 19,513 lines)

- `inbound message: platform=telegram user=Roach chat=... msg='...'`
- `response ready: platform=telegram chat=... time=1112.9s api_calls=32 response=6001 chars`
- 플랫폼 연결/해제, 크론 틱커 시작/중지
- 텍스트 배치 플러싱 이벤트

**Argus 활용**: 로그 파싱으로 구조화된 이벤트 추출 → 응답 시간 모니터링, 에러율 추이, 플랫폼별 처리량.

---

## 6. Cron Jobs

### 6.1 `cron/jobs.json` (현재 없음)

```json
{
  "id": "hex12",
  "name": "...",
  "prompt": "...",
  "skills": [],
  "model": null,
  "provider": null,
  "schedule": { "kind": "once|interval|cron", "..." },
  "repeat": { "times": N, "completed": N },
  "enabled": true,
  "state": "scheduled|completed|paused",
  "created_at": "ISO",
  "next_run_at": "ISO",
  "last_run_at": null,
  "last_status": null,
  "last_error": null,
  "last_delivery_error": null,
  "deliver": "origin|local|telegram|..."
}
```

### 6.2 `cron/output/`

실행 결과: `{job_id}/{timestamp}.md`

### 6.3 `cron/.tick.lock`

동시 틱 방지용 파일 락.

---

## 7. Webhook Subscriptions

### 7.1 `webhook_subscriptions.json` (현재 없음)

```json
{
  "<name>": {
    "description": "...",
    "events": [],
    "secret": "...",
    "prompt": "...",
    "skills": [],
    "deliver": "log|telegram|...",
    "deliver_extra": { "chat_id": "..." },
    "created_at": "ISO"
  }
}
```

---

## 8. Session Files (Legacy + JSONL)

**경로**: `~/.hermes/sessions/` (~50MB, 208 files)

| File Type | Format | 설명 |
|-----------|--------|------|
| `session_*.json` | JSON | 레거시 세션 덤프 (~170개) |
| `*_timestamp_hex.jsonl` | JSONL | 최신 세션 트랜스크립트 (4개 활성) |
| `request_dump_*.json` | JSON | API 요청/응답 풀 덤프 (~37개) |
| `sessions.json` | JSON | 게이트웨이 세션 레지스트리 |

**JSONL 라인 포맷**: `{role, content, timestamp, tool_calls, reasoning, ...}`

---

## 9. Other Data Sources

| Source | Path | Size | Argus 활용 |
|--------|------|------|------------|
| Checkpoints | `~/.hermes/checkpoints/` | 160MB, 36 bare git repos | 체크포인트 사용량, 롤백 이력 |
| Skills | `~/.hermes/skills/` | 29 categories | 설치된 스킬 현황 |
| Skill Snapshot | `.skills_prompt_snapshot.json` | 48KB | 스킬 로드 시간/크기 |
| Models Cache | `models_dev_cache.json` | 1.7MB | 모델 메타데이터 캐시 |
| Persona | `notes/user-persona.md` | 3.7KB | 사용자 페르소나 설정 |
| SOUL.md | `SOUL.md` | 537B | 에이전트 정체성 |
| CLI History | `.hermes_history` | 31KB, 858 lines | 명령어 사용 패턴 |
| Interrupt Debug | `interrupt_debug.log` | 1.2KB | 인터럽트 이벤트 |
| Skins | `skins/` | 3 YAML files | 테마 설정 |
| Pastes | `pastes/` | 5 files | 붙여넣기 콘텐츠 |
| Update Check | `.update_check` | - | 버전 체크 상태 |
| Legacy Memory | `memories/MEMORY.md` | 3.4KB | 레거시 자유형 메모리 |
| Legacy User | `memories/USER.md` | 1KB | 레거시 사용자 프로필 |

---

## 10. CLI Analytics Output (`hermes insights`)

`hermes insights --days 7` 기준 실제 데이터:

```
Sessions:   71 sessions, 2964 messages, 1632 tool calls, 163 user messages
Tokens:     51.8M input, 515K output, 40.7M cache read (~93.3M total)
Top Models: gpt-5.4 (13 sessions, 59M tokens), glm-5-turbo (35, 26M), MiniMax (13, 8M)
Top Tools:  read_file 31.8%, terminal 30.7%, search_files 17.8%, patch 5.8%, write_file 4.7%
Peak Hours: 1PM, 12AM | 8-day active streak
Records:    Longest session 6h 3m, Most messages 427, Most tokens 29.6M
```

---

## 11. Profile System — Multi-Instance Monitoring

Hermes의 Profile은 **완전히 독립된 HERMES_HOME 디렉토리**다.
각 Profile마다 위 1~10장의 모든 데이터를 독립적으로 가진다.

### 11.1 Profile 디렉토리 구조

```
~/.hermes/                          ← default profile
├── config.yaml, .env, state.db, ...
│
└── profiles/
    ├── backend/                    ← named profile "backend"
    │   ├── config.yaml             ← 독립 설정 (다른 모델/프로바이더 가능)
    │   ├── .env                    ← 독립 API 키
    │   ├── SOUL.md                 ← 독립 에이전트 정체성
    │   ├── state.db                ← 독립 세션/메시지 DB
    │   ├── memory-decay/
    │   │   └── memories.db         ← 독립 메모리 DB
    │   ├── gateway_state.json      ← 독립 게이트웨이 상태
    │   ├── gateway.pid             ← 독립 게이트웨이 프로세스
    │   ├── auth.json               ← 독립 크레덴셜 풀
    │   ├── sessions/               ← 독립 세션 파일
    │   ├── logs/                   ← 독립 로그
    │   ├── skills/                 ← 독립 스킬 (기본 번들 + 커스텀)
    │   ├── memories/               ← 독립 레거시 메모리
    │   ├── cron/                   ← 독립 크론 잡
    │   ├── checkpoints/            ← 독집 체크포인트
    │   └── home/                   ← 독립 서브프로세스 HOME (git, ssh 등)
    │
    ├── frontend/                   ← named profile "frontend"
    │   └── (동일 구조)
    │
    └── dba/                        ← named profile "dba"
        └── (동일 구조)
```

### 11.2 Profile 메타데이터 (CLI에서 읽을 수 있는 정보)

```python
@dataclass
class ProfileInfo:
    name: str               # "default", "backend", "frontend", ...
    path: Path              # 실제 디렉토리 경로
    is_default: bool        # default = ~/.hermes 자체
    gateway_running: bool   # 게이트웨이 프로세스 생존 여부
    model: Optional[str]    # 설정된 모델명
    provider: Optional[str] # 설정된 프로바이더
    has_env: bool           # .env 파일 존재 여부
    skill_count: int        # 설치된 스킬 수
    alias_path: Optional[Path]  # ~/.local/bin/ 래퍼 스크립트 경로
```

### 11.3 Profile 감지 방법

```python
# 1. Default profile은 항상 존재
default_home = Path.home() / ".hermes"

# 2. Named profiles는 profiles/ 디렉토리 하위
profiles_root = default_home / "profiles"
for entry in profiles_root.iterdir():
    if entry.is_dir() and re.match(r"^[a-z0-9][a-z0-9_-]{0,63}$", entry.name):
        # 유효한 profile
```

### 11.4 Profile별 Argus 추적 데이터 (독립 데이터 소스)

| Profile당 추적 항목 | Source File | 비고 |
|---------------------|-------------|------|
| 세션/메시지/토큰 | `<profile>/state.db` | Profile별 독립 DB |
| 메모리 상태 | `<profile>/memory-decay/memories.db` | Profile별 독립 |
| 게이트웨이 상태 | `<profile>/gateway_state.json` | Profile별 프로세스 |
| 게이트웨이 PID | `<profile>/gateway.pid` | 생존 체크 |
| 크레덴셜 풀 | `<profile>/auth.json` | Profile별 API 키 |
| 설정 | `<profile>/config.yaml` | Profile별 모델/설정 |
| 에이전트 정체성 | `<profile>/SOUL.md` | Profile별 페르소나 |
| 로그 | `<profile>/logs/agent.log`, `errors.log`, `gateway.log` | Profile별 |
| 크론 잡 | `<profile>/cron/jobs.json` | Profile별 |
| 체크포인트 | `<profile>/checkpoints/` | Profile별 |

### 11.5 크로스-Profile 분석 가능 데이터

Profile이 독립적이지만, Argus가 전체를 수집하면 다음 크로스 분석이 가능하다:

| Analysis | Description |
|----------|-------------|
| **Total Cost** | 모든 Profile의 `estimated_cost_usd` 합산 |
| **Model Distribution** | Profile별 모델 사용 현황 비교 |
| **Credential Health** | 모든 Profile의 API 키 상태 종합 |
| **Gateway Uptime** | Profile별 게이트웨이 가동시간 비교 |
| **Token Usage by Profile** | Profile별 토큰 소비량 랭킹 |
| **Tool Usage Patterns** | Profile별 툴 사용 패턴 차이 (예: backend는 terminal 많이, frontend는 browser 많이) |
| **Session Activity Heatmap** | Profile별 활동 시간대 비교 |
| **Memory Footprint** | Profile별 메모리 수/용량 비교 |
| **Error Rate** | Profile별 에러 발생률 비교 |
| **Active Sessions** | Profile별 현재 활성 세션 수 |

### 11.6 Argus 데이터 수집 설계 제안

```
argus.db (Argus 자체 SQLite)
├── profiles                    ← Profile 레지스트리
│   ├── id, name, path, is_default
│   ├── model, provider
│   ├── gateway_running, gateway_pid
│   ├── has_env, skill_count
│   ├── discovered_at, last_scanned_at
│   └── config_hash             ← config.yaml 변경 감지용
│
├── profile_snapshots           ← 주기적 스캔 결과
│   ├── profile_id FK
│   ├── scanned_at
│   ├── state_db_size           ← state.db 파일 크기
│   ├── session_count           ← sessions 테이블 row 수
│   ├── message_count           ← messages 테이블 row 수
│   ├── total_input_tokens
│   ├── total_output_tokens
│   ├── total_estimated_cost
│   ├── gateway_running
│   └── active_session_count
│
├── sessions (aggregated)       ← 모든 Profile의 세션 통합
│   ├── profile_id FK           ← 어느 Profile의 세션인지
│   ├── session_id              ← 원본 state.db의 id
│   ├── source, user_id, model
│   ├── started_at, ended_at, duration
│   ├── message_count, tool_call_count
│   ├── input_tokens, output_tokens, cache_read/write, reasoning
│   ├── estimated_cost_usd, actual_cost_usd
│   └── title
│
├── tool_usage (aggregated)     ← 모든 Profile의 툴 사용 통합
│   ├── profile_id FK
│   ├── session_id FK
│   ├── tool_name
│   ├── called_at
│   └── success (content 기반 추론)
│
├── credential_status           ← 모든 Profile의 크레덴셜 상태
│   ├── profile_id FK
│   ├── provider
│   ├── key_id, label
│   ├── status, error_code
│   ├── request_count
│   └── scanned_at
│
└── gateway_events              ← 게이트웨이 이벤트 (로그 파싱)
    ├── profile_id FK
    ├── event_type (inbound, response, error, lifecycle)
    ├── platform, user_id
    ├── timestamp
    ├── duration_ms (response 시간)
    └── detail (JSON)
```

### 11.7 주의사항

1. **동시 읽기**: `state.db`는 WAL 모드이므로 Argus가 읽는 동안 Hermes가 쓰기 가능. 하지만 `auth.json`은 파일 기반 락(`auth.lock`)이 있음
2. **Default 특수 처리**: Default profile은 `~/.hermes` 자체이며, named profiles는 `~/.hermes/profiles/<name>/`에 있음
3. **Profile 동적 생성/삭제**: `hermes profile create/delete`로 언제든 추가/제거 가능 → Argus는 주기적 스캔으로 감지해야 함
4. **active_profile 파일**: `~/.hermes/active_profile`에 현재 sticky default가 기록됨
5. **Docker 환경**: `HERMES_HOME`이 `~/.hermes`가 아닐 수 있음 → `_get_default_hermes_home()`으로 루트 결정

---

## 12. Argus 추적 가능 데이터 요약 (우선순위별)

### P0 — 핵심 (반드시 수집)

| Category | Data | Source | Granularity |
|----------|------|--------|-------------|
| **Session Lifecycle** | 생성/종료 시간, 지속시간, 종료 사유 | `state.db.sessions` | Per session |
| **Token Usage** | input/output/cache_read/cache_write/reasoning | `state.db.sessions` | Per session + cumulative |
| **Cost** | estimated_cost_usd, actual_cost_usd, provider | `state.db.sessions` | Per session + cumulative |
| **Tool Usage** | 툴 이름, 호출 횟수, 파라미터, 결과 | `state.db.messages.tool_calls` + `tool_name` | Per message |
| **Model Usage** | 모델명, 프로바이더, 토큰 분포 | `state.db.sessions.model` | Per session |
| **Message Flow** | role, timestamp, content, token_count | `state.db.messages` | Per message |

### P1 — 중요 (초기 릴리즈 포함)

| Category | Data | Source | Granularity |
|----------|------|--------|-------------|
| **Gateway Health** | 프로세스 PID, 상태, 가동시간 | `gateway_state.json` | Continuous |
| **Platform Status** | 플랫폼별 연결 상태, 에러 | `gateway_state.json.platforms` | Continuous |
| **Credential Pool** | 키별 상태, 요청수, 에러, 만료 | `auth.json` | Per credential |
| **Memory Health** | 메모리 수, 활성화 빈도, 디케이 상태 | `memory-decay/memories.db` | Per tick |
| **Error Tracking** | 에러 유형, 빈도, 스택트레이스 | `logs/errors.log` | Per event |
| **Response Time** | API 응답 시간, 툴 실행 시간 | `logs/gateway.log` | Per request |

### P2 — 유용 (후속 릴리즈)

| Category | Data | Source | Granularity |
|----------|------|--------|-------------|
| **Cron Jobs** | 스케줄, 실행 이력, 상태, 에러 | `cron/jobs.json` + output | Per job |
| **Webhooks** | 구독 현황, 실행 이력 | `webhook_subscriptions.json` | Per subscription |
| **Session Content** | 전체 대화 내용 검색 | `messages_fts` | Full text |
| **Compression Events** | 압축 트리거, 요약 모델, 비율 | `agent.log` | Per event |
| **Checkpoint Usage** | 생성/롤백 이력, 디스크 사용량 | `checkpoints/` | Per event |
| **Skill Inventory** | 설치된 스킬, 카테고리, 크기 | `skills/` + snapshot | Static + changes |

### P3 — 보너스

| Category | Data | Source |
|----------|------|--------|
| CLI Command History | 사용된 명령어 패턴 | `.hermes_history` |
| Rate Limit Events | 플랫폼별 레이트 리밋 | `pairing/_rate_limits.json` |
| Interrupt Events | 사용자 인터럽트 이력 | `interrupt_debug.log` |
| Model Config Changes | 모델 설정 변경 이력 | `sessions.model_config` diff |
| User Persona Changes | 페르소나 설정 변화 | `notes/user-persona.md` diff |
