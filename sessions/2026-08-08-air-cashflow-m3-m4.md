# [세션] 2026-08-08 에어 캐시플로 M3(감정가·매각)·M4(비용) 완료

- 날짜: 2026-08-08
- 프로젝트/주제: Air Cashflow(Project Air 2) 1단계 Python 엔진, M3·M4 골든마스터 전 셀 통과

## 결정 사항
- `dispositions(inp, scn, months, fv, lease, maint)` 시그니처 확정: Other 렌트에 Maintenance_Other 잔여넷 선택 행이 가산되고, 매각 총계(371행대)에 정비 Remaining PV 럼프가 가산되어 M2 산출물 의존.
- Servicer 3%·Disposition fee 1.5%·Tax는 Cashflow 시트 수식이라 M5 소유로 이동 (ARCHITECTURE §12.4).
- Other_Costs Admin 5·8행과 LOC 12~23행(+Details 29·32·33행)은 Debt Schedule 83·85행(Pre-Sweep 이자, Cashflow 의존)이 필요해 M6로 xfail 이월.
- 참고 표(Other_Costs_Details 3~26·69~89행, LOC_Fee)는 엔진이 아니라 테스트가 골든 원시 셀 산술로 재현.

## 진행 상황
- 완료: M3 future_value.py + disposition.py (커밋 41437e7), M4 costs.py (커밋 826b0bf). 전체 테스트 20 passed, 2 xfailed(M2 Top up, M4 LOC). ARCHITECTURE.md §11·§12 갱신.
- 남은 작업: M5 월CF(Aircraft Valuation + Cashflow Estimation_Monthly 수입~PV 구간) → M6 워터폴(Reserve Top up + LOC 계열 확정) → M7 IRR/집계.

## 다음 세션에서 필요한 맥락
- 위치: `C:\Users\USER\dev\air-cashflow`. 읽는 순서: CLAUDE.md → docs/ARCHITECTURE.md §9~§12 → tests/test_m3_disposition.py(어댑터 패턴).
- 분석 방법: golden_formulas/<시트>.csv를 행별 대표 수식으로 스캔하고, 행별 원시/수식 경계는 rowext류 스크래치 스크립트로 확인. 원본 엑셀은 절대 열지 않는다.
- 이 모델의 단골 함정:
  1) 수식 미확장 구역: 블록마다 우측 끝 열이 다르고 그 밖은 원시 0 또는 빈 셀 (합계 행이 잡아준다).
  2) 한 칸 밀림 직접 참조: FVM 전월 값 참조 (Other S3/S4, PF adjusted 등).
  3) 시트 행 순서는 Assumption(MSN 정렬)이 아니라 리스 시트 순서. 위치 기반 로직(Weight 밀림 등)은 `inp.fvm_upsie.index`를 쓴다.
  4) 행 단위 원본 편집: 38031 원시 리스만기, 32736 F=EOMONTH(E,60) 등.
- 테스트 실행: `$env:PYTHONIOENCODING="utf-8"; python -m pytest tests -q`
