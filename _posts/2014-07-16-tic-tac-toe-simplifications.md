---
layout: post
title: Tic Tac Toe Simplifications
---

Today was all about simplifications in my Tic Tac Toe implementation.

I [could simplify my previous](https://github.com/christophgockel/tictactoe-ruby/commit/2b0422c868a0f79112ddbcacda00b230996406fd) `Player` and `CommandLineIO` attempts to separate a player in the game from its input mechanism. The old version of `Player` didn't have any real behavior in it. It was basically just delegating the `next_move()` method to its so called &ldquo;input object&rdquo; (e.g. an instance of `CommandlineIO`). As shown in the image below.

![Initial design of Player class]({{ site.url }}/assets/ttt_player_inputstrategy.jpg)


So why not merge these things together?! One thing to note though is, I'm thinking about whether &ldquo;`CommandLinePlayer`&rdquo; woud be a better name for `HumanPlayer`, but I'm still undecided about that.

As I had a rough idea where I wanted to go with my code, I started with [implementing a very basic `HumanPlayer` class](https://github.com/christophgockel/tictactoe-ruby/commit/3028dec93eec47f5fd87db82711ecbf90b9153c2).

![Refactored version of Game with players]({{ site.url }}/assets/ttt_game_players.jpg)
The `ComputerPlayer` could be added later by [renaming the existing Negamax implementation](https://github.com/christophgockel/tictactoe-ruby/blob/2b0422c868a0f79112ddbcacda00b230996406fd/lib/computer_player.rb).

The next [big change contains an overhaul of the core game logic](https://github.com/christophgockel/tictactoe-ruby/commit/06dbded4f0d13d4a1174710e8eb39c91080cf9b8). Or to put it better: how the game logic will be triggered. In the old version I passed a &ldquo;display object&rdquo; to my `Game` instance that was responsible to present any messages or UI to the user. I still think abstracting this part from the game itself was a good idea. But an improvable one as well.

One thing that bothered me since quite some time was that [the tests for `Game` looked messy](https://github.com/christophgockel/tictactoe-ruby/blob/2b0422c868a0f79112ddbcacda00b230996406fd/spec/game_spec.rb). Due to the nature of the game logic inside `Game`, the tests needed to set up several collaborators and set specific fake values on them in order to instrument the game loop accordingly (and with &ldquo;game loop&rdquo; I literally mean a do-while loop). For example in order to verify that both players will be asked for a move, I needed to make sure that the game loop runs at least twice in the test.

So the tests in general verified to correct wiring inside `Game`. But they also verified that the game put out the correct messages to this &ldquo;display object&rdquo;. That sounds like there are at least two things going on there, that may can be separated from each other. And so I did - [or at least I started to](https://github.com/christophgockel/tictactoe-ruby/commit/06dbded4f0d13d4a1174710e8eb39c91080cf9b8).
The main idea now is to have the `Game` just to know about the actual game and its rules. Then have a separate `CommandlineUI` that uses a `Game` and displays its state, prints user informations and so on.

I have a list of things I need to work on tomorrow. One part involves removing the duplications in the tests and production code of `Game`. I pretty much implemented the new &ldquo;game loop&rdquo; side by side to the existing implementation. _Now that I write about it, I realize I could have done this in a separated branch and ignore the fear of breaking too much of the existing code..._

What exactly the new game loop looks like will probably topic of tomorrows blog post. Including the main reason to change it in the first place.
