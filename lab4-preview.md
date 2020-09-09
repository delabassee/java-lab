# Lab 4: Java SE Preview Features

## Overview


In this lab, you will get some hands-on experiences with the Java SE *Preview Feature* mechanism.

Preview Feature is one of the 3 mechanisms to add non-final features into the Java platform. The goal is to allow developers to use those non-final features in order to gather feedback and make necessary changes if needed before those features are made final and permanent into the Java platform.

* Preview Features, for new Java platform features fully specified and implemented but yet subject to adjustments

* Experimental Features: mainly for new features in the HotSpot JVM

* Incubator Modules, for potentially new APIs and JDK tools

For more details, please check this [article](https://blogs.oracle.com/javamagazine/the-role-of-previews-in-java-14-java-15-java-16-and-beyond). 


## Hello Preview 


1. Create a basic application

In a directory, create a simple _Hello.java_ application.

```java
class Hello {
	public static void main (String ... args) {
		System.out.println(new Attendee("Jane", "Doe"));
	}
}

record Attendee(String first, String last) {}
```

2. Compile it

`javac Hello.java`

![](./images/lab4-1.png " ")


This simple application uses a Record (see Lab 6) which is a Preview Feature in Java 15. That error simply informs you that preview features are disabled by default. To use those you need to explicitly enable them using the `--enable-preview` java compiler flag. Note that, you also need to confirm to the Java compiler which version of the Preview Feature you are using (ex. using the `--release` flag). 

` javac --enable-preview --release 15 Hello.java`

Those 2 flags are simply a safeguard mechanism to make sure that you are aware that you are using features from the platform that are not final, and hence those might change in a future version.

![](./images/lab4-1.5.png " ")

The compilation now succeeds. Notice that you are still warned that preview features are used in the code.


2. Run the application

`java Hello`

![](./images/lab4-2.png " ")

You are facing the same safeguard as Preview Features are also disabled at runtime! To be used, they should be explicitly enabled using the `--enable-preview`. The difference is that at runtime, you don't need to use a flag to confirm that version that you are using.

`java --enable-preview Hello`

![](./images/lab4-3.png " ")

It now works!

## Preview Features & Helidon

To use Preview Feature in Helidon, those should be enabled at both compile-time and runtime.

#### Compile time configuration

In an Helidon project's `pom.xml`, simply configure the Java compiler plugin to enable Preview Feature.

```xml
…
<plugins>
   …
   <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.0</version>
      <configuration>
        <compilerArgs>--enable-preview</compilerArgs>
        <release>15</release>
      </configuration>
   </plugin>
</plugins>
…
```
#### Runtime configuration

To run the application, use the following command:

```
java --enable-preview -jar target/myapp.jar
```

Similarly, to use Preview Features via the Helidon CLI 'devloop', you need to pass the same `--enable-preview` argument to the JVM running the application:

```
helidon dev --app-jvm-args "--enable-preview"
```

## Wrap-up

In this lab, you have used Record which is a Preview Feature in Java 15. You have also seen how to use Preview Features in Helidon applications.

In summary, the _Preview Feature_ mechanism:

* allows introducing non-final features into the Java platform (ex. Language Feature)

* allow developers to use those, and provide feedback

* enables Oracle to gather that feedback and make changes if needed

* Preview Features are disabled by default, they should explicitly be enabled at both compile time and runtime

* a given Preview Feature is specific to a given Java version


<br>
Java 15 includes multiple *Preview Features* which will be discussed in upcoming labs.

* Sealed Classes (Preview) : JEP 360

* Pattern Matching for instanceof (Second Preview) : JEP 375

* Records (Second Preview) : JEP 384

 

 







