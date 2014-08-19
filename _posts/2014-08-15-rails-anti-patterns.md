---
layout: post
title: Rails (Anti) Patterns
---
Rails enables you to set up and develop new applications real quick.

Unfortunately, over time this speed can decrease. Not necessarily to a screaching halt, but often times the &ldquo;Rails way&rdquo; of putting code into certain places or folders is not the best way to keep a codebase healthy.

## ActiveRecord (or models in general) all over the place
One thing that is very common to see, is that models of the applications are used in every corner and on every layer of an application. Controllers load, update and save models. Views are querying models...
There's no clear separation between what makes a domain object a domain object, and what the controller and the persistency do with it.

That also means that all components directly depend on the model implementation. It doesn't really matter whether it's [ActiveRecord](http://guides.rubyonrails.org/active_record_basics.html), [Sequel](http://sequel.jeremyevans.net/), [Lotus](https://github.com/lotus/model) or whatever. Coupling your views and controllers directly to your model is not a good idea.

Abstracting away from these concrete implementations via a [Repository](http://martinfowler.com/eaaCatalog/repository.html), [Data Mapper](http://martinfowler.com/eaaCatalog/dataMapper.html) or just old fashioned factories, does help in that regard.

## Too much logic in controllers
Another thing that adds up easily is controllers with too much knowledge about details of the application. A controller in Rails should be treated just as a first instance to communicate to with your application via a web-interface. Nothing more. So it handles all the web and HTTP related things that your application _should_ be unaware of. Instead the controller just passes all the necessary information to another _thing_ that is capable of the actual doing. Often times this _thing_ is called an &ldquo;interactor&rdquo; or a &ldquo;use-case implementation&rdquo;. It doesn't really matter how it is called, as long as the concept is applied. There's a really good [blog post on the 8th Light blog](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html) about that. [Uncle Bob's keynote &ldquo;Architecture the Lost Years&rdquo; at Ruby Midwest 2011](http://youtu.be/WpkDN78P884) is also worth watching.
One benefit (among many other) is that the actual application is testable without the need to test it via controller actions. The controllers should be tested though, but they are not responsible for any application- or doman-specific behaviour. That is done and tested via interactors &ndash; independant of any Rails controller.

## What's going on between controllers and views?
There's some weird behaviour between controller and views where I do not have a better solution for yet (if there is any).

To pass data from the controller to the view, instance variables are created in the relevant controller action/method. These instance variables (yes I mean these things like `@variable`) are just available to the view. I don't know why this is the best practice in Rails - maybe I don't use it long enough yet to fully grasp it. But doesn't that just breaks general OO concepts? Okay, these variables are not strictly private, but still. A controller is an object at runtime that has attributes. The view &ldquo;just knows&rdquo; about all these instance variables.

On the same note I still have to figure out why writing ERB templates is good in Ruby, but when people write templates for PHP and mix HTML and PHP code they will get cast out by society for their lack or morals... But this blog today is not about language comparison (I can already hear someone scream _&ldquo;What about Haml?&rdquo;_...)

---

So much for my first shot at some Rails assessments. Since I'm currently tasked with another apprentice to work further on an internal Rails application, this probably won't be my last post about that topic.