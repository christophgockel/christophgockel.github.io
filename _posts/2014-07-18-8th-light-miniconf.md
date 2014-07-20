---
layout: post
title: 8th Light Miniconf
---

In the morning I added a small [change regarding the handling of invalid moves](https://github.com/christophgockel/tictactoe-ruby/commit/2a516f06791e8e79eff99ce35a6eceba3d68aff7) in my Tic Tac Toe game.
The main reason I did this was to get rid of the rescue handler for `Board::InvalidMove` in `CommandlineUI`. The handling of the error is still there, I didn't just removed it completely. But I [pushed it down into the `Game` itself](https://github.com/christophgockel/tictactoe-ruby/commit/c62d2fdccb6d47270745573e5288366126cb1d78). The `Game` knows of a _thing_ like a `Board` in contrast to `CommandlineUI` which doesn't know that there is a `Board` at all. It just seemed odd that a part that knows nothing about `Board` is able to handle an exception from it.

Around noon we all went to the [Makers Academy](http://www.makersacademy.com/) office to attend [a talk Makis gave there](https://twitter.com/jsuchy/status/490098890507812864) (and to support him, of course!). He talked about his journey from his previous job(s), joining Makers Academy, graduating there and starting the apprenticeship at 8th Light. (_Good job there Makis! I'm happy to have you as a fellow apprentice._)

After that we went to Jim's place to watch/attend a Miniconf 8th Light is organizing every quarter. As the conference were held in the Chicago office, we joined remote, but it worked smoothly, and without technical problems. Overall good talks and a lovely afternoon/evening at Jim's home (thanks again for having us!).