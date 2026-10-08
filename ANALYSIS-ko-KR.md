# code-review-graph 전수조사 분석 보고서 (한국어)

> 작성: Claude Code 세션 / 2026-10-08
> 대상 버전: **v2.3.9** (커밋 `2362e73`)

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/code-review-graph |
| **원본 (upstream)** | https://github.com/tirth8205/code-review-graph |
| **PyPI 패키지** | https://pypi.org/project/code-review-graph/ |
| **공식 웹사이트** | https://code-review-graph.com |
| **Discord** | https://discord.gg/3p58KXqGFN |
| **문서 인덱스** | https://github.com/tirth8205/code-review-graph/blob/main/docs/INDEX.md |
| **CHANGELOG** | https://github.com/tirth8205/code-review-graph/blob/main/CHANGELOG.md |
| **한국어 README** | [README.ko-KR.md](README.ko-KR.md) |

---

## 1. 이게 뭐 하는 건가

> **"코드베이스를 미리 지도(지식 그래프)로 만들어 두고, AI 코딩 도구에게 전체 코드가 아니라 '필요한 조각만' 전달하는 엔진"**

### 문제
AI 코딩 도구는 변경 하나를 리뷰하려고 코드베이스 상당 부분을 다시 읽는다.
flask 저장소 전체 = **143,594 토큰**.

### 해결 — 4단계 파이프라인
```
① 저장소 → Tree-sitter 파싱 (AST 추출)
② → SQLite 그래프 저장 (.code-review-graph/graph.db, WAL 모드)
③ → 변경 파일의 blast radius(폭발 반경) 추적
④ → AI에게 "읽어야 할 최소 파일 집합"만 전달
```
결과: 같은 질문에 **2,196 토큰** → **약 65배 절감**

### 쉬운 비유
- 🏢 **빌딩 설계도**: 1,000개 방을 전부 열어보는 인턴 vs 설계도 보고 3개 방만 확인하는 인턴
- 🗺️ **지하철 노선도**: 위성사진 전부 보기 vs 노선도 보고 "2호선" 한 마디
- 🍜 **라면 그램 과금**: 143인분 먹이기 vs 2인분 먹이기 (게다가 AI도 배부르면 둔해진다 — "lost in the middle")

---

## 2. 기본 정보

| 항목 | 내용 |
|---|---|
| 원저작자 | `tirth8205` (이 저장소는 `bmshin94` 포크) |
| 버전 | v2.3.9 |
| 라이선스 | **MIT** (상업적 이용·수정·재배포·클로즈드소스 판매 모두 가능) |
| 언어 | Python 3.10+ |
| 코드 규모 | 핵심 패키지 약 **63,800줄** (`parser.py` 단독 776KB) |
| 테스트 | pytest 모듈 **171개** |
| 브랜치 모델 | `staging`(기본) → `testing` → `main` → 태그 → PyPI |
| 특징 | TrendShift 뱃지 보유, 다국어 README 5종(영/중/일/**한**/힌디) |

---

## 3. 그래프 구조 (실제 SQLite DB)

**노드 9종** — `File`, `Class`, `Function`, `Test`, `Type`, `Endpoint`, `Scheduler`, `ConfigProperty`, `Event`

**엣지 7종** — `CALLS`, `IMPORTS_FROM`, `CONTAINS`, `TESTED_BY`, `REFERENCES`, `DEPENDS_ON_CONFIG`, `EXTRACTED`

**쿼리 패턴 13종** — `callers_of`, `callees_of`, `imports_of`, `importers_of`, `tests_for`, `inheritors_of`, `children_of`, `consumers_of`, `publishers_of`, `listeners_of`, `handlers_of`, `endpoints_for`, `triggers_of`

현재 스키마 버전 **13** (VS Code 확장의 `SUPPORTED_SCHEMA_VERSION`과 일치해야 하며 CI가 검사)

---

## 4. 폴더 전수조사

### `code_review_graph/` — 핵심 엔진

