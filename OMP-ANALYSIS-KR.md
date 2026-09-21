# oh-my-pi (omp) 전수조사 분석 정리 📚

> 작성: Claude Code 세션 대화 정리본
> 정리일: 2026-09-21
> 대상 저장소: https://github.com/bmshin94/oh-my-pi
> 원본(upstream): https://github.com/can1357/oh-my-pi
> 뿌리(fork origin): https://github.com/badlogic/pi-mono
> 공식 사이트: https://omp.sh
> npm: https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent
> Discord: https://discord.gg/4NMW9cdXZa

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [쉬운 설명](#2-쉬운-설명)
3. [설치 및 사용법](#3-설치-및-사용법)
4. [플러그인 vs 스킬 vs MCP](#4-플러그인-vs-스킬-vs-mcp)
5. [API 토큰이 필요한가](#5-api-토큰이-필요한가)
6. [왜 GitHub에서 유명한가](#6-왜-github에서-유명한가)
7. [로컬 에이전트 구축에 도움이 되는가](#7-로컬-에이전트-구축에-도움이-되는가)
8. [React / PHP로 만들 수 있는가](#8-react--php로-만들-수-있는가)
9. [수익화 아이디어](#9-수익화-아이디어)
10. [참고 링크](#10-참고-링크)

---

## 1. 프로젝트 개요

### 한 줄 정의

**oh-my-pi (omp)** 는 터미널에서 동작하는 **오픈소스 AI 코딩 에이전트**다.
Claude Code / Cursor / Codex 와 같은 계층의 제품이며, "배터리 포함(batteries included)" 철학으로
필요한 기능이 처음부터 전부 내장되어 있다.

| 항목 | 내용 |
|---|---|
| 라이선스 | **MIT** (상업적 이용/수정/재배포/SaaS화 자유) |
| 버전 | 18.2.5 |
| 저작권 | © 2025 Mario Zechner, © 2025-2026 Can Bölük, © 2026 Stencil Labs, Inc. |
| 언어 | TypeScript (~68만 줄) + Rust (~26만 줄) |
| 런타임 | Bun (>= 1.3.14) |
| 플랫폼 | macOS · Linux · Windows (WSL 불필요) |

### 규모

| 항목 | 수치 |
|---|---|
| 전체 파일 | 7,501개 |
| TypeScript 파일 | 5,181개 (~688,298줄) |
| Rust 파일 | 467개 (~262,674줄) |
| 지원 AI 제공사 | 60개 이상 |
| 내장 도구 | 31개 |
| LSP 기능 / DAP 기능 | 14개 / 28개 |
| 웹 검색 백엔드 | 23개 |
| 기술 문서 | 80개 이상 (`docs/`) |

### 폴더 구조

```
oh-my-pi/
├── packages/                  TypeScript 워크스페이스 (17개)
│   ├── coding-agent/          [핵심] CLI + SDK, 에이전트 두뇌
│   ├── ai/                    60개 LLM 제공사 통합 클라이언트
│   ├── agent/                 에이전트 런타임 (툴 콜링, 상태 관리)
│   ├── catalog/               모델 카탈로그 DB
│   ├── tui/                   터미널 UI 라이브러리 (차등 렌더링)
│   ├── natives/               Rust N-API 바인딩 래퍼
│   ├── mnemopi/               SQLite 기반 로컬 메모리 엔진
│   ├── snapcompact/           비트맵 프레임 컨텍스트 압축
│   ├── collab-web/            브라우저 게스트 클라이언트 + 로컬 릴레이
│   ├── browser-relay/         Chrome 확장 (기존 탭 제어)
│   ├── metaharness/           벤치마크 러너 + 라이브 대시보드
│   ├── stats/                 AI 사용량 관측 대시보드
│   ├── omptype/               ArkType 호환 스키마 검증 (lazy JIT)
│   ├── utils/ wire/ tsconfig/ 공용 유틸·프로토콜 타입
│   └── typescript-edit-benchmark/  편집 벤치마크 스위트
│
├── crates/                    Rust 네이티브 코어
│   ├── pi-shell/              내장 bash 엔진 (~38,000줄)
│   ├── pi-natives/            N-API 표면 (~25,000줄)
│   ├── pi-builtins/           in-process 커맨드라인 유틸 67개
│   ├── pi-walker/             병렬 ignore-aware 파일 워커 (~5,200줄)
│   ├── pi-iso/                워크스페이스 격리 (APFS/btrfs/zfs/overlayfs/projfs)
│   ├── pi-ast/                tree-sitter + ast-grep (50+ 언어)
│   ├── pi-edit/               해시라인 편집 엔진
│   ├── pi-voice/              오디오 캡처/재생, Opus, WebRTC
│   └── vendor/brush-core/     brush-shell 포크 (vendored)
│
├── docs/                      80+ 기술 문서
├── python/                    omp-rpc (파이썬 클라이언트), robomp
├── .omp/                      이 저장소 자체의 스킬/커맨드/툴 설정
├── .github/                   CI 워크플로우 (ci / nix / bazel-cache-warm)
├── infra/                     셀프호스팅 러너, Kata 런타임, Bazel 원격 캐시
├── scripts/                   릴리스/빌드/벤치마크 스크립트 (50여 개)
├── bazel/ nix/                재현 가능 빌드 시스템
└── AGENTS.md                  28KB 에이전트 행동 규칙서
```

### 내장 도구 31개

**파일 & 검색**
- `read` — 파일/디렉토리/압축파일/SQLite/PDF/노트북/URL/`ssh://` 원격 경로/내부 `://` 스킴을 하나의 인터페이스로
- `write` — 파일·아카이브 엔트리·SQLite 행 생성/덮어쓰기
- `edit` — 해시라인(content-hash anchor) 패치, stale anchor 복구
- `ast_edit` — 구조적 리라이트, 적용 전 미리보기
- `ast_grep` — 50+ tree-sitter 문법 기반 구조 질의
- `grep` — 파일/글롭/내부 URL 위의 정규식 검색
- `glob` — 글롭 경로 탐색

**런타임**
- `bash` — 워크스페이스 셸, in-process coreutils 46개, 선택적 PTY, 백그라운드 잡
- `eval` — 영속 Python/JavaScript 셀, 공유 프렐류드, **에이전트 도구 역호출(re-entry)**

**코드 지능**
- `lsp` — 진단/네비게이션/심볼/리네임/코드액션/raw 요청
- `debug` — DAP 세션 제어 (브레이크포인트·스텝·스레드·스택·변수)
- `security_scan` — 네이티브 보안 리뷰, Codex Security 클라우드 스캔

**조율**
- `task` — 서브에이전트 병렬 팬아웃, 워크스페이스 격리 옵션
- `hub` — 라이브 에이전트 메시징, 백그라운드 잡 대기/취소, 장기 프로세스 감독
- `todo` — 세션 투두 리스트 순서 변경 + 페이즈 추적
- `ask` — 구조화된 후속 질문 (옵션 피커)

**데스크톱 & 웹**
- `browser` — Puppeteer 탭 (헤드리스 Chromium / CDP attach / 릴레이 경유 내 Chrome)
- `computer` — 호스트 데스크톱 제어 (창, 스크린샷, 네이티브 입력, AX 트리, 클립보드)
- `web_search` — 23개 제공사, 답변 + 인용
- `github` — GitHub CLI 오퍼레이션
- `generate_image` — Gemini / GPT / xAI Grok 이미지 모델
- `tts` — xAI Grok Voice (5개 음성, WAV/MP3)

**메모리 & 스킬**
- `checkpoint` / `rewind` — 대화 상태 마킹, 탐색 컨텍스트 정리
- `retain` / `recall` / `reflect` / `memory_edit` — 영속 메모리 뱅크
- `learn` / `manage_skill` — 재사용 가능한 교훈 캡처, 관리형 스킬 생성

> 기본 OFF (설정으로 활성화): `github`, `security_scan`, `generate_image`, `tts`, `checkpoint`, `rewind`, 메모리 도구들

### 차별화 기능 (킬링 포인트)

1. **Hashline 편집** — 줄 내용을 해시 앵커로 지정. 파일이 변경되면 앵커 불일치로 패치를 거부해 손상을 방지. Grok 4 Fast 기준 출력 토큰 61% 절감.
2. **Time-Traveling Stream Rules (TTSR)** — 정규식 매치 시 토큰 스트림을 **중간에 중단**하고 규칙을 시스템 리마인더로 주입한 뒤 같은 지점부터 재시도. 평소에는 컨텍스트 비용 0. 컴팩션을 넘어서도 주입이 유지됨.
3. **Advisor (감시자 모델)** — 두 번째 모델이 별도 컨텍스트로 메인 에이전트의 모든 턴을 읽고 인라인 노트(귀띔/우려/하드 블로커)를 주입.
4. **First-class Subagents** — `task`가 격리된 worktree로 팬아웃, 결과는 **스키마 검증된 객체**. `Alt+A`로 Agent Hub를 열어 실시간 감독·조종·중단.
5. **16개 내부 스킴** — `pr://`, `issue://`, `agent://`, `skill://`, `ssh://`, `conflict://`, `xd://` 등이 모든 FS 도구에서 투명하게 동작. `read pr://1428`이 `read src/foo.ts`와 같은 모양으로 반환.
6. **conflict:// 충돌 해결** — 머지 충돌 하나가 URL 하나. `@theirs` / `@ours` / `@base` 를 쓰면 해결. 일괄은 `conflict://*`.
7. **/collab** — 세션을 릴레이에 올려 링크 + QR 공유. 프레임은 클라이언트단에서 봉인되어 릴레이가 키를 보지 못함. 읽기 전용 모드 지원.
8. **Snapcompact** — 컨텍스트를 비트맵 프레임으로 래스터화 + PNG 인코딩해 압축.
9. **네이티브 일체화** — ripgrep/glob/find/bash를 shell out 하지 않고 프로세스에 링크. 핫패스에 fork/exec 없음. Windows에서도 WSL 없이 동작.
10. **기존 설정 자동 상속** — `.claude`, `.cursor`, `.windsurf`, `.gemini`, `.codex`, `.cline`, `.github/copilot`, `.vscode` 를 네이티브 형태 그대로 읽음. 마이그레이션 스크립트 불필요.

### 벤치마크 (README 공개 수치)

| 모델 | 지표 | 변화 |
|---|---|---|
| Grok Code Fast 1 | 통과율 | 6.7% → 68.3% (약 10배) |
| Gemini 3 Flash | vs str_replace | +5%p |
| Grok 4 Fast | 출력 토큰 | −61% |
| MiniMax | 통과율 | 2.1배 |

> 같은 가중치, 같은 프롬프트. 하네스(harness)만 바꾼 결과.
> 관련 글: https://blog.can.ac/2026/02/12/the-harness-problem/

---

## 2. 쉬운 설명

### 비유

| 구분 | 비유 | 할 수 있는 일 |
|---|---|---|
| ChatGPT 웹 | 요리책 | 코드를 알려줌. 내 컴퓨터는 못 만짐 |
| Cursor / Claude Code | 밀키트 | 내 파일을 직접 수정. 정해진 기능만 |
| **oh-my-pi** | 주방 전체 + 셰프 60명 | 파일 수정 + 터미널 실행 + 브라우저 제어 + 데스크톱 제어 + 디버거 + 소스 개조 |

### 핵심 5가지

**1) 여러 AI가 동시에 일한다 (subagents)**
```
"프로젝트 전체 리팩토링해줘"
   → AI-1 프론트 / AI-2 백엔드 / AI-3 테스트 / AI-4 문서 / AI-5 리뷰
   → 각자 격리된 폴더에서 동시 작업, 충돌 없음
```

**2) 감시자가 붙는다 (Advisor)**
메인 AI가 일하는 동안 다른 AI가 지켜보다가 "그건 요청과 다릅니다"라고 지적.

**3) 삑사리 나면 즉시 멈춘다 (TTSR)**
```
AI: "Box::leak 을 쓰면..."   ← 여기서 스트림 중단
규칙 주입: "프로덕션에서 Box::leak 금지"
AI: "Arc<str> 로 하겠습니다"  ← 같은 지점부터 재개
```

**4) 기억한다 (Memory)**
어제 알려준 프로젝트 규칙을 오늘 새 세션 첫 턴에 자동으로 로드.

**5) 빠르다 (Rust)**
검색 엔진과 셸을 별도 프로세스로 띄우지 않고 프로세스 내부에 링크. 매 호출마다의 fork/exec 왕복 비용이 사라짐.

### 이 프로젝트가 해결해주는 문제

| 문제 | 해결 |
|---|---|
| API 요금이 부담됨 | 로컬 모델(Ollama 등) 또는 기존 구독(Copilot/Cursor) 재활용 |
| 회사 코드 외부 유출 우려 | `secrets` 기능으로 API 키/토큰 자동 마스킹 + 로컬 모델 |
| Windows 환경이라 도구가 안 맞음 | 네이티브 Windows 지원 (WSL 불필요) |
| AI가 엉뚱한 방향으로 감 | Advisor + TTSR |
| 매 세션마다 프로젝트 설명 반복 | 메모리 백엔드 |
| 원하는 기능이 없음 | MIT 라이선스 + TypeScript 익스텐션 API |
| 기존 도구 설정 이전이 귀찮음 | 8개 포맷 자동 상속 |

---

## 3. 설치 및 사용법

### 설치

```sh
# macOS / Linux
curl -fsSL https://omp.sh/install | sh

# Homebrew
brew install can1357/tap/omp

# Bun (README 권장)
bun install -g @oh-my-pi/pi-coding-agent

# Windows (PowerShell)
irm https://omp.sh/install.ps1 | iex

# Nix
nix run github:can1357/oh-my-pi
nix profile install github:can1357/oh-my-pi

# 버전 고정 (mise)
mise use -g github:can1357/oh-my-pi
```

> Alpine/musl 사용 시 먼저: `apk add libstdc++ libgcc`
> 요구사항: bun >= 1.3.14

### 소스 빌드

```sh
bun setup                 # 워크스페이스 설치 + Rust N-API 애드온 빌드
bun dev                   # 개발 모드 실행
bun dev -- --version      # 비대화형 스모크 체크
bun run build:native      # Rust 크레이트 변경 후 재빌드

nix develop               # Nix: 고정된 Bun/Rust 툴체인 진입
```

### 기본 사용

```sh
omp setup                          # 최초 1회 설정 마법사
omp                                # 대화형 TUI
omp "이 버그 고쳐줘"                 # 초기 프롬프트와 함께 시작
omp -p "src의 ts 파일 나열"          # 헤드리스(한 번 처리 후 종료)
omp @design.md @mock.png "구현해"    # 파일/이미지 첨부
omp -c "아까 얘기 뭐였지?"           # 직전 세션 이어서
omp -r                             # 세션 재개(피커)
omp --fork <세션>                   # 세션 분기
omp --from-claude                  # Claude Code 세션 임포트
omp --from-codex                   # Codex 세션 임포트
omp --export <세션>                 # 세션을 HTML로 내보내기
omp --cwd <dir> / --add-dir <dir>  # 작업 디렉토리 지정/추가
omp --profile <name>               # 인증/세션/설정/캐시를 격리한 프로필
omp --no-session                   # 세션 저장 안 함
```

### 슬래시 명령 (세션 내부)

| 명령 | 기능 |
|---|---|
| `/model` | 모델 교체 (`Ctrl+P`로 역할별 순환) |
| `/login <provider>` | 제공사 로그인 (OAuth / 코딩 플랜) |
| `/review` | 리뷰 서브에이전트 병렬 소환 → P0~P3 우선순위 + 신뢰도 + 최종 판정 |
| `/collab`, `/collab view` | 세션 공유 링크 + QR (읽기/쓰기, 읽기 전용) |
| `/advisor` | 감시자 모델 설정/상태 |
| `/vibe` | 바이브 모드 (사용자가 디렉터, 영속 fast/good 워커 세션 구동) |
| `/fresh` | 프로바이더 스트림 상태 리셋 (전사 캐시/스트림 꼬임 해소) |
| `/mcp` | MCP 서버 추가/관리 |
| `/marketplace` | 플러그인 브라우저/설치 |
| `/settings` | 설정 UI |
| `/debug` | 디버깅/리포트/프로파일링 |
| `/reload-plugins` | 플러그인 리로드 |
| `Alt+A` | Agent Hub (서브에이전트 실시간 감독) |

### 매직 키워드

프롬프트 산문 안에 아래 단어를 쓰면 해당 동작이 활성화된다 (코드 블록·경로·식별자 안에서는 발동하지 않음).

- `ultrathink` — 신중한 다단계 추론 + 지원되는 최고 사고 강도
- `orchestrate` — 독립적인 작업을 병렬 서브에이전트로 실행하고 각 단계를 검증
- `workflowz` — `task` 도구로 결정론적 멀티 서브에이전트 워크플로우 구성

### 셸 자동완성

```sh
eval "$(omp completions zsh)"     # ~/.zshrc
eval "$(omp completions bash)"    # ~/.bashrc
omp completions fish > ~/.config/fish/completions/omp.fish
```

---

## 4. 플러그인 vs 스킬 vs MCP

### 결론: omp는 셋 다 아니다. **호스트(Host)** 다.

```
┌──────────────────────────────────────┐
│  oh-my-pi (omp) = 에이전트 본체       │
│  Claude Code / Cursor 와 같은 계층    │
├──────────────────────────────────────┤
│  아래를 "먹는" 쪽:                    │
│   - Plugin    (마켓플레이스)          │
│   - Skill     (SKILL.md)             │
│   - MCP       (외부 도구 서버)        │
│   - Extension (TypeScript 모듈)       │
│   - Command   (슬래시 명령)           │
│   - Hook      (이벤트 훅)             │
│   - LSP       (언어 서버)             │
└──────────────────────────────────────┘
```

### 스킬 (Skills)

```
<root>/skills/
  ├─ postgres/SKILL.md      ✅ 인식됨
  ├─ pdf/SKILL.md           ✅ 인식됨
  └─ team/internal/SKILL.md ❌ 인식 안 됨 (중첩 불가)
```

- frontmatter: `name`, `description`, `globs`, `alwaysApply`, `hide`, `disableModelInvocation`
- 시작 시 **이름 + 설명만** 시스템 프롬프트에 노출 → 토큰 절약
- 본문은 필요할 때만 `read skill://<name>` 으로 온디맨드 로드
- `/skill:<name>` 슬래시 명령으로도 호출 가능
- 디렉토리 스캔은 **비재귀(non-recursive)**
- 이 저장소의 예시: `.omp/skills/{semantic-compression, system-prompts, tool-prompt-optimization}`

### MCP (Model Context Protocol)

omp는 MCP **클라이언트**다. 설정 위치:

| 스코프 | 경로 |
|---|---|
| 프로젝트 | `.omp/mcp.json` (호환: `.omp/.mcp.json`) |
| 유저 | `~/.omp/agent/mcp.json` |
| 프로필 | `~/.omp/profiles/<name>/agent/mcp.json` |
| 폴백 | 프로젝트 루트 `mcp.json` / `.mcp.json` |

**다른 도구의 설정을 그대로 번역해서 읽어온다:**
- Claude Code: `~/.claude.json`, `~/.claude/mcp.json`, `.claude/.mcp.json`
- Codex: `~/.codex/config.toml`, `.codex/config.toml` (`[mcp_servers.*]`)
- Gemini CLI: `~/.gemini/settings.json`, `.gemini/settings.json`
- OpenCode: `~/.config/opencode/opencode.json`, `opencode.json`
- Cursor: `~/.cursor/mcp.json`, `.cursor/mcp.json`
- Windsurf: `~/.codeium/windsurf/mcp_config.json`, `.windsurf/mcp_config.json`
- VS Code: `.vscode/mcp.json` (`mcp.servers`, 프로젝트 전용)

### 플러그인 (Marketplace)

```sh
/marketplace add anthropics/claude-plugins-official
/marketplace install wordpress.com@claude-plugins-official
/marketplace list / update / remove
```

- 카탈로그: `.omp-plugin/marketplace.json` (우선) 또는 `.claude-plugin/marketplace.json` (Claude Code 호환 폴백)
- 식별자: `name@marketplace`
- 스코프: `user` (`~/.omp/plugins/installed_plugins.json`) / `project` (`.omp/plugins/installed_plugins.json`)
  - 활성화된 project 설치가 동일 이름의 user 설치를 가림(shadow)
- 플러그인 내용물: 스킬, 커맨드, 에이전트, 규칙, 훅, 도구, MCP 서버, LSP 서버
- `package.json` 의 `omp.extensions` 로 익스텐션 모듈 선언 가능, `omp-plugins.lock.json` 에 기록

### 익스텐션 (Extension) — 가장 강력

```ts
import type { ExtensionAPI } from "@oh-my-pi/pi-coding-agent";

export default function myExtension(pi: ExtensionAPI) {
  pi.registerTool(...);     // LLM이 호출 가능한 도구
  pi.registerCommand(...);  // 슬래시 명령
  pi.on("tool_call", ...);  // 모든 도구 실행 인터셉트
  // 단축키/플래그, 커스텀 메시지 렌더링,
  // sendMessage / sendUserMessage / appendEntry 로 세션 주입
}
```

> "Nothing is reserved." — 내장 기능이 쓰는 것과 **동일한 API**를 외부 익스텐션도 사용할 수 있다.

---

## 5. API 토큰이 필요한가

**필수가 아니다.** 인증 경로가 4가지 있다.

### 1) API 키 (종량제)

```sh
export ANTHROPIC_API_KEY=...
export OPENAI_API_KEY=...
export GEMINI_API_KEY=...
export XAI_API_KEY=...
# 60개 제공사별 환경변수를 모두 지원 (docs/environment-variables.md)
```

우선순위 예: `ANTHROPIC_FOUNDRY_API_KEY` > `ANTHROPIC_OAUTH_TOKEN` > `ANTHROPIC_API_KEY`

### 2) OAuth 로그인 (키 없이 계정 연결)

`/login` 또는 `omp login`
지원: Anthropic · OpenAI Codex · Google Antigravity · SuperGrok(xAI) · Cursor · GitHub Copilot · Devin · Qwen Portal

### 3) 기존 코딩 플랜 구독 재활용 (추가 비용 0)

Cursor · GitHub Copilot · GitLab Duo · Kimi Code · MiniMax Coding Plan · Alibaba Coding Plan ·
Z.AI / GLM Coding Plan · Zhipu · Umans · Qwen Portal 등

### 4) 로컬 모델 (완전 무료, 키 불필요)

```yaml
# ~/.omp/agent/models.yml
providers:
  local:
    baseUrl: http://localhost:11434/v1   # Ollama
    api: openai-completions
    apiKey: dummy
    models:
      - id: qwen3-coder
        name: Qwen3 Coder
        contextWindow: 128000
        maxTokens: 32000
```

```yaml
# ~/.omp/agent/config.yml
modelRoles:
  default: local/qwen3-coder
```

지원: Ollama · Ollama Cloud · LM Studio · llama.cpp · vLLM · LiteLLM
검증: `omp models <provider>`

### 웹 검색도 키 없이 가능

| 무료 | 키 필요 |
|---|---|
| duckduckgo, startpage, google, ecosia, mojeek, public, searxng(self-host) | perplexity(익명 폴백 있음), exa, kagi, tavily, brave, jina, firecrawl(키리스 폴백 있음), tinyfish, parallel, synthetic, zai |

### 보안: secrets 기능 (기본 OFF, 반드시 켤 것)

```yaml
# config.yml
secrets:
  enabled: true
```

활성화 시 LLM 제공사로 전송되기 전에 자동 마스킹되는 대상:
- 이름이 `KEY`/`SECRET`/`TOKEN`/`PASSWORD`/`PASS`/`AUTH`/`CREDENTIAL`/`PRIVATE`/`OAUTH` 패턴이고 값이 8자 이상인 환경변수
- `secrets.yml` 에 정의한 커스텀 값
- 내장 정규식: GitHub/GitLab/OpenAI/Anthropic 토큰, AWS 액세스 키, Google API 키, Slack 토큰, npm 토큰, Stripe 시크릿/제한 키·웹훅 시크릿, Hugging Face 토큰, SendGrid 키, JWT, Bearer 헤더, PEM 개인키 블록
- `DATABASE_URL` 같은 연결 URL 내부의 패스워드 (변수명 무관)

`$$3P8W5JH1TK2Q$$` 형태의 결정론적 플레이스홀더로 치환되며, 모델이 작성한 도구 인자는 **실행 직전에 원값으로 복원**된다.
모드: `obfuscate`(가역, 기본) / `replace`(비가역).

### 비용 절감 설정

- **역할 기반 라우팅** — `default`/`smol`/`slow`/`plan`/`commit`/`vision`/`task`/`advisor`/`tiny` 9개 역할별로 모델 분리
- **폴백 체인** — `retry.fallbackChains` 로 429/쿼터 초과 시 다음 항목이 턴을 인계, 쿨다운 후 복구
- **라운드로빈 크레덴셜** — 제공사별 API 키를 여러 개 쌓으면 세션 어피니티 + 개별 백오프로 자동 순환
- **경로 스코프 모델** — `enabledModels` / `disabledProviders` 에 `path:` 접두사를 붙여 특정 저장소에만 다른 모델 세트 적용

---

## 6. 왜 GitHub에서 유명한가

1. **배터리 포함 철학** — 31개 도구, 60개 제공사, 23개 검색 백엔드가 기본 내장. 설치 즉시 풀 기능.
2. **실제 기술력** — Rust 26만 줄. ripgrep/glob/bash를 shell out 하지 않고 프로세스에 링크. Windows 네이티브.
3. **충격적인 벤치마크** — 같은 모델·같은 프롬프트에서 하네스만 바꿔 통과율 10배, 출력 토큰 −61%.
4. **아무도 안 한 기능** — Hashline, TTSR, conflict:// 스킴, 16개 내부 스킴, Snapcompact, Advisor, 암호화된 /collab.
5. **마이그레이션 비용 0** — 8개 도구 포맷을 원래 모양 그대로 읽음. Claude Code 세션 임포트까지 지원.
6. **MIT + 완전 공개** — 80개 이상의 내부 아키텍처 문서, Bazel + Nix 재현 빌드, Discord 커뮤니티, PR 오픈.
7. **업데이트 속도** — 버전 18.2.5. 하루에도 여러 번 perf/fix/feat 커밋.

---

## 7. 로컬 에이전트 구축에 도움이 되는가

**매우 큰 도움이 된다.** 세 가지 층위에서 쓸 수 있다.

### A) 그대로 쓰기 — 완전 로컬 에이전트

`models.yml` 에 Ollama/LM Studio/llama.cpp/vLLM 을 등록하면 인터넷 없이, 데이터 유출 없이 동작한다.
참고 문서: `docs/local-models.md`

### B) 임베딩 — 4개 진입점

**SDK (Node/TypeScript)**
```ts
import {
  ModelRegistry, SessionManager,
  createAgentSession, discoverAuthStorage,
} from "@oh-my-pi/pi-coding-agent";

const auth = await discoverAuthStorage();
const models = new ModelRegistry(auth);
await models.refresh();

const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
  authStorage: auth,
  modelRegistry: models,
});
await session.prompt("list .ts files");
```

**RPC (언어 무관, NDJSON over stdio)**
```sh
omp --mode rpc --no-session
```
```
> {"id":"r1","type":"prompt","message":"list .ts files"}
< {"id":"r1","type":"response", ...}
> {"id":"r2","type":"set_model","provider":"anthropic","modelId":"sonnet-4.5"}
> {"id":"r3","type":"abort"}
```
`--mode rpc-ui` 는 도구 카드·셀렉터·다이얼로그를 `extension_ui_request` 프레임으로 넘겨 호스트가 렌더링하게 한다.
파이썬 클라이언트 예시가 `python/omp-rpc/` 에 이미 존재한다.

**ACP (에디터 연동)**
```sh
omp acp
```

| omp 도구 | ACP 경로 |
|---|---|
| `bash` | `terminal/create` + `terminal/output` |
| `read` | `fs/read_text_file` |
| `write` | `fs/write_text_file` |
| `edit`, `bash` | `session/request_permission` |

**TUI** — 기본 대화형 surface.

### C) 설계 레퍼런스로 배우기 (가장 큰 가치)

| 직접 만들 때 부딪히는 문제 | omp의 해법 / 참고 문서 |
|---|---|
| 모델이 편집에 자꾸 실패 | Hashline — `crates/pi-edit`, `docs/tools/` |
| 컨텍스트 초과 | 컴팩션 + Snapcompact — `docs/compaction.md` |
| 도구가 많아 모델이 혼란 | `xd://` 디바이스로 희귀 도구 은닉 |
| 모델 폭주 | TTSR + Advisor — `docs/ttsr-injection-lifecycle.md`, `docs/advisor-watchdog.md` |
| 모델별 프롬프트 튜닝 | `.omp/skills/tool-prompt-optimization` (+ `scripts/probe.ts`) |
| 스트리밍/재시도 | `docs/provider-streaming-internals.md`, `docs/non-compaction-retry-policy.md` |
| 제공사별 함정 | `docs/provider-quirks.md`, `docs/provider-compat-reference.md`, `docs/ERRATA-GPT5-HARMONY.md` |
| 새 제공사 추가 | `docs/adding-a-provider.md` |
| 메모리 설계 | `docs/memory.md`, `docs/mnemosyne-memory-backend.md` |
| 서브에이전트 조율 | `docs/agent-hub.md`, `docs/task-agent-discovery.md` |
| 훅/확장 | `docs/hooks.md`, `docs/extensions.md`, `docs/extension-loading.md` |

### 레이어 구조 (그대로 따라 할 만한 아키텍처)

```
[진입점]   TUI · ACP · RPC · SDK        ← 4개 wrapper, 동일 엔진
    ↓
[에이전트]  pi-agent-core (툴 콜링, 상태)
    ↓
[AI]       pi-ai (60개 제공사 추상화) + pi-catalog (모델 DB)
    ↓
[네이티브]  Rust crates (검색/셸/AST/PTY/데스크톱)
```

---

## 8. React / PHP로 만들 수 있는가

### omp 자체를 재현하는 것은 비현실적

| 부분 | 이유 |
|---|---|
| Rust 26만 줄 | bash 엔진, ripgrep, tree-sitter, PTY, 데스크톱 제어는 React/PHP로 포팅 불가 |
| TypeScript 68만 줄 | 재작성에 수년 소요 |
| 60개 제공사 quirks | 실전 경험 없이는 축적 불가 |
| PHP 구조 | 요청-응답 모델이라 장기 세션/스트리밍/PTY/파일워처에 부적합 |
| React(브라우저) | 샌드박스라 로컬 파일시스템/프로세스 접근 불가 |

### 대신 "위에 얹는" 방식은 매우 현실적

**A) React 웹 UI (추천)**
```
React 앱 (채팅 UI, 파일 트리, 디프 뷰어, 비용 대시보드)
   ↕ WebSocket / SSE
얇은 Node 백엔드 (omp --mode rpc-ui 를 spawn, NDJSON 중계)
   ↕
omp 엔진
```
참고 코드: `packages/collab-web/` (브라우저 게스트 클라이언트), `packages/stats/` (대시보드).
저장소 스택: React 19 + Tailwind 4 + Chart.js + lucide-react + Vite.

**B) PHP 대시보드 / 작업 큐**
```php
$descriptors = [0 => ["pipe","r"], 1 => ["pipe","w"], 2 => ["pipe","w"]];
$proc = proc_open("omp --mode rpc --no-session", $descriptors, $pipes);

fwrite($pipes[0], json_encode([
    "id" => "r1", "type" => "prompt",
    "message" => "이 PHP 파일 리팩토링해줘",
]) . "\n");

while ($line = fgets($pipes[1])) {
    $frame = json_decode($line, true);
    // 처리
}
```
권장 구조: PHP는 **웹 UI + 인증 + 작업 큐 등록**만 맡고, 실제 실행은 워커(Node/Bash)가 omp를 호출한 뒤 결과를 DB에 적재.

**C) 익스텐션 / 스킬 / MCP 서버 제작 (가장 효율적)**
```ts
export default function koreanBizExtension(pi: ExtensionAPI) {
  pi.registerTool({
    name: "check_kisa_security",
    description: "KISA 시큐어코딩 가이드 기준 취약점 점검",
    // ...
  });
  pi.registerCommand("/전자정부", async () => { /* ... */ });
}
```

### 정리

| 접근법 | 난이도 | 기간 | 추천도 |
|---|---|---|---|
| omp 전체 재현 | 극상 | 수년 | ✗ |
| React 웹 UI 래퍼 | 중 | 2~4주 | ★★★★★ |
| PHP 대시보드/큐 | 중 | 2~3주 | ★★★ |
| 익스텐션/플러그인 | 하 | 3~7일 | ★★★★★ |
| 스킬팩 | 최하 | 1~2일 | ★★★★ |
| MCP 서버 | 하 | 2~5일 | ★★★★ |

---

## 9. 수익화 아이디어

> **법적 전제**: MIT 라이선스이므로 상업적 이용·수정·재배포·SaaS화가 모두 허용된다.
> 조건은 저작권 고지와 라이선스 문구 유지. 단, `crates/vendor/brush-core` 등 vendored 코드는
> 각자의 원래 라이선스를 따르므로 `THIRD-PARTY-NOTICES.txt` 확인이 필수다.

### TIER 1 — 개인, 1~4주

#### 1. 한국 특화 스킬팩 / 플러그인 판매
- 전자정부 표준프레임워크 팩 (공공 SI)
- KISA 시큐어코딩 가이드 팩 (보안심사 대응)
- 네이버/카카오/토스 API 팩
- PG 연동 팩 (이니시스/KCP/토스페이먼츠)
- 공공데이터포털 API 팩
- 개인정보보호법 컴플라이언스 검사 팩

가격 모델: 무료(기본 3스킬) / Pro 월 ₩9,900 · 연 ₩99,000 / 팀 인당 월 ₩19,900 / 기업 연 ₩3,000,000~
**근거**: 글로벌 에이전트는 한국 특수 규정을 다루지 않는다. 진입장벽은 낮고 경쟁자는 사실상 없다.

#### 2. AI 코드리뷰 봇 SaaS
```
GitHub PR → Webhook → 내 서버 → omp -p "/review" → PR 코멘트 자동 작성 → 대시보드 시각화
```
차별점: omp `/review` 는 리뷰어 서브에이전트를 병렬로 띄워 P0~P3 우선순위 + 신뢰도 점수 + ship/no-ship 판정까지 제공.
가격: Free(공개 저장소, 월 20 PR) / Starter $19 / Team $99 / **Self-hosted $499**
**핵심**: 한국 금융·대기업은 코드 외부 반출이 금지되므로 **온프레미스 설치형**이 블루오션.

#### 3. React 기반 웹 IDE ("omp Studio")
터미널이 부담스러운 개발자/기획자/디자이너를 타깃. 백엔드는 `omp --mode rpc-ui` spawn.
`packages/collab-web/` 참고로 개발 기간 단축 가능.
가격: Free(로컬 모델 전용) / Pro $29 / Team 인당 $49

#### 4. 콘텐츠 + 교육 (현금화 최속)
| 채널 | 상품 |
|---|---|
| YouTube | AI 에이전트 완전정복 시리즈 |
| 인프런/유데미 | "오픈소스로 나만의 코딩 에이전트 만들기" |
| 전자책 | "oh-my-pi 완벽 가이드 (한국어 최초)" |
| 뉴스레터/블로그 | 기술 심화 분석 + 스폰서십 |
| 기업 세미나 | 사내 도입 컨설팅 (회당 ₩1,000,000~) |

**근거**: 한국어 자료가 거의 없고, 공식 문서 80개가 전부 영어다. 선점 효과가 크다.

### TIER 2 — 팀/투자 필요, 2~6개월

#### 5. 온프레미스 사내 AI 개발 플랫폼 (단가 최대)
타깃: 금융권 · 공공기관 · 대기업 · 방산 · 의료 (망분리 규정으로 SaaS 사용 불가)
제공: 사내 GPU + 로컬 LLM 세팅 / omp 커스터마이징(사내 컨벤션·라이브러리 스킬화) /
사내 위키·Jira·Confluence MCP 연동 / 사용량 대시보드 / 시큐어코딩 검사 / 교육 + 유지보수
가격: 구축비 5,000만~2억 + 연간 유지보수 20~30%

#### 6. 버티컬 특화 에이전트
| 버티컬 | 킬러 기능 |
|---|---|
| 레거시 마이그레이션 | JSP→React, PHP5→8, Java8→21 (`ast_edit` 활용) |
| 테스트 자동생성 | 커버리지 목표 기반 테스트 작성 |
| 보안 감사 | `security_scan` + NVD/OSV/CISA KEV |
| 쇼핑몰/WordPress | 카페24·메이크샵 커스터마이징 자동화 |
| 데이터 파이프라인 | `eval` Python 셀 기반 ETL 작성 |
| QA 자동화 | `browser` + `computer` 로 E2E 테스트 생성·실행 |

#### 7. 에이전트 마켓플레이스 운영
omp에 마켓플레이스 시스템이 이미 내장(`/marketplace`)되어 있고 카탈로그 포맷도 공개되어 있다.
할 일은 카탈로그 호스팅 + 결제 + 큐레이션. 수수료 20~30%.
리스크: 닭-달걀 문제(사용자 선확보 필요).

#### 8. 관리형 클라우드 (omp Cloud)
키 관리 대행, 팀 사용량/비용 통제, 세션 영구 보관·검색, SSO/SAML, 감사 로그. $49/인/월.
리스크: 인프라 비용이 크고 대형 벤더가 진입 가능. Tier 1 검증 후 진행 권장.

### 실행 로드맵

```
1개월차    콘텐츠로 인지도 확보 (한국어 가이드, 무료 스킬 배포)
2~3개월차  스킬팩 유료화 + 강의 오픈 → 고객 리스트 확보
4~6개월차  SaaS(코드리뷰 봇) 또는 React Studio → MRR 구축
7~12개월차 엔터프라이즈 온프레미스 영업 (앞선 결과물을 레퍼런스로)
```

### 리스크 관리

| 리스크 | 대응 |
|---|---|
| 업스트림 유료화 | MIT는 소급 적용 불가. 현재 버전은 영구 자유. 포크 유지 |
| 업스트림이 동일 상품 출시 | 한국 특화 영역으로 차별화 |
| API 비용 급증 | 로컬 모델 + `smol` 역할 라우팅 + 폴백 체인 |
| "공짜인데 왜 사지?" | 판매 대상은 소프트웨어가 아니라 시간·전문성·지원 |
| 라이선스 고지 누락 | LICENSE + THIRD-PARTY-NOTICES.txt 동봉 필수 |
| 빠른 버전 업 추적 부담 | 내 코드는 Extension API에만 의존하도록 설계 |

### 추천 TOP 3

1. 한국어 콘텐츠 + 스킬팩 — 리스크 0, 즉시 시작, 브랜딩 효과
2. React 기반 웹 UI(omp Studio) — 기존 React 역량 활용, 엔진은 무료로 사용
3. 온프레미스 SI — 단가 최대. 1·2번으로 레퍼런스 확보 후 진입

---

## 10. 참고 링크

### 저장소 / 공식
- 이 저장소: https://github.com/bmshin94/oh-my-pi
- 원본(upstream): https://github.com/can1357/oh-my-pi
- 뿌리 프로젝트(Pi): https://github.com/badlogic/pi-mono
- 공식 사이트: https://omp.sh
- 공식 문서: https://omp.sh/docs
- 도구 레퍼런스: https://omp.sh/docs/tools
- 제공사/라우팅: https://omp.sh/docs/providers
- SDK: https://omp.sh/docs/sdk
- npm: https://www.npmjs.com/package/@oh-my-pi/pi-coding-agent
- 체인지로그: https://github.com/can1357/oh-my-pi/blob/main/packages/coding-agent/CHANGELOG.md
- 라이선스(MIT): https://github.com/can1357/oh-my-pi/blob/main/LICENSE
- Discord: https://discord.gg/4NMW9cdXZa
- 관련 글(The Harness Problem): https://blog.can.ac/2026/02/12/the-harness-problem/
- Agent Client Protocol: https://github.com/zed-industries/agent-client-protocol

### 저장소 내부 주요 문서
- `README.md` — 전체 기능 개요
- `AGENTS.md` — 에이전트 행동 규칙 (28KB)
- `CONTRIBUTING.md` — 기여 가이드
- `docs/cli-reference.md` — CLI 전체 레퍼런스
- `docs/skills.md` / `docs/extensions.md` / `docs/marketplace.md` / `docs/mcp-config.md`
- `docs/secrets.md` — 시크릿 난독화
- `docs/local-models.md` — 로컬 모델 설정
- `docs/environment-variables.md` — 환경변수 전체 목록
- `docs/providers.md` / `docs/provider-quirks.md` — 제공사 통합
- `docs/sdk.md` / `docs/rpc.md` — 임베딩
- `docs/memory.md` / `docs/agent-hub.md` / `docs/advisor-watchdog.md`
- `packages/coding-agent/DEVELOPMENT.md` — 아키텍처 및 기여 가이드
