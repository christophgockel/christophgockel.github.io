---
layout: post
title: Don't defer refactorings for too long
---

The title of this blog post is a message I will say to myself for a very long time now.

The reason for this is that I hunted a bug in my Tic Tac Toe implementation for a whole day.

_A. Whole. Day._

First, let me give a short introduction of the general design of my game. There are `Player` objects, a `Board`, a `Game` and a `Rules` object.
While test driving the API of my `Rules` class I was primarily interested in whether a given board has a winner or not (i.e. `rules.has_winner?(board)` returning `true` or `false`).

Later then, a method like `rules.winner(board)` was needed to get the actual winner of the given board. In a game with just two players the method can just return two values, can't it? It's either the one player or the other.

Yes, but: what about a draw? The method returned `nil` for the case that there is no winner at all. (Note to myself: _ugh!_)

I felt that returning `nil` is probably not the best solution I could come up with, but it was &ldquo;okay&rdquo; for the moment.

&ldquo;_I can refactor that at a later point in time easily_&rdquo; I said to myself.

> _Remark_:
I like the way how Kent Beck explained it in his book &ldquo;Test-Driven Development by Example&rdquo;. While you are working on a particular feature, you keep notes of refactoring that you can't make right now, but want to keep in mind.

As things progressed I forgot about that tiny gap I had in my `Rules` public API.

And then it was _this_ very gap that caused me so much trouble.
Because the method didn't differentiate between having a real winner or having a draw, the [Negamax](http://en.wikipedia.org/wiki/Negamax) implementation chose wrong locations. But just under certain constellations of a board, it didn't chose the wrong locations all the time. That's why I was so confused.

After rewriting the Negamax solution several times from scratch to check that I didn't typed or used something wrong, I just couldn't find the problem. I even switched to a Minimax solution in between, just to verify it's not the algorithm itself that is the problem in my code.

But after finally realising what my problem is, there was a great relief! The moment every programmer knows how it feels: You finally slayed the dragon. Made fire with your bare hands. Or, as in my case, finally fixed that not-so-clever mistake you did yourself.

![I HAVE MADE FIRE]({{ site.url }}/assets/ihavemadefire.gif)
(Image source: [mildlyamused.tumblr.com](http://mildlyamused.tumblr.com/post/42460622900/my-gif-spree-continues-unabated))