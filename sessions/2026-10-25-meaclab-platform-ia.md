# [세션] 2026-10-25 meaclab 플랫폼 IA 계약 + 1단계 구현

- **날짜**: 2026-10-25
- **프로젝트/주제**: meaclab 사내 플랫폼 IA 구조화 → 6포지션 5회차 검토 → 아키텍처 계약 → 1단계 구현(미배포) → 애플 디자인 시스템 목업은 새 세션으로 핸드오프

## 결정 사항
- 보스 목표 확정(D-37): **그룹웨어·사내 메신저·이카운트(ERP)를 한 플랫폼(meaclab)에 구축**. "한곳에서 보기"가 아니라 대체. 7단계(그룹웨어 흡수)·7b(ERP 쓰기 이전)는 확정 로드맵, 착수 조건(게이트·서류 5종)은 유지.
- D-25~D-36 CEO 에이전트 제안 답 그대로 확정: 메일 흡수 제외 / 쓰기 주체 하나(운영=폼 DB, 회계=이카운트, 시트=투영) / 스테이징 월 7달러 / 시트 소유권 회사 계정 + Apps Script 트리거 / 권한 원천=권한지기 대장(인사 확정·자동 초안) / 7b 별도 심의·그 전 이카운트 키 읽기 전용만 / 묻지 않고 진행 상한 월 10만·건당 3만 / 권한 사고 책임(대장 일치=인사, 불일치=시스템) / 부담당 1명 / 수수료표·프로모션 선입력 마케팅·검증 재무 / 지표 명칭 "채널 기여이익" / 퇴사 비활성화=최종 근무일 종료.
- Q1(주 3일 가용)·Q2(반발 시 게이트 정지)는 미답 → 가정으로 표기.
- 거부권 6(CEO 대장·퇴사 비활성·로그 불변 / CFO 읽기 전용 키 / COO 부담당 전 폼 잠금 금지 / 사용자 스톱워치 10건 / 인사 로그 목적 제한 규정화 / CMO 요율 이력 없는 기여이익 공개 금지).
- 5회차에서 되돌린 과도 반영: 권한 키 7종 → 2단계 4키(fin.cost·price·fee·ar), 시뮬레이터 제외, 자가 기록은 1회만·게이트는 로그 기반, 확정률은 관찰 지표.
- 세션 한도 50% 초과 시 묻지 않고 새 세션(fresh)으로 이어가라는 상시 지시(메모리 session-limit-handoff).

## 진행 상황
- 완료: 계약 `docs/ARCHITECTURE_PLATFORM_IA.md`, 검토 보고 `docs/IA_PLATFORM_REVIEW_2026-10-25.md` + `docs/ia_review/`(5개), `DECISIONS.md` 5차 D-25~D-37, `PROGRAM_PLAN.md` §6 포인터 (커밋 d282594).
- 완료: 1단계 구현 /dev 파이프라인(에이전트 18개) → 커밋 **55bad2c**, CI 46 스모크·통합 73검사 통과. 모듈 7: registry(매니페스트 10개, /api/modules) · hub-home(/app/home, 메뉴바 보기>홈 1줄) · session-unify(새 단일 세션 쿠키 `aho_s`, aho_sm 병행) · search-api(/api/search menu·person·recent, 초성) · my-targets(/app/my, /api/my/targets, 번들 mtime 캐시) · ops-floor(deploy/backup, /api/health, 계정 deactivate/reactivate, docs/OPS_STAGE1.md) · activity-metrics(기본 꺼짐). 설계 `docs/ARCHITECTURE_STAGE1.md`.
- **서버 배포 보류(보스)**. 보스가 크롬 로컬에서 허브 1차 골격 확인 → "애플 홈페이지처럼, 목업 먼저, 모든 페이지에 적용 가능한 디자인 시스템으로" 지시.
- 미리보기 서버: `C:\.claude\launch.json` `stage1-preview`(uvicorn 8799, app-dir ahovye) 패널에 떠 있음. 운영 8765 는 그대로(새 코드 반영하려면 Web Guard 태스크 재기동 필요).
- 핸드오프: 새 세션 `apple-ds-mock`(local_d39e1cc5-3008-494c-8048-5b0fe27ae42e)이 애플 KR 실측 → `docs/DESIGN_SYSTEM_APPLE.md` + `web/static/ds/{ds.css,ds.js}` + `docs/mock/APPLE_DS_MOCK.html` + 패널 시연. 승인 뒤 hub/my 실제 교체는 그 다음.

## 다음 세션에서 필요한 맥락
- 단일 출처: `dev\ahovye\docs\ARCHITECTURE_PLATFORM_IA.md`(계약) → `ARCHITECTURE_STAGE1.md`(1단계 구현) → `OPS_STAGE1.md`(사람 할 일·런북).
- 검증은 `python tools/ci_check.py`(스모크는 pytest 수집 불가: 모듈 최상위 sys.exit). 통합 스모크 `tests/test_stage1_integration_smoke.py`.
- 배포 전 확인: 서버에 sales-map 번들(`data/sales-map/web/{index.html,data/sales.js,data/targets.js}`) 없으면 `/api/health` 503 → deploy.sh curl 실패. 배포 명령 `ssh meaclab 'bash /opt/meaclab/app/deploy/deploy.sh'`(보스 승인 뒤).
- 사람 몫: `data/modules_external.json`(시트·하이웍스·이카운트 URL, example 복사), `acl.json users[id].reps`(내 달성률 매핑), `settings.json activity_log.enabled`(규정 명문화 전 끄기), 부담당 후보 2명(1월), 팀장 회의 담당 지정(결재선 조사·수수료 입력·검증), 전사 공지 문구, 개인정보 처리방침 초안(2단계 전).
- 검토 운영 패턴(재사용): 포지션별 sonnet 에이전트 6개 병렬 × 5회차(구조 → 순서 → 산출물 → 교차 검증 → 최종·거부권·과도 반영 되돌리기), 회차마다 v2.x 파일 갱신해 읽게 하고 세션 모델이 종합.
- 비용: 이 세션 컨텍스트 47%에서 핸드오프. 5시간 한도 63%.

## 플레이북 반영
- PLAYBOOK §5 검증·작업 프로세스에 추가: 다회차 설계 검토의 마지막 회차에 "지금까지 반영된 것 중 과도한 것" 질문을 반드시 넣는다(회차마다 조건을 더하기만 하면 범위가 부푼다 — 이번에 권한 키 7종·측정 3축이 그렇게 잡혔다).
