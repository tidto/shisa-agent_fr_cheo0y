# macOS 12 (Monterey)에서 시사 쓰기 — 패치 내용과 빌드 방법

## 왜 막혔나
공식 dmg는 Electron 44.7.0으로 만들어졌고, Electron 44부터 Chromium이 macOS 12 지원을 끊어서 **macOS 13 이상**이 필요합니다.
이 폴더는 Electron을 **43.7.11**(마지막 43.x, Chromium 150, Node 24.21.0 — 44와 같은 Node)로 내려서 macOS 12에서도 돌게 한 버전입니다.

## 바꾼 것 (3가지, 앱 코드는 그대로)
- `desktop/package.json`: electron `44.7.0` → `43.7.11`, engines node `>=26` → `>=22`
- `desktop/package-lock.json`: 위에 맞춰 갱신
- `desktop/package.json`의 test 스크립트: `node --test test/` → `node --test "test/*.test.js"` (Node 22에서도 돌게)

## 빌드 (내 맥에서, 인터넷 필요)
1. **Node 22 LTS** 설치 (nodejs.org). Node 24 이상 공식 빌드는 macOS 13.5+ 필요라 Monterey에선 안 돌아요.
2. 터미널에서:
   ```sh
   cd shisa-agent-0.1.0-macos12/desktop
   npm ci
   npm run dist          # 애플 실리콘이면 arm64, 인텔이면 자동으로 x64
   ```
   (인텔인데 못 맞추면 `npm run dist:x64`)
3. `desktop/dist/Shisa-0.1.0-<arch>.dmg` 가 생깁니다. 열어서 「시사」를 응용 프로그램으로 끌어다 놓으세요.
   직접 빌드한 앱은 다운로드 격리 표시가 없어서 보통 「그래도 열기」 없이 열립니다.
   그래도 막히면 앱을 우클릭 → 열기.

## macOS 12에서 안 되는 것
- **Claude Code 뇌 / Claude Agent SDK**: 번들된 `claude` 바이너리가 macOS 13.0 이상용으로 빌드돼 있어 Monterey에선 실행되지 않습니다.
  → 코드 고치기·워크트리·자동 머지·MCP 모드·지식그래프 같은 Claude Code 전용 기능은 못 씁니다.
- 쓸 수 있는 것: AI 없이 쓰기(할 일·오늘 할 일·집중하기·일정 알림), 또는 Gemini/OpenAI/Claude **API 키** 뇌(채팅·브리핑 요약).
- Codex 뇌는 macOS 12에서 되는지 확인 못 했습니다.

## 「완전 로컬」인가?
- **로컬인 것**: 데이터는 `~/.cy-agent/`(SQLite, config.json)에만 저장. 데몬은 `127.0.0.1` 전용이고 Host 헤더 검사도 합니다. 분석·텔레메트리 코드는 못 찾았습니다.
- **로컬이 아닌 것**:
  1. AI 뇌를 연결하면 할 일·대화 내용이 해당 회사(Google/OpenAI/Anthropic)로 전송됩니다. 로컬 LLM은 없습니다.
  2. 하루 한 번 GitHub Releases(api.github.com)에서 새 버전 확인 → 설정 「이 앱」에서 끌 수 있습니다.
  3. 연결한 것만: Slack(소켓 모드), 캘린더 ICS 주소, GitHub, 원격 MCP 서버.
- 처음 설치 직후(뇌 없음, 연결 없음)에는 설치 마법사를 마치기 전까지 아무것도 읽지 않고 모델도 부르지 않습니다.
