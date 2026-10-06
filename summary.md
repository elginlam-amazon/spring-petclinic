# Java 25 upgrade summary

## Result
- `./mvnw clean verify` on JDK 25 (Corretto 25.0.4.1): BUILD SUCCESS. 0 compilation errors. 79 tests ran: 0 failures, 0 errors, 2 skipped.
- `./gradlew build -x test` (Gradle 9.5.1, toolchain 25): BUILD SUCCESSFUL.
- No manual code fixes were needed after the OpenRewrite pass.

## Stack
- Spring Boot 4.1.0 (Spring Framework 7, jakarta namespace). It was already on this version before the upgrade, so there was no javax to jakarta migration and no Spring version change.
- `javax.cache:cache-api` (JSR-107) stays as it is. It is a separate API and not part of the jakarta migration.

## OpenRewrite changes (reviewed and kept)
- `pom.xml`: set `java.version` to 25. Added `maven-dependency-plugin:properties` and a surefire `argLine` that loads Mockito as a `-javaagent`, since JDK 21+ warns about agents that load themselves dynamically. Added an empty `<argLine>` property so `@{argLine}` resolves when JaCoCo is not active.
- `build.gradle`: set the toolchain to 25.
- Tests now use Java 21–25 language features:
  - `List.getFirst()` (sequenced collections)
  - `catch (Exception _)` (unnamed variables)
  - `IO.println` and instance `void main()` in the IDE launcher classes (`PetClinicIntegrationTests`, `MysqlTestApplication`, `PostgresIntegrationTests`)

## Next steps
- The 2 skipped tests are `MySqlIntegrationTests` and `PostgresIntegrationTests`. They need Docker, which this environment does not have. Run them in CI or on a machine with Docker.
- The test launcher `main` methods are now instance methods (JEP 512). They work on JDK 25 and in recent IDEs, but older IDE run configurations may need to be recreated.
- Gradle compile shows a harmless `org.apiguardian.api.API$Status not found` warning. It was there before the upgrade and is unrelated to it.
- Update `README.md` ("Java 17 or later is required") and the CI workflow JDK versions under `.github/workflows` to 25. This cycle left them unchanged.
- `rewrite.yml` is untracked. Decide whether to commit it or remove it.
