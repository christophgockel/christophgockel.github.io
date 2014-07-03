---
layout: post
title: RSpec Test Doubles
---

It seems nearly half of the day was dedicated to RSpec. I read a lot about [`rspec-mocks`](https://github.com/rspec/rspec-mocks) today. At first the syntax for creating test doubles didn’t really make sense to me. Granted, I still struggle with it a bit. Especially when it comes to setting up spies and mocks.

It seems there are several ways to instantiate a test double. There’s `double`, `instance_double`, `spy`, `instance_spy`, `class_spy`, `object_spy` and likely more. Although I only use `double`.

Then there are also a few different ways in validating them. Either as a spy or mock for example.
You then have to `allow` certain doubles to receive specific messages or `expect` them to receive them (depending on the kind of test double). The default behaviour of RSpec is that it creates all test doubles as so called _strict doubles_. Which means you explicitly have to declare upfront of your test which messages you expect and how many times you expect them. There is also a way to create a null object as a test double: just create it with `double.as_null_object`. This way the test double can be called with any message.

The tests for my Game class look a bit messy at the moment. For my taste there's too much of stubbing and mocking going on. I'm not happy with it yet and there's definitely room for improvement. `Game`'s responsibility is to manage the collaboration between two `Player` objects, a `Board` and the `Rules` while one round of Tic Tac Toe is being played. Right now the tests pass and I will first focus on finishing the last bits of the game. It has a basic terminal UI now, allowing two human players to play one round.

I'm currently implementing the computer opponent as Negamax solution.

Over the day we apprentices also discussed some of our general approaches on the Tic Tac Toe game. There were some nice ideas, and it's really interesting to see how everyone approaches a relatively easy game like Tic Tac Toe.

One of the main tasks I have in mind for tomorrow is validating the terminal UI and writing proper specs for it.