# Lab 10: Pattern Matching for instanceof

## Overview

In this 10-minutes lab, you will get some hands-on experience with the **pattern Matching for instanceof** feature previewed in Java 14 and Java 15, and slated to be made standard and permanent in Java 16. 

This feature enhances the Java programming language with pattern matching for the instanceof operator. Pattern matching allows common logic in a program, namely the conditional extraction of components from objects, to be expressed more concisely and safely. For more details, please check [Pattern Matching for instanceof](https://openjdk.java.net/jeps/394).

## Using 'pattern matching for instanceof'

💡 Make sure to checkout the lab10 branch as it introduces 2 new classes to the project: `AgendaRepository.java` and `AgendaService.java`

```
<copy>
git checkout -f lab10
</copy>
```

Check those 2 new classes. `AgendaService.java` introduces a new "/sessions" endpoint that returns the details of the sessions. The sessions are stored in `AgendaRepository.java` using a simple `List<Session>`. The `Session` type has been introduced in Lab 8, it is a sealed abstract class that can only be extended by a given set of classes (check Lab 8 for details).

Build and test the application, `curl {public_ip}:8080/sessions`

![](./images/lab10-1.png " ")



Let's pretend that the displayed details should vary based on the session type.

Add the following `getSessionDetails` method to the "AgendaService".

```
<copy>
nano src/main/java/conference/AgendaService.java
</copy>
```

```
<copy>
private void getSessionDetails(final ServerRequest request, final ServerResponse response) {
   LOGGER.fine("getSessionDetails");

   var sessionId = request.path().param("sessionId").trim();

   Optional<Session> session = sessions.getBySessionId(sessionId);

   if (session.isPresent()) {

      record SessionDetail(String title, String speaker, String location, String type) {}

      var detail = "speaker TBC!";
      var s = session.get();

      if (s instanceof Keynote) {

         Keynote k = (Keynote) s;
         var ks = speakers.getById(k.getKeynoteSpeaker());

            if (ks.isPresent()) {
               var spk = ks.get();
               detail = spk.firstName() + " " + spk.lastName() + " (" + spk.company() + ")";
            } else detail = "Keynote speaker to be announced!";

            var keynote = new SessionDetail("Keynote: " + k.getTitle(), detail, "Virtual hall", "General session");
            response.send(keynote);
        }
		
		else if (s instanceof Lecture) {

           Lecture l = (Lecture) s;
           var speaker = speakers.getById(l.getSpeaker());

           if (speaker.isPresent()) {
              var spk = speaker.get();
              detail = spk.firstName() + " " + spk.lastName() + " (" + spk.company() + ")";
           }

           var lecture = new SessionDetail(l.getTitle(), detail, String.valueOf(l.getVirtualRoom()), "Conference session");
           response.send(lecture);
        }
		
		else if (s instanceof Lab) {

           Lab l = (Lab) s;
           var speaker = speakers.getById(l.getSpeaker());

           if (speaker.isPresent()) {
              var spk = speaker.get();
              detail = spk.firstName() + " " + spk.lastName() + " (" + spk.company() + ")";
           }

           var lab = new SessionDetail(l.getTitle(), detail, String.valueOf(l.getVirtualRoom()), "Hands on Lab");
           response.send(lab);
        }

    } else {
        Util.sendError(response, 400, "SessionId not found : " + sessionId);
    }
}
</copy>
```



Update Helidon's routing to add `getSessionDetails` as a handler for the "/detail" path.

```
@Override
public void update(Routing.Rules rules) {
   rules.get("/", this::getAll);
   <copy>rules.get("/detail/{sessionId}", this::getSessionDetails);</copy>
}
```

Although a bit long, the `getSessionDetails` method is easy to grasp. 


```
Optional<Session> session = sessions.getBySessionId(sessionId);
if (session.isPresent()) {
   record SessionDetail(String title, String speaker, String location, String type) {}
   …
```

It first gets, from the Session list, a given session based on an Id. And if that session is found, a local record is defined…

```
if (s instanceof Keynote) {

   Keynote k = (Keynote) s;

   var speaker = speakers.getById(k.getKeynoteSpeaker());
   if (speaker.isPresent()) {
      var spk = speaker.get();
      detail = spk.firstName() + " " + spk.lastName() + " (" + spk.company() + ")";
   } else speakerDetail = "Keynote speaker to be announced!";

   var keynote = new SessionDetail("Keynote: " + k.getTitle(), detail, "Virtual hall", "General session");
   response.send(keynote);
```

Then, `instanceof` is used to test the actual type of the session, and based on this type, the logic to create the details will be slightly different. So this logic is repeated for the 3 session types, Keynote, Lecture, and Lab using a "`if … else if …`" chain.

If you zoom on the `instanceof` pattern, you will notice some verbosity as the type, Keynote in this example, is repeated 3 times. First for the actual `instanceof` test and then for the casting when a new intermediate locale variable is created.

```
if (s instanceof Keynote) {
   Keynote k = (Keynote) s;
   // do something with k
   …
}
```

The 'pattern matching for instanceof' feature reduces that verbosity by defining a binding variable, k in this example, that will be created should the type test be true.

```
if (s instanceof Keynote k) {
   // do something with k
   …
}
// can't use k here!
```

You can now update the code to leverage the `pattern matching for instanceof` features for the 3 Session types.

```
if (s instanceof Keynote k) {
   var speaker = speakers.getById(k.getKeynoteSpeaker());
   …
} else if (s instanceof Lecture l) {
   var speaker = speakers.getById(l.getSpeaker());
   …
} else if (s instanceof Lab l) {
   var speaker = speakers.getById(l.getSpeaker());
   …
}
```

If you test the application, you should session details varying depending on the session type.

```
curl {public_ip}:8080/sessions/detail/001
curl {public_ip}:8080/sessions/detail/010
```


## Wrap-up

In this exercise, you have used the **pattern matching for instanceof** feature, previewed in Java 14 and Java 15, and slated to be made standard and permanent in Java 16. For more details, please check [Pattern Matching for instanceof](https://openjdk.java.net/jeps/394).

The **pattern matching for instanceof** feature unarguably simplifies the code but in this particular scenario, the '`if … else if …`' chain makes this code repetitive and potentially brittle! Wouldn't it be nice to use a `switch` instead of this '`if … else if …`' chain?  In fact, the **pattern matching for instanceof** feature along with the **Switch Expression** feature (see Lab 9) and the **Sealed Class** feature (see Lab 8) will enable, in the near future, powerful pattern matching support in the Java platform, including the ability to do pattern matching with Switch.


```
// Comming soon, a switch on an Object!
// Exact syntax TBC
…
switch(s) {  
   case Keynote kn -> …
   case Lecture lc -> …
   case Lab lb -> …
}
…
```

