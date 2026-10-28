# 🚀 Java 17 → 25 Upgrade: Executive Report (Spring PetClinic, Iteration 20)  
  
## 📋 Executive Summary  
  
- 🎯 Goal: move Spring PetClinic (Maven + Gradle builds) from Java 17 to Java 25 using AWS Transform Custom (AWS Managed transformation, region us-east-1).  
- ✅ Outcome: **COMPLETE**. All five exit criteria passed. `./mvnw clean verify` on JDK 25 gives **BUILD SUCCESS** with **79 tests run, 0 failures, 0 errors, 2 skipped**, the same as the JDK 17 baseline.  
- 🧩 Scope: configuration only. **5 files changed, 6 lines added, 6 lines deleted**. No Java source, test or dependency changes were needed.  
- 💡 Why it was small: the project was already on Spring Boot 4.1.0, used the Jakarta namespace, and had Gradle wrapper 9.5.1. All of these were already above the recipe's minimum versions.  
- ⏱️ The automated run used **22.79 agent minutes**, about 7 minutes of wall-clock time (05:43 → ~05:50 UTC).  
- 🌿 The result is commit `bc21bdf` on branch `atx-result-staging-20261028_054311_7a258bd5`, based on `d1bce54`. The original `main-upgrade` branch was left unchanged.  
  
## 🏗️ Application Changes  
  
- ☕ **Java version raised from 17 to 25** in every place the project declares it:  
  - 📦 Maven build: `pom.xml`  
  - 🐘 Gradle build: `build.gradle` toolchain  
  - 🔁 CI: GitHub Actions `maven-build.yml` and `gradle-build.yml`  
  - 🐳 Dev environment: `.devcontainer/Dockerfile`, which Gitpod uses  
- 🗄️ **Caching (no change, kept on purpose):**  
  - CodeKB mapping of the old code shows that `CacheConfiguration.petclinicCacheConfigurationCustomizer()` registers a `JCacheManagerCustomizer`. That customizer creates the `"vets"` cache using `cacheConfiguration()`, which returns a `javax.cache.configuration.MutableConfiguration` with statistics enabled.  
  - The `vets` cache supports the vet listing endpoints (`VetController.showVetList` → `/vets.html` and `showResourcesVetList` → `/vets`).  
  - These imports come from `javax.cache` (JSR-107 JCache). There is no `jakarta.*` replacement for this API, so they correctly stay as they are.  
- 🧾 **Business behaviour:** unchanged. No controller, repository, entity or template was modified.  
  
## 🛠️ Tools Used  
  
- ☕ **Java SDK**  
  - Baseline: JDK 17 (`/usr/lib/jvm/java-17`)  
  - Target: **Amazon Corretto 25.0.5** (`Corretto-25.0.5.0.9`)  
- 🤖 **AWS Transform Custom CLI (ATX)**  
  - AWS Managed transformation with target Java version 25  
  - Used a lesson from earlier runs (`jakarta-namespace-migration.md`) to keep `javax.cache` instead of attempting a rewrite to a `jakarta.cache` package that does not exist  
  - Committed its changes through version control on a separate staging branch and produced PR instructions (`git_instructions.md`)  
- 📦 **Build tooling**  
  - Maven wrapper 3.9.16, which meets the 3.9.0+ requirement  
  - Gradle wrapper 9.5.1, which meets the 8.11+ requirement  
- 🧠 **CodeKB:** used to map the old code (cache configuration and vet endpoints) for this report.  
  
## 💻 Code Changes  
  
| 📄 File | 🔴 Before | 🟢 After |  
|---|---|---|  
| `pom.xml` | `<java.version>17</java.version>` | `<java.version>25</java.version>` |  
| `build.gradle` | `JavaLanguageVersion.of(17)` | `JavaLanguageVersion.of(25)` |  
| `.github/workflows/maven-build.yml` | `java: [ '17' ]` | `java: [ '25' ]` |  
| `.github/workflows/gradle-build.yml` | `java: [ '17' ]` | `java: [ '25' ]` |  
| `.devcontainer/Dockerfile` | `VARIANT=17-bullseye`, `JAVA_VERSION=17.0.7-ms` | `VARIANT=25-bookworm`, `JAVA_VERSION=25-ms` |  
  
- ⏭️ **Updates skipped on purpose (reasons recorded in the ATX worklog):**  
  - 🌱 Spring Boot: the recipe minimum is 3.2.12 and the project is already on **4.1.0**. Downgrading is not allowed.  
  - 🔣 Jackson: the recipe minimum is 2.15.2 and the Spring Boot 4.1.0 BOM already manages a newer version.  
  - 🐘 Gradle wrapper: the recipe minimum is 8.11 and the project is already on **9.5.1**.  
  - 🌶️ Lombok: the project does not use it.  
  - 🗃️ `javax.cache` imports in `CacheConfiguration.java`: kept, because JSR-107 has no Jakarta replacement.  
  
## ⏱️ Time Savings Estimate  
  
- 🧮 **Complexity: low.** Only configuration changed, there was no source refactoring, and the dependency stack was already modern.  
- 👩‍💻 **Estimated manual effort: about 4–6 hours.** This covers:  
  - finding every place Java 17 is declared (build files, CI, devcontainer)  
  - setting up JDK 25 and running the baseline and target builds and tests  
  - auditing `javax.*` usage and dependency minimums  
  - writing up the validation results and the reasons for skipped updates  
- 🤖 **Automated effort: about 7 minutes wall-clock (22.79 agent minutes),** plus roughly 30 minutes of human review.  
- 📉 **Estimated saving: about 3.5–5.5 hours (around 85–90%).** The saving per repository is modest because the change was small. It grows when the same recipe runs across many repositories, or across projects that do need Jakarta or Spring migrations.  
  
## 🔜 Next Steps  
  
### ✅ Validation  
- 🐘 **Run the Gradle build on JDK 25** (`./gradlew build`). ATX only ran the Maven build, so Gradle has not been verified.  
- 🧪 **Look at the 2 skipped tests.** They were also skipped in the baseline, most likely the MySQL/Postgres tests that need Docker. Run them in an environment with Docker or Testcontainers.  
- 🐳 **Confirm the devcontainer image exists.** Check that `mcr.microsoft.com/vscode/devcontainers/java:0-25-bookworm` is published and that SDKMAN can resolve `25-ms`. If not, use a JDK 25 Corretto or Temurin identifier. This was not verified during the run.  
- 🔁 **Run the CI pipelines** on the staging branch to confirm that `actions/setup-java` provides JDK 25 for both workflows.  
- 🌐 **Smoke-test at runtime:** start the app on JDK 25, then check `/`, `/owners`, `/vets.html` and `/vets`, the H2 console, and the `vets` JCache statistics.  
- 🔀 **Raise a PR** from `atx-result-staging-20261028_054311_7a258bd5` using the generated `git_instructions.md`. Include a DCO `Signed-off-by` trailer.  
  
### 💡 Improvement Recommendations  
- 📝 **Update the README:** it still says "Java 17 or later is required". Change this to Java 25.  
- 🏃 **Container runtime:** make sure `spring-boot:build-image` and any k8s or Docker base images use a Java 25 JRE.  
- ⚡ **Use Java 25 features:** consider virtual threads (`spring.threads.virtual.enabled=true`), records for DTOs, and pattern matching. Benchmark startup and memory against JDK 17.  
- 🛡️ **Add guardrails:** consider pinning the compiler release through `maven.compiler.release`, and add a JDK 25 compatibility check to CI so the version does not drift back.  

## Kiro credit usage

💳 Kiro CLI credits used: 3.21  
Includes 0.23 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
