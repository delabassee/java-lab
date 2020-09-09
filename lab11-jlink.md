# Lab 10: jlink ⛔

## Overview

In this lab, you will use *`jlink`*, a tool that can assemble and optimize a set of modules and their dependencies to create a custom, i.e. optimized, run-time Java image. `jlink` has been part of the JDK toolin since JDK 9.

...

## Using `jlink`

One can use jlink to create a custom Java runtime with just the modules required by an application. Reducing the numbers of modules reduces the overall size of the Java runtime, a concern especially important when it runs within a container.

```
class Test {

   public static void main(String args[]) {
     System.out.println("Hello World");
   }

}
```

The following example will creat a custom Java runtime with just one module.

`jlink --output custom-runtime --add-modules java.base`

This Java runtime is limited but it can, nevertheless, runs application that only requires the `java.base` module.



## Using `jlink` with Helidon applications 


The challenge when creating custom runtime mostly comes from the dependencies as you have to figure out which modules are used, and hence required in your custom runtime image. Do note that JDK tools such as `jdpes` can help to identify those modules.


The good news is that more and more Java frameworks support `jlink`out-of-the-box! As a developer, you don't have to worry about making sure that you have identified all the modules required by your application and its dependencies as the framework tooling will take care of that!

To create a jlink based custom Java runtime image, Helidon is using a Maven profile.

To use it, simply issue the following Maven command.


```
mvn package -Pjlink-image -Djlink.image.defaultJvmOptions="--enable-preview"
```

![](./images/lab11-1.png " ")

To run the application with its custom Java runtime image.
`target/conference-app/bin/start`


conference-app/bin/start --help


DEFAULT_APP_JVM     Overrides "--enable-preview"
--jvm





* [JEP 282: `jlink`](https://openjdk.java.net/jeps/282)
* [The jlink Command](https://docs.oracle.com/en/java/javase/14/docs/specs/man/jlink.html)
* [Helidon SE — Custom Runtime Images with `jlink`](https://helidon.io/docs/v2/#/se/guides/37_jlink_image)
* [Helidon Maven Plugin](https://github.com/oracle/helidon-build-tools/tree/master/helidon-maven-plugin#goal-jlink-image)

## Wrap-up


 







