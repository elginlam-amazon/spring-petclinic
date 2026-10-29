# Java 25 upgrade: build verification

## State
- Spring PetClinic on Spring Boot 4.1.0 (Jakarta namespace), `java.version` 25. JDK in use: Corretto 25.0.5.
- The OpenRewrite changes were: `java.version` set to 25 in `pom.xml` and in the `build.gradle` toolchain; jacoco bumped from 0.8.15 to 0.8.16; Mockito added as a `-javaagent` through surefire `argLine` together with `maven-dependency-plugin:properties`; Java 25 idioms in test code (instance `void main()`, unnamed `_` catch variable, `IO.println`, `List.getFirst()`).
- No code changes were needed in this cycle.

## Verification
- `./mvnw verify`: BUILD SUCCESS. 79 tests ran, 0 failures, 2 skipped (MySQL Testcontainers tests, because Docker is not available). The Postgres test was also skipped by `assumeTrue` for the same reason. Checkstyle, spring-javaformat and jacoco passed.
- `./gradlew compileJava compileTestJava`: success.

## Next steps
- Run `MySqlIntegrationTests` and `PostgresIntegrationTests` in an environment that has Docker.
- `PostgresIntegrationTests.main` became `void main(String[] args)`, while the other test launchers became `void main()`. Both forms are valid in Java 25.
