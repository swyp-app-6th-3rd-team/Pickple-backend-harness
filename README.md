# Pickple Backend Codex Harness

Pickple 백엔드 저장소에서 사용하는 Codex 작업 규칙, 문제 해결 스킬, 교차 플랫폼 훅을 보관하는 전용 저장소입니다.

## 원본 스냅샷

- 원본 저장소: `swyp-app-6th-3rd-team/Pickple-backend`
- 원본 브랜치: `origin/develop`
- 원본 커밋: `9a0acbb3b9d4b0d9e4426a27300a216d92357e6a`
- 이관 기준일: `2026-09-05`

핵심 하네스 파일은 위 커밋에서 의미 변경 없이 복사했습니다. 원본 프로젝트 저장소의 파일은 삭제하지 않았습니다.

## 구성

- `AGENTS.md`: Pickple 백엔드 작업 규칙
- `.agents/skills/resolve-problem/SKILL.md`: 증거 기반 문제 해결 절차
- `.codex/hooks.json`: `SessionStart`와 `Stop` 훅 등록
- `.codex/hooks/*`: Windows PowerShell 및 POSIX 셸 훅
- `.gitattributes`: 셸 훅의 LF 줄바꿈 보장
- `docs/adr`, `docs/prd`: Stop 훅이 요구하는 프로젝트 문서 디렉터리의 자리표시자

## 적용

Pickple 백엔드 프로젝트 루트에 `AGENTS.md`, `.agents`, `.codex`를 복사합니다. `.gitattributes`는 기존 내용을 덮어쓰지 말고 다음 규칙을 병합합니다.

```gitattributes
.codex/hooks/*.sh text eol=lf
```

Stop 훅은 대상 프로젝트에 `docs/adr`와 `docs/prd`가 존재하는지도 검사합니다. 실제 프로젝트 문서는 이 전용 저장소가 아니라 대상 프로젝트에서 관리합니다.
