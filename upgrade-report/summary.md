# 📊 Executive Report: Spring PetClinic Upgrade to Java 25  
  
## 🧭 Executive Summary  
- 🎯 **Goal:** upgrade `spring-petclinic` (4.0.0-SNAPSHOT, Spring Boot 4.1.0) from **Java 17 to Java 25**.  
- ✅ **Outcome:** the Java 25 upgrade is complete. `mvn verify` on JDK 25 ended in **BUILD SUCCESS**, with **0 compilation errors** and **79 tests run: 0 failures, 0 errors, 2 skipped**.  
- 🤖 **How:** OpenRewrite recipe `com.amazonaws.java.migrate.UpgradeToJava25` (took 56.7 s), followed by a Kiro CLI build-verification pass (took 2m 5s). Kiro found no compilation errors, so it made **no code changes**.  
- 🌱 **Spring:** this was **not a Spring upgrade**. The log shows `Skipping Spring Boot upgrade: detected Spring Boot 4.1 is already >= target 4.0`.  
  - The log has no `Verified: effective Spring Framework version…` line and no `still < 5.3` warning.  
  - The Spring stack did not change. A `mvn dependency:list` run after the upgrade resolves **Spring Framework 7.0.8** (jakarta line), which is above every target.  
- 📁 **Scope:** **7 files** actually changed: `pom.xml`, `build.gradle` and 5 test classes. Production code under `src/main` is **unchanged**.  
  
## 🏗️ Application Changes  
- ☕ **Java version:** 17 → **25** in both the Maven and Gradle builds.  
- 🧪 **Test runtime:** Mockito is now attached as a `-javaagent` in Surefire, because recent JDKs restrict loading agents dynamically.  
- 🧹 **Test code:** modernized with Java 21–25 language and API features (details below).  
- 🚫 **Not changed:** business logic, controllers, entities, repositories, configuration, resources and dependency versions.  
- 🌿 **Framework stack (unchanged):** Spring Boot 4.1.0, Spring Framework 7.0.8, Hibernate ORM 7.4.1.Final, Mockito 5.23.0, Byte Buddy 1.18.10. These are the resolved versions after the upgrade.  
  
## 🛠️ Tools Used  
- ☕ **Java SDK:** Amazon Corretto **25.0.4.1** (25.0.4.1+10-LTS) with Apache Maven 3.9.16. Gradle wrapper 9.5.1 was also detected.  
- 🔄 **OpenRewrite:** `rewrite-maven-plugin` **6.42.0**.  
  - Active recipe: `com.amazonaws.java.migrate.UpgradeToJava25`.  
  - Main sub-recipes: `OfficialUpgradeToJava25WithoutIfElseIfToSwitch`, `UpgradeBuildToJava21/25`, `UpgradePluginsForJava25`, `AddMockitoJavaAgentToMavenSurefirePlugin`, `ListFirstAndLast`, `MigrateMainMethodToInstanceMain`, `ReplaceSystemOutWithIOPrint`, `ReplaceUnusedVariablesWithUnderscore`, `UpdateJavaCompatibility` (Gradle).  
  - The recipes scanned 132 source files.  
- 🤖 **Amazon Kiro CLI:** **kiro-cli 2.14.2**, model **claude-opus-5.5**.  
  - It used CodeKB to map the old code, then built, tested and verified the result.  
  - It used 1.93 credits and took 2m 5s.  
  
## 💻 Code Changes  
- 📄 **`pom.xml`**  
  - 🔢 `<java.version>`: 17 → 25 (`UpdateMavenProjectPropertyJavaVersion`).  
  - 🧩 Added the `maven-dependency-plugin` `properties` goal and a `maven-surefire-plugin` `<argLine>@{argLine} -javaagent:${org.mockito:mockito-core:jar}</argLine>`, plus an empty `<argLine/>` property (`AddMockitoJavaAgentToMavenSurefirePlugin`).  
