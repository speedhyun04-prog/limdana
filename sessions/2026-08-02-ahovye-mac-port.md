# [세션] 2026-08-02 아호비 스튜디오 맥 포팅 (윈도우측 코드 전부 완료)

## 프로젝트/주제
아호비 스튜디오 애플(맥) 배포판 포팅. 시작은 "애플 배포버전 호환성 체크 + 어드민의 모든 기능이 다 담겨야함". 관리자 기능은 전부 웹 기반이라 맥에 자동 포함됨을 확인한 뒤, 실제 맥 네이티브 앱 포팅(ARCHITECTURE_MAC.md 1~4단계 + 7단계 플러밍)을 윈도우 PC에서 가능한 데까지 구현.

## 결정 사항
- **관리자 기능 파리티**: /admin은 순수 웹(FastAPI+admin.html), 서버·웹 UI는 맥/윈도우 완전 공유 → 맥 배포판에 자동 포함. 윈도우 전용 관리자 로직 없음.
- **맥 PIN 암호(보스 결정)**: 키체인 + cryptography(AES-GCM). 표준 라이브러리에 AES가 없어서. 기기키(키체인)+PIN(AES) 둘 다 필요 = DPAPI 속성 그대로.
- **맥 단축키(보스 결정)**: ⌥(Option)+Space. ⌘+Space는 Spotlight 예약이라 불가. ⌥는 Alt와 같은 물리 키라 근육기억 유지.
- **updater dmg apply는 7단계 매니페스트 분리와 한 묶음**으로 처리(그래야 find_pending이 .dmg를 찾음).
- **cryptography는 맥 번들 전용 신규 의존**. lazy import라 윈도우/CI 무영향. 5단계 ahovye_mac.spec에 hiddenimports로 반드시 포함(winreg는 excludes).

## 진행 상황 (완료)
윈도우 무변경 원칙 + 검증 게이트(각 모듈 selftest/스모크) 유지하며 맥 분기 구현. 전부 검증 통과.
- **1단계** client/osenv.py 신규(§3 계약 6함수). 앱 데이터 경로 중복 5곳(pinstore/config/updater_common/agent.pyw/combine.common)을 osenv.app_data_dir()로 흡수. 7개 경로값 예전 공식과 바이트 동일 확인.
- **2단계** autostart(winreg 가드+LaunchAgent plist+launchctl), pinstore(DPAPI 가드+키체인 기기키+AES-GCM `_seal`/`_unseal`만 분기).
- **3단계** hotkey(user32 가드+ctypes로 Carbon RegisterEventHotKey, pyobjc 없이). ⌥+Space, 화살표 코드모드. 맥 배선 시뮬 통과.
- **4단계 기동** orb_common launch_shell/launch_orb(osenv.launch_argv+no_window_flags), orb.py 6번째 로그dir, deploy_entry 예외 로그dir.
- **updater** updater_common 플랫폼 분리(MANIFEST_REL, _INSTALLER_RE, installer_name(v,platform), remove_motw 맥=xattr) + updater.py apply_update를 _apply_windows/_apply_mac(hdiutil+ditto 자기삭제 셸) 분기. osenv.mac_app_bundle() 공개.
- **7단계 플러밍** release.py --platform mac(_release_mac: build_mac.sh→dmg→latest-mac.json), apple.html 맥 네이티브 앱 섹션(dmg 다운로드=latest-mac.json fetch 자동활성, Gatekeeper 우클릭-열기, ⌥+Space 안내).

## 다음 세션에서 필요한 맥락 (맥에서만 가능, 보스 맥에서 새 세션)
- 설계 단일 출처: `C:\Users\USER\dev\ahovye\docs\ARCHITECTURE_MAC.md`, 진행은 메모리 [[ahovye-mac-port]].
- **5단계**: `installer/ahovye_mac.spec`(PyInstaller BUNDLE .app, arm64, hiddenimports=cryptography, excludes=winreg) + `installer/build_mac.sh`(spec→.app→hdiutil create→Ahovye-{v}.dmg) 신규. 그 뒤 `python installer/release.py --platform mac`로 첫 빌드.
- **6단계 맥 전용 오류 루프 = 미검증분 실기기 확인**: pinstore 키체인(security CLI 왕복), autostart launchctl bootstrap, hotkey Carbon 실제 ⌥+Space 눌림+런루프, updater hdiutil+ditto dmg 교체, orb/셸 open -na 기동. 맥 전용 스모크 작성.
- 접근성(손쉬운 사용) 권한: Carbon 등록형이라 실제로 불필요할 수 있음 → 맥에서 확인.
- 맥 검증 방법: 실기기 없으면 "구조 시뮬"(가짜 백엔드로 배선/인자/러너텍스트 검증)까지가 한계.

## 교훈 (PLAYBOOK 반영)
6-16 신설: OS 전용 초기화(WinDLL·winreg)를 모듈 최상위에서 실행 금지, 플랫폼 판정으로 가드하되 함수 def는 밖에 남긴다. (from __future__ import annotations로 가드된 타입 어노테이션 지연평가, 얇은 디스패처만 분기)
