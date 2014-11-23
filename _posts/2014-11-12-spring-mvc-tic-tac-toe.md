---
layout: post
title: Spring MVC Tic Tac Toe
---

After completing the tic tac toe implementation in Java with a terminal UI, the next task was to create a web interface for it with [Spring MVC](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html).

Spring itself is &hellip; how can I put that? Complex. Yes, I think that's a good way to put it.

There's a lot available in Spring (MVC) and most of it isn't really needed or useful for a simple tic tac toe implementation.

Interestingly enough that there is a &ldquo;Rails-y&rdquo; approach to it available that is called [Spring Boot](http://projects.spring.io/spring-boot/). The [website](http://projects.spring.io/spring-boot/) states: _&ldquo;Takes an opinionated view of building production-ready Spring applications. Spring Boot favors convention over configuration and is designed to get you up and running as quickly as possible.&rdquo;_

And it really does favour convention over configuration.

The first Spring tic tac toe I did was with Spring Boot. And there's a lot of _automagic_ going on. I remember my fellow apprentices discussing their approaches when they did their Spring version, and I also remember hearing about `web.xml` configurations and such.

Surprisingly, with Spring Boot, that kind of configuration is not even needed. All that is needed is to annotate your controllers accordingly, and Spring Boot will do the rest for you.

Unfortunately there's a downside to it. You really can get an application up and running in a very small amount of time. For example The monday my iteration started I read into Spring Boot at around 09:00am. My first fully working spike of Spring tic tac toe was done by 11:40am. I certainly did not expect that. I never worked with Spring before, and had only very little knowledge about web development with Java in general.

After having a working spike I `git reset --hard` and started from scratch.

Until 17:00pm that day I did not manage to get a single controller into a JUnit test harness. Maybe it was because my little understanding of Spring, or Spring Boot is really hard to test in isolation. I still don't know really.

So I tried to keep the web/controller layer as thin as possible but to get everything else tested 100%. Which worked out fine, since it's just POJOs behind the controllers.

In the IPM with my mentor I went with him through the whole process and explained everything. At least everything I knew about Spring. Which was very little, since Spring Boot did all the hard work for me.

So the next tasks involved creating a Spring MVC application in a more classical Spring way.

That was definitely helpful. Using Spring Boot felt a bit like cheating. I mean, you really can get up and running fast. But I think it's not the best way to learn Spring.

So my [final Spring tic tac toe](https://github.com/christophgockel/tictactoe-spring) is using classical Spring MVC with a `web.xml` configuration.

Creating it a second time without Spring Boot was helpful as I got to understand the wiring and interconnections of the different components in Spring much better.
