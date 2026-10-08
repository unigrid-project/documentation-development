# Janus Config
Some notes for the configuration of the installed [Janus](https://github.com/unigrid-project/janus-java) wallet.

Janus no longer uses JavaFX and update4j. The window is a browser engine (JCEF) hosting pages served by the embedded web server. The
old update4j configuration, with `config.opens` and `config.exports` in `fx/pom.xml`, only exists on the `legacy-javafx` branch.

## JVM options of the installers
The installers are built by the `desktop` module with jpackage, using the `installer` profile. The JVM options an installed Janus
starts with are set as `javaOptions` in `desktop/pom.xml`, once for each package type:

```xml
<javaOptions>
    <option>--add-opens</option>
    <option>java.base/java.lang=ALL-UNNAMED</option>
    <option>--enable-native-access=ALL-UNNAMED</option>
    <option>-Djanus.jcef=$APPDIR/jcef</option>
</javaOptions>
```

* `--add-opens` and `--add-exports` open or export a JDK package to the unnamed module, where Janus and the browser engine run.
  Each option and its value is a separate `<option>`. The macOS package also exports `java.desktop/sun.awt`,
  `sun.lwawt` and `sun.lwawt.macosx`.
* `-Djanus.jcef` points to the browser engine in the installation. `$APPDIR` is replaced by jpackage with the application folder.
* Change the options for all platforms, not only the one you test on.

## Modules in the bundled Java runtime
The installers carry their own Java runtime made with jlink. A JDK module that is not found by jdeps, because it is loaded through
service lookup, has to be added with an `<addModule>` in the same `desktop/pom.xml`, otherwise the installed Janus fails at run time
with a missing class or module.

## Building an installer
```
mvn -Pinstaller verify -DskipTests
desktop/collect-installers.sh <version>
```

The installers end up in `desktop/target/release`. See the README of Janus for the requirements of each platform.
