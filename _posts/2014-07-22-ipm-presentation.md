---
layout: post
title: IPM Presentation
---

After Jim explained to me last week how I should treat the Iteration and Planning Meetings during the apprenticeship, I prepared myself this morning for the meeting. Jim is my &ldquo;customer&rdquo; and I present a demo of the features I completed in the iteration. Then we go through new features, discussing the acceptance criteria and estimates - potentially resulting in the need of splitting features too big for the iteration.

Just as it happened today. For my current iteration I need to create a Qt GUI for my Tic Tac Toe. Having no clue yet about Qt I estimated very pessimistic. Resulting in this story to big for this iteration. So we decided to split it into a spike task getting myself more familiar with Qt by implementing [Coin Change](http://craftsmanship.sv.cmu.edu/exercises/coin-change-kata) in Qt. After that spike is done I should be able to estimate the Qt GUI for Tic Tac Toe more realistically. This also means, I need to get the spike done quickly in order to have enough time left in the iteration for the actual work.

During the IPM I admitted, I didn’t finish two tasks. One being for 100% code coverage of the current Tic Tac Toe implementation and the other being to support 4x4 boards.

The code coverage one still bugs me though. There is one line in my Negamax implementation that seem to be completely useless. But I have not yet understood completely why this is. I could have just removed the line and be done with the story. But as it feels like cheating, I also want to _really_ understand what is going on there. It’s not that the algorithm has a bug (at least it doesn’t seem so). The Negamax solution is [tested thoroughly](https://github.com/christophgockel/tictactoe-ruby/blob/master/spec/computer_player_spec.rb) I’d say. But I will tinker with the coverage of this line in the next evenings, I guess (it isn’t planned officially in this iteration - but I’m curious).

I’m nearly done with supporting a 4x4 board. I have done two optimisations to increase the speed of picking the next best move. Whenever there are less than ~5 moves on the board, it doesn’t really matter which location the computer player will choose. So it isn’t necessary to calculate all 16! possibilities for the computer’s first move. Just pick any location…

The other optimisation is that with boards that have only a few cells marked yet, it may be not necessary to calculate all depths of possible moves. Maybe it’s sufficient to stop after the n-th already?

You see, I can’t tell any facts yet, as I still fiddle around with the numbers. Once I have these, and have it reflected in the tests I can tell more what worked and what not, and what the thresholds are that were sufficient for me.

In the afternoon I paired with Makis again. We added alpha-beta pruning to his Minimax implementation to speed up his computer vs. computer game setup. After some errors and wrong assumptions about what the best move is and how to express it in the code, we even found a subtle bug in the original implementation. It was interesting coming back to a Minimax solution and having to remember exactly how it all worked, trying to explain the idea behind the &ldquo;pruning&rdquo; and why it helps reducing the amount of recursion levels.