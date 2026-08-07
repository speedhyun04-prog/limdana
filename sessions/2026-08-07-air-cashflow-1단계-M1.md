# [세션] 2026-08-07 에어 캐시플로 1단계 착수 (M0+M1 완료)

- 날짜: 2026-08-07
- 프로젝트/주제: Air Cashflow 자동화 1단계 (Python 엔진 + 골든마스터 검증), dev\air-cashflow

## 결정 사항
- 앱 공장 절차로 착수: 템플릿 이식(웹 없음이라 web\ 제외), git init, 아키텍트 설계 선행
- 골든마스터가 유일한 정답: golden\ CSV(원본 저장값)가 입력 소스이자 정답지. 원본 30MB 엑셀은 tools\ 덤프 스크립트만 연다
- 허용 오차: |engine-golden| <= max(1e-6, 1e-8x|golden|), IRR 계열만 절대 1e-5, 날짜는 완전 일치. 테스트별 완화 금지
- 원본은 58시트가 맞다 (HANDOFF의 59는 오기)
- "수식 없는 셀 = 입력" 원칙: 원본 제작자의 수동 덮어쓰기(다운타임 0 등)는 로더가 원시 입력으로 읽는다

## 진행 상황 (완료)
- 골든마스터 추출: golden\ 58시트 973,891셀 + 수식 텍스트 golden_formulas\ 809,920개
- 아키텍트 설계: docs\ARCHITECTURE.md (M0 공유기반 + M1리스료~M7지표, 파일 소유권 표, 부분 대조 전략)
- M0 구현: xlio(golden 파서)·timeline·exceldates(EOMONTH/EDATE/DATEDIF/YEARFRAC 30/360)·types(Scenario)·inputs 로더, tests의 golden_util·conftest(스위치 전제 검사)
- M1 리스료 구현: Average_LR(4사 평균 체인) + GECAS/Other 시나리오 4블록 + 선택(D97/D126) + Summary(D90) 재현
- 골든 테스트 4건 전 셀 통과 (시트 4개, 약 4만 셀)
- 커밋: 92cbb3a(템플릿) → f507e00(골든) → fd75b91(설계) → 3df1cb1(M0+M1)

## 남은 작업
- M2 정비수입 → M3 감정가·매각 → M4 비용 → M5 월별 CF → M6 부채 워터폴 → M7 IRR·집계·run
- 앵커 3종(IRR 10.77%, PV FCFF $1,027M, LTV 67.8%)은 M7의 test_run_smoke에서

## 다음 세션에서 필요한 맥락
- 읽기 순서: dev\air-cashflow\CLAUDE.md → docs\ARCHITECTURE.md §9(M1 발견사항) → tests\test_m1_lease.py(어댑터 패턴)
- 테스트: `$env:PYTHONIOENCODING="utf-8"; python -m pytest tests -q`
- 수식 분석은 golden_formulas\<시트>.csv (reference\dumps는 상위 90행 샘플이라 하부 블록에 못 쓴다)
- 주의 1: 모듈 구현 전 해당 시트의 raw-num 셀 스캔(수동 덮어쓰기 찾기). M1에서 GECAS 베이스에 40개 있었다
- 주의 2: 시나리오 스위치는 Cayman 노란 셀 외에 Key Assumption 행 87~ "Scenario Selection" 구역에도 있다 (D90/D97/D126 확정, M2는 정비 Factor를 이 구역에서 먼저 찾을 것)
- 주의 3: 월축 중복(GECAS HG·HH 같은 달), Other 27행은 상수 참고선

## 플레이북 점검
- 부류급 교훈 후보 "레거시 재현은 공식만 믿지 말고 저장값의 원시 셀(수동 덮어쓰기)을 전수 스캔"은 현재 이 프로젝트 전용 성격이 강해 PLAYBOOK 승격 보류, docs\ARCHITECTURE.md §9에 기록함. 골든마스터형 프로젝트가 또 생기면 그때 승격 검토.
