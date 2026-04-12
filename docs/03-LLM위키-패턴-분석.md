# LLM Wiki 패턴 분석

> 원문을 `peach-wiki` 관점에서 구조화한 해설 문서다.
> 관련 문서: [LLM Wiki 원문](02-LLM위키-원문.md), [qmd 가이드](04-qmd-설치-활용-가이드.md), [종합 분석](05-qmd와-LLM위키-종합분석.md)
> 출처: `wikiSystem` POC에서 이관 후 `peach-wiki` 기준으로 재정리

## 핵심 통찰

```text
RAG:      질문 → 문서 검색 → 임시 답변 → 소멸
LLM Wiki: 소스 → 위키 업데이트 → 질문 → 위키 기반 답변 → 답변도 위키에 저장
```

핵심 차이는 누적성이다. RAG는 질문마다 지식을 다시 찾고 조합한다. LLM Wiki는 소스를 한 번 구조화한 뒤, 이후 질문과 추가 소스를 통해 계속 보강한다.

## 3계층 구조

| 계층 | `peach-wiki`에서의 의미 | 규칙 |
|---|---|---|
| Raw Source | 코드, 문서, 옵시디언 노트 | 읽기만, 수정 금지 |
| Wiki Layer | `docs/wiki/` | LLM이 생성·갱신 |
| Schema | `AGENTS.md`, `skills/peach-wiki/SKILL.md`, `WIKI-AGENTS.md` | 사람과 LLM이 함께 진화 |

`peach-wiki`는 이 구조를 단일 스킬로 운영한다. code/옵시디언 모드를 나누되 위키 저장 위치는 `docs/wiki/`로 고정한다.

## 3가지 오퍼레이션

### INGEST

새 소스를 읽고 `docs/wiki/`에 통합한다.

- source 요약 생성
- 관련 entity, concept, synthesis 문서 갱신
- `wiki-index.md`, `wiki-log.md` 갱신

### QUERY

위키를 먼저 읽고, 부족한 부분만 소스에서 보강한다.

- `wiki-index.md`로 진입
- 관련 wiki 페이지 드릴다운
- 필요 시 qmd로 추가 검색
- 가치 있는 답변은 synthesis 문서로 환류

### LINT

위키의 건강 상태를 정기적으로 점검한다.

- 모순
- 고아 페이지
- 깨진 소스 링크
- 미문서화 항목
- 코드 모드의 경우 DRIFT 위험

## `peach-wiki`에 적용되는 운영 원칙

- 위키는 항상 `docs/wiki/` 하위에만 쓴다.
- qmd가 있으면 검색은 qmd 우선이다.
- `wiki-index.md`, `wiki-log.md`는 구조의 일부이며 임의 변경 대상이 아니다.
- code 모드에서는 DRIFT가 Karpathy 원문을 확장한 실무 기능이다.
- 옵시디언 모드와 code 모드는 소스만 다르고 위키 운영 방식은 동일하다.

## 왜 이 패턴이 지금도 유효한가

사람이 위키를 포기하는 이유는 생각이 아니라 유지보수다. `peach-wiki`는 그 유지보수를 LLM에게 위임하고, 사람은 소스 선정과 질문, 우선순위 결정에 집중하게 만든다.

## 구현자가 기억할 포인트

- 이 프로젝트의 최신 기준은 `wikiSystem`이 아니라 `peach-wiki`다.
- 과거 `.wiki/`, `5-Wiki/`, `wiki-code`는 설계 실험의 흔적일 뿐 정본이 아니다.
- 배경 이론은 이 문서와 원문에서 보고, 실제 동작 규칙은 `skills/peach-wiki/SKILL.md`를 따른다.
