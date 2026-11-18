# Java 25 Upgrade — Iteration 29

## Status

**Both builds pass with zero compilation errors.**

| Build | Command | Result |
| --- | --- | --- |
| Maven | `./mvnw -B clean verify` | BUILD SUCCESS |
| Gradle | `./gradlew clean build` | BUILD SUCCESSFUL |

Tests: **79 run, 0 failures, 0 errors, 2 skipped.**

The 2 skips are `MySqlIntegrationTests`, gated by the pre-existing
`@Testcontainers(disabledWithoutDocker = true)` — this machine has no Docker. No test was
deleted, disabled, skipped or weakened; the only test change this iteration **adds**
assertions.

The Maven `verify` lifecycle ran green end to end: `enforcer` (requires Java >= 25),
`spring-javaformat:validate`, `checkstyle` (nohttp), `compiler` (`release 25`),
`surefire`, `jacoco`, `jar` and `spring-boot:repackage`.

## Scope correction: this is a pure JDK hop

Base commit `d1bce54` **already** declared `spring-boot-starter-parent` 4.1.0 and resolves
Hibernate 7.4.1.Final. Only `java.version` moved, 17 → 25. The Spring Boot 4 / Jakarta /
Hibernate 7 migration happened in earlier work, so this iteration neither performed nor
re-risked it. Risk statements below are scoped accordingly.

## Carry-over from iteration 28 — disposition

| Item | Outcome |
| --- | --- |
| Check byte-buddy constraint | **Resolved — not needed.** See below. |
| Functional check: owners, pets, visits, vets | **Already covered** by the existing suite. See below. |
| Functional check: JCache `vets` cache | **Resolved — now actually asserted.** See below. |
| Start the app / `ENABLE_RUNTIME_VALIDATION=true` | **Not done.** Blocked; see Limits. |
| Run MySQL / Postgres Testcontainers tests | **Not done.** Docker unavailable here (`docker info` fails). |
| Watch first CI run; rebuild devcontainer + Gitpod image | **Not done.** Requires a push and a container build; see Limits. |
| Review commit `09bf56d`, open a PR | **Not applicable / not done.** See Limits. |
| Improvements: startup/GC, virtual threads, AOT cache | **Not implemented** by design — each needs its own benchmark and a running app. |

### byte-buddy: the constraint is not needed

The advisory in `code-upgrade.log:174` wanted `net.bytebuddy:byte-buddy` forced to 1.17.x
"because Byte Buddy older than 1.17 cannot read Java 21+ class files". Measured instead of
assumed:

- Maven: `net.bytebuddy:byte-buddy:1.18.10`, `byte-buddy-agent:1.18.10` (`dependency:tree`).
- Gradle: `1.18.10` for both (resolved cache).
- `./mvnw help:evaluate -Dexpression=byte-buddy.version` → `1.18.10`, i.e. the version is
  **BOM-managed** by `spring-boot-dependencies` 4.1.0, not incidental.

1.18.10 is well clear of the 1.17 floor, identically in both builds, and governed by the
BOM. **Decision: do not add the pin** to `build.gradle` or `pom.xml`. Adding one would
override a BOM-managed version for no benefit and would have to be re-reviewed on every
Boot upgrade — the opposite of the "keep BOM-managed versions" rule.

### Functional coverage of owners, pets, visits and vets already exists

No new tests were written for these, because `ClinicServiceTests` already exercises them
against a real database (`@DataJpaTest` with `@AutoConfigureTestDatabase(replace = NONE)`,
so the configured datasource and real Hibernate 7.4.1 — not mocks). All 12 passed on JDK 25:

- **owners** — `shouldFindOwnersByLastName`, `shouldFindSingleOwnerWithPet`,
  `shouldInsertOwner`, `shouldUpdateOwner`
- **pets** — `shouldFindAllPetTypes`, `shouldInsertPetIntoDatabaseAndGenerateId`,
  `shouldUpdatePetName`, `shouldFailToInsertDuplicatePetNameForSameOwner`,
  `shouldAllowSamePetNameForDifferentOwners`
- **visits** — `shouldAddNewVisitForPet`, `shouldFindVisitsByPetId`
- **vets** — `shouldFindVets` (including specialties)

On top of that, `PetClinicIntegrationTests` boots the **whole** application on a real
Tomcat at a random port and drives `/owners/1` and `/owners?lastName=` over real HTTP.
Duplicating this as new integration tests would have added maintenance cost and no signal.

### The JCache `vets` cache was the one genuinely unverified area — now closed

