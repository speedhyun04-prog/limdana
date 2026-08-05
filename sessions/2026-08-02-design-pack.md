# [세션] 2026-08-02 아호비 자료 품질 디자인 팩 (1~4단계 + v2 룩 대개편)

**프로젝트/주제**: 아호비 스튜디오의 AI가 만드는 자료(HTML 문서·슬라이드·pptx)
품질을 Claude 아티팩트 수준으로 올리는 디자인 팩 구현. 설계는 이전 세션에서
끝나 있었고(docs/ARCHITECTURE_DOC_DESIGN.md), 이번 세션은 1단계부터 4단계까지
구현 + 룩 대개편 + 결함 감사.

## 결정 사항

- **포인트색**: 아호비 인디고 확정. `--accent #6657FF`, 밝은 배경 위 강조
  텍스트는 `--accent-deep #4A38D6`(v2에서 #5543E1에서 조정, 대비 확보).
- **3단계 pptx 실파일 승인**: 웹 스튜디오(서버 소스 실행) 한정. frozen 오브
  미탑재. HTML 덱이 여전히 기본이고 pptx는 "상대가 편집할 실무 파일"일 때만.
- **4단계 승인**: M6 서버측 표시 전용 검사(server.py 접촉) + htmlship 폰트
  임베드 도구. 단, 폰트 임베드 **기본값은 여전히 "안 함"**, 사외 반출을 명시
  요청받았을 때만 `*.ship.html` 사본 생성.
- **룩**: 보스 1차 소견 "규칙은 도는데 아직 많이 부족" → 같은 데이터로 A(현행)/
  B(개선안) 비교 → **B 확정 + 추가 디벨롭** 지시 → 팩 전체 v2로 개편.
- 룩 검수는 최종 단계까지 간 뒤 일괄로 하기로(보스 일정 사유).

## 진행 상황 (완료)

- **1단계**: `design_pack/`(PACK.md·guide-core·guide-slides·tokens.css) +
  템플릿 report·slides + brain.py 꼬리 교체(`_pack_dir()` 절대경로 Read 절차,
  안전망 5규칙) + installer spec datas.
- **2단계**: `tools/doccheck.py`(stdlib 검사기, E 5종/W 5종, exit 0/1/2) +
  템플릿 3종(dashboard·cardnews·onepager) + guide-charts + 꼬리 4)항 활성
  (모델 자가 검사, 재검사 총 2회 상한).
- **3단계**: `tools/pptgen.py`(slides.json → pptx, 레이아웃 5종 + 파워포인트
  네이티브 차트) + `design_pack/pptx/MASTER.md`. python-pptx 1.0.2 서버 설치 확인.
  한국어는 eastAsia 폰트를 별도 지정해야 맑은고딕으로 안 샌다.
- **4단계**: `chat_common.doc_check_step` + web/server.py on_tool 배선(피드에
  "검사 ✅/⚠" 한 줄, 실패는 조용히 None) + `tools/htmlship.py`(Pretendard 3종
  서브셋 woff2 base64 임베드).
- **v2 룩 대개편**: 토큰을 번호 스케일에서 의미 이름으로 재편(`--fs-display`
  56px, `--fs-stat` 54px 등), **룩 문법 8가지를 guide-core 0절에 명문화**,
  템플릿 5종 전면 교체, pptgen 상수·MASTER.md 색 미러, 마커 v1→v2
  (단일 출처 `doccheck.PACK_VERSION`, 구버전은 W MARKER_OLD),
  스모크에 **tokens.css 전수 동기화 검사** 추가.
- **고정 캔버스 결함 감사**: 실측으로 **실제 결함 1건** 발견. guide-slides가
  "불릿 최대 5개"라 했지만 의무 구조인 결론+근거 2줄에서 5개는 6px 넘쳐 잘림
  (overflow:hidden이라 무증상). 가이드를 실측값(2줄 4개/한 줄 6개)으로 고치고
  같은 상한을 doccheck `W CANVAS_DENSE`로 코드에 박음. 납품 템플릿 자체는 안전.
- **검증**: 스모크 5종 전부 통과. 웹 스튜디오 실사용 3턴(보고서 2, 카드뉴스 1)
  전 항목 통과 — 팩 Read → 템플릿 복제(마커 보존) → 모델 자가 doccheck →
  피드 검사 스텝 → 독립 doccheck E 0건.

## 남은 작업

1. **보스의 v2 룩 최종 검수** (유일한 실질 잔여. 조정은 토큰 한 줄로 5종 반영)
2. 흐름형 3종(report·dashboard·onepager)의 반응형·가로넘침 확인 — 이번엔 못 함
   (미리보기 창 뷰포트가 0으로 잡혀 측정 불가). 실기 브라우저 필요.
3. frozen 오브 동봉 실검증 — 다음 릴리스 스모크에 편입.

## 다음 세션에서 필요한 맥락

- 설계 문서: `dev\ahovye\docs\ARCHITECTURE_DOC_DESIGN.md`
  (§13 v2 개편 전문, §13.5 결함 감사 실측표, §6.2 검사 항목, §8 계약)
