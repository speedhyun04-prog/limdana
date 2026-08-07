# [세션] 2026-08-08 에어 캐시플로 M2 정비수입 완료

- 날짜: 2026-08-08
- 프로젝트/주제: Project Air 2 엑셀 자동화 1단계 (dev\air-cashflow), M2 정비수입 엔진

## 결정 사항
- Reserve Top up(Summary ⓖ175·ⓗ176행)은 전월 Cashflow-Debt 산출물이 필요해 M6에서 재현 (test_m2에 strict xfail로 명시).
- 새 시나리오 스위치 4종을 Scenario에 추가: Key D111(GECAS 정비수입 선택=4), D118(GECAS 정비지출=4), D104(Remaining Net=4), Cayman C10(Other Remaining 게이트=1). conftest 사전검사에도 등록.
- MaintResult 계약 확정: MaintPlan(gecas/other 시트별) + summary/expenses(D90 선택) + rls(환입 = IF(ⓐ+ⓒ>=0,-ⓒ,ⓐ), M5가 한 달 시프트로 소비) + MaintReserve(ⓐⓑⓒⓓⓕ).
- 원본 특이점은 위치가 아니라 MSN 상수로 재현 (시트마다 행 순서가 달라 위치 기반은 틀림).

## 진행 상황
- 완료: M0~M2. M2 골든마스터 전 셀 통과 (테스트 10 passed + 1 xfailed), 커밋 "feat: M2 정비수입 엔진..." (7파일 +645줄).
  - engine/maintenance.py: 수입·지출 4시나리오 블록, Net, Remaining Net·PV·PV Table, Reserve ⓐ~ⓕ
  - core 확장: exceldates.yearfrac_actact(basis 1), inputs 로더에 정비 Check 원시표(maint_rev_raw/maint_exp_raw)·Floor 표(Key B280:G310)·24개월 가중치(F248:AC248)·시드(Summary I169=6M) 추가
- 남은 작업: M3 감정가·매각 → M4 비용 → M5 월CF → M6 워터폴(Reserve Top up 포함) → M7 IRR/집계.

## 다음 세션에서 필요한 맥락
- 읽기 순서: dev\air-cashflow\CLAUDE.md → docs\ARCHITECTURE.md §9·§10 (M1·M2 발견사항) → tests\test_m2_maintenance.py (어댑터 패턴).
- M2에서 발견한 원본 특이점 (§10에 전부 기록): MSN 38031 행만 GECAS 하부 표 E열이 EOMONTH 없이 원시 리스만기(2020-03-24, GE 매각일과 같은 달이라 분기 갈림), GECAS Remaining S3 표는 데이터 셀이 빈 표(합계 0), Other 지출 헤더 GU열부터 +1개월 시프트 오기(결과 무해), Summary 월축은 GT(2032-12)까지 198열, 엑셀 VLOOKUP 빈 셀=0 재현.
- 시트 하부 블록 분석 요령: golden_formulas\<시트>.csv를 E열(grep '^E[0-9]+,') 기준으로 스캔하면 블록 구조가 빨리 나온다.
- M3 대조 시트: Future Value_Monthly/Yearly, Disposition_GECAS/Other/Summary, (Mx_Upsie). Future Value 보간은 월축이 더 넓다(2015-12~2041, ARCHITECTURE §4.1).
- 실행: $env:PYTHONIOENCODING="utf-8"; python -m pytest tests -q. 원본 엑셀은 절대 열지 않는다.

## 플레이북 점검
- 부류급 후보 "위치 인덱스 대신 도메인 키(MSN)로 매핑"은 이 프로젝트 문서(ARCHITECTURE §10-5a)에 기록하는 것으로 충분하다고 판단, PLAYBOOK 추가 없음.
