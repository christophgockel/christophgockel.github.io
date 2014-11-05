---
layout: post
title: Rails and Timezones
---

This week I work on an internal Rails project that included some date and time crunching (among other things).

Some specs for the various controllers and classes included stubbed instances of [`Time`](http://api.rubyonrails.org/classes/Time.html) and [`Date`](http://api.rubyonrails.org/classes/Date.html). Surprisingly when I first cloned the project and ran the test suite I got failures.

The test code contained a line like the following:

```ruby
today = Time.new(2014, 10, 1)
```

Not too overly complicated. But the test failed, and the failure message showed that the actual date that has been compared was this: `"2014-09-30 23:00:00"`.

That was not the date that the test set up before. At least that's not the date the developer had in mind when writing the test.

And the problem is with timezones. Or the seemingly infinite possibilities to create `Time` instances &ndash; which can all use different timezone settings. I don't know really why Ruby/Rails behaves that way it does, but take the following commands as an example of what I mean (executed in the Rails console):

```ruby

irb(main):001:0> Time.now.zone
=> "GMT"

irb(main):002:0> Time.new.zone
=> "GMT"

irb(main):003:0> Time.zone.now.zone
=> "UTC"

irb(main):004:0> Time.new(2014, 10, 1).zone
=> "BST"

```

So whenever `Time.now` or `Time.new` is called, it returns a GMT time. This was mostly used in the production code. The test code though, used `Time.new(2014, 10, 1)`. Which is &ldquo;British Summer Time&rdquo; for my local machine. So when Rails (resp. ActiveRecord) stores timestamps, it converts the given object to GMT internally. My guess at the moment is (haven't digged through it completely), that the test setup created the test data in the (in-memory) database, where it was converted to GMT. When the test expectation happened it compared the converted timestamp to the expected one. Which was one hour off by then (`"2014-09-30 23:00:00"`).

There is a pretty good [blog post](http://www.elabs.se/blog/36-working-with-time-zones-in-ruby-on-rails) about what to do in a situation like that. One of the quintessences is that instead of calling `Time.now` one should use `Time.zone.now` which uses the help of some neat ActiveSupport helpers.

I think it's generally a good idea to know what is going on when it comes to timestamps and timezones in your applications. I don't think every advice should be followed blindly. If this application would have been developed in one location only, this behaviour would have never been uncovered. Which is also not necessarily the worst thing, because the application would still be valid and the tests right. It's just that when you have development teams spread across different timezones working on the same codebase, these timezones can turn on you.

