# wiki-code 사용 가이드

> 작성일: 2026-04-08  
> 대상: staff-www 프로젝트 개발자  
> 목적: qmd + wiki-code 스킬을 활용한 코드베이스 지식 관리

---

## 개요

**wiki-code**는 코드베이스를 Raw Source로 삼아 `.wiki/` 폴더에 LLM이 관리하는 지식 레이어를 구축하는 시스템입니다.  
**qmd**는 코드 파일을 의미 기반으로 검색하는 로컬 벡터 DB 도구입니다.

두 도구를 결합하면:
- 코드가 변경되면 위키가 드리프트 감지 → 자동 갱신 제안
- AI가 코드를 읽지 않고도 qmd 검색으로 정확한 답변 가능
- 아키텍처·도메인 지식이 `.wiki/`에 누적

---

## 시스템 구성도

```
┌─────────────────────────────────────────────────────────────┐
│                     개발자 워크플로우                        │
└─────────────────────────────────────────────────────────────┘
         │ 코드 수정               │ 질문 / 문서화 요청
         ▼                         ▼
┌─────────────────┐       ┌─────────────────────┐
│   소스 코드      │       │  Claude Code (AI)   │
│  (staff-www)    │◄──────│  + wiki-code 스킬   │
│  *.ts *.vue     │ 읽기  │                     │
│  *.sql *.md     │       └────────┬────────────┘
└────────┬────────┘                │ 검색 / 질의
         │ qmd update              ▼
         ▼                ┌─────────────────────┐
┌─────────────────┐       │     qmd (로컬 AI)   │
│  qmd 인덱스     │◄──────│  벡터 임베딩 DB      │
│  (SQLite)       │ embed │  하이브리드 검색     │
│  staff-www      │       │  리랭킹 모델 포함    │
│  컬렉션         │       └─────────────────────┘
└─────────────────┘
         │ 검색 결과
         ▼
┌─────────────────────────────────────────────────────────────┐
│                      .wiki/ (지식 레이어)                    │
│                                                             │
│  WIKI-AGENTS.md     ← 운영 규칙 (AI가 세션 시작 시 읽음)   │
│  wiki-index.md      ← 전체 페이지 카탈로그                 │
│  wiki-log.md        ← 작업 타임라인 (append-only)          │
│                                                             │
│  concepts/          ← 아키텍처 패턴, 도메인 개념           │
│  entities/          ← 모듈·API·DB 스키마 페이지            │
│  synthesis/         ← 데이터 흐름, 의사결정 기록           │
│  diagrams/          ← Mermaid 다이어그램                   │
└─────────────────────────────────────────────────────────────┘
```

---

## qmd 작동 원리

```
┌──────────────────────────────────────────────────────────────┐
│                    qmd 인덱싱 파이프라인                      │
└──────────────────────────────────────────────────────────────┘

  소스 파일                qmd update              qmd embed
  *.ts / *.vue   ──────►  파일 변경 감지  ──────►  벡터 임베딩
  *.sql / *.md            (SHA 해시 비교)           (gemma-300M)
       │                        │                       │
       │                        ▼                       ▼
       │               ┌──────────────┐       ┌──────────────┐
       │               │ 문서 파싱    │       │ index.sqlite │
       │               │ 청크 분할    │       │ (벡터 저장)  │
       └──────────────►│ 메타데이터   │──────►│              │
                       │ 추출         │       │              │
                       └──────────────┘       └──────────────┘


┌──────────────────────────────────────────────────────────────┐
│                    qmd 검색 파이프라인                        │
└──────────────────────────────────────────────────────────────┘

  qmd query "인증 흐름"
       │
       ▼
  ┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
  │ 쿼리 확장   │────►│  하이브리드 검색  │────►│   리랭킹     │
  │ (qmd-1.7B)  │     │  벡터 유사도 +  │     │ (Qwen3-0.6B) │
  │             │     │  키워드 BM25    │     │              │
  └─────────────┘     └─────────────────┘     └──────┬───────┘
                                                      │
                                                      ▼
                                              상위 N개 청크
                                              (파일경로 + 줄번호)
```

