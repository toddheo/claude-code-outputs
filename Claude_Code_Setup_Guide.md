# Claude Code 고급 설정 노하우 가이드

> 작성: Todd Heo | 기준 버전: Claude Code (claude-sonnet-4-6) | 작성일: 2026-04-18

---

## 목차

1. [설정 파일 구조 이해](#1-설정-파일-구조-이해)
2. [MCP 서버 설정 완전 가이드](#2-mcp-서버-설정-완전-가이드)
3. [하네스 엔지니어링 (Hooks 설정)](#3-하네스-엔지니어링-hooks-설정)
4. [권한(Permissions) 최적화](#4-권한permissions-최적화)
5. [업무별 추천 MCP 조합](#5-업무별-추천-mcp-조합)
6. [실전 설정 파일 전체 예시](#6-실전-설정-파일-전체-예시)

---

## 1. 설정 파일 구조 이해

Claude Code는 3단계 설정 파일을 계층적으로 로딩합니다. 나중에 로딩되는 파일이 이전 설정을 덮어씁니다.

```
~/.claude/settings.json          ← 1) 사용자 전역 설정 (모든 프로젝트 적용)
.claude/settings.json            ← 2) 프로젝트 설정 (팀 공유, git commit)
.claude/settings.local.json      ← 3) 로컬 개인 오버라이드 (gitignore)
```

**MCP 서버 설정 파일 위치:**
```
~/.claude/mcp.json               ← 사용자 전역 MCP (모든 프로젝트)
.mcp.json                        ← 프로젝트별 MCP (프로젝트 루트)
```

> **핵심 원칙:** `settings.json`에는 `mcpServers` 필드를 쓸 수 없습니다. MCP 서버는 반드시 `.mcp.json` 또는 `~/.claude/mcp.json`에 정의해야 합니다.

---

## 2. MCP 서버 설정 완전 가이드

### 2-1. 기본 설정 형식 (`~/.claude/mcp.json` 또는 `.mcp.json`)

```json
{
  "mcpServers": {
    "서버이름": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-패키지명", "추가인자"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

### 2-2. 현재 설정된 MCP 서버 12종

#### ① Filesystem — 파일 시스템 직접 접근
```json
"filesystem": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem",
           "C:/Users/사용자명/Desktop",
           "C:/Users/사용자명/Documents"]
}
```
- **기능:** 지정 폴더 내 파일 읽기/쓰기/검색/이동
- **API 키:** 불필요
- **활용:** 로컬 파일을 Claude가 직접 열어서 분석, 리포트 자동 생성
- **주의:** 허용할 경로를 args에 명시적으로 지정해야 함 (보안)

#### ② Fetch — URL 웹 콘텐츠 수집
```json
"fetch": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-fetch"]
}
```
- **기능:** URL에서 HTML/텍스트/JSON 콘텐츠 가져오기
- **API 키:** 불필요
- **활용:** 경쟁사 웹사이트 내용 수집, 뉴스 기사 요약, 공공 데이터 API 호출
- **한계:** JS 렌더링 불가 (SPA 사이트는 Puppeteer 사용)

#### ③ Memory — 세션 간 지식 영속 저장
```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"]
}
```
- **기능:** Knowledge Graph 방식으로 정보를 entities/relations로 저장 및 검색
- **API 키:** 불필요
- **활용:** 프로젝트 맥락 기억, 고객사 정보 축적, 반복 작업 패턴 저장
- **저장 위치:** `~/.config/mcp-memory/` (기본값)

#### ④ Sequential Thinking — 단계적 복잡 추론
```json
"sequential-thinking": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
}
```
- **기능:** 복잡한 문제를 여러 단계 사고 스텝으로 분해하여 처리
- **API 키:** 불필요
- **활용:** 전략 수립, 다변수 의사결정, 복잡한 분석 태스크
- **특징:** Claude가 스스로 추론 단계를 관리하여 일관성 향상

#### ⑤ Puppeteer — 브라우저 자동화
```json
"puppeteer": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
}
```
- **기능:** 헤드리스 Chromium 브라우저 제어, 스크린샷, JS 렌더링 페이지 스크래핑
- **API 키:** 불필요 (Node.js + Chrome 필요)
- **활용:** 로그인 필요 페이지 자동화, 동적 웹사이트 데이터 수집, UI 테스트
- **주의:** 처음 실행 시 Chromium 다운로드 (~170MB)

#### ⑥ GitHub — GitHub 저장소 관리
```json
"github": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxxxxxxxxxx"
  }
}
```
- **기능:** PR 생성/리뷰, 이슈 관리, 코드 검색, 저장소 파일 접근
- **API 키:** 필요 → [github.com/settings/tokens](https://github.com/settings/tokens) (Fine-grained 권장)
- **활용:** 코드 리뷰 자동화, PR 설명 초안 생성, 이슈 트래킹
- **필요 권한:** `repo`, `issues`, `pull_requests`

#### ⑦ Brave Search — 실시간 웹 검색
```json
"brave-search": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-brave-search"],
  "env": {
    "BRAVE_API_KEY": "BSA_xxxxxxxxx"
  }
}
```
- **기능:** 실시간 웹 검색 (Google 대체), 뉴스 검색, 이미지 검색
- **API 키:** 필요 → [brave.com/search/api](https://brave.com/search/api) (무료: 2,000건/월)
- **활용:** 시장 조사, 경쟁사 동향 파악, 최신 뉴스 수집
- **장점:** Google과 달리 광고 없는 독립 인덱스, 프라이버시 강화

#### ⑧ Playwright — 안정적 브라우저 자동화 (Puppeteer 대체)
```json
"playwright": {
  "command": "npx",
  "args": ["-y", "@executeautomation/playwright-mcp-server"]
}
```
- **기능:** Chromium/Firefox/WebKit 크로스브라우저 제어, JS SPA 스크래핑
- **API 키:** 불필요
- **활용:** Puppeteer보다 안정적, 동적 웹사이트 데이터 수집, 로그인 자동화
- **장점:** Puppeteer 대비 병렬 처리·에러 복구 우수

#### ⑨ SQLite — 로컬 데이터베이스 쿼리
```json
"sqlite": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-sqlite", "C:/Users/사용자명/Documents/data.db"]
}
```
- **기능:** SQLite DB 파일 쿼리, 테이블 생성/조회, CSV 데이터 분석
- **API 키:** 불필요
- **활용:** 헤비유저 데이터 분석, 재무 모델 데이터 저장, 로컬 집계

#### ⑩ Time — 시간대 변환 및 날짜 계산
```json
"time": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-time"]
}
```
- **기능:** 타임존 변환, D-Day 계산, 현재 시각 조회
- **API 키:** 불필요
- **활용:** 글로벌 BD 미팅 스케줄, 프로젝트 데드라인 계산

#### ⑪ Slack — 팀 커뮤니케이션 자동화
```json
"slack": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-slack"],
  "env": {
    "SLACK_BOT_TOKEN": "xoxb-YOUR_TOKEN_HERE",
    "SLACK_TEAM_ID": "YOUR_TEAM_ID_HERE"
  }
}
```
- **기능:** 채널 메시지 조회·발송, DM, 파일 공유, 채널 검색
- **API 키:** 필요 → Slack API 앱 생성 후 Bot Token 발급
- **설정 방법:**
  1. [api.slack.com/apps](https://api.slack.com/apps) → Create New App
  2. OAuth & Permissions → Bot Token Scopes: `channels:history`, `chat:write`, `channels:read`
  3. Install to Workspace → `xoxb-...` 토큰 복사
  4. Team ID: Slack 워크스페이스 URL에서 확인 (`T0XXXXXXX`)

#### ⑫ Google Drive — 구글 드라이브 파일 접근
```json
"gdrive": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-gdrive"],
  "env": {
    "GDRIVE_CREDENTIALS_FILE": "C:/Users/사용자명/.claude/gdrive-credentials.json"
  }
}
```
- **기능:** Google Drive 파일 검색·읽기, Docs/Sheets 내용 접근
- **API 키:** OAuth 2.0 설정 필요 (1회)
- **설정 방법:**
  1. [Google Cloud Console](https://console.cloud.google.com) → 새 프로젝트 생성
  2. Google Drive API 활성화
  3. 사용자 인증 정보 → OAuth 2.0 클라이언트 ID 생성 (데스크톱 앱)
  4. JSON 다운로드 → `~/.claude/gdrive-credentials.json`으로 저장
  5. 최초 실행 시 브라우저 OAuth 승인 팝업 뜸

### 2-3. 추가 추천 MCP 서버

#### BD/전략 업무에 유용한 MCP

| 서버 | 패키지 | 기능 | API 키 |
|------|--------|------|--------|
| **Playwright** | `@executeautomation/playwright-mcp-server` | Puppeteer보다 안정적인 브라우저 자동화, 크로스브라우저 | ❌ |
| **Excel/Spreadsheet** | `@modelcontextprotocol/server-excel` | Excel/CSV 파일 직접 읽기·쓰기·수식 계산 | ❌ |
| **SQLite** | `@modelcontextprotocol/server-sqlite` | 로컬 DB 쿼리, 데이터 분석 | ❌ |
| **Slack** | `@modelcontextprotocol/server-slack` | Slack 메시지 조회·발송, 채널 검색 | ✅ OAuth |
| **Google Drive** | `@modelcontextprotocol/server-gdrive` | GDrive 파일 검색·읽기 | ✅ OAuth |
| **Perplexity** | `mcp-server-perplexity` | 더 정확한 실시간 리서치 (Brave보다 질 높음) | ✅ |
| **Time** | `@modelcontextprotocol/server-time` | 타임존 변환, 날짜 계산, 일정 관련 | ❌ |
| **Everything** | `@modelcontextprotocol/server-everything` | Windows 전체 파일 초고속 검색 | ❌ |

#### Playwright 설정 예시 (Puppeteer 대체 권장)
```json
"playwright": {
  "command": "npx",
  "args": ["-y", "@executeautomation/playwright-mcp-server"]
}
```

#### Perplexity 설정 예시 (시장 조사 최강)
```json
"perplexity": {
  "command": "npx",
  "args": ["-y", "mcp-server-perplexity"],
  "env": {
    "PERPLEXITY_API_KEY": "pplx-xxxxxxxxx"
  }
}
```

---

## 3. 하네스 엔지니어링 (Hooks 설정)

하네스(Harness)란 Claude Code의 생명주기 이벤트에 자동 실행되는 커맨드를 붙여 워크플로우를 자동화하는 것입니다.

### 3-1. Hook 이벤트 종류

| 이벤트 | 타이밍 | 주요 용도 |
|--------|--------|-----------|
| `PreToolUse` | 툴 실행 **전** | 위험 명령 차단, 명령 로깅 |
| `PostToolUse` | 툴 실행 **후** | 파일 변경 로깅, 자동 포맷팅, 테스트 실행 |
| `PostToolUseFailure` | 툴 실행 **실패 후** | 에러 알림, 롤백 |
| `Stop` | Claude 응답 **완료 후** | 결과 동기화, 알림 발송 |
| `SessionStart` | 세션 **시작 시** | 컨텍스트 주입, 초기화 |
| `PreCompact` | 컨텍스트 압축 **전** | 중요 내용 보존 처리 |
| `UserPromptSubmit` | 사용자 입력 **제출 시** | 프롬프트 전처리, 검증 |

### 3-2. 현재 설정된 Hooks

#### Hook 1: 세션 종료 시 자동 동기화 (기존)
```json
"Stop": [{
  "matcher": "",
  "hooks": [{
    "type": "command",
    "command": "powershell -ExecutionPolicy Bypass -File \"~/.claude/scripts/sync.ps1\" -ProjectDir \"%PROJECT_DIR%\""
  }]
}]
```
- 매 세션 종료(응답 완료) 시 PowerShell 동기화 스크립트 실행

#### Hook 2: 파일 수정 이력 자동 로깅 (신규)
```json
"PostToolUse": [{
  "matcher": "Write|Edit",
  "hooks": [{
    "type": "command",
    "command": "jq -r '.tool_input.file_path // empty' | { read -r f; [ -n \"$f\" ] && echo \"$(date '+%Y-%m-%d %H:%M:%S') EDIT  $f\" >> \"C:/Users/사용자/.claude/file-history/changes.log\"; } 2>/dev/null || true",
    "shell": "bash",
    "async": true
  }]
}]
```
- Write/Edit 툴 실행 후 수정된 파일 경로를 타임스탬프와 함께 `changes.log`에 기록
- `async: true`로 비동기 실행 (Claude 응답 블로킹 없음)
- 로그 위치: `~/.claude/file-history/changes.log`

#### Hook 3: Bash 명령 이력 자동 로깅 (신규)
```json
"PreToolUse": [{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "jq -r '.tool_input.command // empty' | { read -r cmd; [ -n \"$cmd\" ] && echo \"$(date '+%Y-%m-%d %H:%M:%S') $ $cmd\" >> \"C:/Users/사용자/.claude/bash-history.log\"; } 2>/dev/null || true",
    "shell": "bash",
    "async": true
  }]
}]
```
- Bash 툴 실행 전 실행될 명령어를 `bash-history.log`에 기록
- 작업 감사(Audit Trail) 용도, 반복 패턴 파악에 유용

### 3-3. 추가 추천 Hook 패턴

#### 패턴 A: 마크다운 파일 생성 후 Notion 자동 업로드 (고급)
```json
"PostToolUse": [{
  "matcher": "Write",
  "hooks": [{
    "type": "command",
    "command": "jq -r '.tool_input.file_path // empty' | { read -r f; [[ \"$f\" == *.md ]] && echo \"[AUTO] $f 생성됨 → Notion 업로드 검토\" >> ~/.claude/md-created.log; } 2>/dev/null || true",
    "shell": "bash",
    "async": true
  }]
}]
```

#### 패턴 B: 위험 Bash 명령 차단
```json
"PreToolUse": [{
  "matcher": "Bash",
  "hooks": [{
    "type": "prompt",
    "prompt": "다음 bash 명령이 데이터를 삭제하거나 되돌리기 어려운 작업을 수행하는지 확인하세요: $ARGUMENTS. 'rm -rf', 'format', 'DROP TABLE' 등이 포함되어 있으면 {\"continue\": false, \"stopReason\": \"위험한 명령어 감지됨\"}을 반환하세요."
  }]
}]
```

#### 패턴 C: 세션 시작 시 업무 컨텍스트 주입
```json
"SessionStart": [{
  "matcher": "",
  "hooks": [{
    "type": "command",
    "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"SessionStart\", \"additionalContext\": \"사용자는 아임인 Global BD Director입니다. 주요 업무: 헤비유저 관리, 군장병 채널 BD, Notion 기반 전략 문서 작성. 기술 수준: 비개발자이나 Claude Code 숙련 사용자.\"}}'",
    "shell": "bash"
  }]
}]
```

---

## 4. 권한(Permissions) 최적화

### 4-1. defaultMode 옵션 비교

| 모드 | 동작 | 추천 상황 |
|------|------|-----------|
| `default` | 모든 작업 승인 요청 | 처음 사용, 보수적 환경 |
| `acceptEdits` | 파일 편집 자동 승인, bash는 물어봄 | **일반 업무 (현재 설정)** |
| `auto` | AI가 판단해서 자동 처리 | 빠른 반복 작업 |
| `dontAsk` | 모든 작업 자동 승인 | 신뢰된 환경에서 최대 효율 |

### 4-2. 현재 설정된 allow 규칙

```json
"allow": [
  "Bash(git *)",      // git 명령 전체 자동 승인
  "Bash(npm *)",      // npm 명령 전체
  "Bash(npx *)",      // npx 명령 전체
  "Bash(node *)",     // node 실행
  "Bash(jq *)",       // JSON 처리
  "Bash(ls *)",       // 디렉토리 조회
  "Bash(cat *)",      // 파일 읽기
  "Read",             // 모든 파일 읽기
  "Glob",             // 파일 패턴 검색
  "Grep",             // 파일 내용 검색
  // Notion MCP 11개 툴 전체
  "mcp__claude_ai_Notion__notion-search",
  "mcp__claude_ai_Notion__notion-fetch",
  // ... (생략)
]
```

### 4-3. 권한 규칙 문법

```
"ToolName"                  → 해당 툴 전체 허용
"ToolName(exact command)"   → 정확히 일치하는 명령만 허용
"ToolName(prefix *)"        → prefix로 시작하는 모든 명령 허용
"mcp__서버명__툴명"          → MCP 특정 툴 허용
```

---

## 5. 업무별 추천 MCP 조합

### BD/전략 기획 (현재 업무)
```
필수: fetch + brave-search + sequential-thinking + memory
선택: playwright (웹 스크래핑) + perplexity (심층 리서치)
```

### 데이터 분석
```
필수: filesystem + sqlite + sequential-thinking
선택: excel + puppeteer (웹 데이터 수집)
```

### 개발/코딩
```
필수: filesystem + github + sequential-thinking
선택: fetch (API 문서) + brave-search (스택오버플로우)
```

### 문서/콘텐츠 작성
```
필수: filesystem + memory + sequential-thinking
선택: fetch (레퍼런스 수집) + brave-search (최신 정보)
```

---

## 6. 실전 설정 파일 전체 예시

### `~/.claude/settings.json` (전역 설정)

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "powershell -ExecutionPolicy Bypass -File \"~/.claude/scripts/sync.ps1\" -ProjectDir \"%PROJECT_DIR%\""
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path // empty' | { read -r f; [ -n \"$f\" ] && echo \"$(date '+%Y-%m-%d %H:%M:%S') EDIT  $f\" >> \"~/.claude/file-history/changes.log\"; } 2>/dev/null || true",
        "shell": "bash",
        "async": true
      }]
    }],
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command // empty' | { read -r cmd; [ -n \"$cmd\" ] && echo \"$(date '+%Y-%m-%d %H:%M:%S') $ $cmd\" >> \"~/.claude/bash-history.log\"; } 2>/dev/null || true",
        "shell": "bash",
        "async": true
      }]
    }]
  },
  "permissions": {
    "allow": [
      "Bash(git *)", "Bash(npm *)", "Bash(npx *)", "Bash(node *)",
      "Bash(jq *)", "Bash(ls *)", "Bash(cat *)",
      "Read", "Glob", "Grep",
      "mcp__claude_ai_Notion__notion-search",
      "mcp__claude_ai_Notion__notion-fetch",
      "mcp__claude_ai_Notion__notion-create-pages",
      "mcp__claude_ai_Notion__notion-update-page",
      "mcp__claude_ai_Notion__notion-get-users",
      "mcp__claude_ai_Notion__notion-get-teams",
      "mcp__claude_ai_Notion__notion-create-comment",
      "mcp__claude_ai_Notion__notion-get-comments",
      "mcp__claude_ai_Notion__notion-create-database",
      "mcp__claude_ai_Notion__notion-duplicate-page",
      "mcp__claude_ai_Notion__notion-move-pages"
    ],
    "defaultMode": "acceptEdits"
  },
  "enableAllProjectMcpServers": true
}
```

### `~/.claude/mcp.json` (전역 MCP 서버)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem",
               "C:/Users/사용자명/Desktop",
               "C:/Users/사용자명/Documents"]
    },
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    },
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "YOUR_GITHUB_TOKEN_HERE"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "YOUR_BRAVE_API_KEY_HERE"
      }
    }
  }
}
```

---

## 부록: 트러블슈팅

### MCP 서버가 안 뜰 때
1. `claude mcp list` 로 등록 여부 확인
2. `claude --debug` 로 실행하면 MCP 연결 로그 확인 가능
3. Node.js 18+ 설치 여부 확인 (`node --version`)
4. npx 캐시 문제: `npx clear-npx-cache` 후 재시도

### Hook이 실행 안 될 때
1. `/hooks` 명령으로 hooks 설정 재로딩
2. `jq` 설치 확인 (`jq --version`)
3. settings.json JSON 문법 검증: `jq . ~/.claude/settings.json`
4. `async: true`인 hook은 실패해도 무시됨 (의도적 설계)

### 권한 프롬프트가 계속 뜰 때
- 해당 명령 패턴을 `permissions.allow` 배열에 추가
- `Bash(명령어 *)`  형식으로 prefix 매칭 활용
