# Java 25 upgrade summary

## Result
- `./mvnw verify` passes on JDK 25 (Corretto 25.0.5): zero compilation errors, 79 tests, 0 failures, 2 skipped. The format and checkstyle checks pass too.
- I made no manual code changes. The OpenRewrite output compiled as-is.

## Changes made by OpenRewrite (reviewed)
- `pom.xml`: `java.version` 17 -> 25, jacoco 0.8.15 -> 0.8.16 (supports Java 25), Mockito attached as a `-javaagent` for surefire, using `maven-dependency-plugin:properties` (dynamic agent loading is restricted on JDK 21+).
- `build.gradle`: toolchain 17 -> 25 (Gradle wrapper 9.5.1 supports Java 25).
- Tests: instance `void main()` methods (JEP 512), unnamed catch variable `_`, `IO.println`, `List.getFirst()`.

## Next steps
- `MySqlIntegrationTests` and `PostgresIntegrationTests` were skipped because Docker isn't available. Run them in an environment that has Docker.
- I did not run the Gradle build in this cycle. Only the Maven build was checked.
- The Mockito javaagent is set only in the Maven surefire config. If Gradle tests show the dynamic-agent warning, add the same setup to `build.gradle`.
