# Java 25 Upgrade – Iteration 17 Executive Report (Spring PetClinic)  
  
## 📋 Executive summary  
  
- 🎯 OpenRewrite (`com.amazonaws.java.migrate.UpgradeToJava25`) moved Spring PetClinic from Java 17 to Java 25. Kiro CLI then ran a build-fix pass, which found no compilation errors to fix and made no further code changes.  
- ✅ The post-upgrade verification build ended in **SUCCESS** with **79 tests run** (0 failures, 0 errors, 2 skipped).  
- 🌱 The Spring Boot upgrade step was skipped. The log says: "detected Spring Boot 4.1 is already >= target 4.0". The project uses `spring-boot-starter-parent` 4.1.0, which manages the Spring Framework version.  
- 🔍 The runtime-validation static check passed: "OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it)."  
- ⚠️ "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." The application was **not started**, so its runtime behaviour on Java 25 has not been verified.  
- 📦 7 files were changed (35 insertions, 15 deletions). Only the build files and test sources changed; nothing under `src/main` was touched.  
  
## 🛠️ Application changes  
  
- ☕ **Java level:** Maven `java.version` changed from 17 to 25, and the Gradle toolchain from `JavaLanguageVersion.of(17)` to `JavaLanguageVersion.of(25)`.  
- 🧪 **Test runtime:** surefire now attaches the Mockito jar as a `-javaagent`. The `maven-dependency-plugin` `properties` goal provides the jar's path.  
- 📈 **Coverage tooling:** the `jacoco.version` property changed from 0.8.15 to 0.8.16.  
- 🧹 **Test sources:** five test classes were modernised to Java 21/25 idioms (details under Code changes).  
- ➕ **Dependencies added (report.json):**  
  - 🔌 `org.apache.maven.plugins:maven-dependency-plugin`, version LATEST. The diff declares it without a version, so `spring-boot-starter-parent` 4.1.0 resolves it. It is a build plugin, so it has no dependency scope and is not packaged with the application.  
  - 🔌 `org.apache.maven.plugins:maven-surefire-plugin`, version LATEST. The diff declares it without a version, so `spring-boot-starter-parent` 4.1.0 resolves it. It is a build plugin, so it has no dependency scope and is not packaged with the application.  
  - ✅ Neither addition puts a server- or runtime-provided API into the application artifact, so this change carries no classloader-conflict risk.  
- ➖ **Dependencies removed:** none.  
- ⬆️ **Dependencies upgraded:** none.  
- 📂 **Changed files (7):**  
  - `build.gradle`  
  - `pom.xml`  
  - `src/test/java/org/springframework/samples/petclinic/MysqlTestApplication.java`  
  - `src/test/java/org/springframework/samples/petclinic/PetClinicConcurrencyTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/PetClinicIntegrationTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/PostgresIntegrationTests.java`  
  - `src/test/java/org/springframework/samples/petclinic/service/ClinicServiceTests.java`  
  
## 🧰 Tools used  
  
- ☕ **Java SDK:** Amazon Corretto OpenJDK 25.0.5 (`Corretto-25.0.5.0.9`, build 25.0.5+9-LTS).  
- 🔄 **OpenRewrite:** `rewrite-maven-plugin` 6.42.0 running the `com.amazonaws.java.migrate.UpgradeToJava25` recipe. This includes:  
  - `UpgradeBuildToJava25`  
  - `UpgradePluginsForJava25` / `AddMockitoJavaAgentToMavenSurefirePlugin`  
  - `UpgradeJaCoCo`  
  - `ListFirstAndLast`  
  - `MigrateMainMethodToInstanceMain`  
  - `ReplaceSystemOutWithIOPrint`  
  - `ReplaceUnusedVariablesWithUnderscore`  
  - `UpdateJavaCompatibility`  
- 🤖 **Amazon Kiro CLI:** version 2.14.2 with model `claude-opus-5.5`. It handled the build-fix and verification pass and succeeded on the first attempt.  
- 🏗️ **Build tools:** Apache Maven 3.9.16 for the build of record. The project's Gradle wrapper was used for an extra test-compile check.  
  
## 💻 Code changes  
  
