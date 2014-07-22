---
layout: post
title: Tic Tac Toe Enhancements
---

Today I enhanced my Tic Tac Toe game a bit. When playing the game, there are four game types now to choose of:

 1. Human vs. Computer
 2. Human vs. Human
 3. Computer vs. Human
 4. Computer vs. Computer

Depending on the choice made, the appropriate game setup will be created by a `GameFactory`. With the extraction of `CommandlineUI` out of `Game` I made last week, the changes needed to support different game types went relatively smooth. 

This is also reflected in [the diff of the changes](https://github.com/christophgockel/tictactoe-ruby/commit/9e21a34c702b88ea56876d2204bfb47e82725159), just a few red lines where clients of the changed code needed to be adjusted, but mostly additions to the existing code. Which is not that much of a surprise. There were new features added, and new features should result in new code to be added, and not necessarily in too many modifications of existing code.

After that I tinkered a bit with the support of a 4x4 board. The general support for it is there, even the Negamax implementation. But the calculation of the best move still takes too much time in a real game. I have a very simple optimization in my code that it just picks a random location whenever there have been < 5 moves made. The Negamax solution is then only triggered for boards that have more moves on them. But even with this solution, it took 56 seconds to have the algorithm chose the next best move.

As seen in the following `rspec` example output:

```bash
ComputerPlayer
  4x4 board
    handles real world game setups

Finished in 1 minute 56.32 seconds (files took 0.17151 seconds to load)
1 example, 0 failures
```
Which was a board with this setup:

     o |  o |  3 | 4 
     ----------------
     5 |  6 |  7 | 8 
     ----------------
     9 | 10 | 11 | o
     ----------------
     x | 14 |  x | x

(The expected move was that the Computer would choose location `14`.)
I'll look further into it tomorrow.

Later today I paired with Maki on his Qt GUI. I finally had the chance to help him a bit, after he helped me so many times in the last weeks with Ruby topics. Even though I don't know the Qt framework for Ruby, I could at least explain a bit the general approach of GUIs with all the event handling going on.

_On a side note:_ is the term &ldquo;hollywood principle&rdquo; still being used today? I can't remember the last time I heard it in a discussion. Maybe it's just common knowledge nowadays...?
