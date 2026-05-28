# Maven Project Template

This project is a ready-to-use `pom.xml` and `checkstyle-suppressions.xml` with defaults designed to fit well when
working with a team of developers. It is designed to promote a standard way of developing by providing common commands
and enforcing strict code quality.

- [Customization Guide](#customization-guide)
- [The Development Lifecycle](#the-development-lifecycle)
- [CI/CD Integration](#cicd-integration)
- [Versioning and Release Conventions](#versioning-and-release-conventions)

## Customization Guide

When using this template, consider updating the following in `pom.xml`:

### 1. Project Metadata

* **`groupId`**
* **`artifactId`**
* **`version`**: 1.0.0-SNAPSHOT is default to adhere to [SemVer](https://semver.org/) while also allowing Windows native
  builds by starting
  at v1.x.x

### 2. Dependency & Plugin Updates

When starting a new project, it is recommended to check for newer versions of the included dependencies and plugins:

* `mvn versions:display-dependency-updates`
* `mvn versions:display-plugin-updates`

### 3. Main Class Paths

Update the `<mainClass>` to point to the correct program entrypoint.

### 4. SCM Configuration

Update the `<scm>` section with the remote repository URLs.

### 5. Style Rule Adjustments

The `checkstyle-suppressions.xml` file can be used to relax specific Google Java Style rules. Common suppressions for
abbreviation naming and variable usage distance are included by default.

### 6. Optional Components

If the project does not require specific features, they can be removed:

* **Integration Tests**: `maven-failsafe-plugin`
* **Test Coverage**: `jacoco-maven-plugin`
* **Code Styling**: `spotless-maven-plugin` and `maven-checkstyle-plugin`
* **Release Management**: `maven-release-plugin`
* **Documentation**: `maven-javadoc-plugin`
* **Dependency Updates**: `versions-maven-plugin`
* **Native Binary Support**: `jpackage-maven-plugin` (and the `native` profile)

## The Development Lifecycle

The build system is designed to use `mvn verify` as the source of truth; if it passes verify, it is ready to be merged.

### 1. Local Development

* **Rapid Iteration**: Use `mvn clean package -DskipTests -Dspotless.check.skip -Dcheckstyle.skip` for fast builds that
  bypass styling and tests.
* **Auto-Formatting**: Run `mvn spotless:apply` to fix styling issues automatically.
* **Testing**:
    * `mvn test`: Runs unit tests only.
    * `mvn verify -DskipITs`: Runs style checks, dependency analysis, and unit tests, but skips integration tests.
    * `mvn verify`: Runs the full project validation including integration tests.
* **Quality Gate**: Before opening a PR, run `mvn clean verify` locally. This executes:
    * **Style Checks**: Google Java Style (Spotless & Checkstyle).
    * **Dependency Hygiene**: Fails the build if dependencies are unused or undeclared.
    * **Environment Check**: Enforces Java 25+ and Maven 3.9+.

### 2. Creating a Release

The template uses the **Maven Release Plugin** to manage the project lifecycle.

* **Command:** Run `mvn release:prepare` and push the resulting commits and tags to your remote repository.
* If you are unfamiliar with how Maven handles `-SNAPSHOT` vs. release versions, please read
  the [Versioning and Release Conventions](#versioning-and-release-conventions),
  or refer to the [official Maven versioning guide](https://maven.apache.org/guides/mini/guide-naming-conventions.html).

### 3. Native Packaging (`-Pnative`)

The `native` profile leverages `jpackage` to create platform-specific installers.

* **Platform Neutrality**: The configuration is intentionally "bare." It relies on the host OS to decide the output:
    * **Windows**: Produces `.msi` or `.exe`.
    * **macOS**: Produces `.dmg` or `.pkg`.
    * **Linux**: Produces `.deb` or `.rpm`.

## CI/CD Integration

This `pom.xml` is created with GitHub Actions in mind. A typical release workflow could:

1. Run on `windows-latest`, `macos-latest`, and `ubuntu-latest` runners.
2. Execute `mvn clean verify -Pnative -DskipTests`.
3. Upload the resulting artifacts from `target/native/*` to a GitHub Release.

## Versioning and Release Conventions

You are free to use whatever versioning you'd like, but by default this template
uses [Semantic Versioning (SemVer)](https://semver.org/) combined with Maven's standard lifecycle suffixes.
Versions follow the `MAJOR.MINOR.PATCH` format (e.g., `1.0.0`).

* **Development:** Use the `-SNAPSHOT` suffix (e.g., `1.0.0-SNAPSHOT`) to indicate a work-in-progress state.
* **Release:** Remove the `-SNAPSHOT` suffix (e.g., `1.0.0`) for project milestones or final deliveries. This signifies
  a stable version.
* **Update Cycle:** Upon completing a release, the version is typically incremented (e.g., to `1.1.0-SNAPSHOT`) to begin
  the next development cycle.

### Automating the Release

Version updates, Git tagging, and local build validation can be handled by the Maven Release Plugin:

```bash
mvn release:prepare
```

This will suggest a default next version number which can be accepted by pressing **Enter**, or overwritten.

### Remote Sync

This template is designed to work with CI/CD pipelines (e.g., GitHub Actions). Once `mvn release:prepare` is finished,
the local repository will contain the new version commit and the release tag. To trigger the CI/CD pipeline, push
manually:

```bash
# Push the branch and the new release tag
git push origin <branch-name>
git push origin <tag-name>
```

