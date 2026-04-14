# qmd CLI 레퍼런스

> qmd(Quick Markdown Search): 마크다운·코드 파일 로컬 하이브리드 검색 도구
> BM25 + 벡터 + LLM 리랭킹 조합

---

## 설치

```bash
npm install -g @tobilu/qmd
qmd --version
```

전제: Node.js v22+, 디스크 ~3GB, Apple Silicon Metal 권장

---

## Named Index 분리 운영 (필수 패턴)

### 왜 `--index`를 써야 하는가

`qmd update/embed`는 컬렉션 범위 지정 플래그가 없다.
plain 실행 시 `~/.config/qmd/index.yml`에 등록된 **전체 컬렉션**이 처리된다.
여러 프로젝트·개인 옵시디언·CloudStorage 경로가 섞인 환경에서는 의도치 않은 재인덱싱이 발생한다.

`--index <name>` 플래그를 사용하면 **DB와 설정 파일이 동시에 분리**된다:

| 항목 | 기본 (`--index` 없음) | `--index tang` |
|------|----------------------|----------------|
| DB 파일 | `~/.cache/qmd/index.sqlite` | `~/.cache/qmd/tang.sqlite` |
| 설정 파일 | `~/.config/qmd/index.yml` | `~/.config/qmd/tang.yml` |

- DB는 `--index` 지정 즉시 생성된다.
- 설정 파일(yml)은 `collection add` 시점에 생성된다.
- 모델 파일(`~/.cache/qmd/models/`)은 인덱스 간 **공유** — 추가 디스크 불필요.
- 공식 문서(README.md 739행): `qmd --index work search "quarterly reports"` — "Use separate index for different knowledge base"

### 표준 패턴

```bash
# 프로젝트명을 인덱스명으로 고정 (세션 시작 시 선언)
QMD_INDEX=$(basename $(pwd))   # 예: tang, peach-wiki, my-project

# 초기 등록
qmd --index "$QMD_INDEX" collection add . --name "$QMD_INDEX" --mask "**/*.{ts,vue,md,sql,py,go,js}"
qmd --index "$QMD_INDEX" context add "qmd://$QMD_INDEX/" "프로젝트 한 줄 설명"
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed

# 갱신 (대량 변경)
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed

# 갱신 (소규모 수정)
qmd --index "$QMD_INDEX" update

# 검색
qmd --index "$QMD_INDEX" query "키워드" -c "$QMD_INDEX"

# 상태 확인
qmd --index "$QMD_INDEX" status
qmd --index "$QMD_INDEX" collection list
```

### `--index` 누락 위험

`--index`를 빠뜨리면 기본 `index.sqlite`에 컬렉션이 등록되고,
이후 plain `qmd update/embed`가 기본 인덱스의 전체 컬렉션(다른 프로젝트 포함)을 처리한다.
반드시 `--index "$QMD_INDEX"`를 붙여라.

### MCP 서버 연동 주의

`qmd mcp`는 항상 기본 `index.sqlite`만 참조한다.
named index 데이터를 MCP로 노출하려면 별도 인스턴스를 띄워야 한다:

```bash
qmd --index "$QMD_INDEX" mcp
```

---

## 컬렉션 등록

```bash
# 코드 프로젝트
qmd --index "$QMD_INDEX" collection add . --name "$QMD_INDEX" --mask "**/*.{ts,vue,md,sql,py,go,js}"

# 옵시디언 노트
qmd --index "$QMD_INDEX" collection add . --name "$QMD_INDEX" --mask "**/*.md"

# 컨텍스트 설명 추가 (검색 품질 핵심)
qmd --index "$QMD_INDEX" context add "qmd://$QMD_INDEX/" "프로젝트 한 줄 설명"

# 인덱싱 + 임베딩
qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed
```

