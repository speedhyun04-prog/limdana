# [세션] 2026-07-18 pxpipe-proxy 설치·롤백과 사기 DM 대응

- **날짜**: 2026-07-18
- **프로젝트/주제**: 인스타그램 DM에서 받은 `npx pxpipe-proxy@latest` 실행 요청 처리, 전체 롤백, 재발 방지 규칙

## 결정 사항
- pxpipe-proxy(Claude Code 토큰 절감 프록시, 포트 47821)는 사기성 인스타그램 DM 출처로 판단되어 완전 제거하기로 결정.
- 앞으로 SNS DM 등 외부 출처 명령어(npx, 설치 스크립트, 프록시 설정)는 실행 전에 답변 맨 앞에 🚨 강조 경고를 하고, 출처 확인 전에는 기본 실행 보류. (영구 기억 scam-dm-command-warning.md에 저장됨)

## 진행 상황
완료:
- pxpipe-proxy 실행 → ANTHROPIC_BASE_URL env 설정 → SessionStart 자동 기동 훅(start_pxpipe.ps1) 추가까지 갔다가, 보스 요청으로 전부 롤백.
- 제거 내역: settings.json env 블록·훅 항목, start_pxpipe.ps1, ~/.pxpipe 데이터 폴더, node 프로세스(포트 47821), npx 캐시(_npx) 잔여물.
- 포트 47821 닫힘 확인, settings.json JSON 유효성 확인 (기존 로그 훅·ecc 플러그인 설정 보존).
- 피해 평가: 프록시 적용 세션을 연 적이 없어 API 트래픽·토큰이 프록시를 지나가지 않음. 유출 가능성 사실상 없음.

남은 작업:
- 보스가 DM 계정 신고·차단. DM 내 다른 링크 클릭/입력 여부는 미확인 (했다면 추가 점검 필요).

## 다음 세션에서 필요한 맥락
- 영구 기억: `~/.claude/projects/C--/memory/scam-dm-command-warning.md` (외부 출처 명령 강력 경고 규칙).
- settings.json은 원상태 기준: SessionStart 로그 훅 1개, PostToolUse/SessionEnd 로그 훅, ecc 플러그인 항목 존재.
- 이 세션 중 보스가 CLAUDE.md에 NotebookLM 기억 라우팅 섹션을 추가했고, ecc 플러그인을 비활성화(false)로 바꿈.
- 잔여물 재확인 명령: `netstat -ano | findstr 47821` (LISTENING 없어야 정상).
