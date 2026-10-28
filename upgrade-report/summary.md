# ☕ Java 25 Upgrade – Iteration 19 Executive Report (Spring PetClinic)  
  
## 📋 Executive summary  
- 🎯 Spring PetClinic (`spring-petclinic` 4.0.0-SNAPSHOT) moved from Java 17 to Java 25. The OpenRewrite recipe `com.amazonaws.java.migrate.UpgradeToJava25` made the changes, and they are in commit `84f4426` ("Upgrade to Java 25 on Iteration 19") on top of base `d1bce54`.  
- ✅ The post-upgrade verification build succeeded with 79 tests run: 0 failures, 0 errors, 2 skipped.  
- 🛠️ The Kiro CLI build-fix pass found no compilation errors, so it made no code changes. It also confirmed that `./gradlew compileTestJava` compiles successfully.  
- 🌱 Spring: the Spring Boot leg was skipped ("detected Spring Boot 4.1 is already >= target 4.0"). Runtime validation reports "OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it)."  
- ⚠️ Runtime behaviour is unverified: "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." The application was not started.  
- 🐳 The database integration tests did not exercise a real database because there was no Docker environment. `MySqlIntegrationTests` skipped 2 tests and `PostgresIntegrationTests` ran 0 tests.  
- 📦 Only build files and test sources changed (7 files, 35 insertions, 15 deletions). No production source under `src/main` was modified.  
  
## 🧩 Application changes  
- 🔧 `pom.xml`:  
  - `java.version` 17 → 25  
  - `jacoco.version` 0.8.15 → 0.8.16  
  - adds an empty `<argLine>` property  
  - adds the `maven-dependency-plugin` and `maven-surefire-plugin` build plugins  
- 🔧 `build.gradle`: Gradle toolchain `JavaLanguageVersion.of(17)` → `JavaLanguageVersion.of(25)`.  
- 🧪 Test sources: 5 files under `src/test/java/org/springframework/samples/petclinic/` were modernised to Java 21/25 idioms (see Code changes).  
- ➕ Dependencies added (report.json):  
  - `org.apache.maven.plugins:maven-dependency-plugin` (version LATEST). It is a build plugin in `<build><plugins>`, not a dependency scope, and has no explicit `<version>` in the diff. It runs the `properties` goal.  
  - `org.apache.maven.plugins:maven-surefire-plugin` (version LATEST). It is a build plugin in `<build><plugins>`, not a dependency scope, and has no explicit `<version>` in the diff. It configures `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>`.  
  - ℹ️ Neither one is packaged with the application at compile or runtime scope, so they add no classloader-conflict risk.  
- ➖ Dependencies removed: none.  
- ⬆️ Dependencies upgraded: none.  
  
## 🧰 Tools used  
- ☕ Java SDK: Amazon Corretto OpenJDK 25.0.5 (build 25.0.5+9-LTS), with Apache Maven 3.9.16 and Gradle 9.5.1.  
- 🔄 OpenRewrite: `rewrite-maven-plugin` 6.42.0 running `com.amazonaws.java.migrate.UpgradeToJava25`. The applied sub-recipes include:  
  - `UpgradeBuildToJava25` / `UpdateMavenProjectPropertyJavaVersion`  
  - `UpgradeJaCoCo`  
  - `AddMockitoJavaAgentToMavenSurefirePlugin`  
  - `ListFirstAndLast`  
  - `MigrateMainMethodToInstanceMain`  
  - `ReplaceSystemOutWithIOPrint`  
  - `ReplaceUnusedVariablesWithUnderscore`  
  - `gradle.UpdateJavaCompatibility`  
- 🤖 Amazon Kiro CLI 2.34.0 with model `claude-opus-5.5`, which did the build-fix and verification pass. It needed no code fixes.  
  
