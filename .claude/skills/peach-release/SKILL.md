---
name: peach-release
description: |
  peach-wiki 버전 업데이트 → CHANGELOG.md 자동 생성 → develop 커밋/푸시 → main PR 생성 → PR 머지 → GitHub Release 생성까지 일괄 처리하는 릴리스 스킬.
  변경 내용 분석 → CHANGELOG 초안 + 실행 계획을 한 번에 제시 → 대화로 보완 → 승인 1회 후 일괄 실행.
  "릴리스", "버전 업", "release", "main 머지", "배포 준비" 키워드로 트리거.
  peach-wiki 저장소에서만 사용한다.
metadata:
  internal: true
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

# peach-release — 릴리스 일괄 처리

peach-wiki 저장소의 릴리스를 한 번에 처리한다.
두 버전 파일 동기화 → CHANGELOG.md 업데이트 → develop 커밋/푸시 → main PR 생성 → 머지 → GitHub Release 생성까지 자동화한다.

## 전제조건

- **peach-wiki 저장소 루트**에서 실행
- `develop` 브랜치에 체크아웃된 상태
- `gh` CLI 인증 완료

---

## Workflow

### 1단계: 상태 확인

```bash
git status && git branch && git log --oneline -5
```

- develop 브랜치인지 확인한다. 아니면 중단하고 사용자에게 알린다.
- 미스테이지 변경사항이 있으면 사용자에게 보여주고 계속할지 확인한다.

### 2단계: 분석 → 계획 한 번에 제시

#### 2-1. 현재 버전 확인 (2중 교차 검증)

```bash
grep '"version"' .claude-plugin/marketplace.json
grep '"version"' .claude-plugin/plugin.json
```

두 값이 일치해야 정상이다. 불일치 시 **중단하고 사용자에게 알린다**.

#### 2-2. 신규 커밋 추출 (Release 커밋 기준)

`git log main..develop`은 main이 뒤처진 경우 이미 릴리스된 커밋까지 포함하므로 **사용 금지**.
대신 `Release v{현재버전}` 커밋 이후 신규 커밋만 추출한다.

```bash
CURRENT_VER=$(grep -m1 '"version"' .claude-plugin/plugin.json | grep -o '[0-9]*\.[0-9]*\.[0-9]*')
RELEASE_COMMIT=$(git log --oneline --all | grep "Release v${CURRENT_VER}" | head -1 | awk '{print $1}')
git log ${RELEASE_COMMIT}..HEAD --oneline
git diff ${RELEASE_COMMIT}..HEAD --stat
```

> **핵심 원칙**: 분석 대상은 `Release v{현재버전}` 커밋 이후 신규 커밋만이다.
> `Release v{현재버전}` 커밋 자체는 분석에서 제외한다.

> 신규 커밋이 0개이면 "릴리스할 변경사항이 없습니다"를 출력하고 중단한다.

#### 2-3. 분석 + 계획 통합 출력

AI는 커밋을 분석하여 버전 타입을 결정하고, CHANGELOG 초안과 실행 계획을 **한 번에** 출력한다.

**버전 판단 기준** (핵심: peach-wiki 스킬, references):

| 변경 유형 | 버전 | 예시 |
|----------|------|------|
| 문서 수정, 오타, 버그 수정, 보조 파일 추가 | **patch** | SKILL.md 오류 수정, references 파일 추가 |
| 핵심 스킬 추가/개선, 오퍼레이션 로직 변경 | **minor** | 새 오퍼레이션 추가, 워크플로우 개선 |
| 하위호환 파괴, 구조 변경 | **major** | 배포 구조 변경, 스킬 인터페이스 변경 |

```
🚀 Release v{새버전} 계획
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
버전 검증:
  plugin.json      : v{현재버전}
  marketplace.json : v{현재버전}  ← 일치 ✅

커밋 ({N}개):
  {커밋 해시} {커밋 메시지}
  ...

버전: {현재버전} → {새버전} ({patch/minor/major})
이유: {판단 근거 한 줄}

📝 CHANGELOG.md 추가 내용:
## [v{새버전}] - {YYYY-MM-DD}

### Added
- ...

### Changed
- ...

### Removed
- ...

### Fixed
- ...

📦 실행 순서:
  1. marketplace.json / plugin.json 버전 업데이트
  2. CHANGELOG.md 맨 위에 블록 추가
  3. git commit -m "Release v{새버전}"
  4. git push origin develop
  5. gh pr create --base main --head develop --title "Release v{새버전}"
  6. gh pr merge {PR번호} --merge --delete-branch=false
  7. gh release create v{새버전} --target main
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
진행할까요? 수정이 필요하면 말씀해 주세요.
```

**대화 보완**: 사용자가 버전 변경·CHANGELOG 수정·항목 추가 등을 요청하면 해당 부분만 수정하여 위 계획을 다시 출력한다. 승인("진행")이 올 때까지 반복한다.

