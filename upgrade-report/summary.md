# ☕ Java 25 Upgrade: Executive Report (Iteration 21)  
  
## 📋 Executive summary  
  
- 🎯 The Spring PetClinic application (`org.springframework.samples:spring-petclinic` 4.0.0-SNAPSHOT) was upgraded from Java 17 to Java 25 with the OpenRewrite recipe `com.amazonaws.java.migrate.UpgradeToJava25`.  
- 🌱 The Spring Boot upgrade step was skipped. code-upgrade.log says: "detected Spring Boot 4.1 is already >= target 4.0". The build reports "Built with Spring Boot :: 4.1.0".  
- ✅ The post-upgrade verification build succeeded, and 79 automated tests ran (0 failures, 0 errors, 2 skipped).  
- 🔍 Runtime validation's static check reported "OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it)."  
- ⚠️ "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." The application was not started, so its runtime behaviour on Java 25 is unverified.  
- 🛠️ The Kiro CLI build-fix pass found nothing to fix: the build already passed after OpenRewrite. All 7 changed files come from the OpenRewrite pass, and 5 of them are test sources. No production code under `src/main` changed.  
  
## 🧩 Application changes  
  
- 📁 7 files changed, 35 insertions, 15 deletions (`git diff --stat d1bce54..HEAD`, matching report.json).  
- 🏗️ Build files:  
  - `pom.xml`  
  - `build.gradle`  
- 🧪 Test sources (all under `src/test/java/org/springframework/samples/petclinic/`):  
  - `MysqlTestApplication.java`  
  - `PetClinicConcurrencyTests.java`  
  - `PetClinicIntegrationTests.java`  
  - `PostgresIntegrationTests.java`  
  - `service/ClinicServiceTests.java`  
- ➕ Dependencies added (report.json):  
  - `org.apache.maven.plugins:maven-dependency-plugin` (version LATEST)  
  - `org.apache.maven.plugins:maven-surefire-plugin` (version LATEST)  
  - 📦 Scope: both are entries under `<build><plugins>` in the `pom.xml` diff, with no `<version>` element in the POM. They are build-time Maven plugins, not dependencies. They have no compile or runtime scope, so they are not packaged with the application and add no classloader-conflict risk.  
- ➖ Dependencies removed: none.  
- ⬆️ Dependencies upgraded: none.  
  
## 🧰 Tools used  
  
- ☕ Java SDK: Amazon Corretto OpenJDK 25.0.5 (25.0.5+9-LTS). The build ran with Apache Maven 3.9.16 through the Maven wrapper.  
- 🔄 OpenRewrite: `rewrite-maven-plugin` 6.42.0 running `com.amazonaws.java.migrate.UpgradeToJava25`. The sub-recipes that applied changes include:  
  - `UpgradeBuildToJava25` / `UpdateMavenProjectPropertyJavaVersion`  
  - `UpgradeJaCoCo`  
  - `AddMockitoJavaAgentToMavenSurefirePlugin`  
  - `gradle.UpdateJavaCompatibility`  
  - `MigrateMainMethodToInstanceMain`  
  - `ReplaceUnusedVariablesWithUnderscore`  
  - `ReplaceSystemOutWithIOPrint`  
  - `util.ListFirstAndLast`  
- 🤖 Amazon Kiro CLI 2.35.1 with model `claude-opus-5.5`. It handled build verification and the fix pass, and succeeded on attempt 1.  
  
## 💻 Code changes  
  
- 🏗️ `pom.xml`:  
  - `<java.version>` changed from `17` to `25`.  
  - `<jacoco.version>` changed from `0.8.15` to `0.8.16`.  
  - An empty `<argLine></argLine>` property was added.  
  - A `maven-dependency-plugin` execution with the `properties` goal was added.  
  - A `maven-surefire-plugin` configuration was added with `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>`, which loads Mockito as a Java agent.  
- 🐘 `build.gradle`: `JavaLanguageVersion.of(17)` changed to `JavaLanguageVersion.of(25)`.  
  - code-upgrade.log also names a Byte Buddy `UpgradeTransitiveDependencyVersion` recipe against this file. The branch diff contains only the toolchain line, so no Byte Buddy change is on the branch.  
- 🚪 `MysqlTestApplication.java` and `PetClinicIntegrationTests.java`: `public static void main(String[] args)` changed to `void main()` (Java 25 instance main method).  
- 🚪 `PostgresIntegrationTests.java`: `public static void main(String[] args)` changed to `void main(String[] args)`.  
- 🔣 `PetClinicConcurrencyTests.java`:  
  - `catch (Exception e)` changed to `catch (Exception _)` (unnamed variable).  
  - 5 `System.out.println(...)` calls changed to `IO.println(...)`.  
- 📜 `service/ClinicServiceTests.java`: `.get(0)` changed to `.getFirst()` in 3 assertions (SequencedCollection API). `getSpecialties().get(1)` is unchanged.  
  
## ⚠️ Risks  
  
- 🚫 Runtime is unverified: "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." Nothing in this iteration shows the application starting on Java 25.  
- 🌱 Spring: "OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it)." The Spring Framework version comes from the Spring Boot 4.1.0 parent and was not pinned or changed in this iteration.  
- 🐳 Database integration paths are untested. `MySqlIntegrationTests` skipped 2 tests because Testcontainers "Could not find a valid Docker environment", so the MySQL and Postgres paths were not exercised on Java 25.  
- 🐘 The Gradle build is unverified. `build.gradle` was changed, but only the Maven build ran in verification.  
- 🧪 Test configuration risk: the new empty `<argLine>` property and the `@{argLine}` surefire setting must keep working with JaCoCo's `prepare-agent`. They did in verification: the JaCoCo 0.8.16 agent was set and the report was generated. A future plugin change could break the Mockito agent or coverage collection.  
- 🆕 Test helpers now use Java 25 language features: instance `main`, `_` and `IO.println`. These sources no longer compile on JDKs older than 25, so every developer and CI toolchain needs JDK 25.  
- 🔧 `maven-dependency-plugin` and `maven-surefire-plugin` have no explicit `<version>` in the POM, so the versions are inherited from the Spring Boot parent and will move when the parent changes.  
  
## ⏱️ Time savings  
  
- ⌛ Transformation duration: 6 minute(s).  
- 💰 Estimated time saved: 60 minute(s).  
  
## 🚀 Next steps  
  
- 🟢 Run the opt-in smoke-start check (`export ENABLE_RUNTIME_VALIDATION=true`) to confirm the application boots on Java 25.  
- 🐳 Re-run the build where Docker is available so the MySQL and Postgres Testcontainers tests run instead of being skipped.  
- 🐘 Run `./gradlew build` to verify the Gradle build with the Java 25 toolchain.  
- 🛠️ Update CI workflows (`.github/workflows/*.yml`), the `.devcontainer` Dockerfile and any runtime images to JDK 25.  
- 🔍 Review the Java 25 idioms in the test helpers, such as the instance `main` methods and `IO.println`, and confirm the team accepts them before merging.  
- 📌 Optionally pin the `maven-surefire-plugin` and `maven-dependency-plugin` versions if reproducible builds independent of the Spring Boot parent are needed.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: 79 tests run
- ⏱️ Transformation duration: 6 minute(s)
- ⏳ Estimated time saved: 60 minute(s)
- 🚦 Runtime validation (runtime-validation.log):
  - OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it).
  - Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check).

## Kiro credit usage

💳 Kiro CLI credits used: 6.82  
Includes 1.34 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
