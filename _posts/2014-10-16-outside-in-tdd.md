---
layout: post
title: Outside-In TDD
---

For my Java version of [Tic Tac Toe](https://github.com/christophgockel/tictactoe-java) I wanted to test drive it completely outside-in.  
And I did. The end result was a bit surprising, though.

I didn't run into any serious trouble while implementing the game. All pieces worked together just fine when I was done and wired everyting up in a main executable. To be fair though, I think the fact that I did know exactly where I wanted my code and design to go, definitely played into it (all the Ruby Tic Tac Toe from the last weeks surely left its marks&hellip;).

But still, seeing everything working perfectly together after just TDD'ing several small classes [never gets old](http://giphy.com/gifs/MOWPkhRAUbR7i).

So there I was, having a tic tac toe implementation with 100% test coverage. But how valuable were my tests? Not very much it turned out.

Don't get me wrong though, having these kinds of tests is still better than no tests at all. But due to the fact that I verified all behaviour with test doubles only (stubs and spies), lead to tests that were tightly coupled to a concrete implementation.

Take my [implementation of the `Game` class](https://github.com/christophgockel/tictactoe-java/blob/456e612bd0ffeee6e8714027e2328dce56eb7b6b/src/main/java/de/christophgockel/tictactoe/Game.java) for example. It's 60 lines of code with not too much complex logic. But having a look at [the tests for it](https://github.com/christophgockel/tictactoe-java/blob/456e612bd0ffeee6e8714027e2328dce56eb7b6b/src/test/java/de/christophgockel/tictactoe/GameTest.java), reveals that nearly every test-method verifies the correct wiring with a collaborating class. There's no verification of _real_ behaviour.

Instead of checking that [a method on a Player object has been called](https://github.com/christophgockel/tictactoe-java/blob/456e612bd0ffeee6e8714027e2328dce56eb7b6b/src/test/java/de/christophgockel/tictactoe/GameTest.java#L43), I could (and should) have verified, that the player made an actual move on the board &ndash; i.e. the board before the player made a move is a different one compared to the board that exists after the move.

And that is because I didn't had any real `Board` class to use in the tests at that time. Because of the outside-in approach I did. While writing the tests for `Game` I only had the `Board` interface implemented by a test double.

I feels like an outside-in approach like that isn't particular well suited for a focussed problem domain like that. It seems it is more suitable for testing boundaries, like explained thoroughly in &ldquo;[Growing Object-Oriented Software, Guided by Tests](http://www.growing-object-oriented-software.com/)&rdquo; by Steve Freeman and Nat Pryce.
