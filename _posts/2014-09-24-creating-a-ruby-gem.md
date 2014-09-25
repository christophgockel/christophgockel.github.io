---
layout: post
title: Creating a Ruby Gem
---

Today I extracted a separate gem for the core Tic Tac Toe game logic.
Up to now, the game itself with all its logic and collaborators was inside one project together with the terminal UI as well as the Qt UI.

Now there is a gem with the core logic, and that gem is used by the existing project that contains the different UI implementations only.

Creating the gem itself was relatively easy. Just executing `bundle gem tictactoe` and moving the existing files into the appropriate folders. Adjust `tictactoe.gemspec` with some description values. Done. Well&hellip; almost. It also forced me to finally use a proper namespace/module hierarchy for the game. Which is a good thing anyway, as I clearly should have done that earlier. There was just no real need for it yet.

Having the overall game(s) now split into two parts lead to another issue: shared examples were not directly available anymore for the UI project.

There are some ways to configure the gem accordingly. I came up with three possible ways:

**1. Put everything needed under `lib`**

This could be done relatively easy. But it felt wrong placing test specific classes into `lib`.
After the gem would have been required in another project, the difference is not noticeable, whether the exported artifacts were placed in `lib` from the beginning or not.

An example `.gemspec` file would then just include the regular `lib` directory:

```ruby
Gem::Specification.new do |spec|
  # ... other settings left out for brevity
  spec.require_paths = ["lib"]
end
```
But the very idea of placing code from `spec` into `lib` scared me enough to not do it this way.
 
**2. Export whole the `spec` directory**

Another way would be to just include the `spec` directory in the `require_path` setting of the gem.

```ruby
Gem::Specification.new do |spec|
  # ... other settings left out for brevity
  spec.require_paths = ["lib", "spec"]
end
```
The drawback of this is, that _all_ test files and classes from the `spec` directory will be available for every client of the gem.
Even when you ignore the fact that this would increase the total file size of your gem, it also just pushes unnecessary code to every client of the gem.

**3. Export parts of `spec`**

The third option includes adding just enough additional files to `require_paths` so that clients of the gem get what they're really interested in.

```ruby
Gem::Specification.new do |spec|
  # ... other settings left out for brevity
  spec.require_paths = ["lib", "spec/helpers"]
end
```
I [placed the shared examples and helper class in `spec/helpers/tictactoe`](https://github.com/christophgockel/tictactoe-gem/tree/master/spec/helpers/tictactoe).
To keep the namespacing behaviour intact the `tictactoe` subdirectory is needed.
Since `require_paths` includes `spec/helpers` now, everything below that should have the proper namespace to not pollute any client's load path.

This makes it possible that, as a client of the gem, you do not need to be aware that these files are loaded from a directory that is not the gem's `lib` directory originally.

It's sufficient to require the shared examples like this:

```ruby
require 'tictactoe/shared_examples'
```
So for any client the world is good now.

There is one downside though. In the specs for the gem itself, the path for `require` can not be written like in the example above. In the specs of the gem, you need to be aware that the file comes from a directory called `helpers`. The [`require` calls in the gem's spec_helper show that](https://github.com/christophgockel/tictactoe-gem/blob/master/spec/tictactoe/spec_helper.rb#L3-L4):

```ruby
require 'helpers/tictactoe/board_helper'
require 'helpers/tictactoe/shared_examples'
``` 
You could probably get around that with some `$LOAD_PATH` adjustments, but for the moment it didn't felt so bad to leave it as it is. It's a relatively focussed place where the path is specified like that &ndash; only in the `spec_helper`. If I find myself having the need to specify that path in multiple places in the future, I might change the path handling then.