## 💻 Code changes  
- 🚀 `MysqlTestApplication.java`: `public static void main(String[] args)` → `void main()` (instance main method).  
- 🚀 `PetClinicIntegrationTests.java`: `public static void main(String[] args)` → `void main()`.  
- 🚀 `PostgresIntegrationTests.java`: `public static void main(String[] args)` → `void main(String[] args)`.  
- 🖨️ `PetClinicConcurrencyTests.java`:  
  - five `System.out.println(...)` calls → `IO.println(...)`  
  - `catch (Exception e)` → `catch (Exception _)` (unnamed variable)  
- 📚 `ClinicServiceTests.java`: three `get(0)` calls → `getFirst()` (SequencedCollection), for example `owner.getPets().get(0).getType()` → `owner.getPets().getFirst().getType()`. `get(1)` is unchanged.  
- ⚙️ `pom.xml`: Mockito is now loaded as a `-javaagent` through surefire's `argLine`, with the jar path resolved by `maven-dependency-plugin:properties`. `@{argLine}` keeps the JaCoCo agent.  
- ⚙️ `build.gradle`: only the toolchain line changed. The OpenRewrite log lists a byte-buddy `UpgradeTransitiveDependencyVersion` against `build.gradle`, but no byte-buddy change appears in the branch diff.  
  
## ⚠️ Risks  
- 🟡 Runtime unverified: the smoke-start tier was skipped, so application boot and runtime behaviour on Java 25 have not been checked.  
- 🟡 No real-database coverage: there was no Docker environment, so `MySqlIntegrationTests` skipped 2 tests and `PostgresIntegrationTests` ran 0. The MySQL and PostgreSQL paths are untested on Java 25.  
- 🟡 Instance `main` launchers: the IDE/dev launchers in three test classes now use Java 25 instance `main` methods. They only work on JDK 25+, and run configurations may need refreshing.  
- 🟡 Unpinned plugin versions: the new `maven-dependency-plugin` and `maven-surefire-plugin` entries have no explicit `<version>` (report.json: LATEST). Their versions come from the Spring Boot parent's plugin management, so a parent bump can change their behaviour.  
- 🟡 Surefire `argLine` coupling: the empty `<argLine>` property plus `@{argLine}` keeps the JaCoCo agent working. If either is removed, test forks or coverage can break.  
- 🟢 Deprecation notice: the build warns "Parameter 'systemProperties' is deprecated: use systemPropertyVariables instead". It is non-blocking.  
- 🟢 Spring compatibility: runtime validation reports "OK (static)". Spring Boot 4.1 manages Spring Framework, and nothing shows an old Spring version.  
- 🟢 Production code: nothing under `src/main` changed, so the business-logic risk is low.  
  
## ⏱️ Time savings  
- ⏳ Transformation duration: 9 minute(s).  
- 💡 Estimated time saved: 20 minute(s).  
  
## 🔜 Next steps  
- 🐳 Run the full test suite on an agent with Docker so `MySqlIntegrationTests` and `PostgresIntegrationTests` run against real MySQL and PostgreSQL.  
- 🚦 Turn on the smoke-start tier (`export ENABLE_RUNTIME_VALIDATION=true`) or boot the application in a test environment to confirm it starts and serves pages on Java 25.  
- 📌 Consider pinning explicit versions for `maven-dependency-plugin` and `maven-surefire-plugin`, or confirm the Spring Boot parent-managed versions are acceptable.  
- 🧹 Replace the deprecated surefire `systemProperties` parameter with `systemPropertyVariables`.  
- 🧪 Run `./gradlew test` to verify the Gradle test path on Java 25. Only `compileTestJava` was checked in this iteration.  
- 🔍 Check whether the byte-buddy transitive upgrade that OpenRewrite reported for `build.gradle` is needed. It is not in the committed diff.  
- 🏗️ Update CI pipelines, container base images (`spring-boot:build-image`) and developer IDEs to JDK 25.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: 79 tests run
- ⏱️ Transformation duration: 9 minute(s)
- ⏳ Estimated time saved: 20 minute(s)
- 🚦 Runtime validation (runtime-validation.log):
  - OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it).
  - Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check).

## Kiro credit usage

💳 Kiro CLI credits used: 6.73  
Includes 0.68 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
