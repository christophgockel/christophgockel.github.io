---
layout: post
title: Tic Tac Toe Refactorings
---

In yesterdays weekly iteration meeting with Jim, I got new ideas and views how my TTT implementation could be improved. After Jim explained issues with the existing code, I catched myself often asking myself &ldquo;_Why haven't I thought of this in the first place? It seems so obvious now it was explained to me._&rdquo;

Some issues were related to my not-yet-very-Ruby-like implementations. I'm starting to consider every method longer than 4 lines not neccesarilly malformed, but at least as a chance to be improved (i.e. shortened).

I remember, in a Peepcode Play by Play episode (now Pluralsight), Ben Orenstein goes as far as considering every Ruby method longer than one line as _smelly_. Of course, he further explains, no one should take this as the absolute truth or mandatory, but at least as food for thought. [He also wrote about it on his blog](http://codeulate.com/2013/08/simple-rules-for-new-ruby-programmers/).

And today I've experienced it myself. What before was totally fine code (granted, not very fine Ruby code, but still...), could be refactored to very simple methods containing mostly just one line.

A good example for that is my `Board` class. The [old version](https://github.com/christophgockel/tictactoe-ruby/blob/15380caa14fd285018217ae6ab136908c6fc06f2/lib/board.rb) of it passed the tests, but wasn't implemented very _ruby-esque_.

The [refactored version of it](https://github.com/christophgockel/tictactoe-ruby/blob/master/lib/board.rb) is far more concise, and it even got more responsibilities, because the now-deleted [`Rules`](https://github.com/christophgockel/tictactoe-ruby/blob/9cca04d8413f9495311512af9caf27cb25767896/lib/rules.rb) class has been merged into `Board`.

Which brings me to the next topic: [feature envy](http://c2.com/cgi/wiki?FeatureEnvySmell).

My `Rules` class envied my `Board`, which was very apparent because nearly every method in `Rules` either got a `board` passed to it, or queried it to get information about its state.

What I thought was would be a good application of [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns), turned out to result in two classes that actually belong together.

My intention was to have a `Board` just as a &ldquo;bucket&rdquo; for player moves and a separate _thing_ that examines such a board (e.g. a `Rules` object). But there's no point in separating these two things in a context like the Tic Tac Toe game. Because the `Board` isn't an ubiquitous usable board for every game one could think of anyway, separating the &ldquo;interpretation&rdquo; of a board doesn't bring any real value to the software. It's even worse, because the complexity of the overall project was bigger and the feature envy smell was present.

Tomorrow I will work further on the findings. Among others, the I/O parts for a game will be extracted into a separate class and be bested - of course.