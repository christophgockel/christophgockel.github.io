---
layout: post
title: Shallow Tests with Spying
---

This iteration was about improving the tests of my [Java Tic Tac Toe](https://github.com/christophgockel/tictactoe-java).

After adding more substance to the tests I can say that looking back at the previous state of the tests, it feels like I tested _nothing_.

Sure, I verified that every part of the game interacts with one another as expected. But verifying _real_ behaviour between the collaborators now definitely gives me more confidence that everything works.



Instead of just testing that a method has been called with a spy, verify the actual behaviour of the method you want to test.
Take a look at the [old revision of the tests for `Game`](https://github.com/christophgockel/tictactoe-java/blob/e49f75eea34091ca7d7453b24a8a220ba183e4af/src/test/java/de/christophgockel/tictactoe/GameTest.java#L91-L96) and you see that the highlighted test verifies that the method `showNextPlayer()` has been called. So far so good, but I found a slightly better approach to that.

After a refactoring, this test still uses a test double, but [it verifies that `showNextPlayer()` is called correctly](https://github.com/christophgockel/tictactoe-java/blob/364b1e4bb9d2e3f8d54e0f32de5aadbb76bc5e1d/src/test/java/de/christophgockel/tictactoe/GameTest.java#L85-L92) from within `Game`.

Another thing I noticed quite often was that I could replace two tests that verified spies, with one test that verifies behaviour.  

This is visible in the tests for `HumanPlayer`. There were [two tests that made sure it called the correct methods on its `Input` object as well as the `Board` that is passed](https://github.com/christophgockel/tictactoe-java/blob/e49f75eea34091ca7d7453b24a8a220ba183e4af/src/test/java/de/christophgockel/tictactoe/HumanPlayerTest.java#L28-L43). These two tests were really just verifying that the player made a move on the board. This has been refactored so that the outcome of the interaction between the `HumanPlayer` and its collaborators [is verified](https://github.com/christophgockel/tictactoe-java/blob/364b1e4bb9d2e3f8d54e0f32de5aadbb76bc5e1d/src/test/java/de/christophgockel/tictactoe/HumanPlayerTest.java#L28-L35).

The next time I need to use spies to verify something, I will definitely think twice.
