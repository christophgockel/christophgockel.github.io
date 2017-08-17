---
layout: default
title: "Go Build Your Own Tools"
external-url: https://8thlight.com/blog/christoph-gockel/2016/04/21/go-build-your-own-tools.html
---

# Go Build Your Own Tools

In building professions as well as software development there is the saying &ldquo;_don't blame your tools_.&rdquo;
During a recent project my pair and I were reminded of exactly that.
The tools we had at our disposal didn't quite cut it for our task anymore, and what started as a workaround ended up being more effective than initially expected.

We needed to measure the performance of a REST service to find new configuration settings that would help improve its throughput.


## Why JMeter Didn't Work for Us

We started with an access log-file from one of the production servers that contained roughly 450k lines of requests.
Our idea was that we would replay this log-file in order to simulate real-world usage of the service.

At that time it seemed like a reasonable choice to use JMeter for the performance measurements, as it has an extensive list of configuration options.
One of them is an &ldquo;access log sampler&rdquo; that can be configured to use an existing log-file.
Unfortunately there is no setting to tell JMeter something like &ldquo;_replay all of this log-file and use n threads until you're done_.&rdquo;
JMeter couldn't run through the whole log-file—only a limited amount of threads and lines were possible.
Because we didn't want that to block us for too long, we went on and just replayed 200 requests from the original log-file.
This allowed us to get fast feedback and continue testing the actual service.

After a while we had a good idea about how the service reacts and which settings we can adjust to gain better performance.
We started deploying our updates into a testing environment that mirrored the production hardware and configuration.
That environment, though, was hosted in a separate datacenter, isolated from the outside world.
Unfortunately, this also meant that we were not able to access the service from our office.
The only way of accessing the server was via SSH.
Now while opening a tunnel through SSH is technically possible, it is not particularly well suited to run a performance test through it.
Which meant for us that re-running our JMeter tests was off the table.

One option we discussed with the client was to get a bastion host in the datacenter to be able to run a headless JMeter from there.
It would have taken days, if not weeks to get the new hardware up and running, though.

We could have said we can't help the client.
We could have waited until we eventually got the new hardware.
We could have done a lot of things.

But what did we do?

We set out to create a small tool, written in Go, in order to be able to measure the performance of the service ourselves.
With that we'd have control over the features (e.g. provide support to replay the 450k lines log-file), and we could bypass the SSH limitation by running the tool directly on the application server.

## Building a Customised Tool

In two evenings we had a fully tested program running that was ready to be used for our requirements: providing detailed information about the timing of certain endpoints of the service.
On top of that it also supported replaying _all_ requests from our existing log-file.
To make sure we measured the right things _and_ the things right, we constantly verified during development that we get similar results like we got with JMeter.

Our language of choice was Go.
Being able to compile a binary on our machine, move it to the application server, and run it from there without the need of a JVM or any other runtime was very beneficial.

We knew that running the tool to measure the performance from the application server itself might skew the results a bit.
But since we were looking for a baseline measurement and then percentages of performance gains or losses for certain configuration changes we wanted to do, it was good enough to work for us in that scenario.


## Conclusion

Instead of giving up or waiting until we got another server, we tackled the problem from a different direction.
We didn't blame our tools, e.g. JMeter or the lack of HTTP access.
We created a customised tool in order to help us and our client in a pragmatic way.

When we say things like &ldquo;_we pick the right tools for the job_,&rdquo; we also acknowledge the fact that the tools that might usually be the right ones do not always fit unconditionally.
In our case JMeter was just not the right tool anymore because of environmental changes.

In the meantime we brushed up the code during [8th Light's Waza]({{site.baseurl}}/kevin-liddle/2012/05/22/be-a-relentless-programmer.html) and added some more features.
In case you're interested, it is also available on [GitHub](https://github.com/christophgockel/goony).

