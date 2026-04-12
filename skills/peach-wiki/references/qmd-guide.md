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

## 컬렉션 등록

```bash
# 코드 프로젝트
qmd collection add . --name 프로젝트명 --mask "**/*.{ts,vue,md,sql,py,go,js}"

# Obsidian PARA
qmd collection add . --name para --mask "**/*.md"

# 컨텍스트 설명 추가 (검색 품질 핵심)
qmd context add qmd://프로젝트명/ "프로젝트 한 줄 설명"

# 인덱싱 + 임베딩
qmd update && qmd embed
```

---

## 검색

```bash
# 하이브리드 검색 (권장)
qmd query "키워드" -c 프로젝트명

# BM25 키워드 검색 (빠름)
qmd search "키워드" -c 프로젝트명

# 파일 경로만
qmd query "키워드" -c 프로젝트명 --files

# 특정 파일 읽기
qmd get qmd://프로젝트명/경로/파일.md

# 컬렉션 파일 목록
qmd ls 프로젝트명
```

---

## 인덱스 갱신 기준

| 상황 | 명령 |
|------|------|
| 새 파일 추가·이동·이름변경·대량수정 | `qmd update && qmd embed` |
| 소규모 텍스트 수정 | `qmd update` |
| 변경 없음 | 실행하지 않음 |

---

## 상태 확인

```bash
qmd status           # 전체 상태
qmd collection list  # 컬렉션 목록
```
