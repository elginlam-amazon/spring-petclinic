# ☕ Java 25 Upgrade Report: spring-petclinic (Iteration 23)  
  
## 📋 Executive summary  
  
- 🎯 The OpenRewrite recipe `com.amazonaws.java.migrate.UpgradeToJava25` moved the build target of `spring-petclinic` (Spring Boot `spring-boot-starter-parent` 4.1.0) from Java 17 to Java 25.  
- ✅ The post-upgrade verification build succeeded and ran 79 tests with 0 failures and 0 errors. 2 tests were skipped.  
- 🤖 The Kiro CLI build-fix pass found nothing to fix. The build already compiled and passed its tests, so Kiro CLI made no code changes.  
- 🌱 Spring: the Spring Boot upgrade step was skipped because Spring Boot 4.1 is already at or above the 4.0 target (code-upgrade.log). Runtime validation reported: "OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it)."  
- ⚠️ Runtime behaviour is not verified: "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." The application was not started in this iteration.  
- 📦 The iteration branch commit `e6bd121` ("Upgrade to Java 25 on Iteration 23") changes 7 files, with 35 insertions and 15 deletions.  
  
## 🛠️ Application changes  
  
- 🔧 **Build configuration**  
  - `pom.xml`: Java version 17 -> 25, JaCoCo 0.8.15 -> 0.8.16, and the Mockito Java agent is now attached to surefire.  
  - `build.gradle`: Gradle toolchain 17 -> 25.  
- 🧪 **Test sources** (no production code under `src/main` changed):  
  - `src/test/java/org/springframework/samples/petclinic/MysqlTestApplication.java`  
  - `src/test/java/org/springframework/samples/petclinic/PetClinicConcurrencyTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/PetClinicIntegrationTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/PostgresIntegrationTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/service/ClinicServiceTests.java`  
- ➕ **Dependencies added** (report.json):  
  - `org.apache.maven.plugins:maven-dependency-plugin` (version LATEST). In the diff this is a build plugin with no version and no dependency scope. It runs the `properties` goal and is not packaged with the application.  
  - `org.apache.maven.plugins:maven-surefire-plugin` (version LATEST). In the diff this is a build plugin with no version and no dependency scope. It sets `argLine` to `@{argLine} -javaagent:${org.mockito:mockito-core:jar}` and is not packaged with the application.  
- ➖ **Dependencies removed:** none.  
- ⬆️ **Dependencies upgraded:** none.  
- 🛡️ Neither added item is packaged with the application. Because they add no runtime API to the artifact, they carry no classloader-conflict risk.  
  
## 🧰 Tools used  
  
- ☕ **Java SDK:** Amazon Corretto 25.0.5 (OpenJDK 25.0.5 LTS), Apache Maven 3.9.16.  
- 🔄 **OpenRewrite:** `rewrite-maven-plugin` 6.42.0 running `com.amazonaws.java.migrate.UpgradeToJava25`. The sub-recipes that made changes include:  
  - `UpdateMavenProjectPropertyJavaVersion`  
  - `UpgradeJaCoCo` / `UpgradePluginVersion`  
  - `AddMockitoJavaAgentToMavenSurefirePlugin`  
  - `ListFirstAndLast`  
  - `MigrateMainMethodToInstanceMain`  
  - `ReplaceSystemOutWithIOPrint`  
  - `ReplaceUnusedVariablesWithUnderscore`  
  - `UpdateJavaCompatibility` (Gradle)  
- 🤖 **Amazon Kiro CLI:** kiro-cli 2.35.1 with model `claude-opus-5.5`. It succeeded on its first attempt and only built and verified the project.  
- 🔍 **CodeKB:** the CodeKB tools were not in the tool list for this run, so the report is based only on the logs, report.json and the git diff.  
  
## 💻 Code changes  
  
- 🚪 **Instance main methods (JEP 512)**, applied by `MigrateMainMethodToInstanceMain`:  
  - `MysqlTestApplication.java` and `PetClinicIntegrationTests.java`: `public static void main(String[] args)` -> `void main()`.  
  - `PostgresIntegrationTests.java`: `public static void main(String[] args)` -> `void main(String[] args)`.  
