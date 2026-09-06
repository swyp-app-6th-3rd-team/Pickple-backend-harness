# Pickple Backend Codex Harness

Pickple 백엔드 저장소에서 사용하는 Codex 작업 규칙, 작업별 스킬, MCP 설정, 교차 플랫폼 훅을 보관하는 전용 저장소입니다. 제품 코드와 Gradle 테스트는 백엔드 저장소에서 관리합니다.

## 원본 스냅샷

- 원본 저장소: `swyp-app-6th-3rd-team/Pickple-backend`
- 원본 브랜치: `origin/develop`
- 원본 커밋: `9a0acbb3b9d4b0d9e4426a27300a216d92357e6a`
- 이관 기준일: `2026-09-05`

최초 이관은 위 커밋의 핵심 하네스 파일을 의미 변경 없이 복사했습니다. 이후 변경은 이 저장소의 커밋과 PR로 관리합니다. 이 저장소의 PR이 병합돼도 백엔드 작업 경로나 개인 Codex 설정에 자동 동기화되지는 않습니다.

## 구성

```text
Codex
├── MCP
│   ├── GitHub                      기존 연결 재사용
│   └── Context7                    .codex/config.toml
├── Skills                          .agents/skills/<이름>/SKILL.md
│   ├── spring-api-implementation   API 계약부터 구현·검증까지
│   ├── pr-review                   변경 사항 자체 리뷰
│   ├── integration-test            MySQL·LocalStack·HTTP 검증
│   └── resolve-problem             기존 증거 기반 문제 해결
├── AGENTS.md                       공통 규칙과 스킬 선택
└── Hooks                           .codex/hooks.json, .codex/hooks/*
```

스킬은 작업 절차이고 MCP는 외부 도구 연결입니다. 파일을 복사했다고 GitHub 인증이나 Context7 연결까지 완료되는 것은 아닙니다.

| 요청 예시 | 사용할 스킬 |
| --- | --- |
| `$spring-api-implementation 댓글 API를 추가해줘` | SPEC·기존 계층·인증·OpenAPI·관련 테스트를 함께 반영 |
| `$pr-review 이 변경을 PR 전에 검토해줘` | 실제 diff와 호출 경로에 근거한 결함 검토 |
| `$integration-test 이미지 업로드 실패 경로를 검증해줘` | 기존 MySQL·LocalStack 테스트 구성 재사용 |
| `$resolve-problem 배포 후에만 실패하는 원인을 좁혀줘` | 가설·실패 로그·최소 실험으로 원인 진단 |

명시적으로 지정하지 않아도 작업과 스킬의 description이 맞으면 선택할 수 있습니다. 모든 작업에 스킬 전체를 실행하지 않습니다.

## 적용

1. 적용할 백엔드의 실제 루트·브랜치·기존 변경을 확인합니다. 이 하네스 저장소와 백엔드 worktree를 구분합니다.
2. `AGENTS.md`, `.agents/skills`의 각 스킬, `.codex/hooks.json`과 `.codex/hooks`를 대상 파일과 비교해 반영합니다. 기존 사용자 규칙과 훅을 통째로 덮어쓰거나 중복 등록하지 않습니다.
3. `.codex/config.toml`의 Context7 테이블을 대상 프로젝트 설정에 병합합니다. 이미 개인 설정이나 플러그인으로 Context7을 사용한다면 연결 하나를 재사용합니다. 모델·권한·기존 MCP 설정을 덮어쓰지 않습니다.
4. `docs/codex-setup.md`도 함께 반영하고 연결·스킬 발견 상태를 확인합니다. 구체적인 인증과 확인 절차는 [설정 안내](docs/codex-setup.md)를 따릅니다.
5. `.gitattributes`에 아래 줄바꿈 규칙을 병합합니다.

```gitattributes
.codex/hooks/*.sh text eol=lf
```

Stop 훅은 네 스킬의 존재·기본 frontmatter, 훅 구성·스크립트 문법 등 로컬 하네스를 점검합니다. 일반 작업 종료에서는 발견한 문제를 비차단 경고로 알리며, 경고 자체는 요청 범위 밖 파일을 수정할 권한이 아닙니다. 하네스 변경의 완료 검증에는 `docs/codex-setup.md`의 엄격 검사 명령을 사용합니다. 개인 MCP 연결을 재사용할 수 있으므로 프로젝트 MCP 설정 파일은 필수 검사 대상으로 두지 않습니다. MCP 네트워크 연결이나 제품 테스트는 실행하지 않습니다.

`docs/adr`와 `docs/prd`도 필요합니다. 이 저장소의 두 README는 디렉터리 자리표시자이며, 백엔드의 실제 ADR·PRD에 덮어쓰지 않습니다.

## 적용 확인

- 대상 백엔드 루트를 Codex 프로젝트로 열고 신뢰 설정을 확인한 뒤 새 작업을 시작합니다. 상위 폴더에서 시작한 작업이 하위 백엔드의 스킬까지 자동 발견한다고 가정하지 않습니다.
- 스킬 목록에서 네 이름을 확인합니다. 보이지 않으면 작업 루트·복사 경로를 확인하고 Codex를 재시작합니다.
- `codex mcp get context7`은 설정 확인용입니다. 실제 도구 목록과 문서 조회 결과까지 확인해야 연결 검증이 완료됩니다.

스킬 형식과 발견 방식은 [OpenAI 스킬 문서](https://learn.chatgpt.com/docs/build-skills), MCP 설정은 [OpenAI MCP 문서](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)를 참고합니다.
