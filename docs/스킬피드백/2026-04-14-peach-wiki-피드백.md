---
status: completed
target_skill: peach-wiki
severity: 높음 1 / 중간 2 / 낮음 0
completed_at: 2026-04-14
applied_by: Claude Sonnet 4.6
---

# peach-wiki 피드백 — 2026-04-14

> **대상 스킬**: peach-wiki
> **작성 근거**: `~/source/bsi/tang`에서 wiki ingest 후 다른 AI 세션이 `qmd update`와 `qmd embed`를 실행했고, Synology Drive 위의 `Obsidian/PARA`까지 함께 임베딩되면서 `fileproviderd`/`SynologyDriveFileProvider` CPU 상승이 관측됨
> **심각도 요약**: 높음 1건 / 중간 2건 / 낮음 0건

---

## 1. 발견된 문제

| # | 문제 | 심각도 | 현재 스킬에 있는가 | SKILL.md 행 |
|---|------|:---:|:---:|---|
| 1 | `qmd embed`가 현재 프로젝트만 처리한다는 보장이 없음. plain `qmd embed`는 등록된 전체 컬렉션에 영향을 줄 수 있음 | 높음 | X | 99, 187 |
| 2 | code 모드에서 `프로젝트명`을 명시하지만, 실제 `update/embed` 시 컬렉션 범위를 다시 고정하지 않음 | 중간 | 부분 | 93, 99, 187 |
| 3 | `qmd` 전역 설정(`~/.config/qmd/index.yml`)에 다른 컬렉션이 섞여 있을 때의 위험 설명이 없음 | 중간 | X | — |

---

## 2. 해결 방법 / 우회 전략

### 문제 #1: plain `qmd embed`가 전체 컬렉션을 건드릴 수 있음

**원인**:
- `peach-wiki` 스킬은 INIT와 INGEST에서 모두 `qmd update && qmd embed`를 plain 형태로 안내한다.
- 실제 환경의 `~/.config/qmd/index.yml`에는 아래처럼 여러 컬렉션이 등록되어 있었다.
- 그중 `para`는 Synology Drive 경로를 직접 가리킨다.

```yaml
collections:
  para:
    path: /Users/nettem/Library/CloudStorage/SynologyDrive-SynologyDrive/Obsidian/PARA
    pattern: "**/*.md"
  tang:
    path: /Users/nettem/source/bsi/tang
    pattern: "**/*.{php,md,sql,js}"
```

**해결**:
- 스킬이 `qmd` 실행 시 항상 현재 프로젝트 컬렉션명을 먼저 확정하고,
- 이후 모든 `update/embed/query`를 그 컬렉션명 기준으로 제한하도록 바꿔야 한다.
- 최소한 문서에는 `plain qmd embed 금지`를 명시해야 한다.

```bash
# 현재 전역 컬렉션 확인
sed -n '1,220p' ~/.config/qmd/index.yml

# 도움말 확인
qmd embed --help | sed -n '1,120p'
qmd update --help | sed -n '1,120p'
```

**실제 관측 결과**:
- 다른 AI 세션 로그: `qmd update`가 `Updating 6 collection(s)...`로 동작
- 사용자는 `~/source/bsi/tang`에서 작업 중이었지만, `para` 컬렉션도 전역 설정에 포함
- 결과적으로 Synology Drive의 `Obsidian/PARA`가 함께 스캔되었을 가능성이 높음

### 문제 #2: `tang` 컬렉션 자체도 문서 전용이 아니라 소스까지 포함함

**원인**:
- 현재 `tang` 컬렉션 패턴은 `**/*.{php,md,sql,js}` 이다.
- 따라서 `~/source/bsi/tang` 기준으로 `md`뿐 아니라 `php/js/sql` 소스 파일도 함께 인덱싱/임베딩 대상이 된다.

**해결**:
- 이 동작은 현재 설정상 정상이다.
- 다만 스킬 문서에 "code 모드에서는 문서 + 소스 전체를 처리한다"는 설명을 명확히 넣어야 한다.

```bash
sed -n '1,220p' ~/.config/qmd/index.yml
```

**판단**:
- `~/source/bsi/tang`에서 embed를 진행하면, 현재 설정상 해당 폴더 아래의 `md`, `php`, `sql`, `js`를 모두 처리하는 해석이 맞다.
- 단, plain `qmd embed`면 `tang`만이 아니라 등록된 다른 컬렉션도 같이 돌 가능성이 있다.

