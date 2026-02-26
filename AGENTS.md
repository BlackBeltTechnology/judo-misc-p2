# Judo Misc P2 Site - Project Documentation

## Project Overview

**Repository:** BlackBeltTechnology/judo-misc-p2
**License:** Eclipse Public License 2.0 (EPL-2.0)
**Java Version:** 17
**Build System:** Maven 3.9.4 with Maven Wrapper (`./mvnw`)

1. This project creates an **Eclipse P2 update site** by repackaging Maven Central artifacts as OSGi bundles
2. It currently bundles **Apache Commons Text 1.9** (with transitive dependencies) into a P2 feature
3. The output is a self-contained P2 repository (ZIP archive) that Eclipse IDE can consume as an update site
4. It is part of the [judo-community](https://github.com/BlackBeltTechnology/judo-community) aggregator ecosystem

## Directory Structure

```
judo-misc-p2/
├── pom.xml                    # Main build file — artifact declarations, feature definitions, profiles
├── category.xml               # P2 category definitions (groups features in Eclipse update site)
├── src/assembly/assembly.xml  # Maven assembly descriptor — packages P2 repo as ZIP
├── .github/workflows/         # GitHub Actions CI/CD pipelines (build, release, merge)
├── .github/CIFLOW.md          # CI/CD workflow documentation
├── openspec/                  # OpenSpec configuration and change tracking
├── .vscode/settings.json      # VS Code Java settings
├── .zed/settings.json         # Zed editor Java settings
├── README.md                  # Project overview and usage
├── CONTRIBUTING.md            # Contribution guidelines
├── CHANGELOG.md               # Release changelog
└── LICENSE.txt                # EPL-2.0 license text
```

## Technology Stack

### Core Technologies

- **p2-maven-plugin 2.0.0** (org.reficio) — converts Maven artifacts to OSGi bundles and generates P2 repository metadata
- **maven-assembly-plugin 3.4.2** — packages the `target/repository/` directory into a distributable ZIP
- **flatten-maven-plugin 1.3.0** — resolves CI-friendly `${revision}` version property

### Build & Quality

- **Maven 3.9.4** via wrapper (`./mvnw`)
- **JaCoCo 0.8.8** for code coverage
- **SonarQube** integration (sonar-maven-plugin 3.9.1.2184)
- **maven-surefire-plugin 3.0.0-M7** for test execution

## Build Commands

```bash
# Full build and install
./mvnw clean install

# Package P2 site (produces target/repository/ and ZIP)
./mvnw clean package

# Deploy to JudoNG Nexus
./mvnw -Prelease-judong deploy

# Upload P2 repository to Nexus via WebDAV
./mvnw -Prelease-p2-judong deploy

# Deploy to Maven Central (with signing)
./mvnw -Psign-artifacts,release-central deploy

# Test deploy to /tmp/
./mvnw -Prelease-dummy deploy
```

### Maven Profiles

| Profile | Purpose |
|---------|---------|
| `sign-artifacts` | GPG-sign artifacts using `sign-maven-plugin` |
| `release-dummy` | Deploy to local `/tmp/` directory for testing |
| `release-judong` | Deploy to JudoNG Nexus (`nexus.judo.technology`) |
| `release-central` | Deploy to Maven Central via OSSRH (auto-release, 15min timeout) |
| `release-p2-judong` | Upload P2 repository to Nexus via WebDAV |
| `generate-github-asciidoc-diagrams` | Generate PlantUML diagrams from AsciiDoc sources |
| `update-source-code-license` | Update file headers with EPL-2.0 license |

## Key Configuration Files

| File | Purpose |
|------|---------|
| `pom.xml` | Declares bundled artifacts in `<featureDefinitions>`, all build plugins, and deployment profiles |
| `category.xml` | Maps P2 features to categories (`judo_misc`, `judo_misc_source`) displayed in Eclipse |
| `src/assembly/assembly.xml` | Defines the ZIP assembly that packages `target/repository/` |
| `.github/workflows/build.yml` | Main CI — builds, deploys, tags, creates GitHub releases |
| `.github/workflows/release.yml` | Manual release trigger — creates PRs to master and bumps develop version |

## How to Add a New Dependency

1. In `pom.xml`, add an `<artifact>` entry inside an existing `<feature>` block (or create a new `<feature>` in the `<featureDefinitions>` section):
   ```xml
   <artifact>
       <id>group.id:artifact-id:version</id>
       <transitive>true</transitive>
       <source>true</source>
   </artifact>
   ```
2. In `category.xml`, add `<feature>` and `<category-def>` entries for the new feature
3. Run `./mvnw clean package` and verify bundles appear in `target/repository/plugins/`

## Development Environment

**Required:**
- Java 17 JDK (Zulu distribution recommended)
- Maven 3.9+ (or use `./mvnw` wrapper)

**No Java source code exists in this project.** It is purely a Maven POM project that orchestrates the p2-maven-plugin to generate an Eclipse update site.

## Git Workflow

- **Main Branch:** `develop`
- **Release Branch:** `master`
- **Versioning:** `${revision}` property (currently `1.0.1-SNAPSHOT`), resolved by flatten-maven-plugin
- **Branching Model:** GitFlow — see [CIFLOW.md](.github/CIFLOW.md) for details
- **Commit Rule:** Every commit must reference a JIRA ticket (`JNG-xxx`)

## Important Notes

1. This project has **no Java source code** — it is a packaging/aggregation project only
2. The `p2-maven-plugin` does the heavy lifting: resolving Maven artifacts, wrapping them as OSGi bundles, and generating P2 metadata
3. The `${revision}` property in `pom.xml` is the single source of truth for the version; CI pipelines manipulate it during releases
4. When modifying bundled artifacts, always test locally with `./mvnw clean package` and inspect `target/repository/` before pushing
5. The P2 site ZIP is the primary distributable artifact — it can be used as an Eclipse update site URL

## Related Documentation

- [README.md](README.md) — Project overview and usage instructions
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to contribute
- [CIFLOW.md](.github/CIFLOW.md) — CI/CD workflow and branching strategy
- [CHANGELOG.md](CHANGELOG.md) — Release history
