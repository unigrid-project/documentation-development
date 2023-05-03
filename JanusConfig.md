# Janus Config
Some notes for the update config when using [update4j](https://github.com/update4j/update4j/wiki/Documentation)

## Config add-opens and add-exports
"add-opens" and "add-exports" for module fx is done by adding it to fx/pom.xml

```xml
<config.opens>
    target=maven.path/artifact.Id@package.name
</config.opens>
<config.exports>
    target=maven.path/artifact.Id@package.name
</config.exports>
```

This is used when module-info is not enough to open or export a module.

Example

```xml
<config.opens>
    fx=jakarta.xml.bind/jakarta.xml.bind-api@jakarta.xml.bind
</config.opens>
<config.exports>
    jakarta.json=org.glassfish/jakarta.json@org.glassfish.json
</config.exports>
```

in this example we open fx to jakarta.xml.bind and exports org.glassfish.json to jakarta.json.

When opening or exporting between dependenciesyou have to find witch dependency the package is part of.