| 파일 | 크기 | 역할 | 쉬운 이름 |
|---|---|---|---|
| `parser.py` | 776KB | Tree-sitter 다국어 파서 | 📖 번역가 |
| `graph.py` | 164KB | SQLite 그래프 스토어 + impact 분석 | 🗄️ 창고 관리인 |
| `incremental.py` | 139KB | 전체 빌드 / Git·SVN 변경감지 / 증분 업데이트 / watch | 👀 감시원 |
| `visualization.py` | 120KB | D3.js 인터랙티브 그래프 HTML | 🎨 화가 |
| `skills.py` | 119KB | 16개 플랫폼 자동 설치기 | 🔧 설치 기사 |
| `cli.py` | 99KB | `code-review-graph` 명령 (22개 서브커맨드) | 💻 조종석 |
| `daemon.py` | 61KB | 멀티 저장소 감시 데몬 (`crg-daemon`) | 🛰️ 관제탑 |
| `main.py` | 58KB | **FastMCP 서버** — 툴 30개 + 프롬프트 5개 | 📞 교환원 |
| `embeddings.py` | 56KB | 임베딩 5공급자 (Local/OpenAI/Google/MiniMax/Voyage) | 🧠 의미 검색기 |
| `uninstall.py` | 44KB | 설치한 것만 정확히 되돌리기 (JSONC 주석 보존) | 🧹 청소부 |
| `changes.py` | 40KB | 리스크 점수 변경 분석 | 🚦 위험 판정관 |
| `search.py` | 39KB | FTS5 + 벡터 하이브리드 검색 | 🔍 검색기 |
| `communities.py` | 38KB | Leiden(igraph) 모듈 클러스터링 | 🏘️ 구역 나누기 |
| `refactor.py` | 38KB | 리네임 미리보기 / 데드코드 | 🏗️ 리모델러 |
| `flows.py` | 26KB | 실행 흐름 + 중요도 | 🌊 동선 추적기 |
| `migrations.py` | 26KB | 스키마 마이그레이션 (현재 v13) | 📐 설계 변경 이력 |
| `hints.py` | 12KB | `next_tool_suggestions` | 🧭 길안내 |
| `uncertainty.py` | 16KB | 빈 결과에 `confidence` 노트 | 🤔 겸손 장치 |
| `context_savings.py` | 11KB | 토큰 절감 추정 메타데이터 | 📊 계산기 |