사용자가 **진행**을 승인하면 3단계를 순서대로 실행한다. **취소**하면 중단한다.

### 3단계: 일괄 실행

승인 후 아래 작업을 순서대로 실행한다. 각 단계 완료 시 진행 상황을 출력한다.

#### 3-1. 두 버전 파일 동시 업데이트

반드시 두 파일을 동시에 같은 버전으로 업데이트한다. 불일치 시 auto update가 실패한다.

- `.claude-plugin/marketplace.json` → `plugins[0].version`
- `.claude-plugin/plugin.json` → `version`

#### 3-2. CHANGELOG.md 업데이트

2단계에서 작성한 버전 블록을 `CHANGELOG.md` **맨 위에** 추가한다.

- 해당 섹션이 없으면 생략한다 (빈 섹션 작성 금지).
- 각 항목은 한 줄 한국어로 간결하게 작성한다.
- `CHANGELOG.md`가 없으면 신규 생성한다. 파일 상단에 아래 헤더를 포함한다:

```markdown
# Changelog

> [keep-a-changelog](https://keepachangelog.com) 포맷을 따릅니다.
> 버전은 [Semantic Versioning](https://semver.org)을 따릅니다.

```

CHANGELOG.md 포맷 (keep-a-changelog 표준):

```markdown
## [v{버전}] - {YYYY-MM-DD}

### Added
- 새로 추가된 스킬, 기능, 파일

### Changed
- 기존 기능 개선, 워크플로우 변경, 구조 개편

### Removed
- 제거된 스킬, 파일, 기능

### Fixed
- 버그 수정, 오타 수정
```

분류 기준:

| 커밋 prefix / 파일 변화 | 섹션 |
|----------------------|------|
| `feat:`, 새 SKILL.md 추가 | Added |
| `refactor:`, `docs:`, 기존 SKILL.md 수정, references 재구성 | Changed |
| 파일 삭제, 이전 | Removed |
| `fix:`, 오타 수정 | Fixed |

#### 3-3. 커밋

```bash
git add .claude-plugin/marketplace.json .claude-plugin/plugin.json CHANGELOG.md
git commit -m "Release v{버전}"
```

#### 3-4. develop 푸시

```bash
git push origin develop
```

#### 3-5. main PR 생성

실행 전 기존 PR 여부를 확인한다. 이미 열린 PR이 있으면 생성을 건너뛰고 해당 번호를 사용한다.

```bash
PR_NUM=$(gh pr list --base main --head develop --state open --json number --jq '.[0].number')
if [ -z "$PR_NUM" ]; then
  gh pr create \
    --base main \
    --head develop \
    --title "Release v{버전}" \
    --body "..."
  PR_NUM=$(gh pr list --base main --head develop --state open --json number --jq '.[0].number')
fi
```

PR body는 CHANGELOG.md에 방금 작성한 버전 블록 내용을 그대로 사용한다.

```
## Release v{버전}

### 변경 사항
{CHANGELOG.md 해당 버전 블록의 내용}

### 버전
- {이전 버전} → {새 버전}
- 변경 유형: {patch/minor/major}

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

#### 3-6. PR 머지

```bash
gh pr merge {PR_NUM} --merge --delete-branch=false
```

> `--delete-branch=false`: develop 브랜치는 삭제하지 않는다.

#### 3-7. GitHub Release 생성

실행 전 동일 버전 Release가 이미 존재하는지 확인한다. 존재하면 생성을 건너뛴다.

```bash
if ! gh release view v{버전} > /dev/null 2>&1; then
  gh release create v{버전} \
    --title "v{버전}" \
    --notes "..." \
    --target main
else
  echo "ℹ️ Release v{버전} 이미 존재 — 건너뜀"
fi
```

릴리즈 노트는 CHANGELOG.md 해당 버전 블록을 그대로 사용한다.

### 4단계: 완료 보고

```
✅ Release v{버전} 완료
- develop 커밋: {커밋 해시}
- PR: {PR URL}
- main 머지: 완료
- GitHub Release: {Release URL}
```

---

## 규칙

- **develop 브랜치에서만** 버전을 업데이트한다. main 직접 작업 금지.
- 버전 타입(major/minor/patch)은 **AI가 분석하여 결정**한다.
- 분석 → CHANGELOG 초안 → 실행 계획을 **한 번에** 출력한다. 분석과 계획을 별도로 나눠 묻지 않는다.
- 사용자가 수정을 요청하면 해당 부분만 고쳐 계획을 다시 출력한다. 승인("진행")이 올 때까지 대화로 보완한다.
- 승인은 **1회만** 받는다. 단계별 개별 승인 금지.
- 두 버전 파일은 항상 동일한 버전으로 유지한다.
- CHANGELOG.md는 항상 최신 버전이 맨 위에 위치한다.
- PR body와 GitHub Release 노트는 CHANGELOG.md 내용을 기준으로 작성한다 (중복 작성 금지).
