# 보류: AhoPlus 수신자 칩 + 메일 발송 (2026-07-29)

보스 지시로 기능 보류. auto-dev 파이프라인으로 구현·검증까지 끝난 상태에서
프런트 진입점(web/static/msgtag.js)만 HEAD로 되돌려 비활성화했다.

## 남아 있는 것 (ahovye 작업 트리, 무해·비활성)
- web/mail_api.py, web/mailer.py, web/recipients.py (신규, server.py에 배선됨)
- web/rooms_api.py 의 POST /api/rooms/sendmany
- web/static/chat.js 의 칩 분기 (MsgTag.hasChips 가드라 msgtag 구버전에서는 미작동)
- tests/test_mail_chips_smoke.py
- docs/ARCHITECTURE_MESSAGING_SEND.md 증보 절, docs/ARCHITECTURE.md §4.5
- mail_config.example.json (SMTP 설정 템플릿, 실설정 파일은 data/mail_config.json)

## 재개 방법
1. `msgtag.chips.js` 를 `web/static/msgtag.js` 로 복사
   (또는 `git apply msgtag.chips.patch`)
2. web/index.html 의 msgtag.js ?v= 를 새 값으로 올리기 (소모된 문자열 재사용 금지,
   보류 시점 기준 최신은 20260730c)
3. `python tests/test_mail_chips_smoke.py` 로 검증
4. SMTP 실발송 쓰려면 data/mail_config.json 작성 후 1회 실발송 확인
