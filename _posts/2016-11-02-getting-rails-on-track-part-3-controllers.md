---
layout: default
title: "Getting Rails on Track - Part 3: Controllers"
external-url: https://8thlight.com/blog/christoph-gockel/2016/11/02/getting-rails-on-track-part-3-controllers.html
---

# Getting Rails on Track - Part 3: Controllers

_In this multi-series blog post, we're going hands-on through the steps that can help transform the design of a default Rails application into one with clearer responsibilities and that is easier to test._

_Have a look at the other parts, too!_

- _[Getting Rails on Track - Part 1: Models]({{ site.baseurl }}/posts/getting-rails-on-track-part-1-models)_
- _[Getting Rails on Track - Part 2: Views]({{ site.baseurl }}/posts/getting-rails-on-track-part-2-views)_

_You can follow along with a [repository available on GitHub](https://github.com/christophgockel/rail-track) that contains all the code discussed here._

After we [extracted domain logic out of our models]({{ site.baseurl }}/posts/getting-rails-on-track-part-1-models) and [separated presentation logic from the UI]({{ site.baseurl }}/posts/getting-rails-on-track-part-2-views), it's time to work on the code that glues our web application together: the controllers.

Rails controllers, and controllers of web frameworks in general, tend to attract behaviour.
Unfortunately, often times it's behaviour that does not necessarily belong in a controller.
If we think of Rails purely in the way of being a facility to expose our application to the web, a first step toward a better design is done already!

In the [movie example](https://github.com/christophgockel/rail-track), that means the core domain of the application is organising movies.
It's not storing or viewing movies on a website.
That's a nice &ldquo;byproduct&rdquo; but not the main intent, even when we can be certain the application will always be running in the web.
Separating the core domain logic and behaviour from the so-called _delivery mechanism_ is a good design approach.

There are several names for this kind of application architecture.
Uncle Bob calls it [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) and Alistair Cockburn used [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture) or Ports &amp; Adapters for it.
The main idea behind it is that the application's core logic shouldn't worry or even know whether users interact with it through a web interface, a GUI or a terminal, etc.

A similar concept has also been around the Rails community for some time now under the name of _skinny controllers_.
Keeping controllers _skinny_ means there shouldn't be much code in them.
Ideally, the only responsibility of a controller is to translate a web request for our application.

_Translating_ can be as simple as just triggering an action because a user clicked on a button, or converting data of a form submission that contains a lot of parameters into a data structure the application can use.

But where to put all the code that currently lives in the controller then?

Into something we call service classes.
A service class encapsulates a certain action or behaviour specific to a use case.
It can also be seen as a command object, to use a design pattern name for it.


## Movie Example

How can this look in our movie example?
This is the implementation of the [`MoviesController`](https://github.com/christophgockel/rail-track/blob/part-2-views/app/controllers/movies_controller.rb) (shortened for readability purposes).

{% highlight ruby %}
class MoviesController < ApplicationController
  def create
    @movie = Movie.new(movie_params)
    validation = MovieValidator.validate(movie)

    if validation.okay?
      @movie.save
      redirect_to root_path, notice: "Successfully created movie #{@movie.title}."
    else
      flash.now[:alert] = validation.messages
      render :new
    end
  end

  private

  def movie_params
    params.require(:movie).permit(:id, :title, :release_date, :description)
  end
end
{% endhighlight %}

Here, the responsibilities of `create` are
a) creating a movie;
b) showing a success message, redirecting when the movie has been created; and
c) showing an error message depending on whether the movie couldn't be created while re-rendering the form.
And not to forget, actually persisting the movie if sufficient data has been provided.

Having all of these responsibilities is not necessarily bad, but having the controller know about the specifics of how to create and validate a movie is.
This &ldquo;obsession&rdquo; about details is reflected in the tests for the controller, too.

{% highlight ruby %}
RSpec.describe MoviesController do
  describe "#create" do
    def create_movie(attributes = {})
      post :create, movie: {
        "title":        "new movie",
        "release_date": "2016-10-01",
        "description":  "new description"
      }.merge(attributes)
    end

    it "redirects to movie overview" do
      create_movie
      expect(response).to redirect_to(root_path)
    end

    it "contains a success message" do
      create_movie
      expect(flash[:notice]).to include("created movie new movie")
    end

    it "creates a new movie" do
      create_movie

      newest_movie = Movie.last

      expect(movie.title).to eq(movie_attributes[:title])
      expect(movie.release_date).to eq(movie_attributes[:release_date])
      expect(movie.description).to eq(movie_attributes[:description])
    end

    it "does not create a new movie with validation errors" do
      create_movie(title: "")

      expect(Movie).not_to exist
    end

    it "re-renders the form again on errors" do
      create_movie(title: "")
      expect(response).to render_template("new")
    end

    it "contains an error message for invalid movies" do
      create_movie(title: "")
      expect(flash.now[:alert].first).to include("Title")
    end
  end
end
{% endhighlight %}

Most of the tests verify web-related behaviour, which is what we want the controller tests to focus on.
But that makes a test like `"creates a new movie"` stand out, as it is too concerned about details.
We do need to verify that a movie gets created, but the controller's tests shouldn't verify the details of that action.


### Adding a Create Service

The controller should not be responsible for actually creating a movie.