> **plain 등록 사용 시 주의**: `--index` 없이 실행하면 기본 인덱스의 전체 컬렉션이 처리된다.
> CloudStorage/Synology 경로가 포함된 경우 fileproviderd 부하가 발생할 수 있다.
> 반드시 `--index "$QMD_INDEX"` 패턴을 사용한다.

---

## 검색

```bash
# 하이브리드 검색 (권장)
qmd --index "$QMD_INDEX" query "키워드" -c "$QMD_INDEX"

# BM25 키워드 검색 (빠름)
qmd --index "$QMD_INDEX" search "키워드" -c "$QMD_INDEX"

# 파일 경로만
qmd --index "$QMD_INDEX" query "키워드" -c "$QMD_INDEX" --files

# 특정 파일 읽기
qmd --index "$QMD_INDEX" get "qmd://$QMD_INDEX/경로/파일.md"

# 컬렉션 파일 목록
qmd --index "$QMD_INDEX" ls "$QMD_INDEX"
```

---

## 인덱스 갱신 기준

| 상황 | 명령 |
|------|------|
| 새 파일 추가·이동·이름변경·대량수정 | `qmd --index "$QMD_INDEX" update && qmd --index "$QMD_INDEX" embed` |
| 소규모 텍스트 수정 | `qmd --index "$QMD_INDEX" update` |
| 변경 없음 | 실행하지 않음 |

---

## 전역 컬렉션 설정과 위험 (plain 실행 시)

`--index` 없이 plain `qmd update/embed`를 실행하면 `~/.config/qmd/index.yml`에
등록된 전체 컬렉션이 처리된다.

**실제 사례**:
```yaml
# ~/.config/qmd/index.yml 예시 — 여러 컬렉션이 섞인 상태
collections:
  para:
    path: /Users/nettem/Library/CloudStorage/SynologyDrive-SynologyDrive/Obsidian/PARA
    pattern: "**/*.md"
  tang:
    path: /Users/nettem/source/bsi/tang
    pattern: "**/*.{php,md,sql,js}"
```

`tang` 프로젝트에서 `qmd update`를 실행하면 `Updating 6 collection(s)...` 형태로
`para` 컬렉션까지 함께 처리될 수 있다.

**해결책**: `--index "$QMD_INDEX"` 패턴을 사용하면 이 문제가 완전히 해소된다.
각 프로젝트는 별도 인덱스(`tang.yml`, `peach-wiki.yml` 등)를 가지므로
서로 영향을 주지 않는다.

**code 모드 인덱싱 범위**: mask 패턴에 `php/js/sql`이 포함되면 md뿐 아니라
소스 파일 전체가 인덱싱 대상이 된다. 의도된 동작이지만 첫 실행 시 확인 필요.

---

## CloudStorage/동기화 폴더 주의사항

`~/Library/CloudStorage/*`, Synology Drive, OneDrive, iCloud Drive 경로가
컬렉션에 등록된 상태에서 `qmd embed`를 실행하면:

- `fileproviderd`, `SynologyDriveFileProvider` 등 동기화 데몬이 재색인을 시작한다
- 대량 파일 스캔으로 CPU 상승 및 체감 성능 저하가 발생할 수 있다
- background 실행(`qmd embed &`)과 동기화 경로 임베딩이 겹치면 영향이 더 커진다

**권장 대응**:
1. `--index` 패턴으로 프로젝트별 인덱스를 분리하면 다른 인덱스의 CloudStorage 컬렉션은 건드리지 않는다
2. CloudStorage 컬렉션 자체는 별도 타이밍(동기화 안정 후)에만 임베딩
3. 이상 감지 시 확인 명령:
   ```bash
   ps -Ao pid,%cpu,comm,args | grep 'fileproviderd\|SynologyDriveFileProvider'
   ```

---

## 상태 확인

```bash
qmd --index "$QMD_INDEX" status    # 해당 인덱스 상태
qmd --index "$QMD_INDEX" collection list  # 해당 인덱스 컬렉션 목록
```
