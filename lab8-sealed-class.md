# Lab 8: Sealed Classes

## Overview

In this 10-minutes lab, you will get some hands-on experiences with _Sealed Classes_ (JEP 360), a preview feature of JDK 15. Sealed classes and interfaces restrict which other classes or interfaces may extend or implement them.

💡 Despite its name, the *Sealed Classes* feature applies to both **classes** and **interfaces**.

## Restricted  Class Hierarchies

In Java, a class hierarchy enables the reuse of code via inheritance: The methods of a superclass can be inherited (and thus reused) by many subclasses. However, the purpose of a class hierarchy is not always to reuse code. Sometimes, its purpose is to model the various possibilities that exist in a domain, such as the kinds of shapes supported by a graphics library or the kinds of loans supported by a financial application. When the class hierarchy is used in this way, restricting the set of subclasses can streamline the modeling, that is what Sealed Classes enable.

A Sealed Class (or interface) can be extended (or implemented) only by those classes (and interfaces) explicitly permitted to do so.

* A new `sealed` modifier has been introduced to *seal* a class
* A new `permits` clause is then used to explicitly specify the class(es) that is(are) permitted to extend the sealed class  

## Your first Sealed Classes

💡 "Sealed Classes" is a preview feature in JDK 15 so make sure that preview features are enabled both at compile time and run time.

For the sake of this exercise, let us suppose that the conference application needs to deal with sessions. 

Broadly speaking, the conference has the following sessions :

* **Breakout** session, each has a virtual room
	* Lecture: a traditional conference session
	* Lab: a hands-on lab
* **Keynote** session, a traditional general session

All extends the **Session** abstract class

1. Create a `session` directory (`mkdir src/main/java/conference/session/`) and create the abstract sealed `Session.java` superclass.

`nano src/main/java/conference/session/Session.java`

```
package conference.session;

sealed public abstract class Session
permits Keynote, Breakout {

   private String uid;
   private String title;

   public Session(String title) {
      this.title = title;
      this.id= id;
   }

   public String getUid() {
      return uid;
   }

   public String getTitle() {
      return title;
   }
}
```

🔎 `sealed public … class Session` ➞ declares it to be a **sealed** class.

🔎 `permits Keynote, Breakout …` ➞ explicitly declares that only the `Keynote` and the `Breakout` classes can extend it.


2. Now you need to create both `Keynote.java` and `Breakout.java` classes

`nano src/main/java/conference/session/Keynote.java`

```
package conference.session;

final public class Keynote extends Session {

    String keynoteSpeaker;

    public Keynote(String keynoteSpeaker, String title) {
        super(title);
        this.keynoteSpeaker = keynoteSpeaker;
    }
}
```
🔎 `Keynote.java` is **final**, it can't be extended.


`nano src/main/java/conference/session/Breakout.java`

```
package conference.session;

import java.util.Random;

public sealed abstract class Breakout extends Session
permits Lab, Lecture {

    private String speaker;
    private int virtualRoom;

    public Breakout(String title, String speaker) {
        super(title);
        this.speaker = speaker;
        this.virtualRoom = new Random().nextInt(3) + 1; // random room
    }

    public String getSpeaker() {
        return speaker;
    }

    public int getVirtualRoom() {
        return virtualRoom;
    }

}
```
🔎 `Breakout.java` is also **sealed**, it **permits** both the `Lab` and the `Lecture` classes to extend it.

3. Create the `Lecture.java` and `Lab.java` classes

`nano src/main/java/conference/session/Lecture.java`

```
package conference.session;

final public class Lecture extends Breakout {

    String slidesUrl;

    public Lecture(String title, String speaker, String slidesUrl) {
        super(title, speaker);
        this.slidesUrl = slidesUrl;
    }

    public String getslidesUrl() {
        return slidesUrl;
    }

}

```

