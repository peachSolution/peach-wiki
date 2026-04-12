# qmd 설치·활용 가이드

> `peach-wiki`에서 qmd를 어떻게 쓰는지에 집중한 운영 문서다.
> 관련 문서: [패턴 분석](03-LLM위키-패턴-분석.md), [종합 분석](05-qmd와-LLM위키-종합분석.md)
> 출처: `wikiSystem` POC에서 이관 후 현재 저장소 기준으로 재작성

## qmd가 필요한 이유

`peach-wiki`는 위키 자체를 누적하지만, 소스 탐색은 여전히 효율적이어야 한다. qmd는 코드와 마크다운을 하이브리드 검색해서 다음 두 가지를 해결한다.

- 토큰을 아끼면서 관련 소스를 먼저 좁힌다.
- 단순 키워드와 의미 기반 검색을 함께 써서 누락을 줄인다.

## 설치

```bash
npm install -g @tobilu/qmd
qmd --version
```

권장 전제:

- Node.js 22 이상
- 로컬 디스크 여유 공간 약 3GB
- Apple Silicon 또는 GPU 가속 환경이면 더 빠름

## 컬렉션 등록

### 코드 프로젝트

```bash
qmd collection add . --name 프로젝트명 --mask "**/*.{ts,vue,md,sql,py,go,js}"
qmd context add qmd://프로젝트명/ "프로젝트 한 줄 설명"
qmd update
qmd embed
```

### 옵시디언 노트

```bash
qmd collection add . --name para --mask "**/*.md"
qmd context add qmd://para/ "옵시디언 노트 컬렉션"
qmd update
qmd embed
```

## `peach-wiki`에서의 기본 사용 순서

1. `qmd status`
2. `qmd collection list`
3. `qmd query "키워드" -c 컬렉션명`
4. 필요 시 `qmd get qmd://컬렉션명/경로/파일`

이 원칙은 `skills/peach-wiki/SKILL.md`의 qmd 우선 규칙과 동일하다.

## 자주 쓰는 명령

```bash
qmd status
qmd collection list
qmd query "키워드" -c 프로젝트명
qmd search "키워드" -c 프로젝트명
qmd query "키워드" -c 프로젝트명 --files
qmd get qmd://프로젝트명/경로/파일.md
qmd update
qmd embed
```

## 언제 `embed`까지 돌릴까

| 상황 | 명령 |
|---|---|
| 새 파일 추가, 파일 이동, 대량 수정 | `qmd update && qmd embed` |
| 소규모 텍스트 수정 | `qmd update` |
| 변경 없음 | 실행하지 않음 |

## `peach-wiki`와의 관계

- qmd는 검색 레이어다.
- `docs/wiki/`는 지식 레이어다.
- `AGENTS.md`, `SKILL.md`, `WIKI-AGENTS.md`는 운영 규칙 레이어다.

즉, qmd는 위키를 대체하지 않는다. 소스를 더 잘 찾게 도와줄 뿐이다.

## 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| `qmd: command not found` | 미설치 또는 PATH 문제 | `npm install -g @tobilu/qmd` |
| 새 파일 검색 안 됨 | 인덱스 미갱신 | `qmd update` |
| 벡터 검색 품질이 낮음 | 임베딩 미생성 | `qmd embed` |
| 검색이 느림 | CPU only 환경 | `--no-rerank`로 임시 완화 후 환경 개선 |

## 참고

- 제품 사용 진입점: [루트 README](../README.md)
- 제품 규칙 진입점: [peach-wiki 스킬](../skills/peach-wiki/SKILL.md)
- 레퍼런스: [qmd 가이드](../skills/peach-wiki/references/qmd-가이드.md)
