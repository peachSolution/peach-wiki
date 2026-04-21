# peach-wiki

Andrej Karpathy의 [LLM Wiki 패턴](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)을 적용한 Claude Code 플러그인.

코드 프로젝트와 옵시디언 노트를 Raw Source로 삼아 `docs/wiki/`에 **누적형 지식베이스**를 구축합니다.

## 핵심 개념

```
RAG:      질문 → 문서 검색 → 임시 답변 → 소멸
LLM Wiki: 소스 → 위키 업데이트 → 질문 → 위키 기반 답변 → 답변도 위키에 저장
```

wiki는 쓸수록 깊어지는 **복리형 지식 자산**입니다.

배경 이론과 설계 해설은 [docs/01-문서-안내.md](docs/01-문서-안내.md)에서 정리합니다.

## 설치

### 권장 설치 조합

- **Claude Code만 사용**: Claude Code 플러그인 방식 설치를 권장한다.
- **Claude Code + 다른 AI 함께 사용**: Claude Code는 플러그인으로 설치하고, Codex / Cursor / Gemini / Antigravity 등 다른 AI에는 `skills.sh`로 별도 설치한다.
- **비-Claude AI만 사용**: `skills.sh`로 설치한다.

> **중요**: Claude Code 플러그인으로 설치한 내용은 Cursor, Codex, Gemini 등 다른 AI가 직접 로드하지 못한다.
> 다른 AI에서도 사용하려면 해당 AI 대상에 `skills.sh` 방식으로 별도 설치해야 한다.
> 기본 권장 시나리오는 "Claude는 플러그인, 나머지는 `skills.sh`"이다.

### Claude Code 플러그인 (권장)

Claude Code에서는 플러그인 방식 설치를 기본 권장한다.
`agents/`, `hooks/`를 포함한 Claude Code 전용 구성을 함께 사용할 수 있다.

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add peachSolution/peach-wiki

# 2. 플러그인 설치
/plugin install peach-wiki
```

### skills.sh 설치 (비-Claude AI 권장 / Claude에서는 대안)

SKILL.md 오픈 스탠다드 기반으로 Cursor, Codex CLI, Antigravity, Copilot, Gemini CLI 등을 지원한다.

> **권고**: `-g` (global) 옵션을 사용하면 모든 프로젝트에서 스킬을 공유할 수 있다.
> `-g` 없이 실행하면 현재 디렉터리 기준의 project 스코프에만 설치된다.

**macOS / Linux**

```bash
# Codex 글로벌 설치
npx skills add peachSolution/peach-wiki -a codex -g

# 여러 AI 동시 글로벌 설치
npx skills add peachSolution/peach-wiki \
  -a codex \
  -a cursor \
  -a gemini-cli \
  -a antigravity \
  -g

# 프로젝트 스코프 설치 (현재 디렉터리에만 적용)
npx skills add peachSolution/peach-wiki -a cursor
```

**Windows PowerShell**

```powershell
# Codex 글로벌 설치
npx skills add peachSolution/peach-wiki -a codex -g

# 여러 AI 동시 글로벌 설치
npx skills add peachSolution/peach-wiki `
  -a codex `
  -a cursor `
  -a gemini-cli `
  -a antigravity `
  -g

# 프로젝트 스코프 설치 (현재 디렉터리에만 적용)
npx skills add peachSolution/peach-wiki -a cursor
```

> **`--skill '*'` 비권장**: `--skill '*'`은 내부 전용 스킬(`metadata.internal: true`)까지 포함한다.
> `--skill` 옵션 없이 실행하면 배포용 스킬만 자동 설치된다.

**지원 에이전트 ID**

| AI 도구 | 에이전트 ID |
|--------|-----------|
| Claude Code | `claude-code` |
| OpenAI Codex CLI | `codex` |
| Cursor / Cursor CLI | `cursor` |
| Gemini CLI | `gemini-cli` |
| Google Antigravity | `antigravity` |
| GitHub Copilot | `github-copilot` |
| Windsurf | `windsurf` |
| Roo Code | `roo` |

### 업데이트

**Claude Code 플러그인:**
- `/plugin` → Installed 탭 → peach-wiki 선택 → Update
- "Enable auto update" 설정 시 Claude Code 실행마다 자동 최신화

**skills.sh:**

**macOS / Linux**

```bash
# 업데이트 확인
npx skills check -g

# 재설치로 업데이트 (권장 — 글로벌)
npx skills add peachSolution/peach-wiki -a codex -g -y

# 여러 AI 도구 동시 업데이트
npx skills add peachSolution/peach-wiki \
  -a codex \
  -a cursor \
  -a gemini-cli \
  -a antigravity \
  -g -y
```

**Windows PowerShell**

```powershell
# 업데이트 확인
npx skills check -g

# 재설치로 업데이트 (권장 — 글로벌)
npx skills add peachSolution/peach-wiki -a codex -g -y

# 여러 AI 도구 동시 업데이트
npx skills add peachSolution/peach-wiki `
  -a codex `
  -a cursor `
  -a gemini-cli `
  -a antigravity `
  -g -y
```

> **`npx skills update` 비권장**: 새로 추가된 스킬이 설치되지 않는 버그가 있다.
> `npx skills add ... -g -y` 재설치를 사용한다.

## 사용

```bash
# 아무 프로젝트나 Obsidian 폴더에서 호출
/peach-wiki

# 자동 감지:
# .git 있음 → code 모드 (DRIFT 활성)
# .obsidian 있음 → 옵시디언 모드
```

### 오퍼레이션

| 오퍼레이션 | 동작 |
|-----------|------|
| **INIT** | docs/wiki/ 생성 + qmd 등록 + AGENTS.md wiki 규칙 추가 |
| **INGEST** | Raw Source → 위키 페이지 생성/업데이트 |
| **QUERY** | qmd 검색 + 위키 기반 질문 답변 |
| **DRIFT** | git 변경 감지 → 위키 갱신 (code 모드만) |
| **LINT** | 모순/고아/드리프트/미문서화 점검 |

### wiki 폴더 구조

```
docs/wiki/
├── WIKI-AGENTS.md    ← 운영 규칙
├── wiki-index.md     ← 카탈로그
├── wiki-log.md       ← 타임라인
├── concepts/         ← 패턴·개념
├── entities/         ← 모듈·인물·프로젝트
├── synthesis/        ← 분석·의사결정
├── sources/          ← 원본 요약
└── diagrams/         ← Mermaid 다이어그램
```

## qmd 연동

[qmd](https://github.com/tobi/qmd) 설치 시 토큰 절약 + 높은 검색 정확도:

```bash
npm install -g @tobilu/qmd
```

설계 배경과 qmd 활용 맥락은 [docs/04-qmd-설치-활용-가이드.md](docs/04-qmd-설치-활용-가이드.md)에서 다룹니다.

## 라이선스

MIT