- 🖨️ **Console output:** in `PetClinicConcurrencyTests.java`, 5 `System.out.println(...)` calls -> `IO.println(...)`.  
- 🕳️ **Unnamed variable:** in `PetClinicConcurrencyTests.java`, `catch (Exception e)` -> `catch (Exception _)`.  
- 📚 **Sequenced collections:** in `ClinicServiceTests.java`, 3 `.get(0)` calls -> `.getFirst()`. `getSpecialties().get(1)` was left as it was.  
- 🏗️ **Build files:**  
  - `pom.xml` adds an empty `<argLine></argLine>` property, sets `java.version` to 25 and `jacoco.version` to 0.8.16, and adds the 2 plugin blocks listed above.  
  - `build.gradle` changes `JavaLanguageVersion.of(17)` -> `JavaLanguageVersion.of(25)`.  
- ℹ️ code-upgrade.log attributes the `build.gradle` change to a recipe chain that names a Byte Buddy 1.17.x transitive upgrade. The branch diff shows only the toolchain line changing in `build.gradle`, so no Byte Buddy constraint is on the branch.  
  
## ⚠️ Risks  
  
- 🚫 **Runtime not verified:** "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." The build and tests passed, but nothing in this iteration shows that the application starts on Java 25.  
- 🐳 **Docker-dependent tests did not run:** the 2 skipped tests are in `MySqlIntegrationTests`. `PostgresIntegrationTests` ran 0 tests because Testcontainers "Could not find a valid Docker environment". The MySQL and PostgreSQL paths, and the changed `PostgresIntegrationTests.java`, have no test coverage in this iteration.  
- 🧩 **Instance-main behaviour:** the test helper classes now use instance `void main()`, which requires Java 25. IDE run configurations or scripts that call these classes as `public static void main` may need updating. `MysqlTestApplication` and `PetClinicIntegrationTests` no longer accept command-line arguments.  
- 🐘 **Gradle build not run:** `build.gradle` now targets toolchain 25, but only the Maven build was verified. The Gradle wrapper is 9.5.1.  
- 🔌 **Unpinned plugin versions:** the 2 added plugins have no explicit version in `pom.xml` (report.json records "LATEST"). Their versions come from the Spring Boot parent's plugin management, so a parent upgrade can change them.  
- 🧪 **Mockito agent wiring:** surefire's `argLine` now depends on the `maven-dependency-plugin` `properties` goal resolving `${org.mockito:mockito-core:jar}`, and on JaCoCo's `@{argLine}`. A build that skips either goal can break test forks.  
- 📝 **Deprecation warning:** post-upgrade-verification.log shows "Parameter 'systemProperties' is deprecated: use systemPropertyVariables instead". This existing configuration was not changed in this iteration.  
  
## ⏱️ Time savings  
  
- 🕒 **Transformation duration:** 6 minute(s).  
- 💰 **Estimated time saved:** 30 minute(s).  
  
## 🚀 Next steps  
  
- 🐳 Run the test suite on a host with Docker so that `MySqlIntegrationTests` and `PostgresIntegrationTests` actually run on Java 25.  
- ▶️ Enable the smoke-start tier (`ENABLE_RUNTIME_VALIDATION=true`) or boot the application in a test environment to confirm runtime behaviour on Java 25.  
- 🐘 Run `./gradlew build` to confirm that the Gradle build works with the Java 25 toolchain, and decide whether the Byte Buddy 1.17.x constraint named in the log is needed there.  
- 🔧 Replace the deprecated surefire `systemProperties` parameter with `systemPropertyVariables`.  
- 📌 Optionally pin the versions of `maven-dependency-plugin` and `maven-surefire-plugin` if the build must not depend on the parent's plugin management.  
- 🧑‍💻 Update any IDE run configurations or docs that start the test helper `main` classes with arguments.  
- 🚢 Make sure the CI images, Docker base images and deployment runtimes (`k8s/`, `.devcontainer`, `.github` workflows) provide a Java 25 JDK/JRE. These files were not changed in this iteration.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: 79 tests run
- ⏱️ Transformation duration: 6 minute(s)
- ⏳ Estimated time saved: 30 minute(s)
- 🚦 Runtime validation (runtime-validation.log):
  - OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it).
  - Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check).

## Kiro credit usage

💳 Kiro CLI credits used: 5.93  
Includes 1.31 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