Three tests called `vets.findAll()` twice with a `// served from cache` comment, but
**nothing asserted the cache was engaged**; a broken cache would have passed silently. The
assertion was added to the existing `PetClinicIntegrationTests.findAll()` (the full-boot
test that runs without Docker) rather than in a new test:

```java
Cache vetsCache = cacheManager.getCache("vets");
assertThat(vetsCache).as("the 'vets' cache should be configured").isNotNull();
vetsCache.clear();
assertThat(vetsCache.get(SimpleKey.EMPTY)).as("the 'vets' cache should start empty").isNull();

Collection<Vet> firstCall = vets.findAll();
assertThat(vetsCache.get(SimpleKey.EMPTY)).as("findAll() should populate the 'vets' cache").isNotNull();

Collection<Vet> secondCall = vets.findAll(); // served from cache
assertThat(secondCall).hasSameSizeAs(firstCall);
```

The empty-cache assertion after `clear()` is deliberate: it makes the test self-validating,
proving the later non-null entry comes from the `@Cacheable` interception rather than
passing vacuously. This verifies `@EnableCaching` + `JCacheManagerCustomizer` + the
`javax.cache` JSR-107 path + Spring proxying all still work on JDK 25. It is written
against Spring's `CacheManager`/`Cache` abstraction, so it is not coupled to the provider.

### Hibernate 7 silent-break checklist (KB 36 / 10b) — assessed, none apply

Hibernate 7.4.1 is in use, so this checklist was worked rather than assumed. A green build
proves nothing about these, so each was scanned:

| Checklist item | Finding |
| --- | --- |
| Native queries now return `java.time` types | **Not applicable.** The only `@Query` is JPQL (`PetTypeRepository:36`); no `nativeQuery = true`, no `createNativeQuery`. Date fields are already `java.time.LocalDate` via JPA. |
| `StatelessSession` now uses the L2 cache | **Not applicable.** No `StatelessSession` anywhere. |
| `refresh()` / `lock()` on a detached entity throws | **Not applicable.** No `refresh()`, `lock()` or direct `EntityManager` use; data access is via Spring Data repository interfaces. |

The CDI 4 items do not apply either (Spring application, no `beans.xml`).

## Other changes made this iteration

### Reverted three instance-main conversions

OpenRewrite's `MigrateMainMethodToInstanceMain` had converted the Spring Boot launcher
entry points to JDK 25 instance main methods:

| File | Recipe output | Restored to |
| --- | --- | --- |
| `MysqlTestApplication.java` | `void main()` | `public static void main(String[] args)` |
| `PetClinicIntegrationTests.java` | `void main()` | `public static void main(String[] args)` |
| `PostgresIntegrationTests.java` | `void main(String[] args)` | `public static void main(String[] args)` |

Reverted per `05-java-21-to-25-migration.md:107` — keep the traditional main for framework
code, reserve instance main for scripts and CLI tools. These three are developer launchers
for the app against MySQL / Postgres / Docker Compose. The instance form compiles and is
launchable under JEP 512, but it drops the `public static void main` signature that IDE and
tooling "run as application" discovery looks for, and `PostgresIntegrationTests` also lost
the `args` pass-through to `SpringApplicationBuilder.run(args)`. These files now match
their pre-upgrade content exactly.

The other recipe modernizations were kept: `.getFirst()`, `catch (Exception _)` and
`IO.println` (`java.lang.IO`, final in JDK 25 via JEP 512) all compile on `release 25` and
their tests pass. The recipe's Mockito `-javaagent` surefire wiring was also kept — it is
the correct JDK 25 fix, loading Mockito as an explicit agent instead of relying on
self-attach, which modern JDKs restrict.

### Aligned the JDK pins the build depends on

The enforcer rule now requires Java >= 25, so every environment that runs the build had to
move with it:

- `.github/workflows/maven-build.yml`, `.github/workflows/gradle-build.yml`: JDK matrix
  `'17'` → `'25'`, and `setup-java` distribution `adopt` → `temurin`. `adopt` is the
  retired AdoptOpenJDK and publishes no JDK 25, so the version bump alone would not have
  resolved.
- `.devcontainer/devcontainer.json`: Java feature version `21-oracle` → `25`.
- `.devcontainer/Dockerfile` (used by Gitpod, not the devcontainer): base image
  `mcr.microsoft.com/vscode/devcontainers/java:0-17-bullseye` →
  `mcr.microsoft.com/devcontainers/java:25-bookworm` (the `vscode/devcontainers` path is
  deprecated in favour of `devcontainers`), and SDKMAN `JAVA_VERSION` `17.0.7-ms` →
  `25.0.5.0+2-ms`, keeping the existing Microsoft vendor.