### 문제 #3: Synology/CloudStorage 경로와의 충돌 위험이 문서화돼 있지 않음

**원인**:
- `para` 컬렉션이 `/Users/nettem/Library/CloudStorage/SynologyDrive-SynologyDrive/Obsidian/PARA`를 바라보는 상태에서,
- `qmd embed`를 대량 실행하면 `fileproviderd`와 `SynologyDriveFileProvider`가 반응한다.
- 이번 조사에서도 `Reindex`, `Syncer`, `backgroundReindex()` 경로와 `fileproviderd`의 CloudStorage 접근이 관측됐다.

**해결**:
- 스킬 문서에 "CloudStorage/Sync 폴더 컬렉션은 embed 시 fileproviderd를 자극할 수 있음"을 주의사항으로 추가해야 한다.
- 특히 background 실행(`qmd embed &`)과 동기화 경로 임베딩이 겹치면 체감 성능 저하가 커진다는 점을 넣어야 한다.

```bash
lsof -p 667 | rg 'CloudStorage|SynologyDrive|OneDrive'
ps -Ao pid,%cpu,comm,args | rg 'fileproviderd|SynologyDriveFileProvider'
```

---

## 3. 스킬 업데이트 제안

### 3-1. SKILL.md 변경

**수정 대상 1**: INIT 섹션의 인덱싱 명령
- 현재: [skills/peach-wiki/SKILL.md](/Users/nettem/source/peachSolution2/peach-wiki/skills/peach-wiki/SKILL.md:99)
- 문제: `qmd update && qmd embed`가 전역 컬렉션을 건드릴 수 있음

**제안 문구**:

```md
### qmd 실행 범위 고정 (필수)

컬렉션 등록 후에는 반드시 "현재 프로젝트 컬렉션명"을 변수로 확정한다.
plain `qmd update`, `qmd embed`는 사용하지 않는다.
전역 `~/.config/qmd/index.yml`에 다른 컬렉션(예: para, 개인 옵시디언, 타 프로젝트)이 등록돼 있을 수 있기 때문이다.

예:
```bash
QMD_COLLECTION="tang"   # 또는 현재 프로젝트명
qmd update -c "$QMD_COLLECTION"
# embed는 컬렉션 범위를 지원하지 않으면 실행 전 컬렉션 구성을 별도 확인하고,
# 지원하면 반드시 컬렉션을 명시한다.
```
```

**수정 대상 2**: INGEST 섹션의 qmd 반영 규칙
- 현재: [skills/peach-wiki/SKILL.md](/Users/nettem/source/peachSolution2/peach-wiki/skills/peach-wiki/SKILL.md:186)
- 문제: 대량수정 시 `qmd update && qmd embed`가 그대로 안내됨

**제안 문구**:

```md
8. **qmd 반영** (qmd 설치 시):
   - 먼저 현재 컬렉션명이 실제 대상 프로젝트를 가리키는지 확인
   - 전역 컬렉션 목록에 Synology/OneDrive/개인 옵시디언 경로가 섞여 있으면 plain `qmd embed` 금지
   - 새 파일 추가·이동·대량수정:
     - `qmd update -c "$QMD_COLLECTION"`
     - embed는 현재 컬렉션으로 제한 가능한 경우에만 실행
     - 제한 불가 시 사용자가 전체 컬렉션 임베딩을 의도했는지 먼저 확인
   - 소규모 수정: `qmd update -c "$QMD_COLLECTION"`
```

**수정 대상 3**: 주의사항 섹션 신규 추가

