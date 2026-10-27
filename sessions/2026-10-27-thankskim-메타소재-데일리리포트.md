# [세션] 2026-10-27 땡스킴 메타 소재 성과 리포트 · 보스 지시 1~7 · 데일리 메타 리포트(제미나이)

- **날짜**: 2026-10-27
- **프로젝트/주제**: 땡스킴 대시보드(`C:\Users\USER\dev\thankskim`, 운영 https://dash.thankskimadmin.com) 메타 화면 확장과 AI 데일리 리포트
- **운영 커밋**: 1720ec8(서버 반영) · 저장소 HEAD 7232323(문서만)

## 결정 사항
- 사이드바 "메타"는 그룹: **메타 광고**(캠페인·광고세트·소재 표) · **소재 성과 리포트**(목표 칩·KPI 4·썸네일 표·미리보기 모달).
- 1차안 샘플 공구 데이터는 운영에서 삭제(톰스톤, 백업 `/opt/thankskim/backup/thankskim_20261027_051947.db`).
- 브랜드 화면 "옵션별 판매 · 브랜드 전체" 카드, 메타 그룹 "인스턴스별 성과 비교" 표 삭제.
- 인스타 메뉴는 계정 2개가 바로 화면(드롭다운 없음), 문구 "대표님 오피셜".
- 데일리 메타 리포트: 보스 프롬프트 원문 그대로, **노션 저장 안 함 → 대시보드 안 날짜 열람**.
- AI 는 **제미나이(땡스킴 키, AI Studio 무료 등급) 플래시 모델로 유지**(보스 결정, 다시 묻지 않기). 결제를 켜면 `gemini.model` 만 프로로.

## 진행 상황
완료(전부 운영 배포·검증):
- 소재 성과 리포트 + 미리보기 모달, 소재 이미지 서버 캐시(`data/creatives/`, `/api/meta/creative/{id}/image`).
- 메타 수집기: 소재 목록 limit 25 + 실패 시 최소 필드 폴백(운영 Graph 500 수정), 일별 분해 표 `meta_breakdown_daily`(90일 백필, 실패 격리).
- 소재 리포트 기간 선택(날짜바 → `?since&until` 합산).
- 데일리 메타 리포트: `server/dailyreport.py`(입력 트리·프롬프트·제미나이/Claude 호출·후처리·저장·CLI), `server/report_api.py`, 화면 `dailymeta`(날짜 열람 바·①②③ 탭·마크다운 렌더러), 타이머 `thankskim-report.timer` 09:00 KST.
- 10/26 첫 실생성 성공(Gemini 3.8 Flash, 234초, 경고 0, 화면 렌더링 확인). 10/28 09:01 부터 자동.
- 매 커밋 독립 리뷰(sonnet 에이전트) + pre-commit 브라우저 스모크(17항목) 통과.

남은 것(보스가 다음 세션에서 "추가 수정사항" 줄 예정):
- 카페24 매출 정의 결정(이전부터 대기, deploy/README D절 1-2).
- 대표님 인스타 계정(ceo) 토큰 — 비즈니스 인증 대기.
- 제미나이 키가 채팅에 노출됨 → 필요 시 AI Studio 에서 키 제한·재발급(보스 판단).
- 10/28 09:00 첫 자동 생성 결과 확인(`journalctl -u thankskim-report`).

## 다음 세션에서 필요한 맥락
- 단일 출처: `docs/ARCHITECTURE.md`(§2 모듈, §4 API, §5 표), 변경 이력 `docs/CHANGELOG.md`, 남은 일 `deploy/README.md` D절.
- 검증: `python -I tools/check_js.py`, `python tools/test_report.py`, `python tools/test_collect.py`, `python -I tools/smoke.py`(PYTHONUTF8=1). 커밋 훅이 `node tools/smoke_ui.js` 를 돌린다(`git config core.hooksPath tools/hooks` 설치됨).
- 배포: 작업 폴더가 아니라 `git archive HEAD` 사본을 tar → scp → `deploy/deploy.sh`(같은 저장소에 다른 세션이 있으면 WIP 가 섞인다). 판정은 `/api/health` 의 commit.
- 서버 수동 실행은 `cd /opt/thankskim/app && sudo -u thankskim -H env TK_DATA_DIR=/opt/thankskim/data PYTHONUTF8=1 .venv/bin/python -m server.dailyreport --date YYYY-MM-DD`(또는 `server.collect --source meta --force`). ssh 는 Bash 도구로.
- 비밀: AI 키는 서버·로컬 `secrets/credentials.json` 의 `gemini` 블록에만(값을 요약·로그에 적지 않음). 서버 파일은 0600, 표준입력으로 병합.
- 화면 규칙: UI 신규·룩 변경은 목업 승인 뒤, 정적 파일 수정 시 `index.html` 의 `?v=` 올리기(현재 20261027i).
- 로컬 화면 검증은 scratchpad 의 격리 데이터 + `C:\.claude\launch.json` 에 임시 `thankskim-local`(포트 8782) 등록 후 끝나면 삭제.

## 플레이북 반영(app-factory/PLAYBOOK.md)
- 신규 **2-15**: 무거운 확장 필드 목록 호출은 작은 페이지 + 최소 필드 폴백, 부가 수집은 본 수집과 격리.
- 신규 **2-16**: 재생성 실패가 마지막 정상 결과를 덮어쓰지 않게(시도 상태와 결과물 분리).
- 보강 **4-12**: 재발 — 배포물은 `git archive HEAD` 사본에서, 남의 WIP 가 섞인 공유 파일은 내 몫만 스테이징.
- 보강 **6-5**: `python -I` 는 PYTHONIOENCODING 을 무시 → `sys.stdout.reconfigure(encoding="utf-8")`.
