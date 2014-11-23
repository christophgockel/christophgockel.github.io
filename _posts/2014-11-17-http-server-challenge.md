---
layout: post
title: HTTP Server Challenge
---

For this iteration I got the HTTP Server challenge. We have to implement an HTTP Server that adheres to a specific set of requirements. These requirements are documented and verified in a [FitNesse](http://fitnesse.org) test suite.

It is a tough timeframe to get all tests pass in one week. But that's exactly the point: Deliver a big task in a tight timeframe.

To succeed with the challenge, I often had to remind myself not to &ldquo;waste&rdquo; too much time with features that aren't necessary needed.

The challenge is not to implement an HTTP server that is fully compliant with the HTTP/1.1 specification. But &ldquo;just&rdquo; enough to make it compliant with the current specifications documented in the acceptance tests.

&ldquo;Just enough&rdquo; means also to take shortcuts. Shortcuts are okay &ndash; as long as you don't leave a complete mess. One example from my implementation is the [`Content-Range` header](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.16). [The specification](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.16) defines all the different format possibilities this header can have. And supporting every possibility may me needed in the future, but the current requirement is only to support the very basic form of `bytes 0-4`. And that's what I focussed on.

I'm not saying that it isn't possible to support every `Content-Range` format in one week (IIRC Daniel, my fellow apprentice, does support all the different possibilities in his implementation &ndash; good work by the way). But is it really needed, or could the time spent on that used for another feature?

That's definitely the most valuable take-away for me from that challenge. I see myself often drift away into details that aren't really needed at the time.

So I'll try to remind myself more often: _&ldquo;What would be the most simplest thing to work?&rdquo;_

&ldquo;Simplicity&rdquo;, as one of [XP's core values](http://www.extremeprogramming.org/values.html) is defined as:

> Simplicity: We will do what is needed and asked for, but no more. This will maximize the value created for the investment made to date. We will take small simple steps to our goal and mitigate failures as they happen. We will create something we are proud of and maintain it long term for reasonable costs.

(From: [http://www.extremeprogramming.org/values.html](http://www.extremeprogramming.org/values.html))
