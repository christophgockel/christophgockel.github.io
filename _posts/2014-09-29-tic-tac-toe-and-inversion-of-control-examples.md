---
layout: post
title: Tic Tac Toe and Inversion of Control (Examples)
---

To support the ideas from my [last post about inversion of control](/tic-tac-toe-and-inversion-of-control), I want to demonstrate some of the things mentioned with code examples.

### Hidden Assumptions

One thing that were subtly hidden in my `Game` implementation was, that it had the inherent assumption of blocking players. So the game loop didn't run endlessly but blocked naturally whenever a move from a human player was requested.

Imagine something like the following setup:

```ruby
class Game
  # other parts left out for brevity

  def play_next_round
    place_move_of(next_player)
    switch_players
  end
end
```

And a client of `Game` could implement a game loop like this:

```ruby
while game.is_ongoing?
  game.play_next_round
end
```

At this abstraction level the flaw isn't directly visible. Looking more into `place_move_of()` though, reveals more information:

```ruby
def place_move_of(player)
  move = player.next_move(board)
  board.set_move(move, player.mark)
end
```
The interesting part is the call to `player.next_move(board)` here.
Imagine now `player` being an object of class `HumanPlayer` that is defined as follows:

```ruby
class HumanPlayer
  attr_reader :mark, :input

  def initialize(mark, input = $stdin)
    @mark = mark
    @input = input
  end

  def next_move(board)
    input.gets.to_i
  end
end
```

There is a small abstraction for what can be treated as an `input` for the human player. At the time of writing that, I only had a terminal UI in mind, therefore using `$stdin` as the default input mechanism seemed reasonable. Any `input` object that responds to `gets` could be used. Injecting this dependency made it [easier to test](https://github.com/christophgockel/tictactoe-gem/blob/2b0422c868a0f79112ddbcacda00b230996406fd/spec/human_player_spec.rb). But that's not the most interesting thing about it.

Remembering the `while game.is_ongoing?` game loop from earlier shows that the loop will be naturally stepwise executed due to the blocking nature of `$stdin.gets`.

Everything was fine until a new UI has been introduced. An event based GUI.

### Blocking IO and event based GUIs

Most graphical user interfaces are event driven. Controls/widgets emit events and other parts &ldquo;listen&rdquo; to these events. An example for that would be when a user clicks on a button. This click emits an event (e.g. `click`, or `buttonClicked`, or whatever the framework calls an event like that). Anyone/anything that wants to do something when that particular button has been clicked needs to register itself as a listener for that event. How that is done is different for every framework. Qt for example uses signal and slots for that, where an event is a signal, and a slot is a piece of code that gets executed when that signal is emitted. You connect a slot to a signal by calling `connect()` and passing the appropriate parameters of what slot will be executed by which signal. Java has event listeners that can be added to an object that is capable of emitting events. For a button object that can be done with `attachActionListener()`.

All of these events can happen asynchronously. So there is not a real order or sequence going on when a listener will be executed. (_Well, technically there is some sort of scheduling going on, but I'll skip that for the sake of this blog post._)


### Blocking IO and responsive UIs

What does that mean for the Tic Tac Toe game loop?

Having the existing `HumanPlayer` implementation in mind, a GUI can not (read: should not) wait until a human player clicks a button to place a move. Not even mentioning how you would need to bend a fake `$stdin` version to respond to `gets` and inject it into `HumanPlayer`. Even if you would introduce a fake `$stdin` to handle `gets`, there's still the question from where do you get the actual move the player wants to play?

By the nature of how GUIs work, if the call to `gets` is a blocking call, you are not able to click a button to actually provide a new move. Sounds not like a valuable approach.

### A Way Out

One way out of this is to not constantly keep asking a player to provide the next move. So the game loop in the GUI can still look like we've seen before:

```ruby
while game.is_playable?
  game.play_next_round
end
```
The one thing that changed is a call to `is_playable?` instead of `is_ongoing?`. At that time I'm not interested anymore if the game is ongoing, but whether it can be actually played (i.e. the next player can provide a move).

So `is_playable?` looks like that:

```ruby
def is_playable?
  is_ongoing? && is_ready?
end

def is_ready?
  next_player.ready?
end
```
A player object can now be asked whether it is ready or not (the `HumanPlayer` class has been extended at this point). Ready means if it can provide a new move. If the object can not provide a move the game loop ends.

So when the game loop asks the human player if it is ready, it will return false from that method, because the user has not clicked a button to choose a move yet. The game loop does not block on that call anymore and because the player is not ready the loop itself ends too. This leaves a fully responsive GUI.

But how do we get into the game again? Because the game is not over yet (just in a paused state), it is the user who triggers the next round by clicking on a button. Then this button click sets the value of the next move in the human player object. After that, it triggers the game loop again, which asks the player object if it is ready again. And this time it is, because the value has been set just before. So the game loop can run again just fine. Until the next move of a human player is requested. I left out the details of the `HumanPlayer` class by intention. How the value for the next move is not as important as the concept itself (for example in my implementation there is even [another object involved that the `HumanPlayer` asks for the next move](https://github.com/christophgockel/tictactoe-ruby/blob/master/lib/tictactoe-gui/game_widget.rb#L102)).

This cycle is done until the game is over (by having a draw or a winner).

### Conclusion

All code examples aside, what I wanted to demonstrate is the inversion of control that happened between the game and its players.

Instead of having the `Game` as the main object responsible for program control, this control has been given to &ldquo;the outside world&rdquo; of a game.

In the first version the game asked a player for the next move and waited until that player has made a decision. It was always the game that was in charge.

This has been changed now to as soon as the game realises it can not go on any further (i.e. a player has to provide a move), it gives up the control and leaves it up to its client to trigger the next round(s) again. The _client_ in this case is for example either the command line runtime or a Qt application.
