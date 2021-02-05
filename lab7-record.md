# Lab 7: Records

## Overview

In this 10-minutes lab, you will use *Records*, a new Java language feature that went through 2 preview rounds (JDK 14, JDK 15), and is, starting Java 16, a final and permanent feature.

Records provide a compact syntax for declaring classes which are transparent holders for shallowly immutable data. A record can be best thought of as a nominal tuple that enables us to easily and quickly model immutable "plain data" aggregates.

## A simple Record


Similar to enums, Records are technically a special form of classes optimized for a specific situation, i.e. modeling data aggregates.

1. Create a Person record

Create a `Person.java` class with the following content.

➥ `cd ~ && nano Person.java`

```
public record Person(String lastname, String firstname) {}
```

2. Compile the Person record


➥ `javac Person.java`

3. Decompile the Person record

Using `javap`, you can see what methods are available.

💡 the `-p` parameter instructs `javap` to display the private members of the decompiled class.

```
> javap -p Person
Compiled from "Person.java"
public final class Person extends java.lang.Record {
  private final java.lang.String lastname;
  private final java.lang.String firstname;
  public Person(java.lang.String, java.lang.String);
  public final java.lang.String toString();
  public final int hashCode();
  public final boolean equals(java.lang.Object);
  public java.lang.String lastname();
  public java.lang.String firstname();
}
```
We can observe that this Record has 

* 2 private final fields (`lastname` and `firstname`), i.e. the Record's components, they are immutable
* a public constructor taking 2 parameters that match the Record's components: `lastname` and `firstname`.
* 3 'common' methods implementations: `toString()`, `hashCode()` and `equals()`
* 2 public accessor methods aptly named `lastname()` and `firstname()` to access the Record's components

In addtion, we can see that the Person record extends the [`java.lang.Record`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/Record.html) abstract class.

To use the Record simply create a simple `TestRecord.java` class.

➥ `nano TestRecord.java`

```
class TestRecord {

   public static void main(String ...args) {
      var attendee = new Person("Doe", "Jane");
      System.out.println(attendee);
   }
}
```

Compile and run the test class.

```
> javac TestRecord.java

> java TestRecord
Person[lastname=Doe, firstname=Jane]
```

Do note that the generated implementations can be overridden. For example, change the record as follow...

```
public record Person(String lastname, String firstname) {

   @Override
   public String toString() {
      return ("Person{firstname="+firstname+", lastname="+lastname+"}");
   }
}
```

Compile and run the test class.

```
> javac --enable-preview --source 15 TestRecord.java
...
> java --enable-preview TestRecord
Person{firstname=Jane, lastname=Doe}
```

## A Speaker Record

Back to the Conference application (`cd ~/odl-java-hol`). It exposes simple REST endpoints to get speaker-related information.

* http://{public-ip-address}:8080/speakers ➞ Get all speakers 

* http://{public-ip-address}:8080/speakers/company/oracle ➞ Get speakers for a given company

* http://{public-ip-address}:8080/speakers/lastname/delabassee ➞ Get speaker by its lastname

* http://{public-ip-address}:8080/speakers/track/java ➞ Get speakers for a given track

* http://{public-ip-address}:8080/speakers/029 ➞ Get speaker details for a given id


Browse the source code to understand how things work.


* `Main.java` defines the routings, including the `"/speakers"` path and its related `speakerService` handler.

* `SpeakerService.java` defines the various handlers under the `"/speakers"` path, ex. `"/speakers/lastname"`, `"/speakers/company"`, etc.

The `Speaker.java` class is interesting as it models the Speaker type with all its details (last name, first name, etc.), i.e. it is a data aggregate that represents a speaker. Once created a speaker is effectively immutable as the class is `final`. Moreover there is no way to change the fields (ex. all fields are `private` and there are no setters).

Migrating this regular class into a Record is straightforward. Just replace the `Speaker.java` class content with the definition of the Speaker record. That definition should include the different components related to a speaker. 

➥ `nano src/main/java/conference/Speaker.java`

```
package conference;
public record Speaker (String id,
                       String firstName,
                       String lastName,
                       String title,
                       String company,
                       Track track) {}
```