```md
### 주의: CloudStorage/동기화 폴더

`~/Library/CloudStorage/*`, Synology Drive, OneDrive, iCloud Drive 아래의 컬렉션은
`qmd embed` 시 `fileproviderd`/동기화 재색인을 유발할 수 있다.
특히 background 실행(`qmd embed &`)은 체감 성능 저하를 크게 만들 수 있으므로,
해당 컬렉션은 별도 타이밍에만 임베딩하거나 사용자의 확인을 받는다.
```

### 3-2. references/ 추가/수정

신규 문서 제안:
- `skills/peach-wiki/references/qmd-프로젝트범위-실행규칙.md`

핵심 내용:
- 전역 qmd 컬렉션 구조 설명
- 현재 프로젝트 컬렉션명 확정 절차
- plain `qmd update/embed` 금지 사례
- Synology/CloudStorage 경로가 포함될 때의 부작용
- `code 모드 = md만이 아니라 소스까지 인덱싱` 규칙

### 3-3. 기존 레퍼런스 보완

대상:
- [qmd-가이드.md](/Users/nettem/source/peachSolution2/peach-wiki/skills/peach-wiki/references/qmd-가이드.md)

추가할 내용:
- `~/.config/qmd/index.yml` 예시와 컬렉션별 path/pattern 해석
- `tang` 패턴 `**/*.{php,md,sql,js}`처럼 소스까지 포함된다는 설명
- `para`가 Synology Drive 경로일 때 embed가 동기화 부하를 만들 수 있다는 사례

---

## 4. 검증 시나리오

### 시나리오 1: plain embed가 전역 컬렉션을 건드리는지 확인

**목적**: `peach-wiki` 문서의 기존 안내가 타 프로젝트/개인 노트까지 임베딩할 위험이 있는지 재현

**전제**:
- `~/.config/qmd/index.yml`에 `tang`, `para` 등 복수 컬렉션이 등록돼 있어야 함

```bash
sed -n '1,220p' ~/.config/qmd/index.yml
qmd update
qmd embed
```

**기대 결과**:
- update 단계에서 복수 컬렉션이 처리되는 로그가 보이거나,
- embed 직후 CloudStorage 경로 접근 및 fileproviderd 반응이 관측된다.

**통과 기준**:
- 스킬 문서가 plain embed를 그대로 안내하면 실패
- 수정 후에는 현재 프로젝트 컬렉션 범위 확인 절차가 먼저 나오면 통과

### 시나리오 2: `tang` 컬렉션 범위 확인

**목적**: `~/source/bsi/tang`에서 embed 시 어떤 파일이 대상이 되는지 명확히 검증

```bash
sed -n '1,220p' ~/.config/qmd/index.yml
```

**기대 결과**:
- `tang` 컬렉션의 `pattern: "**/*.{php,md,sql,js}"`가 확인된다.

**통과 기준**:
- 문서에 "code 모드는 md + source 모두 처리"가 명시되면 통과

### 시나리오 3: Synology 경로와 fileproviderd 영향 확인

**목적**: `para` 컬렉션이 Synology Drive 위에 있을 때의 부작용을 재확인

```bash
ps -Ao pid,%cpu,comm,args | rg 'fileproviderd|SynologyDriveFileProvider'
lsof -p 667 | rg 'CloudStorage|SynologyDrive|OneDrive'
```

**기대 결과**:
- `fileproviderd`가 CloudStorage/Synology 경로를 열고 있는 흔적이 보인다.

**통과 기준**:
- 수정된 문서에 CloudStorage 컬렉션 주의사항이 포함되면 통과

---

## 5. 다음 에이전트를 위한 작업 지시

이 문서를 읽은 AI 에이전트는 아래 순서로 작업한다:

1. 대상 스킬 현재 상태 확인
```bash
sed -n '1,260p' /Users/nettem/source/peachSolution2/peach-wiki/skills/peach-wiki/SKILL.md
```

2. `INIT`, `INGEST`에서 plain `qmd update`, `qmd embed` 안내가 있는 부분을 수정
   - 현재 프로젝트 컬렉션명 확정 단계 추가
   - plain embed 금지 문구 추가
   - CloudStorage 경로 주의사항 추가

3. `references/qmd-가이드.md` 또는 신규 레퍼런스에 아래를 추가
   - 전역 컬렉션 설정과 위험
   - code 모드의 실제 인덱싱 범위
   - Synology/OneDrive 경로 부하 사례

4. 검증 시나리오(4장) 실행

5. 반영 완료 후
   - frontmatter의 `status`를 `completed`로 변경
   - `completed_at`, `applied_by` 기입
   - 파일명에서 `TODO-` 접두어 제거

## 반영 기록
- 2026-04-14 Claude Sonnet 4.6: SKILL.md INIT/INGEST qmd 실행 전 컬렉션 확인 절차 추가, plain embed 전역 실행 주의 문구 추가, qmd-가이드.md에 전역 컬렉션 위험·CloudStorage 부하 섹션 신규 추가
