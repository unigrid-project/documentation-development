<p align="center">
  <img src="images/library.png" alt="A library" width="100%">
</p>

# Development Documentation
Notes, rules and tips for everybody working at Unigrid or contributing code to the project. The documents describe how we work
and how the tools we use are set up in [Hedgehog](https://github.com/unigrid-project/hedgehog) and
[Janus](https://github.com/unigrid-project/janus-java). When something here is out of date, fix the document in the same way as
you would fix the code.

## Contents

* [Working together](#working-together)
	* [Committing](Committing.md) - Commit messages, branches and merging
	* [AI Code of Conduct](AICodeOfConduct.md) - How AI may and may not be used
* [Building and code](#building-and-code)
	* [Maven](Maven.md) - The preferred build tool, how it is used and recommended plugins
	* [Lombok](Lombok.md) - Getters, setters, builders and logging without boilerplate
	* [JAX-RS](JaxRS.md) - REST with Jersey, Jackson and the Java module system
	* [GraalVM](GraalVM.md) - Native bindings and the native executable of Hedgehog
* [Projects](#projects)
	* [Hedgehog](Hedgehog.md) - Multi sign of grid sporks
	* [Janus Config](JanusConfig.md) - JVM options and modules of the installed Janus wallet
* [Learning](#learning)
	* [Resources](Resources.md) - Books to read, tools and the technology stack

## Working together
Start with [Committing](Committing.md). It tells you how to write commit messages, how to name branches and when to rebase instead
of merge. Everybody who uses AI tools also has to read the [AI Code of Conduct](AICodeOfConduct.md): AI may be used, but its output
always needs to be thoroughly reviewed, and the repository must be kept free of AI slop.

## Building and code
[Maven](Maven.md) is the preferred build tool, and [Lombok](Lombok.md) is used instead of hand-written getters, setters, builders
and loggers. The [JAX-RS](JaxRS.md) document covers the pitfalls of REST under Java SE and the module system, and
[GraalVM](GraalVM.md) collects what we know about native bindings and how the native executable of Hedgehog is built.

## Projects
Practical how-to notes for the projects. [Hedgehog](Hedgehog.md) explains how a grid spork is proposed and co-signed by two board
keys, and [Janus Config](JanusConfig.md) explains how the options of the installed wallet are set.

## Learning
The [Resources](Resources.md) document lists the books to read, the tools we recommend and the technology stack to get familiar with.

## Contributing
Keep the documents short and practical, with commands and snippets that can be copied. Check statements against the code of the
project before writing them down, remove what is no longer true, and add new documents to the contents above.
