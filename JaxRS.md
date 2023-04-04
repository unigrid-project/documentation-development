# JAX-RS
Some general notes and tips related to working with [JAX-RS (Jakarta RESTful Web Services)](https://projects.eclipse.org/projects/ee4j.rest).

## Working with Jersey and Java SE
While EE projects can easily scan the classpath for classes, when working with Java SE and the Java Platform Module System (JPMS),
extra care needs to be taken to allow the API's to access model classes. If packages are not opened to the right module, it can lead
to unexpected behavior and bugs.

When working with Jersey, REST endpoint classes (resources) should be registered with Jersey when the container is started and
bootstrapped:

```java
final ResourceConfig config = new ResourceConfig(MyRestResource.class, ...);


@Path("/api")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class MyRestResource extends CDIBridgeResource {
	@CDIBridgeInject
	private SomeModel model;


	@GET
	public Response list() {
		...
	}
```

In this example, `CDIBridgeResource` and `@CDIBridgeInject` have been introduced in order to avoid duplicate Weld containers. This
introduces some custom injection handling in order to avoid ending up with additional containers - something that is a common
problem under Java SE when multiple API's, threads and frameworks all use CDI.

## A word of warning when using Jackson & Jersey
The Java module system can play tricks on you. When using Jackson serialization in a Java SE application together with JAX-RS,
it is important that any package that needs to be accessed by the module `com.fasterxml.jackson.databind` is exported appropriately.
For example, if you have a JSON model in the package `com.company.product.model` in the module `product`, you either need to export
the package with `--add-exports product/com.company.product.model=com.fasterxml.jackson.databind` or add a dependency in the
`module-info.java` of `product`:

```java
module product {
	requires com.fasterxml.jackson.databind;
}
```

Not doing this will often result in JAX-RS and Jersey reporting a HTTP 400 (BAD_REQUEST) error whenever you try to encode an
entity to JSON or vice versa.
