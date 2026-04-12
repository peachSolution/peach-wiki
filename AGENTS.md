# AI 에이전트 가이드

> peach-wiki 스킬 개발·유지보수를 위한 가이드

---

## 1. 스킬 개발 규칙

### SKILL.md frontmatter 필수 필드
```yaml
---
name: peach-wiki
description: |
  한 줄 설명 (트리거 키워드 포함)
---
```

### references 정책
- 스킬 내부 `references/` 폴더: 스킬별 상세 가이드
- 조건부 참조: 필요한 참조만 로드 (토큰 절약)

### 버전 관리 규칙

두 파일의 version을 **항상 동일하게** 유지한다.
- `.claude-plugin/marketplace.json` → `plugins[0].version`
- `.claude-plugin/plugin.json` → `version`

| 변경 유형 | 버전 |
|----------|------|
| patch | 문서 수정, 오타, 버그 수정 |
| minor | 오퍼레이션 추가, 기능 개선 |
| major | 하위호환 파괴, 구조 변경 |

---

## 2. Bounded Autonomy

### Must Follow
- wiki 저장 위치: `docs/wiki/` 고정 (변경 금지)
- Raw Source 수정 금지 (읽기만)
- wiki-index.md, wiki-log.md 형식 유지
- qmd 1순위 검색 원칙

### May Adapt
- 위키 페이지 내부 구조 세부 조정
- qmd 컨텍스트 설명 내용
- 점검 항목 세부 조정

### May Suggest (사용자 확인 필요)
- 오퍼레이션 추가/삭제
- WIKI-AGENTS.md 템플릿 구조 변경
- Bounded Autonomy 규칙 변경
