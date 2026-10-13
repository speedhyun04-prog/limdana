# [세션] 2026-10-13 아호비 두뇌 CLI 로그인 만료 재발 복구

- **날짜**: 2026-10-13
- **프로젝트/주제**: 아호비(dev\ahovye) 채팅이 "클로드 CLI 로그인이 풀렸습니다(구독 인증 만료)" 안내를 띄움. 9/11 사고와 동일 원인 재발, 복구 완료.

## 결정 사항
- 로그인은 보스가 직접(자격증명 입력은 Claude 가 못 함). Claude 는 터미널 탭에 로그인 명령을 띄워 두는 것까지만.
- 정식 명령은 `claude auth login --claudeai`. `claude login` 은 2.1.288 에서 REPL 로 들어가 "login" 을 일반 프롬프트로 먹어버리므로 쓰지 않는다.
- 이 사고는 코드 변경이 아니므로 릴리스·배포 대상이 아니다(배포자에게 나갈 것 없음).

## 진행 상황
- 완료
  - 원인 확정: `~/.claude/.credentials.json` 의 OAuth expiresAt=0, `claude -p` 프로브가 terminal_reason=api_error.
  - 보스가 브라우저 승인 + 코드 붙여넣기로 로그인 완료("Login successful").
  - 검증: `claude auth status` loggedIn=true(max 구독), 자격증명 만료 시각 미래로 갱신, `claude -p` 프로브 is_error=False 로 정상 응답.
  - 서버 재기동 불필요(두뇌는 턴마다 CLI 를 새로 띄움). 로그인용 터미널 탭 정리.
  - 메모리 `ahovye-brain-cli-login-expired.md` 에 정식 명령·재발 날짜 반영.
- 남은 작업: 없음. (만료가 주기적으로 재발하므로 다음에 같은 안내가 뜨면 같은 절차.)

## 다음 세션에서 필요한 맥락
- 진단 한 줄: `claude -p "hi" --output-format json` → terminal_reason=api_error 면 인증 문제.
- 복구: 보스 터미널에서 `claude auth login --claudeai` → 브라우저 승인 → 코드 붙여넣기. 확인은 `claude auth status`.
- 영향 범위: 사원은 본인 클로드 키가 CLI 자식 프로세스에 주입(web/chat_api.py api_key 분기)되므로 영향 없음. 끊기는 건 키 없이 구독 인증을 타는 admin(보스) 뿐.
- 아호비 저장소는 0.7.9(맥 첫 배포) 이후 문서 커밋만 있고 작업 트리 변경 없음(data/accounts_audit.jsonl 은 런타임 로그).
- 플레이북 추가 없음: 프로젝트 전용 사실이라 메모리에만 기록.