---

## 5가지 오퍼레이션 플로우

### INIT — 최초 1회 세팅

```
개발자                  Claude Code              qmd
  │                         │                    │
  │  /wiki-system:wiki-code  │                    │
  │  init                   │                    │
  │────────────────────────►│                    │
  │                         │  qmd search        │
  │                         │  "architecture"    │
  │                         │───────────────────►│
  │                         │◄───────────────────│
  │                         │  프로젝트 구조 파악 │
  │                         │                    │
  │◄────────────────────────│                    │
  │  ".wiki/ 생성 계획 확인" │                    │
  │                         │                    │
  │  승인                   │                    │
  │────────────────────────►│                    │
  │                         │  .wiki/ 생성        │
  │                         │  WIKI-AGENTS.md    │
  │                         │  wiki-index.md     │
  │                         │  wiki-log.md       │
  │                         │  project-overview  │
  │◄────────────────────────│                    │
  │  완료                   │                    │
```

### INGEST — 모듈 문서화

```
  /wiki-system:wiki-code ingest auth 모듈
         │
         ▼
  1. qmd query "auth 모듈" -c staff-www
         │
         ▼
  2. 관련 파일 목록 수집
     - auth.service.ts
     - auth.controller.ts
     - auth.plugin.ts
         │
         ▼
  3. 각 파일 핵심 분석
     - 역할 / 인터페이스 / 의존성
         │
         ▼
  4. .wiki/entities/module-auth.md 생성
         │
         ▼
  5. wiki-index.md 갱신
  6. wiki-log.md 기록
```

### QUERY — 코드 구조 질문

```
  "auth 토큰 갱신 흐름 설명해줘"
         │
         ▼
  1. wiki-index.md 읽기
     → module-auth.md 발견
         │
         ▼
  2. .wiki/entities/module-auth.md 읽기
         │
         ▼
  3. 부족한 부분 → qmd query 보완
         │
         ▼
  4. 출처 포함 답변
     (파일경로:줄번호 인용)
         │
         ▼
  5. 가치 있으면 synthesis/ 저장 제안
```

### DRIFT — 코드 변경 후 위키 갱신

```
  (리팩토링 / 기능 추가 완료 후)
  /wiki-system:wiki-code drift
         │
         ▼
  1. git diff --name-only HEAD~1
     변경 파일 목록 추출
         │
         ▼
  2. grep -r "related_files" .wiki/
     영향받은 위키 페이지 파악
         │
         ▼
  3. 각 위키 페이지 업데이트
     updated 날짜 갱신
         │
         ▼
  4. 신규 모듈 감지 → INGEST 제안
  5. wiki-log.md 기록
```

### LINT — 정합성 점검

```
  /wiki-system:wiki-code lint
         │
         ▼
  ┌──────────────────────────────┐
  │ 점검 항목                    │
  │                              │
  │ ✓ 드리프트 탐지              │
  │   (git log vs wiki updated)  │
  │                              │
  │ ✓ 고아 페이지                │
  │   (wiki-index에 없는 페이지) │
  │                              │
  │ ✓ 깨진 소스 링크             │
  │   (삭제된 파일 참조)         │
  │                              │
  │ ✓ 미문서화 모듈              │
  │   (코드는 있는데 위키 없음)  │
  └──────────────────────────────┘
         │
         ▼
  리포트 출력 + 조치 목록 제안
```

---

## 일상적인 활용 패턴

### 신규 모듈 개발 후

```bash
# 1. 개발 완료 후 qmd 갱신
qmd update && qmd embed

# 2. 위키에 반영
# Claude Code에서:
/wiki-system:wiki-code ingest [모듈명] 문서화해줘
```

### 코드 리뷰 전 구조 파악

```bash
# Claude Code에서 바로 질문:
/wiki-system:wiki-code kacta_staff 테이블과 approval 서비스 관계 설명해줘
```

### 대규모 리팩토링 후

```bash
# 1. 변경사항 커밋 후
/wiki-system:wiki-code drift

# 2. 정합성 검증
/wiki-system:wiki-code lint
```

