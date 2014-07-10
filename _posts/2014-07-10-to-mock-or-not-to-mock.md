---
layout: post
title: To mock or not to mock
---
Part of the feedback I got in this week's iteration meeting with Jim was to clean up the specs for my Game class.

I am not particular happy the way the [specs](https://github.com/christophgockel/tictactoe-ruby/blob/c9cb4accba7b0a856a445bea4e86432f2114740a/spec/game_spec.rb) looked like either. For my taste there is too much stubbing and mocking going with no real concept visible.

So, today I started working on adding [more expressiveness to the specs](https://github.com/christophgockel/tictactoe-ruby/blob/935329c345fd9549ae7aac5b43a1b00ff87aaa1d/spec/game_spec.rb) for a while. They're not quite in the shape I want them to be, but I think it's at least a small improvement compared to the [initial version](https://github.com/christophgockel/tictactoe-ruby/blob/c9cb4accba7b0a856a445bea4e86432f2114740a/spec/game_spec.rb).

It's just that they're still not as readable as other tests that you see in other RSpec examples. Will work further on that tomorrow.

Maybe part of the problem is the nature of the `Game` class itself. It has the most collaborators of all classes in the game. It is responsible for the correct sequence of the game itself. Asking the players for moves, placing their moves on the board, switching players, redo. All that until there's a winner or a draw.

And as I want to focus on the logic/behaviour inside `Game`'s methods, I inject most of the collaborators as test doubles. And these test doubles need to be properly set up before a test. That lead quickly to a lot of set up code scattered throughout the test class.

Another thing is that some test doubles are used as spies, some as mocks. As a first approach I started to change the mocks to spies. Mostly because I don't neccessarily like that the use of a mock changes the order of the [Arrange, Act, Assert Pattern](http://c2.com/cgi/wiki?ArrangeActAssert).

Take the following spec for example:

```ruby
  it 'displays the board content for each round plus at the end of a game' do
    expect(display).to receive(:show_contents).with(board).exactly(4).times
    @game.start
  end
```

It sets a mock-expectation on the test double `display` that method `.show_contents()` should be called exactly four times.

In contrast, this is the same spec using a spy:

```ruby
  it 'displays the board content for each round plus at the end of a game' do
    prepare_two_rounds_game
    @game.start
    expect(display).to have_received(:show_contents).exactly(4).times
  end
```
Ignoring `prepare_two_rounds_game` for the moment, what stands out the most, is that the actual expectation comes _after_ the production code has been executed.

For me this feels more natural. But maybe it's just because I'm more used to have (manual) verifications at the end of a test case.
