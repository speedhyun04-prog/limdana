# [세션] 2026-08-08 Air Cashflow M5 월별 CF 완료

- **날짜**: 2026-08-08
- **프로젝트/주제**: Project Air 2 엑셀 모델 자동화 1단계, M5 월별 현금흐름 엔진 (dev\air-cashflow)

## 결정 사항
- Tax(Cashflow 17행)는 단순 시트 내 수식이 아니라 **TAX Study 시트 전체 의존**으로 확인. TAX Study는 Tax analysis(Debt 이자 기체 배분)와 Other Costs 15행(LOC)에 의존하므로 Tax·Admin(25행)·LOC(26행)과 파생 행(18·19·21·22·29·30·32), Aircraft Valuation 205행 Check는 전부 **M6로 이월** (test_m5에 strict xfail 명시).
- monthly_cashflow 계약: M6가 tax/admin/loc Series(최종 CF 행 값, 음수·게이트 적용 완료)를 공급하면 FCFF~PV of FCFF를 확정. None이면 해당 필드 None (0 대입 금지).
- M2 문서의 "Rls는 한 달 앞 열 참조" 표현은 부정확했음. 실제는 **동월 참조**. ARCHITECTURE §10-8 정정, maintenance.py 주석 수정.

## 진행 상황
- 완료: engine/aircraft_valuation.py(AV 시트 46,763셀 전 셀 통과), engine/cashflow.py(CF 수입~PV 구간 약 2,600셀 통과), tests/test_m5_cashflow.py, ARCHITECTURE §13 신설. **커밋 ea97954**. 전체 테스트 22 passed, 4 xfailed(전부 의도된 이월).
- 남은 작업: M6 워터폴(Debt Schedule + Reserve Top up ⓔⓖⓗ + Other_Costs LOC/Admin + TAX Study/Tax analysis + CF 17~61행) → M7 IRR/집계.

## 다음 세션에서 필요한 맥락
- 읽기 순서: `dev\air-cashflow\CLAUDE.md` → `docs\ARCHITECTURE.md` §13(M5)·§12(M4)·§10(M2) → `tests\test_m5_cashflow.py`(어댑터 패턴).
- **M6 착수 전 의존 그래프 확인 필수**: LOC은 향후 10개월 이자 창(Debt 83·85행의 F..O SUM)을 미리 보고, Cash Sweep은 FCFF에 의존. 순환처럼 보이지만 전월 참조(Reserve ⓔ = Cashflow E30 - Debt E139 전월분)로 시간축이 끊겨 있을 가능성이 큼. `golden_formulas\Debt_Schedule.csv`를 행별 스캔(스크래치 scan_m5.py 방식)으로 먼저 구조 파악.
- 발견된 원본 퀴크: AV PV 블록(232~251행)은 CF 20행 할인계수를 **2개월 뒤 열**에서 참조, CF 27행 AOG는 F열부터 AOG!25행의 **3개월 앞 열** 참조(E열만 동월), AV C252 총합은 F252(행합계 열)까지 포함해 실총합의 2배.
- 시트 하부 분석은 원본 엑셀 열지 말고 `golden_formulas\<시트>.csv`를 E열 기준 스캔. 검증은 `$env:PYTHONIOENCODING="utf-8"` 후 `python -m pytest tests -q`.

## 플레이북 점검
- 부류급 교훈 없음. 열 밀림·시트 간 의존 함정은 이 프로젝트 전용 사실이라 ARCHITECTURE §13에 기록 (PLAYBOOK 추가 없음).
