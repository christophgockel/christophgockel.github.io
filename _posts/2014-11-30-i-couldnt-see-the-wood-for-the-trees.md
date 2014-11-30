---
layout: post
title: I couldn't see the Wood for the Trees
---

Apart from getting into [Clojure this week](/first-week-with-clojure), I had a code review with my mentor for my [HTTP server](/http-server-challenge).

I struggled with testing the part where the blocking server socket accepts new connections and spawns a new thread.

A solution to that however was as simple as brilliant.

The code I had looked similar to the following:

```java
final Socket clientSocket = socket.accept();
WorkerThread thread = new WorkerThread(clientSocket);
threadPool.execute(thread);
```

Line one is a blocking call, as it waits until the server sockets gets a new connection. When a new connection has been accepted, this connection is passed to a worker thread do to further work with it.

So, how to test that?

One approach could be to use [Mockito](https://code.google.com/p/mockito) &ndash; but I will not go into that here. Mostly because [you shouldn't mock what you don't own](http://blog.8thlight.com/eric-smith/2011/10/27/thats-not-yours.html).

Another approach is to invert the dependencies to the sockets (and maybe on the thread pool).
By using custom `ServerSocket` and `ClientSocket` classes, calls to methods like `accept()` can easily be stubbed out for testing. The implementation of `ServerSocket` for the production environment would include simple delegating methods to the real [`java.net.ServerSocket`](https://docs.oracle.com/javase/7/docs/api/java/net/ServerSocket.html) object.

After my mentor proposed this approach to me, it felt like I knew nothing or had forgotten everything I knew so far. _Of course_ you should invert the dependency to &ldquo;hard&rdquo; dependencies like sockets!

As I said in the beginning, this is brilliant yet simple.

I do know why it is important to invert dependencies, but not thinking of that myself before, makes this another eye opening experience for me.

Which is what I really enjoy about the apprenticeship here at 8th Light. Even though I have a few years of experience in software development, I keep getting [_Aha! moments_](http://en.wikipedia.org/wiki/Eureka_effect) regularly.