**후처리 리졸버 10종**: `python_resolver`, `jedi_resolver`, `spring_resolver`(Spring DI), `event_resolver`, `temporal_resolver`, `config_keys`, `scoped_resolver`(PHP/Rust/C#), `rescript_resolver`, `hcl_resolver`(Terraform), `tsconfig_resolver`(TS 경로 alias)

### 그 외 폴더

| 폴더/파일 | 내용 |
|---|---|
| `code_review_graph/tools/` | MCP 툴 30개 구현 (5,935줄) — `query.py` 1,454줄, `review.py` 1,121줄 등 |
| `skills/` | AI 스킬 7개: `build-graph`, `review-changes`, `review-delta`, `review-pr`, `explore-codebase`, `debug-issue`, `refactor-safely` |
| `hooks/` | `SessionStart`(상태 체크), `PostToolUse(Write\|Edit\|Bash)`(자동 증분 업데이트), Git `pre-commit` |
| `code-review-graph-vscode/` | VS Code 확장 (TypeScript, `graph.db` 직접 읽기) |
| `action.yml` | GitHub Action — PR에 리스크 점수 리뷰 코멘트 자동 작성, **local-first(코드 외부 전송 없음)** |
| `evaluate/` + `eval/` | 6개 저장소 × 13커밋 고정 SHA 재현 가능 벤치마크 (CSV 커밋됨) |
| `docs/` | 문서 13개 (USAGE, COMMANDS, FAQ, TROUBLESHOOTING, GITHUB_ACTION, REPRODUCING, ROADMAP, architecture, schema, CUSTOM_LANGUAGES, FEATURES, INDEX, LEGAL) |
| `diagrams/` | 다이어그램 PNG 9장 + 데모 GIF + 생성 스크립트 |
| `tests/` | 171개 모듈, `fixtures/` 67개 언어별 샘플 |
| `scripts/` | `auto_promote.py`, `promotion_gate.py`, `render_pr_comment.py` 등 |

---

## 5. 실측 벤치마크

| 저장소 | 스냅샷 SHA | 전체 코퍼스 토큰 | 평균 그래프 토큰 | 절감률 |
|---|---|---:|---:|---:|
| fastapi | `22381558` | 948,793 | 2,653 | **357.6x** |
| flask | `a29f88ce` | 143,594 | 2,196 | **65.4x** |
| code-review-graph | `84bde354` | 208,821 | 3,190 | **65.5x** |
| gin | `5c00df8a` | 166,868 | 2,766 | **60.3x** |
| httpx | `b55d4635` | 142,356 | 2,661 | **53.5x** |
| express | `b4ab7d65` | 136,052 | 3,936 | **34.6x** |

- 중위값 **약 63배** 절감, Impact 정확도 평균 **F1 0.69**, 멀티홉 검색 점수 **0.909**
- 성능: 3,000파일 콜드 빌드 **약 40초**, 2파일 증분 업데이트 **약 2.5초**(중 1.4초가 프로세스 시작)
- 재현 레시피: `docs/REPRODUCING.md` (고정 SHA, 고정 Leiden 시드, 결정론적 CPU 임베딩)

> ⚠️ "65배"는 **전체 코퍼스 읽기 기준** 상한선이다. 실제 에이전트는 grep 후 상위 파일만 읽으므로,
> 이를 측정하는 `agent_baseline` 벤치마크가 별도로 있으나 **공식 수치는 아직 미발표**다.
> 마케팅 시 이 조건을 반드시 명시해야 신뢰를 유지할 수 있다.

---

## 6. 지원 범위

**AI 도구 16종 자동 설치**
Claude Code, Codex, Cursor, Windsurf, Zed, Continue, OpenCode, Antigravity, Gemini CLI, Qwen Code, Kiro, Qoder, GitHub Copilot, Copilot CLI, Hermes Agent, CodeBuddy Code

**언어 40종+**
Python, JS/TS/TSX, Go, Rust, Java, C/C++, C#, VB.NET, Ruby, Kotlin, Swift, PHP, Scala, Solidity, Dart, R, Perl, Lua/Luau, Objective-C, Shell, Elixir, Zig, PowerShell, Julia, ReScript, GDScript, Nix, Verilog/SystemVerilog, SQL, Terraform/OpenTofu, Ansible YAML, Spring Boot 설정, Vue/Svelte SFC, Astro, Jupyter/Databricks 노트북, Perl XS
➕ `.code-review-graph/languages.toml` 로 **포크 없이 커스텀 언어 추가** 가능

---

## 7. 사용자에게 주는 이득

1. 💸 **AI 코딩 비용 절감** — 토큰 과금이 최대 1/65
2. 🎯 **AI 답변 정확도 향상** — 컨텍스트를 꽉 채우지 않으므로 품질 ↑
3. 🚨 **"이거 고치면 뭐 깨지나" 즉답** — `get_impact_radius_tool`
4. 🧪 **테스트 공백 자동 발견** — `TESTED_BY` 엣지 부재 + v2.3.9의 "호출자 경유 커버리지" 구분
5. 👶 **신규 입사자 온보딩** — 아키텍처 개요 + 커뮤니티별 마크다운 위키 자동 생성
6. 🔒 **보안·프라이버시** — local-first, 텔레메트리 0, 클라우드 기본값 0
7. 🏗️ **AI 에이전트 설계 교과서** — 컨텍스트 엔지니어링 레퍼런스 구현체

### 한계 (솔직하게)
- 📈 첫 빌드 시간/디스크 소모
- 🔍 정적 분석 한계 — 동적 호출(`getattr`, 리플렉션, 메타프로그래밍) 미탐지
- 📉 작은 단일 파일 변경은 그래프 메타데이터 고정 오버헤드로 **오히려 토큰이 더 들 수 있음** (문서에 명시됨)
- 🌀 Python 3.10+ 필수, igraph/sentence-transformers는 선택 의존성

---

## 8. 설치 및 사용법

### 최소 설치 (3줄)
```bash
pip install code-review-graph     # 또는 pipx install code-review-graph
code-review-graph install         # AI 도구 자동 감지 → 설정 주입
code-review-graph build           # 그래프 빌드
```
→ 이후 **에디터 재시작** 필수

### 플랫폼별 설정 파일

| 플랫폼 | `--platform` | 설정 파일 |
|---|---|---|
| Claude Code | `claude-code` | `.mcp.json` + `.claude/settings.json` |
| Codex | `codex` | `~/.codex/config.toml` + `hooks.json` |
| Cursor | `cursor` | `.cursor/mcp.json` |
| Windsurf | `windsurf` | `~/.codeium/windsurf/mcp_config.json` |
| Zed | `zed` | `~/.config/zed/settings.json` (macOS는 Application Support) |
| Continue | `continue` | `~/.continue/config.json` |
| OpenCode | `opencode` | `opencode.jsonc` |
| Antigravity | `antigravity` | `~/.gemini/antigravity/mcp_config.json` |
| Gemini CLI | `gemini-cli` | `.gemini/settings.json` |
| Qwen Code | `qwen` | `~/.qwen/settings.json` |
| Kiro | `kiro` | `.kiro/settings/mcp.json` |
| Qoder | `qoder` | `.qoder/mcp.json` |
| GitHub Copilot | `copilot` | `.vscode/mcp.json` |
| Copilot CLI | `copilot-cli` | `~/.copilot/mcp-config.json` |
| CodeBuddy | `codebuddy` | `.mcp.json` + `CODEBUDDY.md` + `.codebuddy/` |
| Hermes Agent | `hermes` | `~/.hermes/config.yaml` |

### install이 하는 일
1. MCP 서버 엔트리 작성 (`uvx code-review-graph serve`)
2. 훅 설치 (파일 수정 시 자동 그래프 업데이트)
3. 스킬 7개 복사 (지원 플랫폼)
4. 룰 파일에 지침 추가 (`CLAUDE.md`, `AGENTS.md` 등)
5. Git `pre-commit` 훅 (Codex / Claude Code / Qoder) — 커밋 전 `update` + `detect-changes --brief`

스킵: `--no-hooks`, `--no-skills`, `--no-instructions`, 미리보기: `--dry-run`

### 제거
```bash
code-review-graph uninstall --dry-run     # 미리보기
code-review-graph uninstall               # 확인 후 적용
code-review-graph uninstall --yes         # 즉시
code-review-graph uninstall --keep-data   # 설정만 제거, DB 유지
code-review-graph uninstall --all-repos   # 등록된 모든 저장소
```
JSONC 주석 보존, 다른 MCP 서버는 건드리지 않음, 원자적 교체(실패 시 원본 보존)

### 주요 CLI 명령
```bash
# 그래프 관리
code-review-graph build / update / status / watch / init
code-review-graph update --brief                      # 업데이트 + 리스크 패널
code-review-graph forget PATH                         # 특정 파일 제거

# 리뷰·분석
code-review-graph detect-changes --brief              # HEAD~1 대비 (읽기 전용)
code-review-graph detect-changes --brief --base main  # main 머지베이스 대비
code-review-graph detect-changes --brief --verify     # tiktoken 실측 검증
code-review-graph impact <target> / query <pattern> <target> / search "질의"
code-review-graph large-functions / refactor

# 구조 파악
code-review-graph architecture / communities / community <id> / flows / flow <id>

# 출력
code-review-graph visualize                           # D3.js HTML
code-review-graph visualize --format json|graphml|svg|obsidian|cypher
code-review-graph visualize --seed-symbol X --depth 2 # 부분 그래프
code-review-graph wiki                                # 마크다운 위키

# 기타
code-review-graph serve / serve --http
code-review-graph register /path --alias mylib / repos
code-review-graph enrich / embed / eval
crg-daemon                                            # 멀티 저장소 데몬
```

### AI에게 말로 쓰는 법
```
"이 프로젝트 코드 리뷰 그래프 빌드해줘"
"내 최근 변경사항 리스크 점수로 리뷰해줘"
"이 프로젝트 아키텍처 보여줘"
"login 함수 고치면 뭐가 영향받아?"
```
슬래시 커맨드: `/code-review-graph:build-graph`, `:review-delta`, `:review-pr`

### 선택 의존성 (extras)
```bash
pip install "code-review-graph[embeddings]"          # 로컬 임베딩 (의미 검색)
pip install "code-review-graph[google-embeddings]"   # Gemini
pip install "code-review-graph[communities]"         # igraph Leiden
pip install "code-review-graph[enrichment]"          # jedi 보강
pip install "code-review-graph[wiki]"                # ollama
pip install "code-review-graph[eval]"                # 벤치마크
pip install "code-review-graph[all]"                 # 전부
```

---

## 9. 플러그인? 스킬? MCP?

### 결론: **MCP 서버가 본체이고, 스킬·훅·CLI·확장·Action을 모두 품은 패키지**

```
🏛️ 본체: MCP 서버 (FastMCP) — main.py, 툴 30개 + 프롬프트 5개
📋 레이어 2: 스킬 7개 (skills/*/SKILL.md) — AI가 읽는 작업 순서 매뉴얼
⚙️ 레이어 3: 훅 — SessionStart / PostToolUse / Git pre-commit
💻 레이어 4: CLI — AI 없이 터미널 단독 사용 가능 (22개 서브커맨드)
🧩 레이어 5: VS Code 확장 + GitHub Action
```

| 질문 | 답 |
|---|---|
| MCP야? | ✅ 예, 이게 본체 (`serve`로 FastMCP 구동, 툴 30개) |
| 스킬이야? | ✅ 예, 7개 포함 |
| 플러그인이야? | 🟡 Claude Code 플러그인 마켓플레이스 형식은 아님. 다만 **VS Code 확장은 진짜 플러그인** |
| 훅? | ✅ 3종 |
| CLI 단독? | ✅ 완전 독립 실행 가능 |

**MCP vs 스킬**: MCP = 🔌 능력(공구함) / 스킬 = 📖 지침(설명서). 둘은 세트다.

---

## 10. API 토큰 필요한가?

### **기본 사용은 API 키 0개. 완전 무료·완전 로컬 (local-first)**

| 기능 | API 키 | 비용 |
|---|:---:|---|
| 그래프 빌드/업데이트 | ❌ | 무료 |
| 40개 언어 파싱 (Tree-sitter) | ❌ | 무료 |
| MCP 툴 30개 전부 | ❌ | 무료 |
| 영향 반경·리스크 점수 | ❌ | 무료 |
| FTS5 키워드 검색 | ❌ | 무료 |
| 시각화·위키·내보내기 | ❌ | 무료 |
| 커뮤니티 탐지 (igraph) | ❌ | 무료 |
| 로컬 의미 검색 (sentence-transformers) | ❌ | 무료 (CPU/디스크만) |
| **클라우드 임베딩 (선택)** | ⚠️ 필요 | 유료 |

### 클라우드 임베딩 환경변수 (유일한 유료 경로)
```bash
CRG_OPENAI_API_KEY / CRG_OPENAI_BASE_URL / CRG_OPENAI_MODEL   # OpenAI 호환
GOOGLE_API_KEY                                                 # Gemini
VOYAGE_API_KEY / CRG_VOYAGE_MODEL                              # Voyage AI
MINIMAX_API_KEY                                                # MiniMax
CRG_ACCEPT_CLOUD_EMBEDDINGS=1                                  # 명시 동의 필수
```
→ 클라우드 공급자 사용 시 **egress 경고 출력** + 명시 동의 변수 필요 (사내 코드 유출 방지)

### 보안 불변식 (CLAUDE.md 명시)
- `eval()`, `exec()`, `pickle`, `yaml.unsafe_load()` 금지
- `shell=True` 금지
- SQL은 항상 `?` 파라미터 바인딩
- API 키는 환경변수에서만 읽음
- `serve --http`는 Host/Origin 검증 (`http_origin_guard.py`)
- D3.js 번들 + SRI 해시
- CI에 bandit 보안 스캔 포함

> 💡 "API 토큰"이 Claude/GPT 사용 토큰을 뜻한다면 — **이 도구는 토큰을 쓰는 게 아니라 아껴주는 도구**다.

---

## 11. AI 에이전트 구축에 도움이 되는가

### **YES — 두 방향 모두**

### 방향 ① 그대로 사용

| 에이전트 종류 | 사용 툴 |
|---|---|
| 코드 리뷰 봇 | `detect_changes_tool` + `get_review_context_tool` |
| 버그 수정 에이전트 | `query_graph_tool(callers_of)` |
| 문서 자동 생성 봇 | `generate_wiki_tool` + `get_architecture_overview_tool` |
| 테스트 생성 에이전트 | `get_knowledge_gaps_tool` |
| 리팩토링 에이전트 | `refactor_tool` + `apply_refactor_tool` |
| 온보딩 에이전트 | `get_suggested_questions_tool` |
| 코드 검색 에이전트 | `semantic_search_nodes_tool` |
| 멀티레포 에이전트 | `cross_repo_search_tool` |

MCP 호환이므로 Claude Agent SDK, LangGraph, CrewAI, OpenAI Agents SDK 등 어디든 붙는다.

### 방향 ② 읽어서 배우기 — 이게 더 가치 있다

**컨텍스트 엔지니어링 레퍼런스 구현체**로서 배울 포인트 7개:

1. **토큰 예산 설계** — 스킬 문서에 "툴 5회·800토큰" 예산을 명시적으로 박음
2. **응답 바운딩 계약** — v2.3.8에서 `get_affected_flows`가 **247,000 토큰**을 뱉던 버그 발견 → 30개 툴 전체 감사 → 통일 계약(`total`/`truncated`/`summary`) + `tests/test_token_budget.py`가 CI에서 툴별 예산표 고정
3. **`detail_level` 계층화** — `minimal`/`standard`, 기본은 minimal
4. **`next_tool_suggestions`** (`hints.py`) — 모든 응답에 "다음 툴" 안내 → 에이전트가 헤매지 않음
5. **불확실성 표현** (`uncertainty.py`) — 빈 결과에 "없다"가 아니라 "인덱스 안 됐을 수도 / 정적으로 안 보일 수도" → 환각 방지
6. **타임아웃 설계** — 읽기 전용 툴만 타임아웃(`CRG_TOOL_TIMEOUT`), 쓰기 툴은 제외. 이유: "타임아웃은 기다림을 취소하지만 워커를 취소하지 않는다"
7. **프로그레시브 디스클로저** — `get_minimal_context_tool`(~100토큰) 먼저, 비싼 건 나중에

### 바로 적용할 3가지
1. 모든 툴 응답에 상한 두고 CI로 고정
2. `next_tool_suggestions` 패턴 도입
3. 빈 결과에 `confidence` 노트 붙이기

---

## 12. React / PHP로 만들 수 있는가

### 결론: **부분적으로 가능. 통째로 재작성은 비추천.**

| 레이어 | React/TS | PHP |
|---|:---:|:---:|
| 프론트엔드 UI | 🟢🟢🟢 최고 | 🟡 가능 |
| MCP 서버 | 🟢🟢 좋음 (공식 TS SDK) | 🟡 가능 |
| SQLite 그래프 저장 | 🟢 OK | 🟢 OK |
| **Tree-sitter 파싱** | 🟡 web-tree-sitter (WASM) | 🔴 거의 불가 |
| 임베딩/벡터 | 🟡 transformers.js | 🔴 API만 |
| Leiden 클러스터링 | 🟡 JS 라이브러리 희소 | 🔴 없음 |

### 추천 아키텍처 — 엔진 재사용 + React 대시보드
```
🎨 React 프론트엔드 (신규 창작)
   react-force-graph / @xyflow/react, d3, TanStack Query, Recharts
          ↓ HTTP / JSON
🐍 code-review-graph serve --http  (Python 엔진 그대로 재사용)
```
`serve --http`가 이미 있고 `exports.py`가 JSON/GraphML/Cypher/SVG를 내보내므로,
React는 받아서 그리기만 하면 된다. **난이도 중하, MVP 2~3주.**

### TypeScript 포팅 가능성
가능하다 (`web-tree-sitter`, `@modelcontextprotocol/sdk`, `better-sqlite3`, `transformers.js`, 그리고 이미 VS Code 확장이 TS로 `graph.db`를 읽는다).
단, **40개 언어 WASM 번들 수집** + `parser.py` 776KB에 축적된 언어별 노드타입 매핑 노하우 재현 = 수개월.

### PHP는 비추천
Tree-sitter 실용 바인딩 없음, `nikic/php-parser`는 PHP 코드만 파싱, MCP PHP SDK 생태계 빈약, 장기 실행 데몬/watch와 체질 불일치.
PHP로 할 수 있는 건 **Laravel 웹 대시보드(graph.db 읽기)** 또는 Python CLI 래퍼 정도.

---

## 13. 유튜브 강의 제작 가능성

### **매우 적합** — 소재·시각자료·숫자가 모두 준비돼 있다

| 이유 | 설명 |
|---|---|
| 트렌드 정중앙 | MCP + AI 에이전트 + 컨텍스트 엔지니어링 |
| 훅이 선명 | "AI 코딩 비용 65배 줄이는 방법" |
| 구체적 숫자 | 143,594 → 2,196 토큰 |
| 한국어 자료 희소 | MCP 심화 콘텐츠 거의 없음 = 블루오션 |
| 시각 자료 완비 | `diagrams/` 에 PNG 9장 + 데모 GIF 이미 존재 |
| MIT 라이선스 | 코드 시연·설명 자유 (출처 표기만) |
| 재현 가능 | `REPRODUCING.md` 로 "직접 측정" 가능 |

### 커리큘럼 20화 기획

**시즌 1 — 입문**
1. AI 코딩 비용, 65배 줄이는 방법 (훅)
2. MCP가 뭔데? 10분 완전정복
3. 3줄로 끝내는 설치
4. 내 PR을 AI가 리스크 점수로 리뷰해준다
5. 이 함수 고치면 뭐가 깨질까? (blast radius)

**시즌 2 — 실전**
6. 내 코드베이스 지도 자동 생성 (visualize + wiki)
7. 테스트 없는 함수 자동 색출
8. GitHub Action으로 PR 자동 리뷰봇 만들기
9. 의미 검색: API 키 없이 공짜로
10. Cursor/Copilot/Windsurf 전부 세팅하기

**시즌 3 — 심화 (차별화 구간)**
11. Tree-sitter로 40개 언어 파싱하는 원리
12. SQLite로 지식 그래프 만들기 (스키마 13 해부)
13. **AI 에이전트 토큰 예산 설계법** (247k 토큰 버그 사례)
14. 환각 막는 confidence 설계 (`uncertainty.py`)
15. `next_tool_suggestions`: 에이전트 길안내 (`hints.py`)
16. MCP 서버 직접 만들기 (FastMCP)

**시즌 4 — 창작 (최고 가치)**
17. React로 코드 그래프 대시보드 만들기
18. 커스텀 언어 추가하기 (`languages.toml`)
19. 우리 회사 코드리뷰 자동화 구축기
20. MCP 서버 수익화 전략

### 제작 팁
- 썸네일: "143,594 → 2,196" 숫자 대비
- 첫 15초에 `diagrams/diagram1_before_vs_after.png` 노출
- `diagrams/context-savings-demo.gif` 그대로 활용 가능
- `detect-changes --brief --verify` 실행 화면(Token Savings 패널 + tiktoken 검증) 녹화
- 수익 연계: 유튜브 광고 → Inflearn/Udemy 유료 강의 → 기업 교육 → SaaS 홍보
- 법적: 설명란에 *"원본: tirth8205/code-review-graph (MIT)"* 표기. "제가 만들었습니다" ❌ / "분석·확장했습니다" ✅

---

## 14. 수익화 아이디어 상세

### 법적 베이스 — MIT
상업적 이용 ✅ / 수정 ✅ / 재배포 ✅ / **클로즈드소스 판매 ✅** / 리브랜딩 ✅ / SaaS 호스팅 과금 ✅

**지켜야 할 것**
1. 저작권 고지 + MIT 전문을 배포물에 포함 (원저작자 `Tirth`)
2. "처음부터 만들었다" 금지 → "기반: code-review-graph (MIT)"
3. 상표/로고는 라이선스 범위 밖 → 독자 브랜드명 사용
4. `docs/LEGAL.md` 확인

> 포크해서 "내 제품"으로 파는 것보다, **커뮤니티에 기여하며 주변 가치로 수익화**하는 쪽이 지속가능하다.
> 업스트림이 계속 개선되는 이득을 무료로 받는다.

### 티어 1 — 즉시 시작 (투자 거의 0)

**① 유튜브 + 유료 강의 패키지**
- 준비 2~4주 / 투자 ≈0 / 예상 월 50만~500만원 / 난이도 ⭐⭐
- 유튜브 무료(시즌1~2) → Inflearn·Udemy 유료(시즌3~4, ₩55,000~99,000) → 기업 워크샵(1일 200만~500만원)
- 차별화: 한국어 "MCP 서버 개발 + 컨텍스트 엔지니어링" 강의가 거의 없다

**② 한국 시장 현지화 + 컨설팅**
- 준비 1~2주 / 투자 0 / 건당 300만~2,000만원 / 난이도 ⭐⭐
- 세일즈 포인트: 한국 기업의 "코드 외부 유출 금지" 요구 ↔ local-first + 텔레메트리 0
- 패키지: 베이직 300만(그래프 구축+도구 세팅+교육 2h) / 스탠다드 800만(+PR 자동리뷰+커스텀 리졸버+위키 파이프라인) / 엔터프라이즈 2,000만+(+멀티레포 데몬+사내 임베딩 연동+6개월 유지보수)
- 타겟: 금융·공공·제조 대기업

**③ 기술 블로그 → 뉴스레터 → 스폰서십**
- 즉시 시작 / 월 20만~200만원 / 난이도 ⭐
- 소재: "247,000 토큰 버그를 어떻게 발견했나", "AI 에이전트 토큰 예산 설계법", "Tree-sitter 40개 언어 파싱", "스키마 마이그레이션 13단계"

### 티어 2 — 중기 (1~3개월)

**④ React 대시보드 SaaS — 최우선 추천 🏆**
- 준비 2~3개월 / 인프라 월 10만~50만 / MRR 100만~1,000만원+ / 난이도 ⭐⭐⭐
- Python 엔진 재사용, 오빠는 UI/UX + 비즈니스 레이어만 제작
- 아키텍처: React 대시보드 → Next.js/Laravel 백엔드(멀티테넌트·SSO·과금·RBAC) → code-review-graph 엔진
- 가격: Free ₩0(1레포) / Pro ₩19,000월(5레포) / Team ₩49,000시트월 / Enterprise 문의
- **킬러 기능(원본에 없는 것)**: 아키텍처 드리프트 추적(`graph_diff.py` 활용), 팀 리더보드, Slack/Teams 알림, 경영진용 기술부채 지수 PDF, AI 자동 리팩토링 제안 PR

**⑤ VS Code 확장 Pro 버전**
- 준비 1~2개월 / 월 30만~300만원 / 난이도 ⭐⭐⭐
- 무료: 그래프 뷰·영향 반경 / Pro(₩9,900월): 인라인 리스크 렌즈, 테스트 공백 하이라이트, 히스토리 비교, 멀티레포 통합 뷰, AI 리팩토링 제안
- 마켓플레이스가 결제를 지원하지 않으므로 라이선스 키 방식(Lemon Squeezy, Gumroad)

**⑥ 특화 언어/프레임워크 리졸버 판매**
- 준비 2~6주(언어당) / 건당 50만~500만원 / 난이도 ⭐⭐⭐
- 빈 틈: **전자정부 프레임워크(eGovFrame)** 🏛️, NEXACRO/WebSquare, **COBOL/메인프레임**(금융 레거시), Django/FastAPI 심화, Angular/NestJS DI, Flutter/RN 브릿지, Unity C#, **ABAP(SAP)**
- 모델: 오픈소스 코어 + 유료 엔터프라이즈 리졸버 (open-core)

### 티어 3 — 장기 고수익 (6개월+)

**⑦ 레거시 마이그레이션 진단 서비스**
- 준비 3~6개월 / **건당 5,000만~3억원** / 난이도 ⭐⭐⭐⭐⭐
- 핵심 문제: 대기업·공공 레거시 현대화에서 "현재 시스템을 아는 사람이 없다"
- 제공: 전체 구조 지도화, 모듈 결합도 → 분리 경계 도출, 테스트 공백 리스크 매트릭스, 데드코드 탐지(= 옮길 필요 없는 코드 제거로 비용 절감 증명), `graph_diff.py` 진행률 추적
- 산출물: 진단 리포트(구조 지도 / 분리 로드맵 / 데드코드 제거 권고 / 리스크 매트릭스 / Phase 1~5 계획 / 월별 대시보드)
- 타겟: SI 대기업(삼성SDS·LG CNS·SK C&C) 협력 포지션 또는 직접 수주

**⑧ AI 에이전트 플랫폼 (코드 특화)**
- 준비 6~12개월 / 투자 유치 가능 규모 / 난이도 ⭐⭐⭐⭐⭐
- 그래프를 백본으로 PR 리뷰·테스트 생성·문서 유지·버그 추적·리팩토링 에이전트 조합 → "코드베이스 전용 AI 팀"

**⑨ 교육 기관 / 부트캠프 라이선싱**
- 연 2,000만~1억원 / 난이도 ⭐⭐⭐
- 상품: 수강생 프로젝트 구조 품질 자동 채점, 테스트 커버리지 평가, 복잡도·결합도 성장 추적, 리더보드
- 타겟: 코드스테이츠·패스트캠퍼스·우테코, 대학 SW학과

**⑩ 마켓플레이스 / 템플릿 판매 (패시브)**
- 월 10만~100만원 / 난이도 ⭐⭐
- GitHub Action 템플릿 팩(₩29,000), 커스텀 스킬 팩(₩19,000: 보안 리뷰·성능 리뷰), `languages.toml` 번들(₩9,900), Notion/Obsidian 템플릿(`exports.py` Obsidian 출력 활용)

### 전략 비교표

| # | 아이디어 | 투자 | 기간 | 수익 | 리스크 | 추천도 |
|---|---|:---:|:---:|:---:|:---:|:---:|
| 1 | 유튜브+강의 | 💵 | 2~4주 | 💰💰💰 | 🟢 | ⭐⭐⭐⭐⭐ |
| 2 | 현지화 컨설팅 | 💵 | 1~2주 | 💰💰💰💰 | 🟢 | ⭐⭐⭐⭐⭐ |
| 3 | 블로그/뉴스레터 | 💵 | 즉시 | 💰💰 | 🟢 | ⭐⭐⭐⭐ |
| 4 | **React SaaS** | 💵💵 | 2~3개월 | 💰💰💰💰💰 | 🟡 | ⭐⭐⭐⭐⭐ |
| 5 | VS Code Pro | 💵💵 | 1~2개월 | 💰💰💰 | 🟡 | ⭐⭐⭐⭐ |
| 6 | 특화 리졸버 | 💵💵 | 2~6주 | 💰💰💰 | 🟡 | ⭐⭐⭐⭐ |
| 7 | **레거시 진단** | 💵💵💵 | 3~6개월 | 💰💰💰💰💰💰 | 🔴 | ⭐⭐⭐⭐ |
| 8 | 에이전트 플랫폼 | 💵💵💵💵 | 6~12개월 | 💰💰💰💰💰💰 | 🔴 | ⭐⭐⭐ |
| 9 | 교육 라이선싱 | 💵💵 | 2~4개월 | 💰💰💰 | 🟡 | ⭐⭐⭐ |
| 10 | 템플릿 판매 | 💵 | 2~4주 | 💰 | 🟢 | ⭐⭐⭐ |

### 추천 로드맵

**Phase 1 (0~1개월) — 신뢰 쌓기, 투자 0**
① 유튜브 시즌1(EP1~5) ② 기술 블로그 3편 ③ **오픈소스 기여 2~3개** (컨트리뷰터 타이틀 = 이후 컨설팅·강의 신뢰도의 핵심)

**Phase 2 (1~3개월) — 현금 흐름**
④ Inflearn 유료 강의 출시 ⑤ 컨설팅 1~2건 수주 ⑥ React 대시보드 MVP 착수

**Phase 3 (3~6개월) — 제품화**
⑦ React SaaS 베타(Free+Pro) ⑧ 특화 리졸버 1개(eGovFrame 추천) ⑨ VS Code Pro 출시

**Phase 4 (6개월~) — 확장**
⑩ 레거시 진단 서비스 패키징 → 대형 수주 ⑪ 에이전트 플랫폼 확장 / 투자 유치 검토

### 리스크 체크

| 리스크 | 대응 |
|---|---|
| 원저작자가 상업 버전 출시 | 포크 의존 대신 **창작 레이어(UI·특화 리졸버)**에 가치 집중 |
| GitHub/Cursor가 유사 기능 내장 | **local-first 보안** + **한국 특화**로 차별화 |
| "65배" 과장 논란 | 전체 코퍼스 기준임을 명시 (문서에도 그렇게 적혀 있다) |
| 정적 분석 한계 지적 | 한계를 먼저 공개하는 편이 신뢰에 유리 |
| MIT 라이선스 이슈 | 고지 유지로 해결 |

### 핵심 조언

> **엔진을 다시 만들지 말고, 엔진 위에 자기 가치를 올려라.**
> Python 엔진 63,800줄은 MIT로 받은 자산이다.
> 만들어야 할 가치는 **UI/UX · 한국 시장 지식 · 특화 리졸버 · 교육 콘텐츠** —
> 이것이 복제 불가능한 자산이다.

---

## 15. 참고 문서 링크

| 주제 | 경로 |
|---|---|
| 사용 가이드 | [docs/USAGE.md](docs/USAGE.md) |
| 명령어 전체 | [docs/COMMANDS.md](docs/COMMANDS.md) |
| FAQ (LSP·RAG·grep 비교, 안 써야 할 때) | [docs/FAQ.md](docs/FAQ.md) |
| 트러블슈팅 | [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) |
| GitHub Action | [docs/GITHUB_ACTION.md](docs/GITHUB_ACTION.md) |
| 벤치마크 재현 | [docs/REPRODUCING.md](docs/REPRODUCING.md) |
| 로드맵 | [docs/ROADMAP.md](docs/ROADMAP.md) |
| 아키텍처 | [docs/architecture.md](docs/architecture.md) |
| DB 스키마 (v13) | [docs/schema.md](docs/schema.md) |
| 커스텀 언어 추가 | [docs/CUSTOM_LANGUAGES.md](docs/CUSTOM_LANGUAGES.md) |
| 버전별 기능 | [docs/FEATURES.md](docs/FEATURES.md) |
| 법적 고지 | [docs/LEGAL.md](docs/LEGAL.md) |
| 기여 가이드 / 브랜치 정책 | [CONTRIBUTING.md](CONTRIBUTING.md) |
| 보안 정책 | [SECURITY.md](SECURITY.md) |
| 라이선스 (MIT) | [LICENSE](LICENSE) |

---

*이 문서는 저장소 전수조사 기반 분석 결과이며, 수익 금액은 시장 추정치입니다.*