### Corrected an obsolete constraint comment

`pom.xml` carried `<!-- Checkstyle needs to stay on v12 since v13 sets minimal jdk to 21 -->`,
which no longer applies on JDK 25; the comment was corrected. Checkstyle itself was
deliberately **left at 12.3.1** — it builds and is consistent, and bumping it is not needed
for a green build.

## Verification performed

- `./mvnw -B clean verify` — full lifecycle, green, 79 tests.
- `./gradlew clean build --no-daemon` — green (Gradle 9.5.1 supports Java 25).
- Resolved and cross-checked byte-buddy and Hibernate versions in **both** builds.
- Scanned `src/` for the Hibernate 7 and CDI 4 silent-break patterns from KB 36 — none present.
- Swept `src/` for legacy patterns: no `SecurityManager`, no `AccessController`, no
  `finalize()`, no JUnit 4 imports, no Java EE `javax.*` imports. The single remaining
  `javax.*` import (`javax.cache.configuration.MutableConfiguration`) is JSR-107 JCache and
  correctly stays `javax` — Jakarta EE never adopted JCache and there is no `jakarta.cache`.
- Swept the repository for stale JDK pins; all now read 25.

## Limits of this iteration — what was NOT verified

- **The application was never started.** My operating instructions prohibit executing the
  compiled application for verification (build + tests only), and
  `ENABLE_RUNTIME_VALIDATION` is a harness-level toggle outside this session's control.
  The closest available evidence is `PetClinicIntegrationTests`, which does boot the full
  app on real Tomcat — but no browser-level or manual functional pass was done.
- **Docker is unavailable here** (verified: `docker info` fails), so
  `MySqlIntegrationTests` (2 skipped) and `PostgresIntegrationTests` (0 tests executed —
  its `@BeforeAll` assumes Docker) have still never run on JDK 25. **The MySQL and
  Postgres paths remain unexercised**, and the Hibernate 7 dialect behaviour for those two
  databases is therefore unverified — H2 is the only database any test has touched.
- **No container image was built**, so the devcontainer and Gitpod changes are unvalidated.
  `devcontainers/java:25-bookworm` and SDKMAN `25.0.5.0+2-ms` are intended values. If the
  Gitpod image fails to build, the SDKMAN patch identifier is the likely cause — run
  `sdk list java | grep 25` and pin a published identifier.
- **No CI run was observed and nothing was pushed.** Pushing and opening a PR are
  outward-facing actions I was not authorized to take.
- **Commit `09bf56d` is not in this working tree.** That was iteration 28's commit; this
  iteration restarted from base `d1bce54`. The work here is **uncommitted** on branch
  `main-upgrade`: 9 files changed, 60 insertions, 23 deletions. No production source under
  `src/main` changed.

## Next steps

- 👀 **Review and commit.** The changes are uncommitted; commit and open a PR when ready.
- 🐳 **Run the Docker-based tests on a machine with Docker** — this is the largest remaining
  gap, since MySQL and Postgres have never been exercised on JDK 25 / Hibernate 7.4.1.
- ▶️ **Do a manual functional pass** over owners, pets, visits and vets in a test
  environment, to cover what the automated suite cannot.
- 🐳 **Build the devcontainer and Gitpod images** to validate the image tag and SDKMAN
  identifier.
- 🤖 **Watch the first GitHub Actions run** on JDK 25 / Temurin — both workflows changed
  distribution as well as version. Note `gradle-build.yml` still has `cache: maven` on its
  `setup-java` step, a pre-existing oddity for a Gradle build, left untouched as out of scope.
- 🔁 **Optional: mirror the cache assertion** into `MySqlIntegrationTests` and
  `PostgresIntegrationTests`, which still carry the unasserted `// served from cache`
  comment. Deliberately not done here because those tests cannot run in this environment
  and the edit would be unverifiable.
- 🖨️ **Optional: replace `IO.println`** in `PetClinicConcurrencyTests` with a logger.
  It compiles and passes, but `05-java-21-to-25-migration.md:187` prefers structured
  logging for server-side code. Cosmetic, not a correctness issue.
- ⬆️ **Optional: Checkstyle 13** is now viable (JDK 25 satisfies its minimum of 21). Keep it
  a separate, independently tested change — v13 adds checks that may flag existing code.
- 📈 **Optional JDK 25 improvements, each benchmarked on its own:** compare startup, memory
  and GC against Java 17; try virtual threads (`spring.threads.virtual.enabled=true`); try
  the JDK 25 AOT cache.
