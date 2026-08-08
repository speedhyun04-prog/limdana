# [세션] 2026-08-09 air-cashflow: MV/BV 스트레스 구현 + 입력 DB 단계 완결

**프로젝트/주제**: `C:\Users\USER\dev\air-cashflow` — 마지막 엔진 갭(MV/BV) 구현으로 변형 골든 16종 전부 통과, 이어서 3단계(입력 DB + 임포터) M9-0~M9-6 완결.

## 결정 사항

- **v9 MV/BV 스트레스: 지금 구현한다** (보스 결정). 문서는 "사실상 새 기능"이라 적어 뒀지만 실측하니 소비처가 2시트 6블록뿐이라 규모가 작았다.
- **DB 설계 3건 (전부 권장안 채택)**
  - 13-1 원본이 수식으로 계산하는 입력 94칸: `editable=0` **배지만** 붙이고 편집은 허용.
  - 13-2 무검증 스위치 조합: `verified_by_golden` 배지로 표시.
  - 13-3 UI 전 편집 수단: `dbctl set` 최소 CLI, 스칼라 한정 + `frozen=0` 인 fork 에만 쓰기.
- **구현 방식**: 멀티에이전트 파이프라인 없이 본 세션에서 순차, 단계마다 커밋. (아키텍트 서브에이전트는 DB 설계 문서 작성에만 1회 사용)
- **requirements.txt 버전 고정**: `pandas==3.0.3` / `numpy==2.4.6`. dtype 이 갈리면 왕복 비트 동치가 깨진다.

## 진행 상황

### 완료: MV/BV 스트레스 (커밋 f304658)
- `engine/mvbv.py` 신설(DATEDIF 연령 + 기종×연령 HLOOKUP 팩터), `inputs.py` 원시 입력 로딩, `disposition.py` 매각가 6블록 × 팩터 + 가드 제거, MVBV_Test 테스트 어댑터.
- 실측으로 확정한 사실: 팩터 소비처는 Disposition_GECAS(S1·S2·S4) / Disposition_Other(S1·S2·S3) 뿐 — **GECAS S3 와 Other S4 에는 안 붙는다**. 생산시점은 Assumption `manufactured` 와 **다른 별도 원시 입력**(20대 전부 불일치). 팩터표 144칸 중 13칸은 수식 유래. `I20` 한 칸만 `>=`.
- 결과: **변형 16종 전부 통과, 엔진 갭 0건.** base 44 → 45 (어댑터 1건 추가).

### 완료: 입력 DB 3단계 (커밋 100f181 → 65e7702)
- 설계 `docs/ARCHITECTURE_DB.md`(아키텍트 서브에이전트 작성 + 내가 결함 보완), 인수인계 `docs/HANDOFF_DB.md`, CLI `tools/dbctl.py`.
- 계약: DB 는 골든의 대체물이 아니라 **`Inputs` 의 저장소**. 스키마는 `Inputs` dataclass 의 투영이고 `db/` 는 셀 좌표를 모른다. 임포터 = `load_inputs()` + `write_inputs()`. 엔진 변경은 `build(..., inp=)` 4줄뿐.
- 인수 조건 **충족**: `build(golden)` 과 `build(DB)` 가 **오차 허용 0 의 비트 동치**. 시나리오 16벌 전부(`AIRCF_DB_FULL=1`, 18 통과 165초).
- 규모: 23테이블, base dataset 22,161행, 쓰기 0.52초. 테스트 base 69 통과(엔진 45 + DB 24, 약 95초), 변형 세션은 DB 24개 스킵.

### 남은 것
- 4단계 FastAPI 웹 대시보드. `run` 테이블과 `api/` 자리는 비워 뒀다.
- 표 전체 편집 UI(지금은 스칼라만), `data/` frozen 분기(%LOCALAPPDATA%).
- 워크북 전체의 배열 수식 개수 미확인(E239 1건만 확인).
- 보스 판단 대기: `%TEMP%\pytest-of-USER` 쓰기 권한 없음(시스템 폴더라 손대지 않았다).

## 부류급 교훈 → PLAYBOOK 3건 추가

- **2-11** 트랜잭션 정리는 커밋 실패 경로까지 덮는다. COMMIT 도 실패한다(지연 외래키). try 밖에 두면 트랜잭션이 열린 채 남아 이후 쓰기 전부가 연쇄로 죽는다.
- **5-22** 왕복·동치 검증에서 집합 비교는 순서 결함을 못 잡는다. 순서가 결과를 바꾸는 데이터(부동소수 합)면 순서를 저장하고 대조에 포함하며, 입력 동치만이 아니라 **결과 동치**를 인수 조건에 넣는다.
- **5-23** "사실상 새 기능"은 인상이지 규모가 아니다. 착수 전 소비처 전수 스캔 + 사전 검증 스크립트로 규모를 실측한다.

## 다음 세션에서 필요한 맥락

읽는 순서: `CLAUDE.md` → `docs/HANDOFF_DB.md` → `docs/ARCHITECTURE_DB.md` §14 요약 카드 → §15 구현 기록.

```powershell
$env:PYTHONIOENCODING="utf-8"
cd C:\Users\USER\dev\air-cashflow
python -m pytest tests -q                  # base 69 통과, 약 95초
python tools\run_variants.py               # 변형 16종 전 회귀
python tools\dbctl.py init                 # 스키마 + 시나리오 16개 시드
python tools\dbctl.py import --golden golden --name base
python tools\dbctl.py verify --dataset base --golden golden
```

주의:
- pytest `tmp_path`/`tmp_path_factory` 를 쓰지 마라(권한 없음). `tests/db_util.py:tmpdb_dir` 사용.
- 엑셀 변형 생성과 pytest 동시 실행 금지(서로 CPU 를 뺏어 몇 배 느려진다).
- `Inputs` 에 필드를 추가하면 `db/tables.py:FIELD_TABLE` 과 문서 §5.2 를 같이 늘린다(`test_field_coverage` 가 잡는다).
- pandas/numpy 올리기 전에 변형 16종 회귀 선행.
