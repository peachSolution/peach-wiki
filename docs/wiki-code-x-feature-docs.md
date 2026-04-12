# wiki-code × peach-gen-feature-docs 조합 분석

> 작성일: 2026-04-08  
> 작성자: 스파이크 (Spike)  
> 목적: 두 스킬의 역할 분리 및 조합 전략 정리

---

## 결론 먼저

**조합 가능. 단 역할이 다르다 — 보완 관계.**

두 스킬은 같은 코드베이스를 보지만, 서로 다른 질문에 답한다.

| 스킬 | 핵심 질문 |
|------|----------|
| `wiki-code` | **"무엇이 어디에 있고, 어떻게 연결되는가"** |
| `peach-gen-feature-docs` | **"왜 이렇게 만들었고, 어떤 히스토리가 있는가"** |

---

## 각 스킬 한 줄 정의

### wiki-code
코드베이스를 Raw Source로 삼아 `.wiki/` 폴더에 **구조·의존성·흐름** 지식을 누적하는 시스템.  
git 변경 감지(DRIFT)로 위키가 코드와 항상 동기화된다.

### peach-gen-feature-docs
기존 기능 코드에 숨어 있는 **암묵지(tacit knowledge)를 명세서로 변환**하는 스킬.  
AI 자동 분석 + 개발자 CDM 인터뷰 2단계로 동작. 산출물은 `docs/기능별설명/` 폴더.

---

## 두 스킬 비교

| 구분 | `wiki-code` | `peach-gen-feature-docs` |
|------|-------------|--------------------------|
| **출력 위치** | `.wiki/entities·concepts·synthesis/` | `docs/기능별설명/{카테고리}/{기능}/` |
| **독자** | 팀 개발자 전체 / 구조 파악 | 후속 AI 세션 / 구현 컨텍스트 재주입 |
| **생명주기** | 코드 변화에 따라 갱신 (DRIFT) | 기능 단위 스냅샷 (as-is 고정) |
| **특기** | git diff 기반 드리프트 감지 | CDM 질문으로 개발자 암묵지 추출 |
| **qmd 연계** | 핵심 (검색 기반 탐색) | 없음 |
| **다이어그램** | Mermaid 자동 생성 | 없음 |
| **테스트 가이드** | 없음 | TDD-가이드.md 자동 생성 |

---

## 겹치는 영역과 공백

```
peach-gen-feature-docs          wiki-code
       │                              │
  "왜 이렇게 만들었나"           "무엇이 어디 있나"
  설계 결정 · ADR · 히스토리     모듈 구조 · 의존성 · 흐름
       │                              │
       └──────── 겹침 ───────────────┘
           entities 수준에서 동일 모듈을
           각자 다른 깊이로 기술함
```

**두 스킬 모두 안 하는 것:**
- 서로의 문서를 서로가 참조하지 않음 → 연결 없으면 중복 발생

---

## 조합 플로우 (권장 순서)

```
[신규 모듈 완성 후]

  1. /wiki-code ingest kacta-staff
     → .wiki/entities/module-kacta-staff.md 생성
     → 구조·의존성·API 흐름 등록
          │
          ▼

  2. /peach-gen-feature-docs
     카테고리: 직원관리 / 기능명: 직원-등록
     → docs/기능별설명/직원관리/직원-등록/ 생성
     → 설계결정.md + TDD-가이드.md + 처리흐름.md 생성
          │
          ▼

  3. (코드 수정 후) /wiki-code drift
     → git diff 감지 → .wiki 위키 자동 갱신
```

---

## 시나리오별 어떤 스킬을 쓸까

| 상황 | 사용 스킬 |
|------|----------|
| 신규 모듈 팀에 공유할 때 | `wiki-code ingest` |
| 기존 기능 수정 전 맥락 파악 | `peach-gen-feature-docs` |
| "이 코드 왜 이렇게 짰지?" | `peach-gen-feature-docs` (CDM 인터뷰) |
| 리팩토링 후 문서 동기화 | `wiki-code drift` |
| 주간 문서 정합성 점검 | `wiki-code lint` |
| AI에게 재주입할 컨텍스트 팩 | `peach-gen-feature-docs` 산출물 |
| 구조·흐름 다이어그램 | `wiki-code` (Mermaid 자동 생성) |

---

## 중복 방지 원칙 (1개만 지키면 됨)

> `wiki-code entities` = **"무엇·어디·어떻게"**  
> `peach-gen-feature-docs` = **"왜·히스토리·TDD"**

같은 내용을 두 곳에 쓰지 말 것.  
wiki entities 페이지에서 `docs/기능별설명/` 경로를 **크로스 링크**로 연결.

```markdown
# .wiki/entities/module-kacta-staff.md 예시

## 연결된 문서
- 암묵지 명세: `docs/기능별설명/직원관리/직원-등록/`
```

---

## 지금 당장 권장하는 순서

1. **wiki-code INIT → INGEST** (완료됨 — 2026-04-08)
   - 전체 모듈 구조 위키에 등록
   - 팀 공유 기준선 확보

2. **수정이 잦은 핵심 모듈에 peach-gen-feature-docs 선택 적용**
   - 후보: `kacta-staff (직원등록)`, `kacta-approval (승인 워크플로우)`, `sign (인증)`
   - 기준: 개선 예정이거나 암묵지가 많은 것부터

---

## 참고 스킬 위치

| 스킬 | 트리거 명령 |
|------|-----------|
| wiki-code | `/wiki-system:wiki-code` |
| peach-gen-feature-docs | `/peach:peach-gen-feature-docs` |
