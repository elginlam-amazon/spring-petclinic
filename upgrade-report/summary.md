# 🚀 Executive Report: Java 17 → 25 Upgrade (Spring Petclinic)  
  
## 📋 Executive Summary  
  
AWS Transform Custom CLI (AWS-managed transformation, region us-east-1) upgraded the Spring Petclinic application from Java 17 to Java 25. The agent first ran a baseline build on JDK 17, which passed. It then raised the Java target in the Maven and Gradle builds, the GitHub Actions workflows and the devcontainer, and rebuilt on Amazon Corretto 25. The result was **✅ BUILD SUCCESS with 79 tests run, 0 failures, 0 errors and 2 skipped**. The 2 skipped tests were already skipped before the upgrade. All five exit criteria passed and the overall status is **COMPLETE**. The work used 14.04 agent minutes and changed 5 configuration files. No Java source or test code was modified.  
  
## 🏗️ Application Changes  
  
- 📦 **Application:** Spring Petclinic, built mainly with Maven and also with Gradle, on Spring Boot 4.1.0  
- ☕ **Java target:** 17 ➡️ 25 in all build and environment configuration  
- 🔁 **CI/CD:** the GitHub Actions build matrices for Maven and Gradle now run on Java 25  
- 🐳 **Dev environment:** the devcontainer now installs Java 25 (Amazon Corretto) on a Debian bookworm base image  
- 📚 **Dependencies:** none changed, because every one already met or exceeded the required minimum version:  
  - 🌱 Spring Boot 4.1.0 (minimum 3.2.12)  
  - 🧩 Jackson, managed by the Spring Boot BOM (minimum 2.15.2)  
  - 🐘 Gradle wrapper 9.5.1 (minimum 8.11)  
  - 🔧 maven-compiler-plugin, managed by the Spring Boot parent POM  
  - ➖ Lombok is not used, so it does not apply  
- 🏷️ **Jakarta namespace:** no migration needed. The only `javax.*` usage left is `javax.cache` (the JSR-107 JCache API) in `CacheConfiguration.java`, and that API has no `jakarta.*` equivalent.  
- 🌿 **Version control:** the change is committed as `fcd41af` on branch `atx-result-staging-20261028_231651_2134eaaa`, branched from base commit `d1bce54`. The original branch `main-upgrade` was left as it was.  
  
## 🛠️ Tools Used  
  
- ☕ **Java SDK:**  
  - Amazon Corretto 17 (`java-17-amazon-corretto.aarch64`) for the baseline build  
  - Amazon Corretto 25 (`java-25-amazon-corretto.aarch64` / `java-25`) for the target build and tests  
- 🤖 **AWS Transform Custom CLI (ATX):**  
  - AWS-managed transformation with automatic tool trust, run in classic output mode  
  - Applied a lesson from earlier runs (`dependency-version-floors`): do not downgrade dependencies that are already above the required minimum, and document why they were skipped  
  - Generated a worklog, a validation summary and pull request instructions (`git_instructions.md`)  
- 🧰 **Build tooling:** Maven Wrapper (`./mvnw clean verify`), Gradle Wrapper 9.5.1, and the maven-enforcer Java version rule  
- 🔍 **Analysis:** `grep` scans for Java version references and `javax.*` imports, plus `git diff` for verification  
  
## 💻 Code Changes  
  
| 📄 File | 🔄 Change |  
|---|---|  
| `pom.xml` | `<java.version>17</java.version>` ➡️ `<java.version>25</java.version>` |  
| `build.gradle` | `JavaLanguageVersion.of(17)` ➡️ `JavaLanguageVersion.of(25)` |  
| `.github/workflows/maven-build.yml` | `java: [ '17' ]` ➡️ `java: [ '25' ]` |  
| `.github/workflows/gradle-build.yml` | `java: [ '17' ]` ➡️ `java: [ '25' ]` |  
| `.devcontainer/Dockerfile` | `VARIANT=17-bullseye` ➡️ `21-bookworm`; `JAVA_VERSION=17.0.7-ms` ➡️ `25-amzn` |  
  
- 📊 **Diff size:** 5 files, 6 lines added, 6 lines deleted  
- 🔗 The `java.version` property in `pom.xml` also sets the maven-enforcer minimum Java version and the `java.source`/`java.target` values, so changing that one property moves all of them to 25  
- 🚫 No changes to `src/main` or `src/test`, and no tests were disabled or removed  
  
## ⏱️ Time Savings Estimate  
  
- 🧮 **Scope:** low complexity. The upgrade only touched configuration, but it covered every place Java 17 was set (two build systems, two CI workflows and the devcontainer), with full build and test validation  
- 👤 **Estimated manual effort:** about 4–6 hours for one engineer, broken down as:  
  - 🔎 Finding every Java version reference and checking whether `javax`→`jakarta` migration is needed: about 1 h  
  - 📚 Checking Spring Boot, Jackson and Gradle compatibility against the required minimum versions: about 1–1.5 h  
  - 🧪 Setting up JDK 25, running baseline and target builds, and analysing test results: about 1–2 h  
  - 📝 Writing up the change, the reasons for skipped items and the PR: about 1 h  
- 🤖 **Automated effort:** 14.04 agent minutes  
- 💰 **Estimated savings:** about 3.5–5.5 hours, roughly **90–95% less engineer time** for this upgrade. The savings would grow across a portfolio of similar repositories.  
  
## 🔜 Next Steps  
  
### ✅ Validation  
- 👀 Review commit `fcd41af` and open the PR using the generated `git_instructions.md`  
- 🐘 Run the Gradle build on JDK 25 (`./gradlew build`). The Maven build was fully verified, but the log shows no full Gradle build after the change  
- 🔁 Confirm both GitHub Actions workflows pass on the Java 25 matrix, and that `actions/setup-java` can resolve JDK 25 with the configured distribution  
- 🐳 Rebuild the devcontainer and confirm that SDKMAN can install `25-amzn` on the `21-bookworm` base image  
- ☸️ Check that the runtime images used by `k8s/` and `docker-compose.yml` run on a Java 25 JRE  
- 🧪 Run smoke and integration tests against the packaged `spring-petclinic-4.0.0-SNAPSHOT.jar`, including the database profiles (MySQL and PostgreSQL)  
- ⏭️ Review the 2 skipped tests (they were skipped before the upgrade too) and confirm they are skipped on purpose  
  
### 💡 Recommendations  
- 📈 Compare startup time, memory use and GC behaviour between JDK 17 and JDK 25 under representative load  
- ✨ Consider adopting newer language features step by step, such as records, pattern matching for `switch`, and virtual threads (`spring.threads.virtual.enabled=true`)  
- ⚠️ Turn on `-Xlint:all` or deprecation warnings in CI to catch APIs that are deprecated or slated for removal  
- 🧹 Consider making a Java 25 LTS image the devcontainer base image so the OS image and the installed JDK match  
- 🔒 Add a dependency vulnerability scan, such as OWASP Dependency-Check or Dependabot, to the CI pipeline  

## Kiro credit usage

💳 Kiro CLI credits used: 2.17  
Includes 0.31 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