Well&hellip; that is not 100% true.
It is and will be responsible for creating a movie, but hopefully only by delegating the details of how to do this to something else.
This will also help clean up the tests for the controller, as the details of verifying that the correct movie has been created can be moved to a more suitable place.

We're going to extract those details to a `MovieCreator` class.

{% highlight ruby %}
class MovieCreator
  def self.execute(attributes)
    new(attributes).execute
  end

  def initialize(attributes)
    @attributes = attributes
  end

  def execute
    movie      = Movie.new(attributes)
    validation = MovieValidator.validate(movie)

    if validation.okay?
      movie.save
    end

    Result.new(movie, validation.messages)
  end

  private

  attr_reader :attributes

  class Result
    attr_reader :movie, :messages

    def initialize(movie, messages)
      @movie    = movie
      @messages = messages
    end

    def successful?
      messages.empty?
    end
  end
end
{% endhighlight %}

The tests for the `MovieCreator` will then make sure that a movie with valid data has been created in the correct way.

{% highlight ruby %}
{% raw %}
RSpec.describe MovieCreator do
  let(:movie_attributes) {{
    title:        "the title",
    release_date: Date.today,
    description:  "the description"
  }}

  def create_movie(attributes = {})
    described_class.execute(movie_attributes.merge(attributes))
  end

  it "creates a movie" do
    create_movie
    movie = Movie.first

    expect(Movie.count).to eq(1)
    expect(movie.title).to eq(movie_attributes[:title])
    expect(movie.release_date).to eq(movie_attributes[:release_date])
    expect(movie.description).to eq(movie_attributes[:description])
  end

  it "returns a successful creation" do
    creation = create_movie

    expect(creation).to be_successful
  end

  it "does not create a movie with invalid data" do
    create_movie(title: "")

    expect(Movie).not_to exist
  end

  it "returns a non-succesful creation" do
    creation = create_movie(title: "")

    expect(creation).not_to be_successful
    expect(creation.movie).to be_a(Movie)
    expect(creation.messages).not_to be_empty
  end
end
{% endraw %}
{% endhighlight %}

Here, `MovieCreator` returns a result object that can be asked whether or not the movie could be created.
With that in place, the controller's `create` method can be simplified into this:

{% highlight ruby %}
def create
  creation = MovieCreator.execute(movie_params)

  if creation.successful?
    redirect_to root_path, notice: "Successfully created movie #{creation.movie.title}."
  else
    @movie = creation.movie
    flash.now[:alert] = creation.messages
    render :new
  end
end
{% endhighlight %}

Now the controller doesn't need to know the details of _how_ to create (and validate) a movie anymore.
It delegates to the `MovieCreator` and decides based on its result what to do.

Which leaves us with another question:
What do we do with the previously mentioned test `"creates a new movie"`?

One option could be to to leave it as it is, since it is still passing and verifying needed behaviour.
An alternative is to refactor it in a way that expresses the intent behind `create` on a higher level:

{% highlight ruby %}
RSpec.describe MoviesController do
  describe "#create" do
    def create_movie
      # ...
    end

    it "creates a new movie" do
      create_movie

      expect(Movie).to exist
    end
  end
end
{% endhighlight %}

Now the controller's tests focus on the essence of what the controller should be doing and nothing else.
All web-related behaviour is still in place and covered by the tests.
But we did change _how_ the movie gets created.
Since on this level we're not interested in the details of how to create a movie, we only verify that _a_ movie has been created.

The final code for the [controller](https://github.com/christophgockel/rail-track/blob/part-3-controllers/app/controllers/movies_controller.rb) as well its [tests](https://github.com/christophgockel/rail-track/blob/part-3-controllers/spec/controllers/movies_controller_spec.rb) are available [on GitHub](https://github.com/christophgockel/rail-track/tree/part-3-controllers).


### Namespacing

Now that we introduced several movie-related classes, it's time to revisit the directory structure of our application.
We created a `MovieValidator`, a `MoviePresenter`, a `MovieCreator`, and a `MovieUpdater`.
They all seem to have something to do with a movie.
So instead of prefixing all classes with &ldquo;`Movie`&rdquo;, Ruby provides modules as a better way to group cohesive classes like that.

This will also help tidying up the `lib` directory, as the class files will then reside in (for example) `lib/movies`.
The classes inside that directory will be namespaced with a `Movies` module (e.g. `Movies::Validator`, `Movies::Presenter`, `Movies::Creator` and `Movies::Updater`).

Another way to name theses service classes is using verbs instead of nouns, e.g. `Movies::Create` or `Movies::Update`.
Which way we choose is not as important as being consistent with it.


## Summary

By extracting service classes, we improved the responsibilities of our application's classes.

We will end up with more classes overall in the end, but all classes focus on a more clearly defined task&mdash;having one responsibility at best.
This not only helps with a better adherence to the Single Responsibility Principle.
It makes testing easier too, as the tests can focus on that one particular task as well.

This was a common theme visible in all parts of this blog post series so far: identifying and defining responsibilities.

Just because we use a framework does not mean we're bound to all its defaults.
Rails (or any framework, for that matter) is not always to blame if our application code becomes messy or the tests become slow.
Using a framework should increase our productivity and not slow us down.
Sometimes, though, we need to work a little bit harder to reap the benefits.

