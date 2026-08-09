# [세션] 2026-08-09 air-cashflow 4단계 웹 대시보드 설계 + 백엔드 완성

**프로젝트/주제**: air-cashflow 4단계 FastAPI 로컬 웹 대시보드. 설계 확정 + M10-0~M10-3
(백엔드 전부) 구현·검증·커밋. 다음은 M10-4 프런트.

## 보스가 확정한 결정

- **배포 형태: 로컬 전용** (127.0.0.1:8791 고정, 인증 없음, `--host` 인자 없음).
  LAN 개방은 만들지 않고 조건만 문서에 예고 (ARCHITECTURE_WEB.md §14.1).
- **결과 저장: 파일 경로만 DB 에.** `run` 행에 status·out_dir·error 만.
  결과 표는 `out/runs/<id>/*.csv`. `run_table` 안 만든다 → **스키마 변경 0
  (schema_version 1 유지)**.
- **범위: 조회 + 실행 + 비교만.** 편집 UI 없음 (5단계로). 표 편집·run 라벨·임포트
  트리거·실행 취소·진행률 퍼센트 전부 5단계.
- 포트 8791 (아호비 8765·8768 과 안 겹치게).
- 진행은 M 단계별로 승인받으며 하나씩 (M10-0 → M10-1 → M10-2 → M10-3).
- 의존성 함정 교훈을 PLAYBOOK 4-9 에 처방 2줄로 승격.

## 진행 상황

| M | 내용 | 커밋 |
|---|---|---|
| 설계 | `docs/ARCHITECTURE_WEB.md` (아키텍트 에이전트 1회) | 4add339 |
| M10-0 | 계약 실작성: `api/deps.py`·`schemas.py`·`server.py`·`meta_api.py` + 라우터 3스텁 + `db/runs.py` 스텁, `docs/GUARDRAILS.md` 채움, `engine/run.py` 에 `inp=` 2줄 | 4add339 |
| M10-1 | `db/runs.py`(웹 층 SQL 전부) + `api/jobs.py`(워커 1개 잡 러너·수명주기·기동 청소) | ca1f2e3 |
| M10-2 | `api/datasets_api.py` (B1~B6 조회) | f8c5d9c |
| M10-3 | `api/runs_api.py`(C1~C4) + `api/results_api.py`(D1~D4·E1~E2 diff) | 769a109 |

**남은 것**: M10-4 프런트(`web/static/*` 9파일 + `tools/webctl.py` + `tools/jscheck.py`,
950~1,250줄) → M10-5 테스트 4파일(W1~W14) → M10-6 문서·인수인계.

**기존 회귀 무해**: `pytest tests -q` base 69 통과 + 6 skip, 84초. 4단계가 손댄 기존
파일은 `engine/run.py` 2줄과 `requirements.txt` 뿐이다.

## 큰 성과: 인수 조건이 이미 성립한다

M10-1 시점에 확인됐다. 잡 러너가 낸 CSV 9개가
(1) 같은 입력으로 `run(scn, inp=inputs_from_db(...))` 직접 호출한 산출물과,
(2) 골든 경로 `run()`(inp 없음) 산출물과 **둘 다 파일 바이트 동치**. 차이 0.
`골든 → DB → 웹 → CSV` 가 `골든 → CSV` 와 같음이 파일 수준에서 증명됐다.
M10-5 의 W5 는 이 대조를 시나리오 바꿔 가며 테스트로 고정하는 일이다.

## 이번 세션에서 잡은 함정 (부류급)

### PLAYBOOK 에 승격한 3건
- **4-9 에 처방 2줄 추가**: 의존성 차집합 목록을 눈으로 훑고 "전이겠지"로 넘기면
  안 된다. 상위 패키지가 **무조건 임포트하는데 메타데이터에 안 적어 둔** 패키지가
  실재한다. 판정은 `importlib.metadata.requires()` 전이 폐쇄(extras 제외) 대조 +
  `sys.meta_path` 차단기로 실증.
- **5-24 신규**: "차이가 나야 한다" 검증은 그 입력이 실제로 차이를 내는지 데이터로
  먼저 확인한다. 음성 대조군을 양성으로 고르면 옳은 구현이 버그로 보인다.
- **5-25 신규**: 검사기는 "위반 0건"과 "대상을 0개 봤다"를 구별해 보고해야 한다.
  프레임워크 내부 구조를 순회하는 검사기는 판이 바뀌면 조용히 항상 통과한다.

### 이 프로젝트 문서(§19)에 남긴 것
- **python-multipart·sniffio 는 필수다.** 설계는 "JSON 바디만 쓰니 필요 없다"고
  단정했지만 틀렸다. starlette 1.3.1 `formparsers.py:18` 과 anyio 4.13.0
  `_core/_eventloop.py:21` 이 `import fastapi` 경로에서 무조건 임포트하는데 둘 다
  상위 메타데이터에 선언이 없어 pip 이 안 끌어온다. 깨끗한 환경에서 ImportError.
