# Java 25 upgrade summary

## Result
- `./mvnw verify`: BUILD SUCCESS, 79 tests run, 0 failures, 0 errors, 2 skipped.
- `./gradlew compileTestJava`: success.
- No manual code changes were required after the OpenRewrite pass.

## Changes applied by OpenRewrite (reviewed, kept)
- `pom.xml`: `java.version` 17 -> 25; empty `argLine` property, `maven-dependency-plugin:properties` and surefire
  `-javaagent:${org.mockito:mockito-core:jar}` so Mockito's inline mock maker loads as an agent (JDK 21+ warns/blocks dynamic agent loading).
- `build.gradle`: toolchain 17 -> 25.
- Tests: `get(0)` -> `getFirst()` (JDK 21 SequencedCollection), unnamed catch variable `_` (JDK 22),
  `System.out.println` -> `IO.println` and instance `void main()` (JDK 25 JEP 512).

## Next steps
- `MySqlIntegrationTests` (2 tests) were skipped and `PostgresIntegrationTests` ran 0 tests because Docker is not available
  in this environment; run them where Docker is available.
- The `main` methods in `PetClinicIntegrationTests`, `MysqlTestApplication` and `PostgresIntegrationTests` are now
  instance `void main()` methods (JEP 512). They work with the Java 25 launcher, but check that your IDE's run
  configurations still pick them up.
- The Gradle build does not have the Mockito `-javaagent` surefire setup that the Maven build has. If Gradle test runs
  print dynamic agent loading warnings, add the equivalent `jvmArgs` to the `test` task.
