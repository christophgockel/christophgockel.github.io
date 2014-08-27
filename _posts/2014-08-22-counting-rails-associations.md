---
layout: post
title: Counting Rails Associations
---

With Rails and ActiveRecord it's relatively easy to forget that you're interacting with a database in the background. There are many convenience methods available to help a developer &ldquo;focussing on the important things&rdquo;.

Unfortunately these conveniences introduced by ActiveRecord often come with a cost &ndash; if you're not aware of what is going on behind the scenes.

For this example I'm using ActiveRecord associations in a blogging application. There's one table `Users` and one `Posts`.

To get the number of post of a user, ActiveRecord provides three ways to get the result.

```
Users.comments.count
Users.comments.length
Users.comments.size
```
They all return the same result but the way they are getting the result differ from each other.

Calling `.count` on the association triggers an execution of a `SELECT COUNT...` statement on the database.

Calling `.length` falls back to the implementation on a collection of things (like [`Array#length`](http://www.ruby-doc.org/core-2.1.2/Array.html#method-i-length)). Which means that all comments are loaded/read and then the resulting collection is asked about its `.length`.

When using `.size` the implementation used depends on whether all entries of that association have been requested before (e.g. with a call to `.length` or `.all`). If this is the case, that value will be used as the return value of `.size` &ndash; it will not trigger a new call to `.length`. If not, it will call `.count`.

At a first glance I thought `.count` would be the preferable way no matter what. A simple `SELECT COUNT...` will probably always be more performant than counting the elements of a list that needs to be loaded from a database.

After I had a second look though, I realised that it's not that simple.

For cases where ActiveRecord knows the number of elements in an association already, calling `.size` will probably be faster than issuing a new statement to the database. Because it will just return an &ldquo;already known&rdquo; value.

Currently `.size` looks like a good tradeoff to me, because it gives you the best of both worlds. When the association has already been read completely, it uses the value that is already known. If not, a `SELECT COUNT...` will be executed to get the size (i.e. the count) of the association.