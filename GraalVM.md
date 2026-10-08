# GraalVM
Some general notes and tips and tricks for [GraalVM](https://www.graalvm.org/).

For official documentation, visit https://www.graalvm.org/latest/docs/.

## How Hedgehog uses GraalVM
Hedgehog requires Java 25 and GraalVM 25 (`GRAALVM_HOME`) for the native build. The whole program is not compiled to native code,
as the CDI container (Weld) builds objects at run time, which ahead-of-time compilation does not support without extensive
configuration. Instead, GraalVM compiles a small launcher in the `native-image` module. The launcher embeds a jlink trimmed Java
runtime together with the Hedgehog jar (`BundleFeature`), unpacks it on the first start (`Unzipper`) and starts Hedgehog on it.

```
cd native-image
mvn package
```

The result is `native-image/target/hedgehog.bin`, or `hedgehog.exe` on Windows. The native bindings described below are only used
by the launcher, e.g to find the Windows known folders in `windows/Shell32Wrapper.java` and `windows/Ole32Wrapper.java`. The wiki
page *Build, testing and native image* of Hedgehog describes the full build.

## Some interesting resources on the Internet
[OpenGL demo using GraalVM](https://www.praj.in/posts/2020/opengl-demo-using-graalvm/)

[Interfacing with native methods on Graal VM](https://cornerwings.github.io/2018/07/graal-native-methods/)

[The many ways of polyglot programming with GraalVM](https://medium.com/graalvm/3-ways-to-polyglot-with-graalvm-fb28c1542b45)

## Creating native bindings to a system library

### When defining bindings from Java to a native library you need:
 * A `@CContext` that defines the header files that are used and associated with anything referenced in the class.
 * A `@Library` definition defining the library to open and/or link with.
 * A `CFunction` with a function signature to a working entry point in the given library. The signature can only use
      primitive Java types and types defined under the `org.graalvm.nativeimage.c.type` package - pure Java classes
      can't be used. However Java interfaces annotated with `@CStruct` or `@RawStructure` can be used.

## A word about native Windows bindings:
 * Native windows bindings to DLL libraries work the same as under Linux. On the Graal side the definition of the bindings is basically the same.
 * A big **WARNING** has to be given when working with the Windows API; internally Windows still uses Unicode (UTF-16).
   That means that most C functions in the API that either take or return C strings require them to be in UTF-16 format.
   Not only that, but they also have to be in UTF-16 **little endian** format.
   * To get this formating in Java, you have to use `StandardCharsets.UTF_16LE` and always convert any Java or raw strings to that format.
   * In GraalVM you can use `CTypeConversion.toString(...)`. Here is an example:
     ```java
     final CCharPointer guidString = UnmanagedMemory.calloc(TEMPORARY_ALLOCATION_SIZE);

     CTypeConversion.toCString(holder.getIdentifier(), StandardCharsets.UTF_16LE,
             guidString, WordFactory.unsigned(TEMPORARY_ALLOCATION_SIZE)
     );
     ```
   * Windows is weird. Both `long` and `int` are actually 32 bit wide and longs are not twice as wide like they usually are on,
     for example, Linux. Be aware of this when you define bindings under Windows - a long on the Java side is not the same as
     a long on the native Windows side.

## What about JNA?
If you plan on using it; **DON'T**. It is extremely problematic and error prone under GraalVM. Instead, create real bindings
using the method discussed above.
