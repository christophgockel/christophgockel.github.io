---
layout: post
title: Tic Tac Toe Qt GUI
---

After spiking a Qt GUI for the Coin Changer ([see relevant kata](http://craftsmanship.sv.cmu.edu/exercises/coin-change-kata)), I could better estimate what it would take to finish the story about a Qt GUI for Tic Tac Toe.

Sure, the GUI for Coin Changer maybe could be a bit more polished, but the main reason for the spike was to get myself familiar with Qt, and not to build the most beautiful UI.

As GUIs and the eventing concepts behind them were not new to me, the main problems I struggled with were with the framework itself. Like what’s the best practice to create a new GUI? How do I register event handlers for (which are _Signals_ and _Slots_ in Qt)… But most of the time I could ask my fellow apprentices how something is done in Qt, since nearly all of them have done their Qt version of Tic Tac Toe already.

A working UI for a human vs. human game with a 3x3 board is ready now.
Next steps will involve isolating logic for the game-play itself from Qt specific behaviour. As well as adding more features to the UI itself like a game mode selection, choosable board sizes and displaying winning messages.