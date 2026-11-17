# Java 25 Upgrade: Iteration 28 Executive Report  
  
## 📋 Executive summary  
- 🎯 Spring PetClinic (`org.springframework.samples:spring-petclinic` 4.0.0-SNAPSHOT) moved its Java target from 17 to 25 in both the Maven and Gradle builds. Commit `09bf56d` ("Upgrade to Java 25 on Iteration 28") on the iteration branch holds the changes, on top of base commit `d1bce54`.  
- ✅ The post-upgrade verification build succeeded, and 79 automated tests ran with 0 failures and 0 errors. 2 tests were skipped because the build machine has no Docker environment for Testcontainers (post-upgrade-verification.log).  
- 🌱 Spring: the project uses a Spring Boot 4.1.0 parent, so no build file pins a Spring Framework version. The static runtime check reported "OK (static)", meaning no Java-17-incompatible Spring was detected.  
- ⚠️ **The application's runtime behaviour is unverified.** The smoke-start tier was skipped (it is opt-in), so the application was never started during this iteration.  
- 🛠️ OpenRewrite made the Java 25 build and test-code changes. The Kiro CLI pass found no compilation errors to fix. It then updated CI, the devcontainer and Checkstyle for JDK 25.  
- ⏱️ The transformation took 18 minutes, and the estimated time saved is 120 minutes.  
  
## 🧩 Application changes  
- 📦 `pom.xml`:  
  - `java.version` changed from 17 to 25.  
  - JaCoCo changed from 0.8.15 to 0.8.16.  
  - Checkstyle changed from 12.3.1 to 13.11.0.  
  - An empty `argLine` property and two build plugins were added (see Code changes).  
- 🐘 `build.gradle`:  
  - The toolchain `languageVersion` changed from 17 to 25.  
  - `checkstyleVersion` changed from 12.3.1 to 13.11.0.  
- 🤖 `.github/workflows/maven-build.yml` and `.github/workflows/gradle-build.yml`: the JDK matrix changed from `'17'` to `'25'`, and the `setup-java` distribution changed from `adopt` to `temurin`.  
- 🐳 `.devcontainer/devcontainer.json`: the Java feature version changed from `21-oracle` to `25`. The distro is still `oracle`.  
- 🐳 `.devcontainer/Dockerfile`, used by Gitpod:  
  - The base image changed from `mcr.microsoft.com/vscode/devcontainers/java:0-17-bullseye` to `mcr.microsoft.com/devcontainers/java:25-bookworm`.  
  - `JAVA_VERSION` changed from `17.0.7-ms` to `25.0.5.0+2-ms`.  
- 🧪 Five test sources were updated to Java 25 idioms: `MysqlTestApplication.java`, `PetClinicConcurrencyTests.java`, `PetClinicIntegrationTests.java`, `PostgresIntegrationTests.java` and `ClinicServiceTests.java`.  
- 📝 `summary.md` (repository root) was added with the Kiro CLI iteration notes.  
- 🚫 No production source under `src/main` changed.  
- 📊 Totals: 12 files changed, 74 insertions and 26 deletions (report.json and `git diff --stat`).  
  
## 🧰 Tools used  
- ☕ **Java SDK:** JDK 25, Amazon Corretto 25.0.5 (code-upgrade.log).  
- 🔁 **OpenRewrite:** `rewrite-maven-plugin` 6.42.0 running the active recipe `com.amazonaws.java.migrate.UpgradeToJava25`.  
  - Contributing sub-recipes include `UpgradeBuildToJava25`, `UpgradeJaCoCo`, `AddMockitoJavaAgentToMavenSurefirePlugin`, `ListFirstAndLast`, `ReplaceSystemOutWithIOPrint`, `ReplaceUnusedVariablesWithUnderscore`, `MigrateMainMethodToInstanceMain` and `gradle.UpdateJavaCompatibility`.  
- 🤖 **Amazon Kiro CLI:** kiro-cli 2.35.1 with the model `claude-opus-5.5`. Every pass succeeded on its first attempt.  
  
## 💻 Code changes  
- 🔧 `pom.xml`: added `maven-dependency-plugin` with the `properties` goal. This exposes artifact paths as Maven properties.  
- 🔧 `pom.xml`: added `maven-surefire-plugin` with `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>`, which loads Mockito as a Java agent during tests. An empty `<argLine></argLine>` property was also added as its default.  
- ✂️ `pom.xml`: removed the comment that pinned Checkstyle to v12 because v13 required JDK 21.  
- 🚪 `MysqlTestApplication.java` and `PetClinicIntegrationTests.java`: `public static void main(String[] args)` became `void main()`, the Java 25 instance main method.  
- 🚪 `PostgresIntegrationTests.java`: `public static void main(String[] args)` became `void main(String[] args)`.  
- 🖨️ `PetClinicConcurrencyTests.java`: five `System.out.println(...)` calls became `IO.println(...)`.  
- 🔣 `PetClinicConcurrencyTests.java`: `catch (Exception e)` became `catch (Exception _)`, an unnamed variable.  
- 📚 `ClinicServiceTests.java`: three `.get(0)` calls became `.getFirst()`, from the Sequenced Collections API. One remaining `.get(1)` is unchanged.  
- ✅ The tests' assertions and expected values did not change.  
  
