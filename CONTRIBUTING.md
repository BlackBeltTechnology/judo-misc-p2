# Contributing to JUDO

## Development Environment

Make sure your environment meets the requirements described in the parent project's [CONTRIBUTING guide](https://github.com/BlackBeltTechnology/judo-community/blob/develop/CONTRIBUTING.adoc). At minimum you need:

- **Java 17** JDK (Zulu distribution recommended)
- **Maven 3.9+** (or use the bundled `./mvnw` wrapper)

## Code Structure

This project is a single-module Maven POM project (no Java source code). The key files are:

| File | Purpose |
|------|---------|
| `pom.xml` | Declares Maven artifacts to bundle and P2 feature definitions |
| `category.xml` | Defines P2 categories that group features in the Eclipse update site |
| `src/assembly/assembly.xml` | Configures how the P2 repository is packaged into a ZIP |

Build output is generated under the `target/` directory.

## Submitting an Issue

Before filing a new issue, search the [issue tracker](https://github.com/BlackBeltTechnology/judo-misc-p2/issues) — your problem may already be reported or resolved.

To help us reproduce and fix bugs quickly, please include:

- Output of `java -version` and `mvn -version`
- Relevant `pom.xml` or `.flattened-pom.xml` snippets
- A minimal reproduction case that demonstrates the failure

## Submitting a PR

This project follows [GitHub's standard forking model](https://guides.github.com/activities/forking/). Fork the repository and submit pull requests from your fork.

> **Important:** Every commit must reference a JIRA ticket number (e.g., `JNG-1234`). See the [CI Flow](/.github/CIFLOW.md) documentation for branching conventions.

## Build Commands

```bash
# Full build
./mvnw clean install

# Package only
./mvnw clean package
```
