# Changelog

> [keep-a-changelog](https://keepachangelog.com) 포맷을 따릅니다.
> 버전은 [Semantic Versioning](https://semver.org)을 따릅니다.

## [v0.1.2] - 2026-04-13

### Added
- peach-release 릴리스 자동화 스킬 추가 (`.claude/skills/peach-release/`)

### Changed
- peach-release 스킬 플로우를 peach-harness 최신 기준으로 동기화 (분석+계획 통합 출력, 대화 보완 루프)
- peach-release `metadata.internal: true` 설정으로 skills.sh 배포 제외 처리
- README 설치 가이드 개선 (Claude Code 플러그인 권장, skills.sh 에이전트 ID 표, 업데이트 방법 추가)

## [v0.1.1] - 2026-04-12

### Changed
- README 설치 안내를 플러그인 + `skills.sh` 병행 기준으로 보강했습니다.
- 문서 구조를 번호형 한글 파일명 중심으로 재정리하고 레거시 `wiki-code` 문서를 제거했습니다.
- 제품 문서 전반의 옵시디언 표기를 정리하고 참고 문서 체계를 단순화했습니다.

### Fixed
- `INGEST` 단계에서 `related_files` 경로를 실제 파일 기준으로 검증하도록 스킬 설명을 보강했습니다.
