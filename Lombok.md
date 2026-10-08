# Lombok

Lombok is to be used in all Java projects. Do not write getters, setters, builders or loggers by hand, let Lombok generate them.

## Setup

Add Lombok to the pom.xml as a provided dependency

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

Add a `lombok.config` in the project root

```
config.stopBubbling = true
lombok.addLombokGeneratedAnnotation = true
```

The second line makes JaCoCo skip the generated code, so do not write tests just to cover generated getters and setters.

## Getters and setters

Put `@Getter` and `@Setter` on the class instead of writing accessors.

```
@Getter
@Setter
public class Account {
    private String name;
    private BigDecimal balance;
}
```

Use them on a single field if only that field needs an accessor.

Use `@Getter` and `@Setter` explicitly, do not use `@Data`. It also generates equals, hashCode and toString, and that is wrong for JPA entities.

On JPA entities never use `@Data`, `@EqualsAndHashCode` or `@ToString` and add `@NoArgsConstructor(access = AccessLevel.PROTECTED)`.

Do not use `@Accessors(fluent = true)` or `chain = true`. JSF/EL and JPA need the normal `getX` / `setX` names.

## Builders

Use `@Builder` to create request and parameter objects. Do not create an empty object and fill it with setters.

```
@Value
@Builder
public class TransferRequest {
    String from;
    String to;
    BigDecimal amount;
}
```

and use it like this

```
final TransferRequest request = TransferRequest.builder()
    .from(from)
    .to(to)
    .amount(amount)
    .build();
```

`@Value` makes the object immutable. Do not use `@Builder` on JPA entities.

## Logging

Use `@Slf4j` on the class and log with the generated `log` field.

```
@Slf4j
public class TransferService {

    public void transfer(final TransferRequest request) {
        log.info("Transferring {} from {} to {}", request.getAmount(), request.getFrom(), request.getTo());
    }
}
```

Do not create the logger by hand with `LoggerFactory.getLogger`, and do not use `@Log`.

Use `{}` for parameters, not string concatenation.
