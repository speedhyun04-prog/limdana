# [세션] 2026-08-07 Air Cashflow 엑셀 분석·PPT·자동화 설계

- **날짜**: 2026-08-07
- **프로젝트/주제**: 바탕화면 Air_Cashflow Analysis_Draft_20161213_Final.xlsx (Project Air 2) 전수 분석, 설명 PPT 제작, 자동화 아키텍처 제안

## 결정 사항
- 파일 정체 확정: EY Korea TAS의 항공기 20대 리스 포트폴리오($976M) 투자 검토용 월별 현금흐름 모델. 결과값 에쿼티 IRR 10.77%(보수차감후), FCFF PV $1,027M, 초기 LTV 67.8%
- 자동화 방향: 4계층(DB → 임포터 → Python 계산엔진(pandas) → 웹 대시보드 + 엑셀 I/O), 골든마스터(원본 엑셀 저장값과 셀 단위 대조) 통과 후 전환, Python+FastAPI 스택(아호비와 동일)
- 단계별 도입: 1단계 엔진+검증(엑셀 I/O만) → 2단계 DB+감정평가 임포터 → 3단계 웹 대시보드
- 1단계는 **새 세션에서 착수**하기로 결정 (이번 세션이 엑셀 덤프로 맥락이 무거움)

## 진행 상황
- 완료:
  - 59개 시트 값·수식 전량 덤프 및 분석
  - 설명 PPT 16장 납품: 바탕화면 `Air_Cashflow_모델설명.pptx` (비전공자 각주 포함)
  - 할루시네이션 체크: 모델 내 삼중 대사(월/분기/연) 0 확인, #REF!(Cayman Analysis)·#DIV/0!(LTV) 이슈 위치 기록
  - 자동화 아키텍처 제안 및 인수인계 자료 정리
- 남은 작업: 1단계 착수 (앱 공장 절차로 아키텍트 설계 → 골든마스터 기준값 정밀 재추출 → 엔진 구현 순서: 리스료→정비→매각→비용→월별CF→워터폴→IRR/집계)

## 다음 세션에서 필요한 맥락
- 인수인계 문서: `C:\Users\USER\dev\air-cashflow\reference\HANDOFF.md` (모델 요약, 설계, 첫 작업 순서 전부 포함)
- 분석 산출물: 같은 폴더 `dumps/`(시트 덤프 58개), `dump.py`(재추출 스크립트), `make_ppt.js`
- 원본 엑셀은 재분석 금지, 덤프 재활용. 정밀 골든마스터는 dump.py의 MAX_R/MAX_C 확장해 재추출
- 장기 메모리 `air-cashflow-automation.md` 등록됨
- 환경 팁: 이 PC에서 pptx 스킬의 LibreOffice 렌더(soffice.py)와 pdftoppm이 안 됨 → PowerPoint COM(Export PNG)으로 대체. validate.py는 `PYTHONUTF8=1` 필요(한글 XML cp949 오류)

## 플레이북 점검
- 부류급 교훈 없음 (환경 팁 2건은 앱 개발 패턴이 아닌 이 PC의 문서 도구 특성이라 PLAYBOOK 제외, 위 맥락에 기록)
