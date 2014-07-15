---
layout: post
title: Unnecessary Implementation Details
---

I didn't finish all the tasks I had initially as my weekly assignment due to the preparation of the Java introduction document. Which is done for now, and apparently of use - which I'm really pleased about!

So back to the drawing board and finish refactoring my Ruby code! During my IPM with Jim today, I went through the parts I did finish. While explaining some parts to Jim, I already realised myself that sometimes I mad questionable choices when it comes to the design of my TTT implementation.

One example is the usage of a 0-based board implementation. So if a player wanted to place a move in the top left corner of a board, he would call something like `board.set_move(0, 'x')`. This is not a very human friendly way to place a move, as we humans are not really used to start counting at zero. Because I _totally_ knew that, I made a super clever trick! In my UI, I translated locations of the board to more human friendly ones - basically just by adding 1 to them. When a player entered the move he wanted to make, I translated the player given location back to a location that fits to the board locations - so subtracting 1 again. _Hell, what an elaborate move!_ (Pun intended)

But is this _really_ so clever? No.

The checking of the given location is within allowed bounds (i.e. >0 and <= 9) was also done in the code that read the user input. So whenever the board changes, for example its size, the UI code needs to be updated as well. That's no [Single Responsibility](http://blog.8thlight.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html). Why does the UI has to make these decisions and conversions?!

A better approach to this is, letting the `Board` handle all that. Let the `Board` decide whether a given move is within the allowed range. I can already hear my professor from university asking &ldquo;Who is the Information Expert?&rdquo; (Referring to the [GRASP Principles](http://en.wikipedia.org/wiki/GRASP_(object-oriented_design)). Even though I haven't heard anything about GRASP in the last couple of years, the principles still hold true. `Board` knows all that, so it is a good start to let it answer these kinds of questions.

[Because I changed the public API of `Board`](https://github.com/christophgockel/tictactoe-ruby/commit/937ca82ac195e0129eed57bf4e4c9cb728d3368a) I had to adjust many test cases as well. But with this change, I [could get rid of a few lines of now-unnecessary code in my `Player` class as well as the parts that handle user input](https://github.com/christophgockel/tictactoe-ruby/commit/645ce43729780cb9ed1f09f5e904c51da3d96b51).

Tomorrow I will focus on the design issue that exists at the moment between my `Player` class and `CommandLineIO`. This will probably also be the topic of tomorrows blog post.