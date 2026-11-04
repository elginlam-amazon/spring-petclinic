# 📊 Executive Report: Java 17 → 25 Upgrade (Spring PetClinic, Iteration 27)  
  
## 🧭 Executive Summary  
  
- 🎯 **Goal:** Move Spring PetClinic (Spring Boot 4.1.0, Maven primary build, Gradle secondary build) from Java 17 to Java 25, using the AWS Transform Custom CLI managed transformation `AWS/java-version-upgrade` in region `ap-southeast-2`.  
- 🔧 **Work done:** The agent changed the Java version declaration in `pom.xml` and in `build.gradle`. No application source, test, dependency or plugin changes were needed.  
- ✅ **Technical result:** `./mvnw clean verify` on Corretto 25.0.5 ended in BUILD SUCCESS. 79 tests ran with 0 failures, 0 errors and 2 skipped, the same as before the upgrade. The packaged jar started on Java 25 and returned HTTP 200 on `/`. The compiled classes have bytecode major version 69 (Java 25).  
- ⚠️ **Reported status:** The agent's validation summary rated the run **PARTIAL**, and the CLI ended with `Execution was not successful, received event with FAILURE status`. The only criterion not met is "no `javax.*` imports". It fails on `javax.cache` (JSR-107 JCache), which has no `jakarta.*` replacement, so no change inside the repository can fix it.  
- ⏱️ **Effort:** 13.65 agent minutes. The result is commit `65961c0` on branch `atx-result-staging-20261104_163932_e9dda0e3`. The original branch `main-upgrade` was left unchanged.  
  
## 🏗️ Application Changes  
  
- 📦 **Build targets:** The Maven and Gradle builds now compile and run against Java 25, up from 17.  
- 🧩 **Application code:** No changes. Before the upgrade, CodeKB shows `CacheConfiguration.petclinicCacheConfigurationCustomizer()` registering a JCache `vets` cache. That cache is built by `cacheConfiguration()` using `javax.cache.configuration.MutableConfiguration` with statistics turned on. This code still works on Java 25 and was correctly left as it is.  
- 🌱 **Framework:** Spring Boot parent 4.1.0 was kept. It is already above the recipe's minimum of 3.2.12 and supports Java 25 bytecode. Downgrading it would have broken Java 24+ support, a lesson the agent reused from an earlier run (`spring-boot-java-compatibility`).  
- 🔁 **Unchanged on purpose:**  
  - ☕ `javax.cache` (JSR-107): no `jakarta.cache` artifact exists.  
  - 🧪 Checkstyle stays pinned at 12.3.1. An existing comment in `pom.xml` notes that v13 needs JDK 21 or later.  
  - 🛠️ The Gradle wrapper is already 9.5.1 (8.11 or later is required), and the Maven wrapper is 3.9.16.  
  - 🚫 The project doesn't use Lombok, and the Spring Boot parent manages the Jackson and Jakarta versions.  
- ⚠️ **Not updated (found by checking the working tree after the run):**  
  - `.github/workflows/maven-build.yml` and `.github/workflows/gradle-build.yml` still use the matrix `java: [ '17' ]`.  
  - `.devcontainer/Dockerfile` still uses `VARIANT=17-bullseye` and `JAVA_VERSION=17.0.7-ms`.  
  - Iteration 26 had updated these files (commit `5892f1a`). This run left them out.  
  
## 🛠️ Tools Used  
  
- ☕ **Java SDKs:** Amazon Corretto 17 for the baseline build and Corretto 25.0.5.0.10 for the target build and runtime. JDKs 8, 11 and 21 were also installed.  
- 🤖 **AWS Transform Custom CLI (`atx`):** `atx custom def exec --transformation-name AWS/java-version-upgrade --non-interactive --trust-all-tools`. It ran as an AWS managed transformation and applied lessons from earlier runs.  
- 📐 **Build tools:** Maven 3.9.16 through `mvnw`, with Gradle 9.5.1 as the secondary build.  
- 🔍 **Checks run:** `javap -v` for the bytecode version, `grep` scans for `javax.*`, `git diff` to confirm scope, and `curl` against the running app.  
- 🌿 **Version control:** The ATX `vcs_control` tool created a staging branch and committed the change there.  
  
## 💻 Code Changes  
  
| 📄 File | 🔄 Before | ✅ After |  
|---|---|---|  
| `pom.xml` (line 19) | `<java.version>17</java.version>` | `<java.version>25</java.version>` |  
| `build.gradle` (line 19) | `JavaLanguageVersion.of(17)` | `JavaLanguageVersion.of(25)` |  
  
- 🔗 `pom.xml` passes `${java.version}` to the enforcer rule that requires a minimum JVM version, and to `java.source`/`java.target` (lines 197–198 and 269–270). One edit therefore updates all of them.  
- 📏 Diff `d1bce54..65961c0`: 2 files changed, 2 insertions, 2 deletions.  
- 🧪 No test files were modified, removed or disabled.  
  
## ⏳ Time Savings Estimate  
  
- 🧮 **Scope:** Small. Two one-line build edits, but they had to be backed by a full compatibility review of the code and dependencies.  
- 👤 **Manual estimate: about 4–6 hours** for one engineer:  
  - 🔎 About 1–1.5 h to check dependencies and frameworks against Java 25 (Spring Boot, Jackson, Jakarta, Lombok, checkstyle and wrapper versions).  
  - 🧹 About 0.5–1 h to scan for `javax` usage and decide what to do about JSR-107.  
  - 🏃 About 1–1.5 h to install the JDK, run the baseline and target builds, and run the full test suite.  
  - 🚀 About 0.5 h to start the app and smoke-test it at runtime.  
  - 📝 About 1 h to document the work and prepare the commit or PR.  
- 🤖 **Automated:** 13.65 agent minutes.  
- 💰 **Saving:** About **3.5–5.5 hours**, roughly 90% or more. The saving would grow across a larger set of similar services.  
  
## 🚀 Next Steps  
  
- 🔴 **Update CI and the devcontainer:** Change the GitHub Actions matrices to `'25'` and update the devcontainer base image and `JAVA_VERSION`. Otherwise CI keeps building on JDK 17, and the Maven enforcer rule (at least Java 25) will fail those builds.  
- ⚠️ **Resolve the FAILURE status:** Find out why the CLI reported FAILURE even though the build, tests and runtime passed. The likely cause is the strict `javax.*` criterion. Ask for a recipe exception for `javax.cache` (JSR-107) so later runs can report PASS.  
- 🔍 **Review and merge:** Review commit `65961c0` on `atx-result-staging-20261104_163932_e9dda0e3`, then raise a PR into `main-upgrade`.  
- 🧪 **Validate more widely:**  
  - Run the Gradle build (`./gradlew build`) on JDK 25. Only Maven was run in this iteration.  
  - Run the 2 skipped tests (for example the Testcontainers/Docker-based MySQL and Postgres tests) in an environment that has Docker.  
  - Run a functional regression of owners, pets, visits and vets, including the JCache `vets` cache.  
- 📈 **Performance and runtime:** Compare startup time, memory use and GC behaviour against Java 17. Then consider Java 25 features such as virtual threads (`spring.threads.virtual.enabled=true`) and AOT cache/CDS for faster startup.  
- 🧰 **Housekeeping:**  
  - Move Checkstyle to v13 now that the minimum JDK is 25.  
  - Add a JDK 25 check to the build pipeline.  
  - Remove the many old `atx-result-staging-*` and `main-upgrade-itr-*` branches.  

## Kiro credit usage

💳 Kiro CLI credits used: 2.78  
Includes 0.31 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
