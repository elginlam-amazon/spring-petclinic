# 🚀 Java 17 → 25 Upgrade: Executive Report (Iteration 26)  
  
**Project:** Spring PetClinic (`/data/code`) · **Branch:** `atx-result-staging-20261029_025258_b727c83c` · **Commit:** `5892f1a` · **Region:** us-east-1  
  
## 📋 Executive Summary  
  
AWS Transform Custom CLI, running as an AWS Managed transformation, moved Spring PetClinic's Java target from 17 to 25. It first ran a baseline `./mvnw clean verify` on Java 17, which passed. It then raised the Java version in the Maven and Gradle build files, both GitHub Actions workflows and the devcontainer. The build was checked on Amazon Corretto 25.0.5 and finished with **BUILD SUCCESS**: 79 tests ran, 0 failed, 0 errored and 2 were skipped, the same result as the baseline. ✅ All five exit criteria passed, and the validation status is **COMPLETE**. No application source, dependency or plugin changes were needed, because the project was already on Spring Boot 4.1.0, already used Jakarta namespaces and already had Gradle wrapper 9.5.1. The run used **16.45 agent minutes** and exited with code 0.  
  
## 🏗️ Application Changes  
  
- ☕ **Java target:** 17 → 25 for both build systems (Maven and Gradle).  
- 🔁 **CI/CD:** the GitHub Actions matrix in `maven-build.yml` and `gradle-build.yml` now builds on Java 25.  
- 🐳 **Dev environment:** the devcontainer Java feature changed from `21-oracle` to `25-oracle`. Before this change, the devcontainer used Java 21 while the build targeted 17.  
- 🧩 **Application code:** no changes.  
  - 🗄️ `system/CacheConfiguration` creates the `vets` JCache with statistics enabled, through `petclinicCacheConfigurationCustomizer()` → `cacheConfiguration()` (old code mapped with CodeKB). It still imports `javax.cache.configuration.*` (JSR-107), on purpose. JCache has no `jakarta.*` replacement, so these imports are correct.  
- 📦 **Dependencies:** no changes.  
  - Spring Boot parent 4.1.0 is above the required minimum of 3.2.12.  
  - Gradle wrapper 9.5.1 is above the required minimum of 8.11.  
  - The project does not use Lombok or Jackson, so their version minimums don't apply.  
  
## 🛠️ Tools Used  
  
- ☕ **Java SDK:** Amazon Corretto **25.0.5** LTS (`/usr/lib/jvm/java-25-amazon-corretto.aarch64`) was the target JDK. JDK 17 was used for the baseline build.  
- 🤖 **AWS Transform Custom CLI (ATX):** ran as an AWS Managed transformation with all tools trusted. It used shell, file read and editor tools, plus `vcs_control` to create the staging branch and commit the change.  
- 📚 **ATX learnings:** the `jakarta-namespace-migration` lesson from earlier runs told the agent to keep the `javax.cache` imports. That stopped it from attempting a rewrite that would have failed.  
- 🔨 **Build tooling:** Maven 3.9.16 through `./mvnw`, the Spring Boot-managed compiler plugin, and the Gradle 9.5.1 wrapper.  
- 🧭 **CodeKB:** used to map the old cache configuration code for this report.  
  
## 💻 Code Changes  
  
| 📄 File | ➖ Before | ➕ After |  
|---|---|---|  
| `pom.xml` | `<java.version>17</java.version>` | `<java.version>25</java.version>` |  
| `build.gradle` | `JavaLanguageVersion.of(17)` | `JavaLanguageVersion.of(25)` |  
| `.github/workflows/maven-build.yml` | `java: [ '17' ]` | `java: [ '25' ]` |  
| `.github/workflows/gradle-build.yml` | `java: [ '17' ]` | `java: [ '25' ]` |  
| `.devcontainer/devcontainer.json` | `"version": "21-oracle"` | `"version": "25-oracle"` |  
  
- 📊 **Change size:** 5 files changed, 5 lines added and 5 removed, in one commit.  
- 🧪 **Tests:** no tests were edited, removed or disabled.  
- 🧹 **Working tree:** `git status` was clean afterwards, with no stray generated files.  
  
## ⏱️ Time Savings Estimate  
  
- 🟢 **Complexity: low.** Only configuration changed. The hard parts of a modernization (Spring Boot 4 and the Jakarta migration) were already done.  
- 👩‍💻 **Manual estimate: about 3–5 hours.** That covers:  
  - auditing the build files, CI and devcontainer for Java version references  
  - checking dependency compatibility with Java 25  
  - checking the remaining `javax.*` imports  
  - running the baseline and target builds (each about 1 minute, several times)  
  - writing the validation notes and the PR  
- 🤖 **Automated:** about 16.5 agent minutes.  
- 💰 **Net savings: about 2.5–4.5 hours (roughly 85–95%).** ATX also produced a validation summary with evidence and PR instructions.  
  
## ✅ Next Steps  
  
- 🔍 **Review and merge:** review commit `5892f1a` on the staging branch. Then open the PR using `~/.aws/atx/custom/20261029_025258_b727c83c/artifacts/git_instructions.md`.  
- 🐘 **Gradle check:** ATX only validated the Maven build. Run `./gradlew build` on JDK 25 to confirm the Gradle path also works.  
- 🧪 **Skipped tests:** the 2 skipped tests need Docker/Testcontainers. Run them in an environment with Docker, or in CI.  
- ⚙️ **CI check:** confirm that the updated GitHub Actions workflows pass with Java 25 on the hosted runners.  
- 🐳 **Container image:** check that the runtime base image (buildpacks or Dockerfile) and the `k8s` deployment use a Java 25 JRE.  
- 🚦 **Runtime smoke test:** start the app on JDK 25 and check the main pages, the `vets` cache and the database profiles (H2, MySQL, PostgreSQL).  
- 📈 **Optional improvements:**  
  - Benchmark startup time and memory on JDK 25, for example compact object headers and the GC defaults.  
  - Look at newer language features, such as records, pattern matching and virtual threads (`spring.threads.virtual.enabled`).  
  - Add a JDK 25 build warning check (`-Xlint:all`) so deprecations show up early.  

## Kiro credit usage

💳 Kiro CLI credits used: 2.59  
Includes 0.31 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