- **널 효과 변형 함정**: 설계 스모크와 GUARDRAILS 가 "다른 시나리오로 돌리면
  diff>0" 예시로 `v4_maturity16` 을 골랐는데 그건 스펙이 스스로
  `expect_downstream_change=False` 라고 선언한 변형이다(base 플랜에서 전 기체가
  만기보다 먼저 매각). 구현이 옳아도 diff 0. `v1_gecas_s1` 로 교체하고, 검사는
  이름 대신 그 필드로 변형을 뽑아 쓰게 했다. 널 효과 2개는 `v4_maturity16`·`v8_handling_fee`.
- **`app.routes` 로 라우트 순서를 검사하면 안 된다.** fastapi 0.139 의 목록에는
  경로를 노출하지 않는 `_IncludedRouter` 가 들어가서 검사기가 라우트를 하나도 못 보고
  초록을 냈다. 소스 AST 로 `@router.get("...")` 등장 순서를 본다.
- **base 결과 9벌의 NaN 은 0개**다. NaN 처리는 합성 데이터로만 검증 가능.
- `api/` 에 `golden` 부분문자열 가드를 걸면 계약 필드 `verified_by_golden` 을 위반으로
  잡는다. `GOLDEN_DIR`·`golden_dir`·`core.inputs` 를 본다.
- `StaticFiles` 는 디렉터리가 없으면 기동을 죽인다 (`check_dir=False` 필요).
- **`with sqlite3.connect(...) as c` 는 연결을 닫지 않는다** (트랜잭션 컨텍스트다).
  윈도우에서 DB 파일 삭제가 WinError 32 로 실패한다. `api/deps.py:open_conn()` 을 쓴다.
- `open_conn` 은 DB 파일이 없으면 연결을 **거부**한다. `sqlite3.connect` 가 없는 파일을
  만들어 버려서, 그냥 열면 웹이 빈 DB 를 조용히 생성하고 사용자는 임포트를 잊는다.
- 기동 청소를 `create_app()` 안에서 부르면 **임포트만 해도 실제 DB 를 고친다**. lifespan 으로.
- `/api/meta` 의 `counts` 는 **톰스톤 제외** = 기본 목록 길이와 같다 (계약으로 명시).
- dataset 톰스톤은 목록에서만 감추고 경로로 직접 물으면 돌려준다. scenario 톰스톤은 409
  (파생값이 본문이라 톰스톤이면 본문이 없다).

## 다음 세션에서 필요한 맥락

**읽을 것**: `docs/ARCHITECTURE_WEB.md` §18(요약 카드) → §19(구현 기록 M10-0~M10-3) →
§9(UI 설계) → `docs/GUARDRAILS.md`. 그 4개면 M10-4 착수 가능.

**M10-4 범위** (설계 §9·§16):
- `web/static/`: `index.html`, `app.css`, `core.js`, `api.js`, `chart.js`,
  `view_run.js`, `view_result.js`, `view_diff.js`, `boot.js` (로드 순서가 계약,
  즉시실행은 `boot.js` 에만)
- `tools/webctl.py` (기동 + `--gc`), `tools/jscheck.py` (top-level 전역 중복 선언 검사)
- 의존성 0, 외부 CDN 0, 차트는 자체 SVG. `?v=__STAMP__` 를 서버가 치환(수동 갱신 폐지)
- 완료 판정: jscheck 0건, `GET /` 200 + 치환 + no-store, 외부 URL 0건,
  script 로드 순서 일치, 탭 3개 DOM 스모크, **짤림 감사 0건(1280x720)**,
  webctl 이 DB 없을 때 exit 2

**실행 방법**:
```powershell
$env:PYTHONIOENCODING="utf-8"
cd C:\Users\USER\dev\air-cashflow
python tools\dbctl.py init
python tools\dbctl.py import --golden golden --name base
python -m pytest tests -q                  # base 69 통과, 84초
python tools\run_variants.py               # 변형 16종 회귀
```
`data/aircf.db` 는 git 미추적이고 이 작업 트리에는 **없다** (검증은 전부 샌드박스로 했다).

**검사 스크립트 3개**: `work/webcheck/m10_0_check.py`, `m10_1_check.py`, `m10_2_check.py`,
`m10_3_check.py` (git 미추적). M10-5 가 `tests/test_web_*.py` 로 이식할 원본이다.
격리는 `AIRCF_DB` + `AIRCF_OUT_ROOT` 환경변수로 한다.

**주의사항**:
- 테스트에 `tmp_path`/`tmp_path_factory` 금지 (이 PC 에서 PermissionError).
  `tests/db_util.py:tmpdb_dir` 또는 `tempfile.mkdtemp()`.
- 엑셀 생성(변형 골든)과 pytest 를 **동시에 돌리지 않는다** (PLAYBOOK 6-21).
- `api/deps.py`·`schemas.py`·`server.py` 는 아키텍트 소유. 구현 세션이 상수·헬퍼를
  추가하지 않는다 (필요하면 문서를 먼저 고친다).
- `db/conn.py`·`reader.py`·`writer.py`·`importer.py`·`verify.py` 와 엔진 계산 모듈 11개는
  **수정 금지**.
- 층 가드 G1~G6 (api 에 SQL·sqlite3·골든 경로 0건, engine 임포트는 허용 3모듈)을
  깨지 않는다.
- 실측: run 1회 3.6초. `v1_gecas_s1` vs base 는 5,726셀 차이.
- 파일 수정은 Edit/Write 만 (PowerShell 치환 금지, 한글 UTF-8 깨진다).
