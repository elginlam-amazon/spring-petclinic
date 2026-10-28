# 🚀 Java 17 → 25 Upgrade: Executive Report (Spring PetClinic, Iteration 18)  
  
## 📋 Executive Summary  
  
AWS Transform Custom CLI moved the Spring PetClinic application from Java 17 to Java 25 (Amazon Corretto 25.0.5 LTS) in about 14 agent minutes. Only the Java target settings had to change. The app already ran on Spring Boot 4.1.0, already used the Jakarta namespace, and already had Gradle wrapper 9.5.1. The four edits went into one commit (`a71b8cd`) on branch `atx-result-staging-20261028_022403_cf2978d5`. `./mvnw clean verify` passes on Java 25: 79 tests run, 0 failures, 0 errors, 2 skipped. All exit criteria passed. ✅ Overall status: COMPLETE  
  
## 🏗️ Application Changes  
  
- ☕ Java target: raised from 17 to 25 in the Maven build, the Gradle build and the CI pipelines  
- 🔁 CI/CD: the GitHub Actions Maven and Gradle workflows now build on JDK 25  
- 🧩 Application source: no changes. Controllers, entities, validators and templates are untouched  
- 🗄️ Caching (`CacheConfiguration`): no change needed. Before the upgrade, `petclinicCacheConfigurationCustomizer` registered the `vets` JCache, and `cacheConfiguration()` built a `javax.cache` `MutableConfiguration` with statistics turned on. That JSR-107 `javax.cache` API is the only `javax.*` usage left. It has no Jakarta replacement, so leaving it is correct  
- 📦 Dependencies: no changes  
  - Spring Boot 4.1.0 is already above the required 3.2.12. Downgrading is not allowed  
  - The Spring Boot BOM manages the Jakarta and Jackson versions  
  - Lombok is not used  
  - Gradle 9.5.1 is above the required 8.11  
- 🧪 Test sources: no changes. No tests were removed or disabled  
  
## 🛠️ Tools Used  
  
- ☕ Java SDK: Amazon Corretto 25.0.5 LTS (`/usr/lib/jvm/java-25-amazon-corretto.aarch64`). The baseline build ran on Java 17  
- 🤖 AWS Transform Custom CLI (ATX):  
  - AWS Managed transformation in region us-east-1, target Java version 25  
  - The agent ran the baseline build, made the edits, committed them to a staging branch with VCS control, and wrote a worklog and a validation summary  
  - 13.72 agent minutes used  
- 🔨 Build tooling: Maven Wrapper with Maven 3.9.16, and the Gradle wrapper 9.5.1  
- 🔍 Validation: `grep` scans for `javax.*`, a `git diff d1bce54..HEAD` review, and the maven-enforcer check that the JDK is at least `${java.version}`  
  
## 💻 Code Changes  
  
4 files changed, 4 lines added and 4 lines deleted. Initial commit `d1bce544`, transformation commit `a71b8cd`.  
  
| 📄 File | 🔧 Change |  
|---|---|  
| `pom.xml` | `<java.version>17</java.version>` → `<java.version>25</java.version>` |  
| `build.gradle` | `JavaLanguageVersion.of(17)` → `JavaLanguageVersion.of(25)` |  
| `.github/workflows/maven-build.yml` | matrix `java: [ '17' ]` → `java: [ '25' ]` |  
| `.github/workflows/gradle-build.yml` | matrix `java: [ '17' ]` → `java: [ '25' ]` |  
  
- ✅ Baseline build on Java 17 (`./mvnw clean install -DskipTests`): PASS  
- ✅ Build with tests on Java 25 (`./mvnw clean install`): BUILD SUCCESS  
- ✅ Re-validation on Java 25 (`./mvnw -o clean verify`): BUILD SUCCESS in 53.4 s and produced `spring-petclinic-4.0.0-SNAPSHOT.jar`  
- ✅ Tests: 79 run, 0 failures, 0 errors, 2 skipped  
  
## ⏱️ Time Savings Estimate  
  
- 🧮 Scope: low complexity. 4 config edits, no source refactoring, no dependency changes  
- 👩‍💻 Manual effort, estimated at about 6–10 hours:  
  - About 1–2 h to assess compatibility across Spring Boot, Jakarta, Gradle and Lombok/Jackson  
  - About 1–2 h to edit the builds and CI and to set up the toolchain  
  - About 2–4 h to run baseline and target builds and triage the test suite  
  - About 1–2 h to write the documentation and validation evidence  
- 🤖 Automated effort: about 14 agent minutes, plus about 30 minutes for a person to review the result  
- 💰 Estimated saving: about 85–90% of the engineering time on this module. The saving grows across a portfolio of similar services  
  
## 🔜 Next Steps  
  
### ✅ Validation  
- 🔀 Review the staging branch `atx-result-staging-20261028_022403_cf2978d5` and open a PR, following `git_instructions.md` in the ATX artifacts folder  
- 🐘 Run `./gradlew build` on JDK 25. Only the Maven build was run in this session  
- ⚙️ Confirm the GitHub Actions Maven and Gradle workflows pass on the `'25'` matrix  
- 🐳 Run the MySQL integration tests (Testcontainers) and the PostgreSQL integration tests (Docker Compose) on Java 25. Check that these are not the 2 skipped tests  
- 📦 Rebuild the container image with `./mvnw spring-boot:build-image` and confirm it uses a Java 25 runtime  
- 🌐 Smoke test the running app: owners, pets, visits, the vets page (including the `vets` cache) and the H2 console  
  
### 💡 Improvements  
- 🧵 Look at Java 21+ virtual threads (`spring.threads.virtual.enabled=true`) to improve request throughput  
- 🧱 Adopt newer language features where they make code clearer: records, pattern matching for `switch`, sequenced collections  
- 📈 Benchmark startup time and memory on JDK 25 against JDK 17, and look at AOT and CDS caching  
- 🔐 Add dependency vulnerability scanning to CI, for example the OWASP dependency-check plugin  
- 📝 Update the README, which still says "Java 17 or later", so it states the new Java 25 requirement  

## Kiro credit usage

💳 Kiro CLI credits used: 2.49  
Includes 0.23 for job setup (startup smoke test and knowledge-base indexing).  

## Codebase index refresh

- Outcome: Index current
- Reason: Index current at d1bce5444713e0e09be66233d6cc4d00a723699d.
- Previously indexed commit: d1bce5444713e0e09be66233d6cc4d00a723699d
- Source commit at transformation start: d1bce5444713e0e09be66233d6cc4d00a723699d
- Duration: 0.0s
