# Claude Memory (로컬 세션 기록)

Claude Code의 wrap-up 스킬이 세션 요약을 저장하는 곳이다.

- `sessions\` : 세션 요약 마크다운. 파일명 `YYYY-MM-DD-주제.md`
- 원래 목적지는 NotebookLM "Claude Memory" 노트북이지만,
  notebooklm-mcp가 Gemini Notebook UI 개편에 아직 대응하지 못해
  (2026-07-18 기준, 이슈 #63/#69/#72) 로컬 저장을 우선 경로로 쓴다.
- MCP가 수정되면 wrap-up이 NotebookLM 저장을 우선하고,
  여기 쌓인 파일은 add_source로 일괄 이관하면 된다.