- 팩: `dev\ahovye\design_pack\` — 진입점 PACK.md, **품질의 핵심은
  guide-core.md 0절(룩 문법 8가지)**. 템플릿이 품질의 상한이라 템플릿 수정은
  코어 편집급으로 취급(한 세션만).
- 도구: `tools\doccheck.py`(stdlib), `tools\pptgen.py`·`tools\htmlship.py`
  (둘 다 서버 한정 의존성: python-pptx / fonttools+brotli)
- 스모크: `python tests\test_doc_pack_smoke.py` 외 doccheck·pptgen·feed_check·
  htmlship 5종. 인코딩은 `$env:PYTHONIOENCODING="utf-8"` 먼저.
- **주의 1**: `tools/` 를 고치면 **웹 서버 재시작 필요**. 상주 서버가 doccheck를
  지연 임포트해 sys.modules에 캐시한다. 실제로 이것 때문에 "저장 파일은 v2인데
  피드만 마커 없음"으로 떴다. design_pack의 md·html은 매 턴 Read라 재시작 불필요.
- **주의 2**: 이 저장소는 **다른 세션과 동시 작업 중**이다. 이번 랩업 시점에
  HEAD가 몇 분 사이 3번 바뀌었다(581bb11 → 48430bf → 96d02d8 → a0852e9).
  커밋 전에 HEAD 안정성부터 확인할 것.
- **커밋 상태**: 디자인 팩 전량(최신 CANVAS_DENSE 작업 포함)이 다른 세션의 커밋
  `a0852e9`("feat(ai): AI 생성을 서버 잡으로 전환…")에 섞여 들어갔다. 내용은
  안전하게 보존됐고 메시지만 이 작업을 설명하지 않는다. 공유 히스토리라
  amend 하지 않았고 별도 커밋도 만들지 않았다.
- 서버 기동: `pythonw C:\Users\USER\dev\ahovye\web\run_server.pyw`
  (숨김 창, 포트 8765, 헬스 `GET /api/health`)

## 다음 세션 첫 작업 (보스 지시, 미완)

**디자인 팩만 별도 커밋으로 분리한다.** 이번 세션 마지막에 받은 지시인데,
다른 세션이 파일을 물고 있어 실행하지 못하고 마감했다.

- 대상: 커밋 `171154f`("feat(studio): 상단 메뉴바 + AI 서버 잡 + 병행 세션 작업 일괄")
- **안전 조건: 아직 푸시 전이다.** origin/main 은 581bb11, 로컬만 ahead 1.
  히스토리를 다시 써도 원격에 영향 없고 reflog 로 되돌릴 수 있다.
  **실행 전에 이 조건(미푸시)이 여전한지 반드시 재확인할 것.**
- 선행 조건: 다른 세션이 조용해질 것(HEAD 와 작업 트리가 몇 분간 무변동).
  마감 시점 상태는 HEAD 171154f, 미커밋 6건(그쪽 것), 미푸시 1건.
- 방법: `git reset --soft HEAD~1` 로 되돌리면 171154f 내용이 전부 인덱스에 남고
  **다른 세션의 미커밋 작업 트리는 건드리지 않는다.** 거기서 디자인 팩 경로만
  갈라 2개 커밋으로 나눈다(디자인 팩 / 나머지는 원문 메시지 그대로).
  원 커밋 메시지 원문은 실행 시 `git log 171154f` 로 다시 뜨거나, 이미
  스크래치패드에 받아 뒀다면 그것을 쓴다.
- 디자인 팩 커밋에 넣을 22개 경로:
  `design_pack/**`(11), `tools/doccheck.py`·`pptgen.py`·`htmlship.py`,
  `tests/test_doc_pack_smoke.py`·`test_doccheck_smoke.py`·`test_feed_check_smoke.py`
  ·`test_htmlship_smoke.py`·`test_pptgen_smoke.py`,
  `docs/ARCHITECTURE_DOC_DESIGN.md`, `chat_common.py`, `installer/ahovye.spec`
  (뒤 두 개는 이번 세션 변경만 들어 있어 파일 단위로 갈라도 안전)
- **가를 수 없는 것**: `brain.py` 와 `web/server.py` 는 내 배선 코드와 다른 세션
  변경이 같은 파일에 섞여 있다. 파일 단위로는 분리 불가. 기본 방침은 그 둘을
  나머지 커밋에 두고 디자인 팩 커밋 메시지에 "배선은 저 커밋" 이라고 적는 것.
  보스가 헝크 단위 분리까지 원하면 그때 patch 필터로 처리(위험도 상승).

## PLAYBOOK 반영 (앱 공장 피드백 루프)

부류급 교훈 2건 신규 + 기존 1건 보강:

- **5-11** 고정 캔버스(overflow:hidden)는 넘쳐도 무증상으로 잘린다. 상한을
  실측해 가이드와 검사기에 함께 박는다. 측정은 `file://` 말고 로컬 HTTP로,
  재기 전에 `innerWidth`가 0이 아닌지 확인.
- **6-20** 상주 서버가 지연 임포트한 모듈은 sys.modules에 캐시된다. 도구·검사기
  코드를 고쳤으면 재시작이 필요하다(데이터 파일은 매번 읽혀 "일부만 반영" 유령
  상태가 된다).
- **5-2 보강** 공유 워킹 트리에서는 커밋 전에 HEAD 안정성을 본다. 내 변경이 이미
  남의 커밋에 들어갔으면 amend로 되돌리지 말고 보고한다.
