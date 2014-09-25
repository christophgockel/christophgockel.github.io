---
layout: post
title: Tic Tac Toe and Inversion of Control
---

One task I had in my last iteration was to decouple my Tic Tac Toe game from any UI specific behaviour.
The general game rules should apply for whatever interface you're playing the game through.


### Rules of the Game

Thinking about it, it's not that hard:

    1. As long as the game is not over
    2. ask the next player for its next move
    3. place the move on the board
    4. switch players
    5. goto 1.

I call these the [Four Rules of Simple Gameplay](https://leanpub.com/4rulesofsimpledesign). Just kidding.

But they really are relatively simple rules. Not a single one of them mentions any specific behaviour depending on whether the game is played via a terminal, a GUI, a web interface or any other &ldquo;delivery mechanism&rdquo;.

That's the way it should be. Unfortunately, my last version of the game wasn't designed like that at all.


### Terminal specific game behaviour

I thought, I had abstracted away from the actual display of the game, but in reality I hadn't (at least I [got rid of the blocking loop in the game itself a long time ago](https://github.com/christophgockel/tictactoe-ruby/commit/9fb40842b5f610c544393ee146565ea825231c5f#diff-bb71a740e71dd9742edf8b2ce2ec7948)).

Due to the nature of a terminal UI, there is blocking IO going on when playing the game. So it pauses naturally when asking for the next move by waiting for user input. That eases the initial development of the game loop itself, yes. But it also hides an important fact about the game logic itself.

It basically relies on actively blocking the whole game until a user has made a move. That's not _that_ bad in general. But with an UI agnostic design in mind it just doesn't cut it. Event driven UIs (as is Qt for example) don't work like that. You can't just block the whole UI process until the user may click somewhere in the UI. Well, technically you could, but that's not the point. Nobody like non-responsive UIs.


### Getting out of the loop (no pun intended)

One way to get around that is actively polling the user for its next move. So there's still some looping going on, but the UI could still be responsive.
The whole concept of polling in an UI environment doesn't really appeal to me (who likes burning CPU cycles?), so I won't dive further into that.


### Just give up! (pun intended)

By &ldquo;give up&rdquo; I mean giving up the control. (In the game. Don't get me wrong.)

So when the game comes to the point where it needs to request something from a user, it should give up the control of &ldquo;being the active part of the game and requesting things&rdquo;.
By just accepting the fact that the game can't do anything until the user is ready for it, we invert the control of the gameplay. The game is done at that point, and when a user has some input for the game, it's the user who will trigger the next round with its move.

I call this [Hakuna matata](https://www.youtube.com/watch?v=abjAqvdGZgM) driven development. Again, just kidding.

But there is some truth in it. It is not the more widely known idea of [Inversion of Control](http://c2.com/cgi/wiki?InversionOfControl) as in the sense of the [Hollywood Principle](http://c2.com/cgi/wiki?HollywoodPrinciple), but it is still an inversion of program flow control.

It's the user who triggers the next round, not the game asking for the next round.

### Food for thought
I wonder, though, if all of this would have been easier if you would just assume an event driven UI from the beginning of a project. I could imagine &ndash; though, haven't tested it &ndash; that it's easier to adapt a terminal UI to an application that has isolated itself from any UI specific eventing, than it is to adapt an application that started with an abstraction of a terminal like UI.
