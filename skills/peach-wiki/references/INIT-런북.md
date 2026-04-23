# INIT Runbook

> INIT 오퍼레이션의 preflight, qmd 등록, 신규 생성 흐름을 정리한 문서.
> 기본 경로는 항상 `docs/wiki/`이며, `.wiki/`와 `5-Wiki/`는 레거시 호환 마이그레이션 대상으로만 취급한다.

---

## 1. Preflight (4단계)

### Step 1: 모드 감지

```bash
ls -d .obsidian 2>/dev/null && echo "OBSIDIAN=true" || echo "OBSIDIAN=false"
ls -d .git 2>/dev/null && echo "GIT=true" || echo "GIT=false"
```

| .obsidian | .git | 모드 | DRIFT |
|-----------|------|------|-------|
| true | false | 옵시디언 | 비활성 |
| false | true | code | 활성 |
| true | true | code | 활성 |
| false | false | 일반 | 비활성 |

### Step 2: qmd 상태 확인

```bash
QMD_INDEX=$(basename $(pwd))
qmd --index "$QMD_INDEX" status 2>/dev/null
```

- 성공 → Step 3으로
- 실패 (`command not found`) → 설치 권고 메시지 출력 후 Step 4로 (qmd 없이 진행)

### Step 3: qmd 컬렉션 확인

```bash
qmd --index "$QMD_INDEX" collection list
```

- 현재 디렉토리가 등록된 컬렉션에 포함? → 스킵
- 미등록 → 등록 진행 (아래 §2)

### Step 4: 기존 wiki 감지

```bash
ls -d docs/wiki 2>/dev/null && echo "docs/wiki/ 이미 존재 → INIT 불필요"
ls -d .wiki 2>/dev/null && echo ".wiki/ 발견 → 마이그레이션 필요"
ls -d 5-Wiki 2>/dev/null && echo "5-Wiki/ 발견 → 마이그레이션 필요"
```

- `docs/wiki/WIKI-AGENTS.md` 존재 → INIT 중단 ("이미 초기화됨" 안내)
- `.wiki/` 또는 `5-Wiki/` 발견 → 레거시 호환이 필요한 경우에만 §3 마이그레이션
- 아무것도 없음 → §4 신규 생성

---

## 2. qmd 컬렉션 등록

### code 모드
```bash
qmd --index "$QMD_INDEX" collection add . --name "$QMD_INDEX" --mask "**/*.{ts,vue,md,sql,py,go,js}"
qmd --index "$QMD_INDEX" context add "qmd://$QMD_INDEX/" "프로젝트 한 줄 설명"
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed
```

### 옵시디언 모드
```bash
qmd --index "$QMD_INDEX" collection add . --name para --mask "**/*.md"
qmd --index "$QMD_INDEX" context add "qmd://para/" "옵시디언 노트 컬렉션"
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed
```

### 등록 실패 시
- 에러 메시지 출력 후 **중단**
- "qmd 컬렉션 등록에 실패했습니다. 수동으로 등록해주세요:" + 명령어 안내
- wiki 생성은 진행하지 않음 (qmd 없이도 wiki를 만들 수는 있으나, 등록 시도 후 실패는 환경 문제이므로 해결 후 재시도 권장)

---

## 3. 레거시 마이그레이션

이 섹션은 과거 POC 구조를 흡수해야 할 때만 쓴다.
현재 `peach-wiki`의 기본 흐름은 **신규 생성 또는 기존 `docs/wiki/` 재사용**이다.

### .wiki/ → docs/wiki/

```bash
# 1. docs/wiki/ 생성
mkdir -p docs/wiki

# 2. 내용 복사
cp -r .wiki/* docs/wiki/

# 3. 백업 (원본 보존)
mv .wiki .wiki.bak

# 4. WIKI-AGENTS.md 내 경로 업데이트
# .wiki/ → docs/wiki/ 로 경로 치환
```

### 5-Wiki/ → docs/wiki/

```bash
mkdir -p docs/wiki
cp -r 5-Wiki/* docs/wiki/
mv 5-Wiki 5-Wiki.bak
```

### 마이그레이션 실패 시
- 복사 실패 → `.bak` 생성 전이므로 원본 보존
- 사용자에게 수동 이동 안내

---

## 4. 신규 생성

```bash
mkdir -p docs/wiki/{concepts,entities,synthesis,sources,diagrams}
```

→ `references/WIKI-AGENTS-템플릿.md` 기반 WIKI-AGENTS.md 생성
→ wiki-index.md 초기화
→ wiki-log.md 초기화
→ entities/project-overview.md 생성

---

## 5. AGENTS.md wiki 참조 규칙 추가

대상 프로젝트의 AGENTS.md에 아래 섹션이 없으면 추가:

```markdown
## wiki 참조 (필수)
코드 생성·분석 전 아래 순서를 따른다.
1. `qmd --index 프로젝트명 query "키워드" -c 프로젝트명` 으로 wiki + 소스 통합 검색
2. qmd 미설치 시 `docs/wiki/wiki-index.md` → 관련 페이지 직접 Read
3. `docs/wiki/`도 없으면 기존 방식대로 진행
```

**중요**: AGENTS.md가 없으면 이 규칙만으로 새 파일 생성하지 않는다. 이미 존재하는 AGENTS.md에만 추가.

---

## 6. 완료 확인

```bash
ls docs/wiki/WIKI-AGENTS.md && echo "INIT 성공"
ls docs/wiki/wiki-index.md && echo "wiki-index 존재"
ls docs/wiki/wiki-log.md && echo "wiki-log 존재"
```

wiki-log.md에 기록:
```
## [YYYY-MM-DD] init | {프로젝트명}
- 모드: {code|para|일반}
- qmd 컬렉션: {등록됨|미등록}
- 마이그레이션: {.wiki/|5-Wiki/|없음}
```
