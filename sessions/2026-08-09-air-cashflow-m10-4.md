# 2026-08-09 air-cashflow 4단계 완결 (M10-4 프런트 · db_conn 수정 · M10-5 테스트 · M10-6 문서)

## 어디까지 했나

**4단계 전부 완료 (M10-0 ~ M10-6). 다음은 5단계(편집 UI), 미착수.**
인수인계는 `docs/HANDOFF_WEB.md` 하나로 충분하다.

커밋 4개 (푸시 안 함, 로컬):
- `51263e8` feat(web): M10-4 프런트 9파일 + webctl/jscheck + 연결 주입 방식 교정
- `2a0944f` fix(web): 폴링 경합 수정 + run 소요 시간 정정
- `f5bb13e` test(web): M10-5 W1~W14, 인수 조건 W-ACC 파일 바이트 동치 통과
- `6e9ec74` docs(web): M10-6 완료 기록 + HANDOFF_WEB.md + README

작업 트리 깨끗함. 미커밋 WIP 없음.

## 만든 것

- `web/static/` 9파일: index.html, app.css, core.js, api.js, chart.js, view_run.js, view_result.js, view_diff.js, boot.js
- `tools/webctl.py` (기동 · `--gc [--yes] [--verify]`), `tools/jscheck.py` (전역 중복 선언 + node --check)
- 판정 (a)~(g) 전부 통과. 자기 검증도 통과: base/base 두 번 돌려 9표 전부 `diff_cells 0`,
  `v1_gecas_s1` 은 5,726셀 (§19.13-7 실측 재현).

## 다시 켜고 이어갈 때

```powershell
$env:PYTHONIOENCODING="utf-8"
cd C:\Users\USER\dev\air-cashflow
python tools\webctl.py            # http://127.0.0.1:8791
```

DB(`data\aircf.db`)와 결과(`out\runs\1~4`)는 그대로 있다 (git 미추적).
dataset `base` 1개, scenario 16개, run 4개(#1·#2·#4 base/base, #3 v1_gecas_s1) 전부 `ok`.

**주의**: 서버를 강제 종료했으므로 다음 기동의 청소가 `running` 잔해를 `failed` 로
확정할 수 있다. 설계대로다 (§6.4).

## 이 세션에서 확정된 사실 (전부 docs 에 기록됨)

1. **`Depends(deps.db_conn)` 크로스 스레드 결함** (WEB §19.14-2, 아키텍트가 수정 완료).
   fastapi 는 동기 의존성의 본체와 정리를 anyio 워커 풀에 각각 넘긴다.
   실측: 동시 16클라 x 25요청에서 **400건 중 227건 HTTP 500**, ProgrammingError 916건.
   순차 폴링에서는 250건 중 2건이라 "간헐적 네트워크 오류"로 오인한다.
   **`TestClient` 로는 재현되지 않는다 (200/200 통과).** 실서버에 동시 요청을 넣어야 한다.
   수정: `db_conn()` 삭제, 라우트 본문이 `with deps.open_conn() as conn:` 으로 연다.
   재현기: `work\repro_db_conn_thread.py` (서버를 띄운 상태로 돌린다). 수정 후 400/400 성공.

2. **`run()` 소요 시간은 고정값이 아니다** (WEB §19.14-1). 같은 기계·같은 조합이
   한 창(50분)에서는 40~75초, 그 뒤에는 4.0초다. `pytest tests -q` 도 그 창에서
   856초(문서 84초의 10.2배)였다. **원인 미확정.**
   교훈: 값을 4번 재도 **같은 창 안에서 재면 표본은 1개다.** 나도 그래서 한 번 틀리게
   단정했다가 되돌렸다.
   **M10-5 는 "몇 초 이내"를 통과 조건에 넣으면 안 된다** (느린 창에서 무작위로 빨간다).

3. 짤림 감사: **말줄임(ellipsis)은 `title` 이 없으면 짤림으로 센다** (GUARDRAILS §5.5 에 추가).

4. 이 환경에서 **브라우저 도구의 마우스 클릭 주입이 페이지에 닿지 않는다**
   (`elementFromPoint` 는 맞는 요소를 주고 `hasFocus()` 는 true 인데 캡처 리스너 수신 0건).
   UI 결함으로 오해하기 쉽다. 왕복 검증은 `dispatchEvent(new MouseEvent('click',...))` 로 했다.
   스크린샷도 "Browser pane is not displayed" 로 실패한다.

## M10-5·M10-6 에서 더 확정된 것

5. **시간 예산 미결은 "테스트가 시간을 판정하지 않는다"로 정리했다** (§19.16-1).
   `pytest tests -q` = **129 통과 + 7 skip, 94~97초** (base 69 유지, 웹 60 추가).
   build 는 기본 2회, `AIRCF_WEB_FULL=1` 이면 3회(골든 경로 대조).
   "이미 실행 중이면 409"는 시간에 기대지 않고 **엔진을 `threading.Event` 로 막아** 판정한다.
6. **텍스트·패턴 가드는 대상 언어의 문법을 알아야 한다** (세 번 밟았다):
   `golden` 부분문자열 → `verified_by_golden` 을 잡음 / "외부 URL 0건" → SVG 네임스페이스
   URI 를 잡음 / "top-level 실행문 0" → `async function` 을 잡음.
   처방은 매번 같다: 예외 목록을 붙이지 말고 검사를 문법 수준으로 올리거나 표현을 없앤다.
7. **`shutil.rmtree(ignore_errors=True)` 가 청소 실패를 삼킨다.** 테스트가 만든 임시 폴더가
   실행마다 %TEMP% 에 쌓였다. `tests/web_util.py:rmtree_hard()` 로 재시도 + 경고.

## 다음 세션 첫 할 일

5단계(편집 UI)다. `docs/HANDOFF_WEB.md` §6 을 먼저 읽어라. 오더가 오면 착수 전에
`GUARDRAILS.md` §5(오더 유형별 체크리스트)와 §7(이 앱에 없는 것)을 대조하고,
계약·스키마 변경이 필요하면 아키텍트 선행이다.