- 📄 **`build.gradle`**  
  - 🔢 Toolchain `JavaLanguageVersion.of(17)` → `of(25)` (`UpdateJavaCompatibility`).  
  - ⚠️ The log warning says Byte Buddy 1.17.x was added via `UpgradeTransitiveDependencyVersion`, but that recipe **failed** on `build.gradle` (`SourcesFileErrors`: `Unable to find GradleProject marker`). **No Byte Buddy change was applied.** Maven resolves Byte Buddy 1.18.10 regardless.  
- 🧪 **`ClinicServiceTests.java`:** `getPets().get(0)` / `getSpecialties().get(0)` → `getFirst()` (Sequenced Collections, `ListFirstAndLast`).  
- 🧪 **`PetClinicConcurrencyTests.java`:**  
  - `catch (Exception e)` → `catch (Exception _)` (unnamed variables).  
  - 5 calls to `System.out.println` → `IO.println`.  
- 🧪 **`PetClinicIntegrationTests.java`, `MysqlTestApplication.java`:** `public static void main(String[] args)` → instance `void main()` (JEP 512).  
- 🧪 **`PostgresIntegrationTests.java`:** `public static void main(String[] args)` → instance `void main(String[] args)`. It keeps the parameter because the method uses `args`.  
- 🤖 **Kiro CLI fixes:** none needed. The post-recipe build already had 0 errors.  
- ℹ️ The `SourcesFileResults` table also lists every `src/main` file under the `UpgradeJavaVersion` parent rows, all with 0 time saved. `git status` shows **no diff** in those files, so they weren't actually modified.  
  
## ⏱️ Time Savings Estimate  
- 🤖 **Reported by OpenRewrite:** `Estimate time saved: 1h 5m`.  
- 👩‍💻 **Manual effort avoided:** about **3–5 hours**, covering:  
  - Researching Java 25 build and plugin requirements and the Mockito agent change.  
  - Editing the Maven and Gradle builds.  
  - Updating the test code.  
  - Running and triaging a full build and tests.  
  - Writing the report.  
- ⚡ **Actual automated runtime:** about **3 minutes** (OpenRewrite 56.7 s + Kiro 2m 5s).  
- 📉 **Why it's modest:** the project was already on Spring Boot 4.x / jakarta. There was no namespace migration, no framework jump and no compilation errors to fix.  
  
## 🚀 Next Steps  
- 🐳 **Container-based integration tests:** the MySQL (Testcontainers) and Postgres (Docker Compose) integration tests were skipped because there was no Docker environment. Run them in CI with Docker available.  
- 🖥️ **Instance-main launchers:** the README tells developers to run `PetClinicIntegrationTests`, `MysqlTestApplication` and `PostgresIntegrationTests` as `main()` launchers. Confirm their IDEs can run instance `void main()` methods; if not, revert to `public static void main(String[] args)`.  
- 🐘 **Gradle build:** run `./gradlew build` on JDK 25 in CI. This cycle only checked the wrapper version (9.5.1).  
- 🔍 **Byte Buddy recipe error:** OpenRewrite's Byte Buddy change failed on `build.gradle` (missing `GradleProject` marker). Check that the Gradle-resolved Byte Buddy is ≥ 1.17. Maven already resolves 1.18.10.  
- 🐳 **Container image:** rebuild it with `./mvnw spring-boot:build-image` and confirm the buildpack uses a Java 25 runtime.  
- 🔁 **CI and dev environments:** move these from JDK 17 to JDK 25:  
  - `.github/workflows/maven-build.yml` and `gradle-build.yml`, which still use the matrix `java: [ '17' ]`.  
  - `.devcontainer/Dockerfile`, which still sets `JAVA_VERSION=17.0.7-ms`.  
  - Also check `.gitpod.yml`.  
- 📖 **README:** it still says "Java 17 or later is required"; change it to Java 25.  
- ✨ **Optional modernization:** consider Java 25 features in production code (records for DTOs, pattern matching, virtual threads via `spring.threads.virtual.enabled=true`), after load testing.  

## Kiro credit usage

💳 Kiro CLI credits used: 10.57  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
