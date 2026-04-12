# peach-wiki

Andrej Karpathy의 [LLM Wiki 패턴](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)을 적용한 Claude Code 플러그인.

코드 프로젝트와 Obsidian PARA 노트를 Raw Source로 삼아 `docs/wiki/`에 **누적형 지식베이스**를 구축합니다.

## 핵심 개념

```
RAG:      질문 → 문서 검색 → 임시 답변 → 소멸
LLM Wiki: 소스 → 위키 업데이트 → 질문 → 위키 기반 답변 → 답변도 위키에 저장
```

wiki는 쓸수록 깊어지는 **복리형 지식 자산**입니다.

## 설치

### Claude Code 플러그인 (권장)

```bash
# 1. 마켓플레이스 등록
/plugin marketplace add peachSolution/peach-wiki

# 2. 플러그인 설치
/plugin install peach-wiki
```

### skills.sh (Codex, Cursor, Gemini 등)

```bash
npx skills add peachSolution/peach-wiki --skill '*' -a codex -g
```

## 사용

```bash
# 아무 프로젝트나 Obsidian 폴더에서 호출
/peach-wiki

# 자동 감지:
# .git 있음 → code 모드 (DRIFT 활성)
# .obsidian 있음 → para 모드
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

## 라이선스

MIT