## 📦 Dependencies (report.json)  
- ➕ Added `org.apache.maven.plugins:maven-dependency-plugin`, version LATEST.  
  - The pom has no `<version>` element, so the Spring Boot 4.1.0 parent's plugin management resolves the version.  
  - Scope: it is a build plugin, not a dependency, so it is not packaged with the application.  
- ➕ Added `org.apache.maven.plugins:maven-surefire-plugin`, version LATEST.  
  - The pom has no `<version>` element, so the Spring Boot parent manages the version.  
  - Scope: it is a build plugin used only for test execution, so it is not packaged with the application.  
- ✅ Neither addition packages an API that the runtime already provides, so there is no classloader-conflict risk.  
- ➖ Removed: none.  
- ⬆️ Upgraded: none in report.json. The pom.xml diff separately shows plugin and tool property bumps: JaCoCo 0.8.15 → 0.8.16 and Checkstyle 12.3.1 → 13.11.0.  
  
## ⚠️ Risks  
- 🚨 **Runtime is unverified.** The smoke-start tier was skipped (opt-in: `ENABLE_RUNTIME_VALIDATION=true`), so the application was not booted. Only the static Spring check ("OK (static)") passed.  
- 🐳 **2 tests were skipped and Postgres tests ran 0.**  
  - The Testcontainers-based database tests could not run because the build machine has no Docker.  
  - In particular, MySQL integration with the database is untested on JDK 25.  
- 🔒 **The JDK floor rose to 25.**  
  - The Maven enforcer requires at least Java 25, and Checkstyle 13 needs a newer JDK.  
  - Any developer machine, CI runner or image still on JDK 17 or 21 will fail the build.  
- 🤖 **The CI and devcontainer edits were not run.**  
  - The GitHub Actions matrices, the `temurin` switch and both container image changes were not exercised in this iteration.  
  - The `25-bookworm` tag and the SDKMAN identifier `25.0.5.0+2-ms` still need a real rebuild to confirm them.  
- 🧪 **The test-only Mockito agent.** Surefire now depends on the `maven-dependency-plugin:properties` goal resolving `${org.mockito:mockito-core:jar}`. If either plugin is removed, the tests break.  
- 🔍 **The log and the branch disagree on byte-buddy.**  
  - code-upgrade.log shows an `UpgradeTransitiveDependencyVersion` change for `net.bytebuddy:byte-buddy` and `byte-buddy-agent` (1.17.x) in `build.gradle`.  
  - The `build.gradle` diff on the branch contains no such constraint, so that change is not on the branch.  
- 🆕 **Java 25 syntax in tests.** Instance `void main()`, `IO.println` and the unnamed `_` mean the test sources can no longer be compiled on an older JDK.  
  
## ⏱️ Time savings  
- 💡 Estimated time saved: **120 minutes** (report.json `estimated_time_saved`).  
- ⌛ Transformation duration: **18 minutes** (report.json `transform_duration_in_minutes`).  
  
## 🚀 Next steps  
- ▶️ **Validate the runtime:**  
  - Rerun with `ENABLE_RUNTIME_VALIDATION=true`, or start the application in a test environment.  
  - Run a functional check of owners, pets, visits, vets and the JCache `vets` cache.  
- 🐳 **Run the Docker-based tests:** run the MySQL and Postgres Testcontainers integration tests on a machine that has Docker.  
- 🤖 **Confirm CI and the devcontainer:** watch the first GitHub Actions run on JDK 25 (Temurin), then rebuild the devcontainer and the Gitpod image.  
- 🔍 **Check byte-buddy:** decide whether the byte-buddy constraint from code-upgrade.log is needed in `build.gradle`. The Gradle build passed without it in the Kiro pass.  
- 👀 **Review and merge:** review commit `09bf56d` and open a pull request.  
- 📈 **Improvements, each to be tested on its own:**  
  - Compare startup, memory and GC behaviour against Java 17.  
  - Try virtual threads (`spring.threads.virtual.enabled=true`).  
  - Try the JDK 25 AOT cache for faster startup.

## Verification results

Copied from this iteration's report.json, progress.json and runtime-validation.log after the final build. This section is not written by the AI.

- 🛠️ Post-upgrade verification build: SUCCESS
- 🧪 Automated tests: 79 tests run
- ⏱️ Transformation duration: 18 minute(s)
- ⏳ Estimated time saved: 120 minute(s)
- 🚦 Runtime validation (runtime-validation.log):
  - OK (static): no Java-17-incompatible Spring detected (effective Spring n/a: no build file pins a Spring Framework version; a Spring Boot parent or imported BOM, if present, manages it).
  - Smoke-start tier skipped (opt-in: export ENABLE_RUNTIME_VALIDATION=true to enable an advisory app-boot check).

## Kiro credit usage

💳 Kiro CLI credits used: 13.82  
Includes 1.16 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
