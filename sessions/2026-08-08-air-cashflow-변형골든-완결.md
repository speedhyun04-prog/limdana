# [세션] 2026-08-08 air-cashflow 변형 골든 단계 완결 (엔진 갭 4건 해소)

**프로젝트/주제**: air-cashflow (EY "Project Air 2" 엑셀 현금흐름 모델의 Python 엔진).
2단계 = 변형 골든 검증. 시작 시점에 남아 있던 G3 갭 + 스펙 정정에서 시작해 단계를 끝냈다.

## 결정 사항 (보스 확정)

1. **G3 는 임계값 가드로 풀지 않는다** (세션 시작 시 보스 지시). 지켰고, 진짜 원인은 다른 데 있었다.
2. **v6·v13 폐기 → v15·v16 으로 대체** (선택지 (b)). 단독 `Cayman C10` 변형은 base 가
   GECAS 플랜(D90=1)이라 채택 경로를 못 건드려 하류 전파 가드에 걸린다. `C4=2` 와 조합하면
   Other 축이 실제 채택 경로가 되고 v7(C4=2+C10=3)과 형태가 일관된다.
3. **v8_handling_fee 는 `expect_downstream_change=False`** (선택지 (a)). 원본 전체에서
   `Cayman C21` 을 읽는 셀이 CF Monthly D165·D184 **2개뿐**이라 핸들링피는 잎사귀다.
   조합으로 하류에 밀어 넣을 방법이 없음을 수식 전수 스캔으로 증명했다.
4. **v8 엔진 갭은 이 단계에서 고친다** (ARCHITECTURE §16.13 보스 결정 C 해소).
5. **MV/BV 스트레스(v9)는 이월**. 사실상 새 기능이라 판단.

## 진행 상황

### 완료

- **G3 수정** (커밋 `627de80`). CF Monthly 241행 2셀. 원인은 부동소수 노이즈가 아니라
  **엑셀 연산 의미 재현 오류 2건**이었다.
  - 엑셀 `^` 는 정수 지수에서 제곱-곱(square-and-multiply). `1.02**3`=1.0612080000000002(파이썬)
    vs `1.02^3`=1.061208(엑셀). 감정사 8시트 명목표 800셀 대조: 제곱-곱 800/800, `**` 240/800.
  - 엑셀 `a+b+c`·`SUM` 은 순차 누적인데 `DataFrame.sum(axis=0)` 은 8개 초과에서 numpy 쌍합.
  - `engine/core/excelmath.py` 신설(`xlpow`, `xlsum_rows`), future_value·aircraft_valuation 이 사용.
  - 역추적 경로: CF 221 ← Debt 32/34 ← Debt 28/29/31 ← CF 11 ← Aircraft Valuation 88행
    ← Disposition ← FV Monthly(Engine) ← IBA_Engine 43행 = 실질원시 × (1+인플)^(연-2015).
- **v7·v2·v3 스펙/어댑터 정정** (커밋 `caf7303`). v7 의 Disposition_Other 40셀은 엔진 갭이
  아니라 어댑터가 원본 `='Key Assumption'!$D$126` 자리에 상수 1.0 을 넣고 있던 것.
  v2 RRR_cost·v3 Lease_Payment_GECAS 는 "집어온 시나리오 블록 값이 base 블록과 전 셀 동일"해서
  안 바뀌는 것이 정상(각 2799셀·4451셀 실측).
- **v15·v16 생성·통과** (커밋 `bb25f35`). 바뀔 시트를 **생성 전에 base 골든만으로 예측**했고
  실측과 일치. v16 은 LTV #DIV/0! 7셀이 v7 과 같은 사유로 발생해 등재.
- **P2 5종 생성** (커밋 `4d30bd7`). v10·v11·v12 통과.
- **v8 핸들링피 갭 수정** (커밋 `3da4f0c`). 핵심은 D165·D184 가 표시값이 아니라
  **E1·E2 수익증권 XIRR 의 최초 현금흐름**이라는 것. `-e1`·`-e2` 쓰던 자리를 전부
  `D166`(=D163+D165)·`D185`(=D181+D184)로 교체(Monthly·Quarterly XIRR 8개 + CoC 4개 +
  Yearly D59·D60). `Scenario.handling_fee_included` 를 이제 엔진이 실제로 읽는다.
  마지막 1건은 또 어댑터였다(Cayman I9 를 `-D95` 아닌 `e1` 로 나눔).
- **README 로드맵 정정** (커밋 `08631f1`). "2단계" 번호 충돌(README=DB vs §16=변형 골든) 명시.
- **PLAYBOOK 3항목 추가**: 5-20(기준 데이터 한 벌이면 우연히 맞는 하드코딩이 통과),
  5-21(1 ulp 를 부동소수로 덮지 마라 = 연산 의미 차이 신호), 6-21(오피스 COM 과 테스트 동시 실행 금지).

### 최종 상태

**변형 16종 중 15종 통과** (base 44 · v0_null 48 · 나머지 13종 각 47).
남은 1종 `v9_mvbv_stress` 는 `disposition.py:318` 이 `NotImplementedError` 로 의도적으로 막아 둔
미구현 기능이다(MVBV_Test 팩터가 Disposition 시트 21,520셀에 곱셈 인자로 들어간다).

## 다음 세션에서 필요한 맥락

- **먼저 읽을 것**: `docs/HANDOFF_M8.md` (이것만 읽고 이어받게 정리돼 있다).
  상세 실측은 `docs/ARCHITECTURE.md` §16.14 (특히 -9 G3, -10 스펙 정정, -11 v15/v16, -13 P2·v8).
- **회귀**: `python tools\run_variants.py` — **base 44 통과가 깨지면 즉시 되돌린다.**
  단일 변형은 `$env:AIRCF_VARIANT="<이름>"; python -m pytest tests -q` (끝나면 환경변수 제거).
- **주의 1**: 변형 생성(엑셀 COM)과 pytest 를 **동시에 돌리지 마라**. 실측으로
  pytest 25초→453초, 생성 160초→778초가 됐다. 완료 판정이 CPU 유휴 기반이라 오판 위험도 있다.
- **주의 2**: 갭이 보이면 **엔진부터 의심하지 말고 검증 어댑터를 먼저 대조**한다.
  이번 세션에서 세 번 다 어댑터였다(PLAYBOOK 5-20).
- **주의 3**: 변형 스펙 변경 시 `spec_hash` 는 inputs+derived+scenario 만 포함한다.
  `expect_changed_sheets`·`known_error_cells` 를 고쳐도 골든 재생성은 필요 없다.
- **남은 일**: (a) v9 MV/BV 스트레스 구현 여부 결정(규모 보고 후), (b) 다음 큰 덩어리는
  DB(기체/계약/감정가/트랜치/시나리오) + 임포터 → 새 구조라 app-factory PLAYBOOK 선행 + 아키텍트.
