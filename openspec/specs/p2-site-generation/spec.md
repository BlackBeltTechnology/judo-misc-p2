# P2 Site Generation Specification

## Purpose

Generates an Eclipse P2 update site by converting Maven Central artifacts into OSGi bundles, packaging them with P2 metadata so Eclipse-based tooling can install them via the standard update-site mechanism.

## Architecture

The build pipeline consists of three stages orchestrated by Maven:

1. **p2-maven-plugin** resolves declared Maven artifacts (with transitive dependencies), wraps each JAR as an OSGi bundle, and generates P2 repository metadata (`content.jar`, `artifacts.jar`)
2. **category.xml** assigns features to named categories (`judo_misc`, `judo_misc_source`) displayed in the Eclipse update site UI
3. **maven-assembly-plugin** packages the `target/repository/` directory into a distributable ZIP archive

Key configuration files:
- `pom.xml` — `<featureDefinitions>` block declares which Maven artifacts to bundle and how features are structured
- `category.xml` — maps features to categories
- `src/assembly/assembly.xml` — defines the ZIP packaging format

## Requirements

### Requirement: Maven artifacts SHALL be converted to OSGi bundles

Each Maven artifact declared in a `<feature>` block must be resolved, wrapped with OSGi `MANIFEST.MF` metadata, and placed in the P2 repository's `plugins/` directory.

#### Scenario: Bundle Apache Commons Text with transitive dependencies
- **GIVEN** the feature `apache.commons.text.feature` declares `org.apache.commons:commons-text:1.9` with `<transitive>true</transitive>`
- **WHEN** `./mvnw clean package` is executed
- **THEN** `target/repository/plugins/` contains OSGi bundles for commons-text and all its transitive dependencies (e.g., commons-lang3)

### Requirement: Source bundles SHALL be generated when requested

When `<source>true</source>` is set on an artifact and `<generateSourceFeature>true</generateSourceFeature>` is set on the feature, corresponding source bundles must be generated.

#### Scenario: Source feature generation for commons-text
- **GIVEN** the `apache.commons.text.feature` has `<generateSourceFeature>true</generateSourceFeature>` and the artifact has `<source>true</source>`
- **WHEN** the P2 site is built
- **THEN** a `apache.commons.text.feature.source` feature is generated with source JARs in `target/repository/plugins/`

### Requirement: P2 metadata SHALL be generated

The P2 repository must contain valid `content.jar` and `artifacts.jar` metadata files so Eclipse can discover and resolve installable units.

#### Scenario: Valid P2 repository structure
- **GIVEN** a successful build
- **WHEN** inspecting `target/repository/`
- **THEN** the directory contains `content.jar`, `artifacts.jar`, `plugins/`, and `features/` directories

### Requirement: Categories SHALL organize features in the update site

Features must be assigned to categories as defined in `category.xml` so they appear grouped in the Eclipse Install New Software dialog.

#### Scenario: Category assignment
- **GIVEN** `category.xml` maps `apache.commons.text.feature` to category `judo_misc`
- **WHEN** an Eclipse user points at the P2 site
- **THEN** the feature appears under the "Judo Misc" category

### Requirement: P2 site SHALL be packaged as a distributable ZIP

The assembly plugin must produce a ZIP file containing the complete P2 repository that can be served as an Eclipse update site.

#### Scenario: ZIP archive creation
- **GIVEN** a successful build
- **WHEN** `./mvnw clean package` completes
- **THEN** `target/judo-misc-p2-*-site.zip` exists and contains the full contents of `target/repository/` at its root (no base directory prefix)

### Requirement: New dependencies SHALL be addable via configuration

Adding a new third-party library to the P2 site must only require configuration changes (no code changes).

#### Scenario: Adding a new Maven artifact
- **GIVEN** a developer wants to add `com.example:new-lib:2.0`
- **WHEN** they add an `<artifact>` entry in the p2-maven-plugin configuration in `pom.xml` and a `<feature>`/`<category-def>` in `category.xml`
- **THEN** running `./mvnw clean package` produces a P2 site containing the new library as an OSGi bundle