`nano src/main/java/conference/session/Lab.java`
```
package conference.session;

final public class Lab extends Breakout {

    String labUrl;

    public Lab(String title, String speaker, String labUrl) {
        super(title, speaker);
        this.labUrl = labUrl;
    }

    public String getLabUrl() {
        return labUrl;
    }

}

```

🔎 Both classes are `final`.


4. Create a fictional 'AgendaRepository.java' class

`nano src/main/java/conference/AgendaRepository.java`

```
package conference;

import conference.session.Keynote;
import conference.session.Lab;
import conference.session.Lecture;
import conference.session.Session;

import java.util.List;
import java.util.stream.Collectors;

public final class AgendaRepository {

   private List<Session> sessionList;

   public AgendaRepository() {

      var keynote = new Keynote("Georges", "The Future of Java Is Now");
      var s1 = new Lecture("Java Language Futures - Mid 2020 Edition", "Brian", "http://speakerdeck/s1");
      var s2 = new Lecture("ZGC: The Next Generation Low-Latency Garbage Collector", "Per", "http://slideshare/s2");
      var s3 = new Lecture("Continuous Monitoring with JDK Flight Recorder (JFR)", "Mikael", "http://speakerdeck/");
      var h1 = new Lab("Building Java Cloud Native Applications with Micronaut and OCI", "Graeme", "http://github.com/micronaut");
      var h2 = new Lab("Using OCI to Build a Java Application", "David", "http://github.com/xyz");

      sessionList = List.of(keynote, s1, s2, s3, h1, h2);
   }

  public List<Session> getAll() {

      List<Session> allSessions = sessionList.stream()
              .collect(Collectors.toList());
      return allSessions;
    }
    
}
```

5. Create `AgendaService.java`

`nano src/main/java/conference/AgendaService.java`

```
package conference;

import conference.session.Keynote;
import conference.session.Lab;
import conference.session.Lecture;
import conference.session.Session;
import io.helidon.webserver.Routing;
import io.helidon.webserver.ServerRequest;
import io.helidon.webserver.ServerResponse;
import io.helidon.webserver.Service;

import java.util.List;
import java.util.logging.Logger;


public class AgendaService implements Service {

   private final AgendaRepository sessions;
   private static final Logger LOGGER = Logger.getLogger(AgendaService.class.getName());

   AgendaService() {
      sessions = new AgendaRepository();
   }

   @Override
   public void update(Routing.Rules rules) {
      rules.get("/", this::getAll);
   }

   private void getAll(final ServerRequest request, final ServerResponse response) {
     LOGGER.fine("getSessionsAll");

     List<Session> allSessions = this.sessions.getAll();
     response.send(allSessions);
   }
}
```


6. Update the `createRouting` method in `Main.java` to instantiate the AgendaService and register its handler under the "/sessions" path.


`nano src/main/java/conference/Main.java`

```
…
AgendaService sessionsService = new AgendaService();

…
return Routing.builder()
      …
      .register("/sessions", sessionsService)
      .build();

```

This new endpoint can new be accessed via `{public_ip}:8080/sessions`, it exposes sessions details. 


7. Create a new session type.

You should now try to create an additional session type, ex. a `Quickie` session type that extends `Breakout`. Given that only `Lab` and `Lecture` are permitted to extend `Breakout`, the Java compiler will simply refuse that `Quickie` tries to extends `Breakout` but you should be able to fix this.


## Wrap-up

In this exercise, you have used Sealed Classes. 

Sealed Classes is a new feature that enables a developer to define a restricted class hierarchy, i.e. a developer has now the ability to explicitly states for a given class (or an interface) which classes (or interfaces) may extend (or implement) it. Sealed Classes is a previewed feature in JDK 15.

For more details, please check [JEP 360: Sealed Classes (Preview)](https://openjdk.java.net/jeps/360)
and [Java Feature Spotlight: Sealed Classes](https://www.infoq.com/articles/java-sealed-classes/).




 