- 📄 **`pom.xml`:**  
  - `<java.version>17</java.version>` → `<java.version>25</java.version>`  
  - `<jacoco.version>0.8.15</jacoco.version>` → `<jacoco.version>0.8.16</jacoco.version>`  
  - Added an empty `<argLine></argLine>` property.  
  - Added `maven-dependency-plugin` with an execution of the `properties` goal.  
  - Added `maven-surefire-plugin` with `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>`.  
- 📄 **`build.gradle`:** `JavaLanguageVersion.of(17)` → `JavaLanguageVersion.of(25)`.  
- 📄 **`MysqlTestApplication.java`** and **`PetClinicIntegrationTests.java`:** `public static void main(String[] args)` → `void main()` (instance main method).  
- 📄 **`PostgresIntegrationTests.java`:** `public static void main(String[] args)` → `void main(String[] args)`.  
- 📄 **`PetClinicConcurrencyTests.java`:**  
  - `catch (Exception e)` → `catch (Exception _)` (unnamed variable).  
  - Five calls changed from `System.out.println(...)` → `IO.println(...)`.  
- 📄 **`ClinicServiceTests.java`:** three calls changed from `.get(0)` → `.getFirst()` (SequencedCollection API). The remaining `.get(1)` is unchanged.  
  
## ⚠️ Risks  
  
- 🚫 **Runtime unverified:** "Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check)." Nobody has checked that the application boots or serves requests on Java 25.  
- 🐳 **Database integration tests not exercised:** of the 79 tests, 2 were skipped (`MySqlIntegrationTests`). `PostgresIntegrationTests` ran 0 tests because the build environment has no Docker. MySQL and PostgreSQL compatibility on Java 25 is therefore untested.  
- 🏃 **IDE launchers:** the test-app entry points now use Java 25 instance `main` methods. Older IDEs and tools that expect `public static void main` may no longer offer them as run targets.  
- 🧪 **Empty `argLine` property:** it exists so `@{argLine}` always resolves. If a later change sets `argLine` in another place (for example, a JaCoCo agent configuration), check that the values combine as intended.  
- 🔧 **Unpinned plugin versions:** the two added plugins take their versions from the Spring Boot parent. Bumping the parent changes them implicitly.  
- 🧩 **Maven/Gradle parity gap:** the Mockito `-javaagent` setup was added only to the Maven build. `build.gradle` changed only its toolchain version.  
- 📝 **Log vs. branch:** `code-upgrade.log` lists a byte-buddy `UpgradeTransitiveDependencyVersion` change for `build.gradle`. That change is **not** on the branch: the only `build.gradle` change in the diff is the toolchain version.  
- ℹ️ **Recipe warnings:** OpenRewrite reported "The recipe produced 2 warning(s)" but did not say what they were. The build still succeeded.  
  
## ⏱️ Time savings  
  
- ⏳ **Transformation duration:** 9 minute(s)  
- 💰 **Estimated time saved:** 40 minute(s)  
  
## 🚀 Next steps  
  
- 🟢 Re-run runtime validation with `ENABLE_RUNTIME_VALIDATION=true` to get an app-boot check on Java 25.  
- 🐳 Run `MySqlIntegrationTests` and `PostgresIntegrationTests` on a Docker-enabled CI agent.  
- 🧪 Run `./gradlew test` on Java 25. Add an equivalent Mockito `-javaagent` setting to the Gradle test task if inline-mock warnings or failures appear.  
- 🖥️ Confirm the team's IDEs can launch the instance-`main` test applications; otherwise restore `public static void main(String[] args)`.  
- 📌 Consider pinning the `maven-dependency-plugin` and `maven-surefire-plugin` versions, or document that the Spring Boot parent manages them.  
- 🔎 Review the two unspecified OpenRewrite recipe warnings.  
- 🐋 Update CI/CD images and container base images (for example, the `spring-boot:build-image` builder) to a Java 25 runtime before deploying.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: 79 tests run
- ⏱️ Transformation duration: 9 minute(s)
- ⏳ Estimated time saved: 40 minute(s)
- 🚦 Runtime validation (runtime-validation.log):
  - OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it).
  - Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check).

## Kiro credit usage

💳 Kiro CLI credits used: 5.78  
Includes 0.93 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
