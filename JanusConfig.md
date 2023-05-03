# Janus Config
Some notes for the update config when using [update4j](https://github.com/update4j/update4j/wiki/Documentation)

## Config add-opens and add-exports
"add-opens" and "add-exports" for module fx is done by adding it to fx/pom.xml

```xml
<config.opens>
    target=maven.path/artifakt.Id@package.name
</config.opens>
<config.exports>
    target=maven.path/artifakt.Id@package.name
</config.exports>
```
This is used when module-info is not enough to open or export a module.