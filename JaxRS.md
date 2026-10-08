# JAX-RS
Some general notes and tips related to working with [JAX-RS (Jakarta RESTful Web Services)](https://projects.eclipse.org/projects/ee4j.rest).

## Working with Jersey and Java SE
While EE projects can easily scan the classpath for classes, when working with Java SE and the Java Platform Module System (JPMS),
extra care needs to be taken to allow the API's to access model classes. If packages are not opened to the right module, it can lead
to unexpected behavior and bugs.

[Hedgehog](https://github.com/unigrid-project/hedgehog) runs Jersey on Netty (`jersey.container.netty.http`) with Weld as CDI
container. The REST endpoint classes (resources) are registered with Jersey in `RestServer` when the server is started, together with
the providers and filters:

```java
final ResourceConfig config = new ResourceConfig(GridSporkResource.class, MintStorageResource.class, ...);

config.register(JacksonJaxbJsonProvider.class);
config.register(JsonExceptionMapper.class);
config.register(new BearerTokenFilter(token));
```

A resource extends `CDIBridgeResource` and gets its collaborators with `@CDIBridgeInject`:

```java
@Slf4j
@Path("/gridspork")
@Produces(MediaType.APPLICATION_JSON)
public class MintStorageResource extends CDIBridgeResource {
	@CDIBridgeInject
	private SporkDatabase sporkDatabase;

	@Path("/mint-storage") @GET
	public Response list() {
		return Response.ok().entity(sporkDatabase.getMintStorage()).build();
	}
}
```

Jersey creates the resource itself, so a plain `@Inject` would not reach the CDI container. `CDIBridgeResource` looks up every field
annotated with `@CDIBridgeInject` in the running container after construction (`CDI.current()`). This avoids ending up with additional
Weld containers - something that is a common problem under Java SE when multiple API's, threads and frameworks all use CDI.

## A word of warning when using Jackson & Jersey
The Java module system can play tricks on you. When using Jackson serialization in a Java SE application together with JAX-RS,
it is important that any package that needs to be accessed by the module `com.fasterxml.jackson.databind` is opened appropriately.
Jackson reads and writes the fields of the model through reflection, so exporting is not always enough. For example, the JSON entities
in `org.unigrid.hedgehog.server.rest.entity` are opened in the `module-info.java` of the module:

```java
module org.unigrid.hedgehog {
	requires com.fasterxml.jackson.databind;

	opens org.unigrid.hedgehog.server.rest.entity to com.fasterxml.jackson.databind;
	exports org.unigrid.hedgehog.model.gridnode to com.fasterxml.jackson.databind;
}
```

Without a `module-info.java` you can instead use `--add-opens product/com.company.product.model=com.fasterxml.jackson.databind`.

Not doing this will often result in JAX-RS and Jersey reporting a HTTP 400 (BAD_REQUEST) error whenever you try to encode an
entity to JSON or vice versa.

## Using the Jersey client
[Janus](https://github.com/unigrid-project/janus-java) talks to Hedgehog as a client only, with `ClientBuilder` from
`jakarta.ws.rs.client` in `HedgehogClient`, and runs on the classpath without `module-info.java`. The REST interface of Hedgehog
requires the header `Authorization: Bearer <token>` on every request, which the client adds with a `ClientRequestFilter`.
