# Java 25 upgrade – build verification

## Result
- Maven: `./mvnw verify` gives BUILD SUCCESS. 79 tests ran: 0 failures, 0 errors, 2 skipped (`MySqlIntegrationTests` needs Docker).
- Gradle: `./gradlew compileJava compileTestJava` succeeds with the Java 25 toolchain.
- The compiled classes are class file major version 69 (Java 25).
- No compilation errors came up, so no code changes were needed after the OpenRewrite pass.

## Changes applied by OpenRewrite (reviewed, kept)
- `pom.xml`: `java.version` 17 → 25. Added `maven-dependency-plugin:properties` and a surefire `argLine` that loads Mockito as a `-javaagent`. This avoids the JDK 21+ dynamic agent loading warning and the planned block on it.
- `build.gradle`: toolchain 17 → 25.
- Test sources:
  - Instance `void main()` methods (JEP 512) in `PetClinicIntegrationTests`, `MysqlTestApplication` and `PostgresIntegrationTests`.
  - `IO.println`.
  - `List.getFirst()`.
  - Unnamed catch variable `_`.

## Next steps
- The project was already on Spring Boot 4.1.0 (jakarta). The framework did not need to change.
- `PostgresIntegrationTests` and `MySqlIntegrationTests` need Docker, which is not available in this environment. Run them in CI.
- The Gradle build does not have the Mockito `-javaagent` surefire equivalent. Consider adding it to the `test` task if Mockito agent warnings show up.
- You need a Java 25 JDK to run the dev launchers in your IDE (the instance `main` methods in test classes).
