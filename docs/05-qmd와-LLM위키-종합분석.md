# qmd와 LLM Wiki 종합 분석

> `peach-wiki`의 배경 설계를 한 번에 이해하기 위한 종합 문서다.
> 읽기 순서: [원문](02-LLM위키-원문.md) → [패턴 분석](03-LLM위키-패턴-분석.md) → [qmd 가이드](04-qmd-설치-활용-가이드.md) → 이 문서
> 출처: `wikiSystem` POC에서 이관 후 `peach-wiki` 정본 기준으로 재작성

## 한 줄 요약

Karpathy가 제안한 누적형 위키 패턴 위에, qmd가 검색 인프라를 제공하고, `peach-wiki`가 이를 단일 스킬과 `docs/wiki/` 구조로 운영하는 형태다.

## 큰 구조

```text
Raw Source
  ├─ 코드
  ├─ 프로젝트 문서
  └─ 옵시디언 노트
        ↓
qmd
  ├─ BM25
  ├─ vector search
  └─ reranking
        ↓
peach-wiki
  ├─ INIT
  ├─ INGEST
  ├─ QUERY
  ├─ DRIFT
  └─ LINT
        ↓
docs/wiki/
  ├─ WIKI-AGENTS.md
  ├─ wiki-index.md
  ├─ wiki-log.md
  └─ concepts/entities/synthesis/sources/diagrams
```

## 역할 분리

| 구성 | 역할 |
|---|---|
| Karpathy 원문 | 방법론의 출발점 |
| qmd | 소스 검색 인프라 |
| `peach-wiki` | 현재 제품화된 운영 방식 |
| `docs/wiki/` | 실제 누적 지식베이스 |

## 왜 `wikiSystem`이 아니라 `peach-wiki`가 정본인가

- `wikiSystem`은 POC였다.
- `wiki-code`, `wiki-para`, `.wiki/`, `5-Wiki/`는 실험 단계 산물이다.
- 현재 제품은 `peach-wiki` 단일 스킬과 `docs/wiki/` 고정 구조로 통합되었다.

즉, POC는 배경 지식으로만 남기고, 운영 기준은 현재 저장소 문서로 단일화해야 한다.

## 현재 저장소에서 이 문서가 담당하는 역할

- 배경 이론과 실제 제품 구조를 연결한다.
- `docs/` 문서들의 읽기 순서를 제공한다.
- 제품 소개 문서와 내부 운영 문서 사이의 중간 설명 층을 담당한다.

## 무엇을 보러 어디로 가야 하나

| 궁금한 것 | 읽을 문서 |
|---|---|
| 아이디어 원형 | [LLM Wiki 원문](02-LLM위키-원문.md) |
| LLM Wiki 핵심 개념 | [패턴 분석](03-LLM위키-패턴-분석.md) |
| qmd를 왜 쓰고 어떻게 붙이는가 | [qmd 가이드](04-qmd-설치-활용-가이드.md) |
| 실제 동작 규칙 | [peach-wiki 스킬](../skills/peach-wiki/SKILL.md) |
| 제품 소개와 설치 | [루트 README](../README.md) |

## 결론

`peach-wiki`는 “원문 아이디어를 이해하고, qmd로 소스를 찾고, `docs/wiki/`에 누적형 위키를 유지하는 시스템”이다. 이 문서는 그 구조를 압축해서 보여주는 허브 문서로만 유지한다.
