# Maven

[Apache Maven](https://maven.apache.org/) is a build automation and project management tool used primarily in the Java ecosystem. It standardizes how a project is compiled, tested, packaged, and published.

Maven is a *build* tool. It can be used in a CI/CD pipeline, but it is not, by itself, a CI/CD platform.

## What Maven does in practice

- **Manages dependencies:** in `pom.xml`, you declare the libraries and versions that your project needs. Maven downloads those libraries — including their transitive dependencies — from repositories such as Maven Central.
- **Standardizes the build lifecycle:** compilation, tests, packaging, local installation, and publishing follow well-defined phases such as `compile`, `test`, `package`, `install`, and `deploy`.
- **Favors convention over configuration:** it uses a familiar directory structure, such as `src/main/java`, `src/test/java`, and `src/main/resources`. This makes Maven projects more predictable to maintain.
- **Centralizes configuration:** the `pom.xml` file gathers project information, dependencies, plugins, version, repositories, and build rules.

Before Maven, it was common for every project to have its own process for compilation, library management, and artifact generation. Maven conventions reduce that inconsistency.

## Installation

See the [official installation instructions](https://maven.apache.org/install.html).

On Windows, the basic process is:

1. Install a JDK compatible with the project and configure the `JAVA_HOME` environment variable.
2. Download and extract the Maven distribution to the desired directory.
3. Add Maven's `bin` directory to the `Path` environment variable, for example: `C:\apache-maven-3.8.8\bin`.
4. Open a new terminal and verify the installation:

```bash
mvn --version
```

!!! tip "Prefer Maven Wrapper when the project provides it"
    Files such as `mvnw`, `mvnw.cmd`, and `.mvn/wrapper` let the team use the Maven version defined by the project. On Windows, run `./mvnw.cmd package` instead of `mvn package`.

## Conventional project structure

A Java Maven project normally follows this structure:

```text
my-project/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
        ├── java/
        └── resources/
```

- `src/main/java`: application source code;
- `src/main/resources`: configuration files and other resources;
- `src/test/java`: test source code;
- `src/test/resources`: resources used by tests;
- `target`: build output; it normally should not be versioned.

Mule projects have a structure of their own, recognized by the Mule Maven Plugin, but they still use `pom.xml` and the core Maven concepts.

## The role of `pom.xml`

The `pom.xml` (*Project Object Model*) is the central file of a Maven project. It describes what the project is, which components it needs, and how it should be built. Without it, Maven does not have the information required to build the project.

### Information declared by a POM

1. **Project identity:** the GAV coordinates — `groupId`, `artifactId`, and `version` — uniquely identify a Maven artifact.
2. **Dependencies:** libraries that must be available on the project's classpath.
3. **Build:** plugins, goals, compilation, tests, and packaging such as `jar`, `war`, or `pom`.
4. **Inheritance and composition:** a POM can inherit settings from a `parent` and aggregate projects in `modules`.
5. **Version management:** `dependencyManagement` can centralize versions, including through BOM import.

Simply put, Maven is the build engine; `pom.xml` is the instruction set that tells it what to build, which components to use, and which rules to follow.

### Common elements

| Element | Purpose |
| --- | --- |
| `groupId` | Identifies the responsible organization or group, for example `com.mycompany`. |
| `artifactId` | Technical name of the project or artifact. |
| `version` | Artifact version. |
| `dependencies` | Libraries actually used by the application. |
| `plugins` | Tools used during build, tests, packaging, or publishing. |
| `properties` | Reusable values, such as the Java version. |
| `repositories` | Locations where Maven looks for dependencies. |
| `build` | Build-process configuration. |

A simplified structure is:

```xml
<project>
    <!-- Project identification -->
    <properties>
        <!-- Reusable values -->
    </properties>
    <build>
        <plugins>
            <!-- Tools used by Maven -->
        </plugins>
    </build>
    <dependencies>
        <!-- Components used by the application -->
    </dependencies>
    <repositories>
        <!-- Locations to search for dependencies -->
    </repositories>
    <pluginRepositories>
        <!-- Locations to search for Maven plugins -->
    </pluginRepositories>
</project>
```

A minimal POM needs the model version and the project coordinates:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</project>
```

The `SNAPSHOT` suffix indicates a version under development; a version without that suffix normally represents a stable, immutable release.

When you run `mvn package`, Maven reads `pom.xml`, resolves the dependencies, compiles the code, runs the tests, and produces the final artifact — usually a `.jar` or `.war`.

In short: the POM declares the application and its needs; plugins define how to build it; dependencies provide the components it uses; and repositories indicate where to find those components.

## Dependencies

Dependencies are external libraries: third-party code that a project needs to compile or run. Instead of implementing common capabilities from scratch — such as reading JSON, connecting to a database, or making HTTP requests — an application can use existing libraries.

For example, a Java application can use Jackson to work with JSON:

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.0</version>
    </dependency>
</dependencies>
```

Maven locates the corresponding JAR in a remote repository, such as Maven Central, and makes it available to the project.

Maven also resolves transitive dependencies. If one library depends on another, Maven normally obtains the second one as well. Still, dependencies directly used by your code should be explicitly declared in the POM.

### Common scopes

A dependency `scope` tells Maven **when that dependency must be available**. Think of three moments:

- **compilation:** when Java transforms source code into classes;
- **runtime:** when the packaged application is running;
- **tests:** when tests are compiled and run.

For example, Jackson is normally used by both application code and the running application, so its default scope is `compile`. JUnit is used for testing, but should not be part of the delivered application, so it uses the `test` scope.

| Scope | Available in | When to use it |
| --- | --- | --- |
| `compile` | Compilation, runtime, and tests. | This is the default. Use it for libraries imported by the application code and used in production, such as Jackson. |
| `provided` | Compilation and tests, but not runtime. | Use it when the production environment supplies the library, such as an API provided by an application server. |
| `runtime` | Runtime and tests, but not main-code compilation. | Use it when the application does not import the API directly but needs its implementation at runtime, as with some JDBC drivers. |
| `test` | Test compilation and execution only. | Use it for test tools such as JUnit and Mockito. It does not accompany the production artifact. |

JUnit example:

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>COMPATIBLE_VERSION</version>
    <scope>test</scope>
</dependency>
```

This makes JUnit available to `mvn test`, but does not include it as a dependency required to run the application.

The `import` scope is different from the others: it does not add a library to the classpath. It can only be used with a dependency of type `pom`, inside `dependencyManagement`, to import versions managed by a BOM.

### Local repository

After the first download, Maven stores artifacts in a local repository and reuses them in other projects that require the same version:

| Operating system | Default location |
| --- | --- |
| Linux and macOS | `~/.m2/repository` |
| Windows | `C:\Users\YOUR_USER\.m2\repository` |

Files are organized by GAV coordinates. For the Jackson example, the path looks like this:

```text
~/.m2/repository/com/fasterxml/jackson/core/jackson-databind/2.17.0/
├── jackson-databind-2.17.0.jar
├── jackson-databind-2.17.0.pom
└── ...
```

In other words: `groupId` (with dots converted to directories) → `artifactId` → `version` → artifact files.

### How resolution works

1. You run a command such as `mvn compile` or `mvn install`.
2. Maven reads `pom.xml` to identify the required dependencies.
3. For each dependency, it checks the local repository first.
4. If it is not available locally, Maven searches a configured remote repository and stores it in `.m2`.
5. When the artifact already exists locally, Maven reuses it without downloading it again.

Besides Maven Central, organizations can use repositories such as Nexus, Artifactory, and Anypoint Exchange.

Every Maven project resolves declared dependencies when required. In offline mode (`mvn -o ...`), Maven only uses what is already in the local repository and fails if an artifact is missing.

!!! tip
    If a dependency is corrupted or has a resolution conflict, inspect the specific artifact directory in `~/.m2/repository`. Avoid clearing the entire cache unless necessary.

## Lifecycle

The Maven lifecycle is the standardized sequence of phases a project goes through during a build: from initial validation to publishing the final artifact. This gives projects a predictable flow for compilation, testing, and packaging.

### The three lifecycles

| Lifecycle | Purpose |
| --- | --- |
| `default` | Builds and publishes the project: compilation, tests, packaging, installation, and deployment. |
| `clean` | Removes artifacts generated by previous builds. |
| `site` | Generates the project's documentation or website. |

Each lifecycle has its own phases. `default` is used most often day to day, so it usually gets more attention. The main phases of the other two lifecycles are:

| Lifecycle | Phases | Result |
| --- | --- | --- |
| `clean` | `pre-clean` → `clean` → `post-clean` | Removes output from earlier builds; the `clean` phase normally removes the `target` directory. |
| `site` | `pre-site` → `site` → `post-site` → `site-deploy` | Prepares, generates, finalizes, and, when configured, publishes project documentation. |

When you run a phase, Maven also runs the preceding phases **in that same lifecycle**. For example, `mvn clean` runs `pre-clean` and `clean`; `mvn site` runs `pre-site` and `site`.

### Phases of the `default` lifecycle

| Phase | What it does |
| --- | --- |
| `validate` | Checks that the project is correct and that required information, such as the POM, is available. |
| `compile` | Compiles the source code. |
| `test` | Runs unit tests without packaging the project yet. |
| `package` | Packages compiled code in the defined format, such as `.jar` or `.war`. |
| `verify` | Runs additional checks, usually related to integration tests. |
| `install` | Installs the package in the local repository, making it available to other local projects. |
| `deploy` | Publishes the package to a remote repository for other teams or projects to consume. |

Phases are cumulative. When you request a phase, Maven runs all previous phases in order. Therefore:

```bash
mvn install
```

runs `validate` → `compile` → `test` → `package` → `verify` → `install`.

Practical examples:

- `mvn compile`: compiles the project;
- `mvn test`: compiles and runs tests;
- `mvn package`: compiles, tests, and produces the `.jar` or `.war`;
- `mvn install`: performs the previous work and installs the artifact in `~/.m2/repository`;
- `mvn deploy`: performs the previous work and publishes the artifact to a remote repository.

A **phase** is a lifecycle stage. A **goal** is a specific task provided by a plugin. In the example below, `clean` and `package` are phases, while `dependency:tree` is a goal:

```bash
mvn clean dependency:tree package
```

## Plugins and useful commands

Maven coordinates the build, but plugins perform the concrete work, such as compiling, testing, and creating packages. For example, `mule-maven-plugin` lets Maven recognize and build projects with `mule-application` packaging.

```bash
# Removes the previous build output and creates a new package
mvn clean package

# Shows the resolved dependency tree
mvn dependency:tree

# Shows the effective POM, including inheritance, profiles, and default values
mvn help:effective-pom

# Shows available and active profiles
mvn help:active-profiles
```

## Maven in MuleSoft applications

In Mule applications, `pom.xml` also sets `packaging` to `mule-application`, uses `mule-maven-plugin`, and declares connectors, modules, and API specifications as dependencies. A BOM can be imported in `dependencyManagement` to centralize versions for those components.

The detailed BOM and MuleSoft examples are currently available in Portuguese: [BOM and version management](../../pt/maven/bom.md) and the [complete MuleSoft example](../../pt/maven/exemplo-bom-mulesoft.md).

## References

- [Apache Maven](https://maven.apache.org/)
- [Maven installation](https://maven.apache.org/install.html)
- [POM reference](https://maven.apache.org/pom.html)
- [Maven dependency mechanism](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)