### 주간 위키 상태 점검

```bash
/wiki-system:wiki-code lint
# → 드리프트 위험 페이지 목록 확인
# → 미문서화 모듈 목록 확인
```

---

## qmd 주요 명령어

```bash
# 인덱스 갱신 (코드 변경 후 반드시 실행)
qmd update

# 벡터 임베딩 (update 후 실행)
qmd embed

# 현재 상태 확인
qmd status

# 시맨틱 검색
qmd query "인증 미들웨어 동작" -c staff-www

# 특정 파일 읽기
qmd get qmd://staff-www/api/src/modules/auth/auth.service.ts

# 컬렉션 파일 목록
qmd ls staff-www

# 컨텍스트 설명 추가 (검색 품질 향상)
qmd context add qmd://staff-www/ "한국세무사회 직원 관리 시스템 API 서버 + 프론트엔드"
qmd context add qmd://staff-www/api/src/modules/ "백엔드 도메인 모듈"
qmd context add qmd://staff-www/front/src/modules/ "Vue 3 프론트엔드 모듈"
```

---

## .wiki/ 폴더 구조

```
staff-www/
└── .wiki/
    ├── WIKI-AGENTS.md          ← AI 운영 규칙 (수정 금지)
    ├── wiki-index.md           ← 전체 페이지 카탈로그
    ├── wiki-log.md             ← 작업 로그 (append-only)
    │
    ├── concepts/               ← 아키텍처·도메인 개념
    │   ├── plugin-system.md    (Elysia 플러그인 패턴)
    │   ├── module-boundary.md  (독립 모듈 원칙)
    │   └── ...
    │
    ├── entities/               ← 모듈·API·DB 페이지
    │   ├── project-overview.md
    │   ├── module-auth.md
    │   ├── module-staff.md
    │   ├── api-approval.md
    │   ├── schema-kacta-staff.md
    │   └── ...
    │
    ├── synthesis/              ← 흐름·의사결정 기록
    │   ├── 2026-04-08-auth-flow.md
    │   └── ...
    │
    └── diagrams/               ← Mermaid 다이어그램
        ├── module-dependency.md
        └── ...
```

> `.gitignore`에 `.wiki/` 추가 여부는 팀에서 결정.  
> 팀 공유 목적이면 git 추적, 개인 메모용이면 ignore.

---

## 시작하기

### 1. 스킬 설치 (개발자 머신 최초 1회)

**방법 A — Claude Code CLI로 설치 (권장)**

```bash
# Claude Code 터미널에서:
/install-github-app
# 또는 마켓플레이스에서:
claude mcp install wiki-system
```

**방법 B — 직접 설치**

```bash
# wiki-system 플러그인 설치
claude plugin install wiki-system
```

> 설치 후 `/wiki-system:wiki-code` 명령이 Claude Code에서 활성화됩니다.

---

### 2. qmd 설치 및 초기화

```bash
# qmd 설치 (최초 1회)
brew install qmd  # 또는 공식 설치 방법 따름

# staff-www 컬렉션 등록
cd /path/to/staff-www
qmd collection add . --name staff-www --mask "**/*.{ts,vue,md,sql}"

# 컨텍스트 설명 추가 (검색 품질 향상)
qmd context add qmd://staff-www/ "한국세무사회 직원 관리 시스템 API 서버 + 프론트엔드"
qmd context add qmd://staff-www/api/src/modules/ "백엔드 도메인 모듈"
qmd context add qmd://staff-www/front/src/modules/ "Vue 3 프론트엔드 모듈"

# 인덱싱
qmd update && qmd embed
```

---

### 3. wiki 초기화 및 모듈 문서화

```bash
# Step 1. qmd 임베딩 최신화
qmd update && qmd embed

# Step 2. wiki 초기화 (Claude Code에서)
/wiki-system:wiki-code init

# Step 3. 핵심 모듈 문서화
/wiki-system:wiki-code ingest auth 모듈 문서화해줘
/wiki-system:wiki-code ingest kacta-staff 모듈 문서화해줘
```