💡 The Java compiler will automatically generate a default constructor. If needed, this constructor can be customized. Moreover, the compiler will also generate default implementations of the `toString`, the `equals`, and the `hashCode` methods. If required, those default implementations can be overridden. Finally, the Java compiler will also generate accessor methods for each component of the Record.


If you now compile the application (`mvn package`), you will get multiple errors. Can you guess why?

```
…src/main/java/conference/SpeakerRepository.java:[52,69] cannot find symbol
   symbol:   method getLastName()
   location: variable e of type conference.Speaker
…src/main/java/conference/SpeakerRepository.java:[62,77] cannot find symbol
   symbol:   method getTrack()
   location: variable e of type conference.Speaker
…
```

Those errors make sense as the `Speaker.java` class is using the old [JavaBeans](https://docs.oracle.com/javase/tutorial/javabeans/writing/properties.html) getter convention to provide access to its private fields. Records on the other hand rely on (automatically generated) accessor methods to enable access to its various (immutable) components. So that needs to be fixed in the conference application code! Go through the `SpeakerRepository.java` class and make sure to use accessor methods for accessing components instead of getters. This needs to be fixed for all components of the Speaker record (lastName, company, etc.).


➥ `nano src/main/java/conference/SpeakerRepository.java`
For example, change 
```
public List<Speaker> getAll() {
   List<Speaker> allSpeakers = speakers.stream()
       .sorted(Comparator.comparing(Speaker::getLastName)) // getter
       .collect(Collectors.toList());
   return allSpeakers;
}
```
to
```
public List<Speaker> getAll() {
   List<Speaker> allSpeakers = speakers.stream()
       .sorted(Comparator.comparing(Speaker::lastName)) // accessor method
       .collect(Collectors.toList());
   return allSpeakers;
}
```


Compile the application and test it (ex. http://{public-ip-address}:8080/speakers). You will notice that the application now works but it doesn't return any data!?

The `Speaker.java` class is relying on the JSONB (JSON Binding) API support to automatically marshall Java objects to their equivalent JSON representations. The issue you are facing is that Record is a fairly recent feature. Not all JSONB providers support them yet. For example Eclipse Yasson, Helidon default JSONB provider, doesn't support them yet. That explains why the returned results are empty. The good news is that things are evolving rapidly, ex. Apache Johnson, and Jackson are already supporting Records, and more will follow.


To fix this problem, we simply need to use a JSONB provider that supports Records, ex. Jackson v2.12 or greater. 

Add the following Jackson dependency in the `pom.xml`

```
<dependency>
    <groupId>io.helidon.media</groupId>
    <artifactId>helidon-media-jackson</artifactId>
</dependency>
```

💡 This lab uses the latest version of Helidon (2.2.1) which in turn uses Jackson v2.12, so you're good to go! If you happen to use an older Helidon version, you can manually bump the Jackson version by updating the application `pom.xml` as follow.


```
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.fasterxml.jackson</groupId>
                <artifactId>jackson-bom</artifactId>
                <version>2.12.1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-annotations</artifactId>
                <version>2.12.1</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

The last thing you need to do is instructing the application to use Jackson as its JSONB provider. 

➥ `nano src/main/java/conference/Main.java`

Simply replace
``` 
   .addMediaSupport(JsonbSupport.create())
```
with
```
   .addMediaSupport(JacksonSupport.create())
```
and update the import's

```
import io.helidon.media.jackson.JacksonSupport;
```

Compile the application and test it again (ex. http://{public-ip-address}:8080/speakers), it now works!

You can notice that using Records leads to a more concise, more readable code when it comes to model data aggregates! Moreover, more and more frameowrks are adding support (ex. Jackson). Overal, this leads to simple and clean code. 

## A Speaker Record with JSONP (optional)


In the previous exercise, we have used Jackson for its ability to automatically marshall Java objects, including records instances, to equivalent JSON representations. Another approach would be to use the JSONP API to manually construct a JSON representation from a record. 

This can be done by adding the following `toJson` method to the `Speaker.java` record.

➥ `nano src/main/java/conference/Speaker.java`

```
JsonObject toJson() {
   JsonObject payload = Json.createObjectBuilder()
      .add("id", id)
      .add("firstName", firstName)
      .add("lastName", lastName)
      .add("title", title)
      .add("company", company)
      .add("track", track.toString())
      .build();
   return payload;
}
```

💡 Make sure to update the Record class's `import`'s accordingly.

```
import javax.json.Json;
import javax.json.JsonObject;
```

and update the code to return a List of `JsonObject` instead of List of `Speaker`.	

For example, update update the `getAll` method in the the `SpeakerService.java` class

➥ `nano src/main/java/conference/SpeakerService.java` 

```
List<Speaker> allSpeakers = this.speakers.getAll();
if (allSpeakers.size() > 0)
   response.send(allSpeakers);
else Util.sendError(response, 400, "getAll - no speaker found!?");
```
to use the Streams API and the newly added `toJson` method to return a `List` of `JsonObject` as follow. 
```
List<Speaker> allSpeakers = this.speakers.getAll();
if (allSpeakers.size() > 0) {
      response.send(allSpeakers.stream()
                .map(Speaker::toJson)
                .collect(Collectors.toList()));
} else Util.sendError(response, 400, "getAll - no speaker found!?");
```


📝 Make sure to do this for all `SpeakerService.java` methods (`getByCompany`, `getByTrack`, `getSpeakersById`) as you just did for the `getAll` method.


This approach, while more cumbersome, shows that methods can be added to customize records.


## Local Records (optional)


When you are developing applications, think how many times you are creating intermediate values that are a simple group of variables? That should be very frequent! The Record feature is perfect to cope with such a use-case.

Local Record is a feature introduced in the second Record preview in JDK 15. Local Records offer a convenient option to declare a record inside a method, close to where it is used.

For this exercise, let's pretend that you want to return a simpler form of Speaker, ex. just the last name/first name pair and the company.

1. Update the `getAll` method to include a local Record, i.e. within the body of the method!

`nano src/main/java/conference/SpeakerService.java`

```
record SpeakerSummary(String last, String first, String company) {}
```

2. Adapt the the `getAll` method to create, using the Streams API, a list of `SpeakerSummary` instead of a list of `Speaker`. 


```
List<Speaker> allSpeakers = this.speakers.getAll();
if (allSpeakers.size() > 0) {
      response.send(allSpeakers.stream()
         .map(s -> new SpeakerSummary(s.lastName(), s.firstName(), s.company()))
         .collect(Collectors.toList()));
} else Util.sendError(response, 400, "getAll - no speaker found!?");
```

If you now test the endpoint, you will get the shorter speaker representation (see the right column below).

💡 Use `curl` or FireFox to test this, not the Web UI as it needs to updated to cope with the updated JSON payload.

![](./images/lab7-1.png " ")


📝 Make sure to update all `SpeakerService.java` methods for the new `SpeakerSummary` record. As an additional exercise, try to create different Records.


## Stream.toList()

Each new Java release comes with a set of new Java language features (ex. Records in JDK 16), with JDK enhancements (ex. the jpackage in JDK 16). But this is just the tip of the iceberg! In addition to bug and security fixes, there are many smaller improvements done across the platform, from the HotSpot JVM to the Core libraries. And Java 16 is no exception to this rule!

One example is the new [java.util.stream.toList()](https://download.java.net/java/early_access/jdk16/docs/api/java.base/java/util/stream/Stream.html#toList()) method introduced in Java 16 which returns an immutable List containing the elements of a stream.

`nano src/main/java/conference/SpeakerService.java`

For example, we could replace the terminal operation 

```
    response.send(allSpeakers.stream()
       .map(s -> new SpeakerSummary(s.lastName(), s.firstName(), s.company()))
       .collect(Collectors.toList()));
```

with this new method

```
    response.send(allSpeakers.stream()
       .map(s -> new SpeakerSummary(s.lastName(), s.firstName(), s.company()))
       .toList());
```

💡 Make sure to read the [javadoc](https://download.java.net/java/early_access/jdk16/docs/api/java.base/java/util/stream/Stream.html#toList()) to understand when and where this method could be used instead of a more traditional collector.


## Wrap-up

In this exercise, you have used Records. 

Records allow to easily and quickly create immutable data aggregates. Records went through 2 previes rounds (Java 14 & Java 15) and are slated to be made final and permanent in Java 16.

For more details on Records, please check the following resources.

* [JEP 395: Records](https://openjdk.java.net/jeps/395)
* [Java Feature Spotlight: Records](https://inside.java/2020/02/04/spotlightrecords/)






