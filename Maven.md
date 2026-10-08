# Maven
[Maven](https://maven.apache.org/) is the preferred build tool for all Java projects at Unigrid. New projects use Maven, and we do
not add Gradle, Ant or other build files next to a Maven build. The reference setups are
[Hedgehog](https://github.com/unigrid-project/hedgehog) and [Janus](https://github.com/unigrid-project/janus-java), so when in doubt
about how to set something up, do what they do.

## Using it
Both projects need JDK 25 or later and Maven. A build on an older JDK is refused by the enforcer plugin.

```
mvn clean install              # build, test and check style
mvn clean install -DskipTests  # build only
mvn verify                     # everything up to and including checkstyle and the integration tests
mvn test -Dtest=MyTest         # a single test class
mvn -pl shell exec:java        # run a module, e.g Janus
```

Heavy modules are kept out of the everyday build and are turned on with a profile. In Janus, `-Pinstaller` builds the native
installers and `-Pbrowser` runs the end-to-end tests in a real browser:

```
mvn -Pinstaller verify -DskipTests
```

Style is checked as part of the build, so `mvn test` alone does not run checkstyle. Run `mvn install` or `mvn verify` before you push.

## How a project is set up
 * A parent `pom.xml` with `<packaging>pom</packaging>` lists the `<modules>`. Hedgehog has `application`, `common` and
   `native-image`. Janus has `core`, `web`, `ui` and `shell`, plus `desktop` and `e2e` in profiles.
 * Set the Java version with `<maven.compiler.release>25</maven.compiler.release>` and `<project.build.sourceEncoding>UTF-8`.
 * Keep every dependency version as a property in the parent (`<lombok.version>`, `<jersey.version>` and so on) and use the property
   in `<dependencyManagement>`. The modules then list dependencies without versions.
 * Pin the version of every plugin, in `<pluginManagement>` of the parent. No version ranges and no `LATEST`.
 * Configure plugins once in the parent and only name them in the modules that need them.
 * Use a profile, not a comment or a flag, for anything that is slow or needs special tools.
 * Indent the pom with tabs, like the Java code, and add a short comment above a plugin or a profile that is not obvious.
 * Keep `.mvn/jvm.config` for JVM options of Maven itself, e.g `--enable-native-access=ALL-UNNAMED`.

## Recommended plugins
These are used in Hedgehog and Janus. Take the same versions as the existing projects, unless you have a reason to move.

| Plugin | Use it for | Notes |
| --- | --- | --- |
| `maven-enforcer-plugin` | Refuse to build on the wrong JDK | `requireJavaVersion` `[25,)` with a message that tells the user to set `JAVA_HOME` |
| `maven-compiler-plugin` | Compile | `-Xlint:unchecked` and `-Xlint:deprecation`, and Lombok under `annotationProcessorPaths`, see [Lombok](Lombok.md) |
| `maven-surefire-plugin` | Unit tests | Hedgehog runs tests in parallel forks (`forkCount` from the core count) and attaches the JaCoCo and JMockit agents in `argLine` |
| `maven-failsafe-plugin` | Integration tests | Goals `integration-test` and `verify`. Used by Janus for the tests that start a server or a browser |
| `jacoco-maven-plugin` | Test coverage | `prepare-agent` before the tests and `report` after them. Janus merges the unit and integration runs into one report |
| `maven-checkstyle-plugin` | Enforce the code style | `check` goal with the project `checkstyle.xml`, engine `com.puppycrawl.tools:checkstyle`. Fails the build on a violation. `module-info.java` is excluded |
| `maven-pmd-plugin` | Static analysis | Janus runs `pmd:pmd` in `verify` as a report only, with the project `pmd.xml`. Nothing fails on a finding |
| `spotbugs-maven-plugin` | Bug patterns | Used as a report in Hedgehog |
| `maven-assembly-plugin` | A fat jar that runs with `java -jar` | Hedgehog sets the `mainClass` in the manifest and uses the `jar-with-dependencies` descriptor |
| `maven-jar-plugin` | The thin jar of a module | |
| `jandex-maven-plugin` | An index of the classes for the CDI container | Used in Hedgehog so Weld finds the beans without scanning |
| `build-helper-maven-plugin` | Small build helpers | Hedgehog uses `cpu-count` to size the test forks |
| `exec-maven-plugin` | Run the application from Maven | `mvn -pl shell exec:java` in Janus. Also runs helper steps in the installer build |
| `maven-dependency-plugin` | Copy dependencies and files into the build | Used when packaging |
| `native-maven-plugin` (GraalVM) | Compile the native launcher | Hedgehog `native-image` module, see [GraalVM](GraalVM.md) |
| `maven-release-plugin` | Tag and version a release | Hedgehog sets `tagNameFormat` to `v@{project.version}` and does not push. Both projects also have a release script |

Plugins for generated reports, like `maven-site-plugin`, `maven-project-info-reports-plugin` and `socomo-maven`, are optional.

## Adding a plugin
 1. Check that the existing plugins can not already do it.
 2. Add it with a pinned version to `<pluginManagement>` in the parent, and name it in the module that uses it.
 3. Run the whole build with `mvn clean install` and check that the CI workflow still passes.
 4. Be careful with plugins that fetch things at build time. Janus checks the SHA-256 of the Hedgehog download it bundles, and a new
    download in the build should be checked in the same way.
