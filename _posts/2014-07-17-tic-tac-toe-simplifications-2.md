---
layout: post
title: Tic Tac Toe Simplifications (2)
---

More simplifications today. [More deletions, too](https://github.com/christophgockel/tictactoe-ruby/commit/9fb40842b5f610c544393ee146565ea825231c5f).

As I extracted some responsibilities out of the old `Game` class, the tests for it became easier and simpler. One reason for the complicated tests for it were because of the blocking game loop inside it.

To verify the correct behavior of `Game`, the test doubles needed to be set up in very specific ways. As for example when you wanted to test that every player will be asked for their next moves, the test doubles for the players needed to be set up in a way, that the game loop actually ran at least two times, and at best stop after the second iteration.

After the refactoring, the public API changed though, but overall the `Game` class was simpler as before and easier to test. [There is no blocking game loop anymore inside `Game`](https://github.com/christophgockel/tictactoe-ruby/blob/9fb40842b5f610c544393ee146565ea825231c5f/lib/game.rb).

`Game` has three methods now: `play_next_round`, `is_ongoing?` and `winner`. A client of `Game` needs to instrument these methods properly (likely also in a _&ldquo;loopy&rdquo;_ fashion).

But having the loop extracted out, these methods can be tested isolated of any looping behavior now. The Game gets passed two players and a board. For testing these can be faked, and then `play_next_round`, `is_ongoing?` and `winner` can be verified in a simple query like manner. As for example &ldquo;when I pass these players and this board, I expect that no player will be asked for a move and that `winner` returns the value of player A&rdquo;.

One question remains: where did the actual loop go, though? The loop, or the responsibility of playing the next round until the game is over, is located in the new class `CommandlineUI` now.

The `CommandlineUI` ensures that a `Game` is instrumented properly according to the rules, as well as that graphical informations will be presented to the user. Displaying the board's content, or showing an error message when a user has entered an invalid move.

And again, this can be tested without having the need to set up fake boards, or fake players that drive the game loop in a particular way. This can be tested just by using a fake `Game` instance.

Overall I'm happy about the results I currently have. The heavy mocking and spying could be reduced - often by using very simple fake objects (either hand rolled or using RSpec's test doubles). Here and there are still some things I want to improve, but in general I think the game design is getting into a good shape.