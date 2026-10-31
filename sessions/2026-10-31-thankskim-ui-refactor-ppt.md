# [세션] 2026-10-31 땡스킴 메인·마진·스케줄 UI 정렬 + 리팩토링·QA + 2주 정리 PPT·플랜 엑셀

- **날짜**: 2026-10-31
- **프로젝트/주제**: 땡스킴 공동구매 대시보드(`C:\Users\USER\dev\thankskim`, 운영 https://dash.thankskimadmin.com). 원 세션 "땡스킴 추가 수정사항 리스트"가 띄운 사이드 세션. 사내 업무와 완전 별개.

## 결정 사항
- 메인(전체) 화면 정렬 A안 승인(KPI 6칸 같은 틀·실적 표 전체 폭·추이+카테고리 같은 높이·회차 그래프 문법). Q2~Q6 추천안 그대로.
- 마진 계산기 카드: 세 줄(머리 칩·입력·요약) 같은 4열(1:1:1.5:1) A안 승인. 요약 줄은 **이름 = 열 왼쪽 끝, 값·근거 = 열 오른쪽 끝**(보스 최종 지시).
- 카테고리 칩: 이름 글자 숨김, 태그만(도형 겹침 없이 글자색만), title·aria "카테고리: X". 회차 칩도 이름 숨김(내 판단, 보고함).
- 삭제 버튼은 휴지통 아이콘(맨 오른쪽 끝).
- 공구 스케줄: 요약 칸을 메인 KPI 틀(dshTile)로, 달력 A안(간트와 같은 상태 색 + 카테고리 점), 기획전(settings.promos) 띠를 달력·간트에 표시.
- 상단 '화면 저장' 버튼 삭제 → 리팩토링에서 capture()·html2canvas 로드까지 제거.
- 보스 피드백: 시인성·정렬 다듬기는 지적 전에 알아서(메모리 visual-polish-autonomy 저장).

## 진행 상황 (전부 운영 배포 완료)
- 커밋: fceced6(메인 A안) → f90a8e9·09bc065(시인성·기간 칸) → 49b38f5(마진 4열) → a465033(요약 줄) → ea02e7b(스케줄 요약) → fc8cd10(화면 저장 삭제) → e5a202c(달력 A안) → 04fd047(머리 칩) → 958af0a·7b3d470·8b6644e·1edd9df·4477b65·7e5da3e(정렬·칩 다듬기) → a86b555(리팩토링·QA) → 45e8878(PPT 생성기).
- 리팩토링·QA: 죽은 CSS 33줄 삭제, smoke.py 수집 상태 기준 3→4행(e7df421 meta_ceo 이후 실패하던 것), 79개 메뉴 순회 콘솔 오류 0.
- PPT: `C:\Users\USER\Desktop\Dashboard\땡스킴_대시보드_2주_작업정리_261031.pptx`(11장), 생성기 `tools/ppt/build_2week_ppt.js`(NODE_PATH=C:/Users/USER/dev/reports/node_modules).
- 엑셀: 원본 `001_PROJECTPLAN (1).xlsx` 가 보스 엑셀에서 열려 있어 새 파일 `C:\Users\USER\Documents\카카오톡 받은 파일\001_PROJECTPLAN_업데이트_261031.xlsx` 로 갱신(Revision History 9행, Action Plan 1주차 완료+19행, 새 시트 Work History 112행). Registration Information 시트 미변경.

## 남은 일
- 땡스킴 결정 대기: 회차 매출 적립금 포함 여부, 카페24 일 매출 정의(다시 묻지 말 것, 다음 주).
- 대표님 인스타 계정 인증, 서버 무료 플랜 2027-04 전 업그레이드, 노션 연동(계층 매핑 설계 뒤).
- 보안 권고: 플랜 엑셀 Registration Information 시트에 계정 비밀번호·API 키가 평문(카톡 전달) → 비밀번호 관리자로 옮기고 키 교체 권장(보스에게 알림).
- 같은 폴더의 `001_PROJECTPLAN_261030_업데이트.xlsx` 내용은 합치지 않았음.

## 다음 세션에서 필요한 맥락
- 흐름: 화면 새로 만들기·룩 변경은 목업(docs/mockups/*) 승인 뒤 → 구현 → 독립 리뷰(sonnet) → pre-commit 스모크(check_js + smoke_ui 17항목) → `powershell -File deploy\push_tar.ps1`. 트리가 더러우면 임시 worktree 에서 배포.
- 로컬 검증: 8794 포트 서버(다른 세션이 띄운 것)에서 localStorage `ia.api` 를 mock 으로 → `?view=all|margin:plan|schedule:gonggu`.
- CSS 는 gongu.css 끝에 블록을 붙이는 방식이 누적됨: 메인 `.dsh-*`, 마진 카드 `.ch4`·`.strip4`, 달력 `.sc-*`. 전역 `.kv`·`.cat`·`.dh` 충돌 주의(PLAYBOOK 6-26 재발).
- 마진 카드 1280 창(카드 약 700px)에서는 공구 기간 칩 끝이 잘림(전체 날짜는 달력에서).
- PLAYBOOK: 6-26 재발 기록 추가, 6-27(엑셀 COM 숫자 문자열 → 숫자·날짜 변환, `NumberFormat='@'`) 신설.
