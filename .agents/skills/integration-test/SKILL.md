---
name: integration-test
description: Pickple의 HTTP·MySQL·S3 통합 테스트를 작성, 실행하거나 실패 원인을 조사한다. 기존 Spring 테스트 지원, MySQL Testcontainers와 LocalStack을 재사용한다. Docker가 필요 없는 단위 테스트만으로 끝나는 변경에는 전체 절차를 적용하지 않는다.
---

# Integration Test

작업 중인 백엔드의 `AGENTS.md`, `build.gradle`, 가장 가까운 `*IT`를 먼저 읽는다. 하네스 저장소 자체에는 Gradle·제품 테스트가 없으므로 백엔드 경로를 확인하기 전에는 테스트 명령을 실행하지 않는다.

## 기존 환경을 재사용한다

- `src/test/java/app/pickple/support/IntegrationTest.java`: Spring 컨텍스트, 테스트 프로파일, MySQL 설정을 묶는 공용 애노테이션이다. HTTP 테스트에 필요한 설정은 가까운 컨트롤러 IT를 확인한다.
- 같은 폴더의 `ContainerConfig.java`, `LocalStackConfig.java`와 `src/test/resources/application-test.yml`을 확인한다. S3 검증에 필요한 경우만 기존 LocalStack 설정을 가져온다.
- `build.gradle`의 Java toolchain·테스트 선택 방식·JVM 타임존과 DB 설정을 확인한다. 테스트마다 컨테이너나 별도 프로파일을 새로 만들지 않는다.
- 순수 규칙은 `*Test`, Spring·DB·스토리지 경계는 `*IT`로 구분한다. 실제 작업 범위에 필요하지 않은 통합 테스트로 단위 테스트를 대체하지 않는다.

## 실행과 실패 판별

1. Wrapper와 Java 버전을 확인하고, 컨테이너 테스트에는 `docker version`으로 클라이언트와 데몬 연결을 확인한다. Docker 부재·권한 문제는 제품 코드 실패와 구분한다.
2. 변경에 가까운 테스트부터 선택한다. 아래 클래스는 기존 아키텍처 테스트 실행 예시이며, 기능 검증에서는 실제 대상 클래스명을 사용한다.

   Windows PowerShell:

   ```powershell
   .\gradlew.bat compileJava compileTestJava test --tests 'app.pickple.architecture.ArchitectureTest*'
   ```

   macOS/Linux:

   ```sh
   ./gradlew compileJava compileTestJava test --tests 'app.pickple.architecture.ArchitectureTest*'
   ```

3. 실패하면 첫 실패 테스트와 예외·관련 SQL·`build/test-results/test` 결과를 확인한다. 연결 실패, 애플리케이션 기동·마이그레이션 실패, 실제 assertion 실패를 나눈다. 원인 불명 상태가 지속되면 `resolve-problem` 절차로 좁힌다.
4. 최초 검증이 완료 조건과 확인된 회귀 위험을 충족하면 종료한다. 새 변경·실패·미해결 위험이 있으면 영향받는 기능의 회귀 테스트로 넓힌다. 전체 테스트는 공통 인증·스키마 등 넓은 영향이나 요청된 완료 조건이 있을 때 실행한다. 검증을 통과시키기 위한 테스트 제외·약한 assertion 변경을 하지 않는다.

## 시나리오를 검증한다

- 요청의 완료 조건마다 관찰할 응답·영속 상태·외부 호출 여부를 정하고 이를 검증하는 기존 테스트부터 확인한다. 예를 들어 검증 실패 시 외부 호출 금지가 계약이면 호출이 0회인지 단언한다. HTTP·DB 검증을 실제 앱·브라우저 E2E 통과로 표현하지 않는다.
- API: 정상 응답뿐 아니라 변경과 관련된 인증·인가, 잘못된 입력, 존재하지 않거나 삭제된 리소스의 경계를 확인한다.
- DB: HTTP 결과와 영속 상태를 함께 검증한다. 시간 경계는 기존 `Clock`과 저장 정밀도를 따르고, 동시성 검증은 관련 불변식을 확인하는 경우에 추가한다.
- S3: 기존 이미지 업로드 IT처럼 HTTP → LocalStack 객체 → DB 연결을 확인한다. 실패 시 보상 동작은 구현된 계약 범위에서 검증한다.
- Flyway 변경: 빈 DB 적용과 이미 존재하는 스키마에서의 업그레이드 경로를 구분한다. 적용된 마이그레이션을 편집해 이력을 숨기지 않는다.
- 비동기 완료는 고정 sleep 대신 관찰 가능한 조건과 제한 시간을 사용해 기다린다. 기존 대기 도구를 우선 재사용하고, 테스트 간 데이터·Clock·컨테이너 상태가 결과를 바꾸는지 확인한다.
- 실패를 제품 결함, 테스트의 잘못된 기대·격리 문제, 환경 문제로 구분한다. 기대값은 현재 요청·계약과 대조한 근거가 있을 때만 고치며, 통과시키기 위해 검증 강도를 낮추지 않는다.

보고에는 작업 브랜치, 실제 명령과 대상, 통과·실패·건너뜀, 환경 차단과 미검증 범위를 적는다. Gradle의 UP-TO-DATE·FROM-CACHE와 이번 실행을 구분하고, LocalStack 결과를 실 AWS 권한 검증으로 보고하지 않는다.
