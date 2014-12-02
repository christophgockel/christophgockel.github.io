---
layout: post
title: IThing and ThingImpl
---

While refactoring my HTTP server last week, I introduced abstractions for the sockets it used, in order to [invert the dependencies to them](/i-couldnt-see-the-wood-for-the-trees).

I created an interface called `ClientSocket` and a class named `DefaultClientSocket` that is used for the production environment. For testing a `StubClientSocket` was used that could be prepared with data for the test.

As the name `DefaultClientSocket` isn't a particular good one, I had a hard time coming up with a better name. In the end it's really just the `ClientSocket`. But that name was already used by the interface I extracted.

I definitely do not want to use `IClientSocket` for the interface or `ClientSocketImpl` for the implementation. I'm not writing software for Windows, so this naming scheme is clearly off the table. Apart from that I do not like having to pre- or postfix class names in general.

So what to do?

One thing my mentor brought up in todays IPM was, that when you cannot find good names for an interface and the corresponding class, the class _is_ the interface.

What does that mean?

There is only a class called `ClientSocket` that has a few public methods. For testing these methods are then redefined. Having to subclass as a tradeoff in order to introduce better naming is worth it.
