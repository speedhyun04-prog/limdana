# [세션] 2026-08-02 NotebookLM MCP 구글 로그인 완료

- **날짜**: 2026-08-02
- **프로젝트/주제**: notebooklm-mcp 구글 로그인 해결 (setup_auth 실패 원인 규명)

## 결정 사항
- setup_auth / re_auth는 앞으로 **재실행 금지**. 실행하면 로그인 쿠키(크롬 프로필)를 먼저 지우고 시작하는데, 성공 판정이 옛 도메인 기준이라 항상 10분 타임아웃으로 실패한다. 악순환.
- 재로그인이 필요하면 수동 절차를 쓴다 (아래 "다음 세션 맥락" 참조).

## 진행 상황
완료:
- setup_auth 3회 실패의 원인 규명: 구글이 NotebookLM을 notebook.google.com("Gemini Notebook")으로 리다이렉트하는데, notebooklm-mcp 2.0.0은 `notebooklm.google.com` URL 도달 여부로만 성공을 판정해서 로그인해도 감지 못 함. npm 최신도 아직 2.0.0(미수정).
- 시스템 크롬을 MCP 전용 프로필(`%LOCALAPPDATA%\notebooklm-mcp\Data\chrome_profile`)로 띄워 보스가 직접 로그인 → patchright 검증 스크립트로 로그인 확인(제목 "Gemini Notebook") + `browser_state\state.json` 생성 → get_health authenticated=true 확인.
- 진행 중 cleanup_data(preserve_library=true)로 구버전 잔재 포함 40MB 정리했음.
- 메모리 저장: `~/.claude/projects/C--/memory/notebooklm-mcp-auth.md`

남은 작업:
- "Claude Memory" 노트북이 MCP 라이브러리에 미등록(total_notebooks=0). 보스가 NotebookLM에서 공유 링크를 주면 add_notebook으로 등록.
- 도메인 개편 여파로 ask_question / add_source 등 다른 도구가 깨질 수 있음. 첫 질의/저장 때 확인 필요 (URL 판정 하드코딩이 곳곳에 있음).

## 다음 세션에서 필요한 맥락
- 인증 실체는 크롬 프로필 쿠키. state.json은 24시간 지나면 get_health가 false로 보여도 실사용은 영향 없음 (런타임은 쿠키 유효성 검사만 함).
- 수동 재로그인 절차: 시스템 크롬을 `--user-data-dir="%LOCALAPPDATA%\notebooklm-mcp\Data\chrome_profile"`로 실행해 구글 로그인 → 창 닫기 → patchright로 notebooklm.google.com 접속해 `context.storageState()`를 `%LOCALAPPDATA%\notebooklm-mcp\Data\browser_state\state.json`에 저장.
- patchright는 npx 캐시 `%LOCALAPPDATA%\npm-cache\_npx\0d29dd9f4e472da9\node_modules\patchright\index.mjs`에서 절대경로 import로 사용 (스크립트 위치 기준 해석이라 cwd만 바꿔선 안 됨).
- MCP 설정: `.claude.json`의 notebooklm 서버 = `npx -y notebooklm-mcp@latest` (stdio).
