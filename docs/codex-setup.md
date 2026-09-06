# Codex 연결과 스킬 적용

이 하네스는 GitHub 연결을 재사용하고, Context7 문서 조회 설정과 네 작업 스킬을 제공합니다. 새 스킬은 API 구현·PR 리뷰·통합 테스트 세 개이며 기존 `resolve-problem`을 유지합니다. 설치·연결·실행 여부는 각 개발 환경에서 확인합니다. 아래 명령은 하네스를 반영한 **대상 백엔드 루트**에서 실행합니다.

## GitHub

1. Codex에 GitHub가 연결돼 있으면 동일한 용도의 MCP를 추가하지 않습니다.
2. 공개 저장소 조회만으로 필요한 모든 권한이 확인되지는 않습니다. 대상 저장소의 Issue·PR·diff·Actions 조회에 필요한 도구가 현재 세션에 있는지 확인합니다.
3. 연결에서 지원하지 않거나 권한이 제한된 작업은 이미 로그인된 GitHub CLI로 처리할 수 있습니다. `gh auth status`와 대상 저장소 조회로 확인합니다. 인증 출력에 포함된 계정·토큰 정보를 공유 문서에 복사하지 않습니다.
4. 어떤 연결을 사용하든 커밋·푸시·PR 생성·리뷰어 요청·댓글·병합은 사용자가 허용한 작업 범위에 따릅니다. `pr-review` 스킬 자체는 GitHub에 리뷰를 게시하지 않습니다.

GitHub 앱의 계정 인증과 대상 저장소 접근 권한은 각 개발자가 설정합니다. 연결된 앱 도구는 로컬 `codex mcp list`에 같은 이름으로 나타나지 않을 수 있으므로 실제 세션의 도구도 확인합니다. GitHub에서 Codex에게 PR 리뷰를 맡기는 설정과 로컬 작업의 GitHub 도구 연결은 별도로 확인합니다.

## Context7

공유하는 [.codex/config.toml](../.codex/config.toml)은 공식 호스팅 주소로 접속하며 문서 검색 도구 두 개만 사용합니다. 공개 문서 조회를 위한 기본 설정에는 키를 넣지 않습니다. Node·npx를 실행하는 로컬 MCP 프로세스도 추가하지 않습니다.

```toml
[mcp_servers.context7]
url = "https://mcp.context7.com/mcp"
enabled_tools = ["resolve-library-id", "query-docs"]
startup_timeout_sec = 20
tool_timeout_sec = 60
required = false
```

이미 같은 이름의 설정이나 Context7 플러그인이 있다면 해당 연결을 확인해 재사용합니다. 별도 연결을 하나 더 등록하거나 기존 사용자 설정을 통째로 교체하지 않습니다. 프로젝트 `.codex/config.toml`은 신뢰한 프로젝트에서 로드됩니다.

### 설정과 실제 조회를 각각 확인

```text
codex mcp get context7
```

이 명령은 설정만 읽습니다. 서버를 재시작하거나 새 Codex 작업을 시작한 뒤 다음을 확인합니다.

1. `resolve-library-id`, `query-docs`가 제공되는지 확인합니다.
2. 예시 요청: `현재 build.gradle의 Spring Boot 버전을 확인하고 Context7에서 그 버전의 설정 문서를 찾아 출처와 버전 일치 여부를 알려줘.`
3. 검색 결과에서 적합한 라이브러리 ID를 선택하고, 제공되는 버전에 맞춰 문서를 조회합니다. 특정 버전이 없으면 있다고 추정하지 않고 Spring 공식 문서로 보완합니다.

두 단계의 결과를 구분해서 보고합니다. 설정 파싱 성공, 서버 초기화·도구 목록 성공, 실제 라이브러리 검색·문서 조회 성공은 서로 다른 검증입니다.

### 인증 또는 사용량 제한이 있을 때

공개 조회의 익명 사용량은 서비스 정책에 따라 제한됩니다. 인증이 필요하면 로컬에서 다음 명령으로 로그인합니다.

```text
codex mcp login context7
```

로그인은 사용자가 자신의 브라우저에서 완료하며 자격 증명은 개인 Codex 환경에서 관리합니다. 키가 필요한 환경에서는 개인 설정의 해당 테이블에 `bearer_token_env_var = "CONTEXT7_API_KEY"`를 사용하고, 변수 값은 개인 환경에서 설정합니다. 변수 이름과 키 값은 별개이며 실제 값을 PR·설정 파일·로그에 넣지 않습니다. 키 방식을 선택한 경우 Codex 프로세스가 그 환경 변수를 받는지도 확인합니다.

401·403이면 인증 또는 권한을 확인하고, 429이면 서비스의 재시도 안내를 따릅니다. DNS·프록시·네트워크 실패는 설정 문법 오류와 구분합니다. 같은 실패를 반복 호출하지 않고 공식 문서 원문으로 필요한 조사를 계속합니다. `required = false`이므로 이 서버의 시작 실패를 Codex 전체 시작의 필수 조건으로 만들지 않습니다.

### 문서 사용 원칙

- `build.gradle`의 버전과 포크를 우선합니다. 예를 들어 QueryDSL은 OpenFeign 포크와 원본 프로젝트를 구분합니다.
- 검색어는 공개 라이브러리명·버전·일반적인 기술 질문으로 구성합니다. 토큰·비밀값·비공개 코드를 전송하지 않습니다.
- 검색 결과의 출처와 버전을 확인합니다. 문서 조회 성공만으로 생성한 코드나 프로젝트 테스트가 검증된 것은 아닙니다.

## 스킬

스킬은 `.agents/skills/<이름>/SKILL.md`에서 읽습니다. `AGENTS.md`는 공통 규칙과 선택 기준을 제공하며, 각 스킬의 본문은 해당 작업에 필요할 때 사용합니다. 하네스만 별도 폴더에 복제해 두면 백엔드 작업에 자동 적용되는 것은 아닙니다.

## 로컬 훅 확인

PowerShell:

```powershell
'{"stop_hook_active":false}' | powershell.exe -NoProfile -ExecutionPolicy Bypass -File .codex/hooks/stop-validation.ps1 -Strict
```

macOS/Linux:

```sh
printf '%s' '{"stop_hook_active":false}' | sh .codex/hooks/stop-validation.sh --strict
```

성공 출력은 `{"continue":true}`이고 종료 코드는 0입니다. 엄격 검사에서 문제가 발견되면 종료 코드 1로 실패합니다. Codex의 일반 Stop 훅은 같은 문제를 비차단 경고로 보고하며, 그 경고는 요청 범위 밖 파일 수정 권한을 뜻하지 않습니다. 이 확인은 하네스 파일의 기본 구조를 검사하며, 실제 MCP 연결이나 제품 테스트 성공을 뜻하지 않습니다.

## 공식 문서

- [OpenAI: MCP 설정·프로젝트 범위·인증](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)
- [OpenAI: 스킬 형식과 발견](https://learn.chatgpt.com/docs/build-skills)
- [Context7: MCP 서버·공개 조회와 선택적 인증](https://github.com/upstash/context7/blob/master/packages/mcp/README.md)
- [Context7: Codex 연결](https://context7.com/docs/clients/codex)
