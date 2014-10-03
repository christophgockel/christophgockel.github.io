---
layout: post
title: Isolate CSS
---
Hiding details is generally a desirable goal when developing software.

The purpose of the above is to increase the cohesion of parts in our software, as well as to decouple them better from each other via well defined interfaces or boundaries.

This is a well known approach when it comes to isolating yourself from external dependencies in your code. But why only apply it for code you write for the back-end? The same abstraction principles apply no matter what external dependency you want to isolate yourself from.

So it can and should be applied to any CSS framework you use. No matter whether you're using [Bootstrap](http://getbootstrap.com/), [Pure](http://purecss.io/), [Less](http://lessframework.com/), [Blueprint](http://www.blueprintcss.org/), [960](http://960.gs/), &hellip;

The application you write should not have any direct coupling to an external framework. There should be no CSS classes directly used that were not defined by you for your project.

For my example code I will use Yahoo!'s Pure framework. But the principles apply no matter what framework you use.

Pure provides `pure-g` classes to create a grid and several `pure-u-*` classes as units/columns within these grids.

A grid with two buttons next to each other can be done like this:

```html
<div class="pure-g">
  <div class="pure-u-1-2">
    <a class="pure-button" href="/">Back</a>
  </div>
  <div class="pure-u-1-2">
    <a class="pure-button" href="/game/restart">Restart</a>
  </div>
</div>
```

As you can see there are many references to the classes defined by Pure. By that we have coupled the view directly to the framework we used.

Directly coupling to a framework is not always the best approach. Sounds familiar? Maybe that's because whenever something like this happens in, for example, Ruby code, many developers immediately react with some hesitation by introducing such a tight coupling. But even for CSS the same abstraction and isolation principles apply.

The view belongs to your application. And therefore you should isolate any external dependency from that.

It would be better if we could write the previous code in a manner that we are in charge of what CSS classes the elements need.

Something like this:

```html
<div class="grid-container">
  <div class="half-width">
    <a class="button" href="/">Back</a>
  </div>
  <div class="half-width">
    <a class="button" href="/game/restart">Restart</a>
  </div>
</div>
```

This does a better job at expressing the intent of what the HTML is supposed to do.

  * Instead of `pure-g` we use `grid-container`, because that's what `pure-g` really is.
  * Instead of `pure-u-1-2` we use `half-width`, because `pure-u-1-2` is really just a div that spans half of the width of its parent element.
  * And instead of `pure-button` we use just `button`, because &hellip; well it's just a button.

### How to achieve CSS isolation?

I used [Sass](http://sass-lang.com) to be able to use the CSS classes just shown.
Using Sass allows to &ldquo;[extend](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend)&rdquo; new selectors from existing selectors.

With this we can create new selectors for us that match the domain of the application we're developing.

The definition for the classes of the last example look like this:

```scss
div.grid-container {
  @extend .pure-g;
}

div.half-width {
  @extend .pure-u-1-2;
}

a.button {
  @extend .pure-button;
}
```
Sass will create the classes with the appropriate content for us. So `div.grid-container` will have the same declarations as `.pure-g` has.

This is not only possible for single classes, though. Also multiple usages of classes can be merged together.

A button that is coloured in Pure's primary colour is declared like that for example:

```html
<button type="submit" class="pure-button pure-input-1 pure-button-primary">Play!</button>
```
With a SCSS declaration like this:

```scss
button.play-game {
  @extend .pure-button, .pure-input-1, .pure-button-primary;
}
```

We are now able to declare the button with better expressed intent:

```html
<button type="submit" class="play-game">Play!</button>
```

### What does that tell us?

By isolating from the CSS framework we do not couple our views to any specific framework anymore.

When we want to switch to another framework, we do not have to update all the views. We only have to update the abstraction layer between our views and the framework &ndash; which is the SCSS file. It's likely to be easier to maintain just that file instead of finding all usages of framework specific CSS classes across all existing views.

If there's one thing to take away from that: Isolation principles are valid for every area in our applications.
