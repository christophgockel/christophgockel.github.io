---
layout: default
title: "Getting Rails on Track - Part 2: Views"
external-url: https://8thlight.com/blog/christoph-gockel/2016/10/26/getting-rails-on-track-part-2-views.html
---

# Getting Rails on Track - Part 2: Views

_In this multi-series blog post, we're going hands-on through the steps that can help transform the design of a default Rails application into one with clearer responsibilities and that is easier to test._

_Have a look at the other parts, too!_

- _[Getting Rails on Track - Part 1: Models]({{ site.baseurl }}/posts/getting-rails-on-track-part-1-models)_
- _[Getting Rails on Track - Part 3: Controllers]({{ site.baseurl }}/posts/getting-rails-on-track-part-3-controllers)_

_You can follow along with a [repository available on GitHub](https://github.com/christophgockel/rail-track) that contains all the code discussed here._

Ruby's `.erb` templates are the default for Rails' views.
Technically they are Ruby code mixed with HTML.
While this provides a tremendous amount of flexibility, it can also lead to a lot of logic in the views.

The downside to this is the more logic we put into the views, the harder it becomes to test that logic.
There are tools like Capybara or Selenium to help us with that, though.
And there are times where this testing approach is worthwhile.
But it shouldn't be the first choice to do that, as these tests tend to be slow and brittle. Frontend testing often involves rendering the UI, which significantly increases the duration of the test suite.
Additionally, tests that rely on class names or even website copy are more likely to fail for the wrong reasons once those change.

Hence, from a general testing point of view, we should try not to test our application through the UI.
Sometimes this is also referred to as _&ldquo;submarine testing&rdquo;_—the testing starts just after the UI layer, i.e. one layer _below the surface_ of the application.

This forces us to keep our views as dumb as possible.
The more logic we put in there, the more logic we can't test with this approach.

One way to achieve that is to introduce presenters that the views can use.
Ideally a view only calls methods on that presenter with as little conditional logic as possible.
Having said that, it is not always easy to end up there&mdash;and also not always needed.
For example it can be okay having something like the following code in a view:

{% highlight html+ruby %}
<% if presenter.logged_in?(current_user) %>
  <a href="/logout">Log out</a>
<% else %>
  <a href="/login">Log in</a>
<% end %>
{% endhighlight %}

As long as `logged_in?` of the presenter has the appropriate unit tests.
It's oftentimes okay having this kind of conditional logic in a view because it is still testable in an easy enough manner, even though it might be a manual test.


## Presenting Movies

Continuing with the movie example, let's have a look at how a movie gets displayed.

{% highlight ruby %}
RSpec.describe MoviesController
  it "renders the show template" do
    movie = Movie.create(title: "the title")
    get :show, id: movie.id

    expect(response).to render_template("show")
  end
end
{% endhighlight %}

Making that test pass is relatively straightforward.
The controller looks up a movie from the database and provides it for the view in the form of an instance variable.

{% highlight ruby %}
class MovieController < ApplicationController
  def show
    @movie = Movie.find(params[:id])
  end
end
{% endhighlight %}

The view then formats the movie's data.

{% highlight html+ruby %}
<h1><%= @movie.title.titleize %></h1>
<h2><%= @movie.release_date.strftime("%e %B %Y") %></h2>

<p><%= @movie.description %></p>
{% endhighlight %}

What&mdash;if anything&mdash;can be improved here?

The smell here is the logic in the view.
To be precise, the calls to `titleize` and `strftime` contain domain logic, and should be tested in isolation.
But since this is directly written in the view, it is not as straightforward to test as it could be.


### Adding a Movie Presenter

The logic for formatting a movie's data can be wrapped in a presenter.
Let's start with a test for its title.

{% highlight ruby %}
RSpec.describe MoviePresenter do
  it "titleizes the title" do
    movie = Movie.new(title: "the movie title")
    presenter = MoviePresenter.new(movie)

    expect(presenter.title).to eq("The Movie Title")
  end
end
{% endhighlight %}

To make that test pass, we move the code we already have in the view to the new `MoviePresenter`.

{% highlight ruby %}
class MoviePresenter
  def initialize(movie)
    @movie = movie
  end

  def title
    movie.title.titleize
  end

  private

  attr_reader :movie
end
{% endhighlight %}

In a similar fashion, we can add a test for the `release_date`.

{% highlight ruby %}
it "formats the release date" do
  movie = Movie.new(release_date: Date.parse("1984-06-08"))
  presenter = MoviePresenter.new(movie)

  expect(presenter.release_date).to eq("8 June 1984")
end
{% endhighlight %}

Making this test pass is also as straightforward as moving the code from the view to the presenter.
As we do this, we can even fix a bug while we're at it, as the original format string `"%e"` adds a space in front of the day.
What we actually want is just the day of the month without a padded zero or space (i.e. `"%-d"`).

{% highlight ruby %}
def release_date
  movie.release_date.strftime("%-d %B %Y")
end
{% endhighlight %}

The presenter should be able to present all the data of a movie, so let's add a last test for the description.

{% highlight ruby %}
it "presents the description" do
  movie = Movie.new(description: "the description")
  presenter = MoviePresenter.new(movie)

  expect(presenter.description).to eq(movie.description)
end
{% endhighlight %}

We don't have any new behaviour around the description for a movie, which makes the implementation a simple delegation.

{% highlight ruby %}
def description
  movie.description
end
{% endhighlight %}

An alternative implementation for a delegation could make use of [Ruby's `Forwardable` module](http://ruby-doc.org/stdlib-2.3.1/libdoc/forwardable/rdoc/Forwardable.html).
The example doesn't use it because it was only one method we need to delegate here.
`Forwardable` can be a bigger benefit when there are many methods to delegate.

The full code of the [`MoviePresenter`](https://github.com/christophgockel/rail-track/blob/part-2-views/lib/movie_presenter.rb) and [its tests](https://github.com/christophgockel/rail-track/blob/part-2-views/spec/movie_presenter_spec.rb) is available [on GitHub](https://github.com/christophgockel/rail-track/tree/part-2-views).


### Putting It Back Together

Now that we have a presenter we can use it in the controller.
For that, we first add a test to reflect that we actually do get a presenter.

{% highlight ruby %}
it "wraps a movie in a presenter" do
  movie = Movie.create(title: "the title")
  get :show, id: movie.id

  expect(assigns(:movie)).to be_a(MoviePresenter)
end
{% endhighlight %}

To make that test pass, we update the controller's code as follows.

{% highlight ruby %}
def show
  @movie = MoviePresenter.new(Movie.find(params[:id]))
end
{% endhighlight %}

And use the presenter in the view.

{% highlight html+ruby %}
<h1><%= @movie.title %></h1>
<h2><%= @movie.release_date %></h2>

<p><%= @movie.description %></p>
{% endhighlight %}

Now the view doesn't know how to format a release date or a title anymore.
It's simply viewing whatever it gets.


## Presenting a List of Movies

Let's have a look at the movie list.
The following code shows the controller test as well as the view code.

{% highlight ruby %}
RSpec.describe MovieController
  it "renders the overview" do
    get :index

    expect(response).to render_template("index")
  end
end
{% endhighlight %}

{% highlight html+ruby %}
<ul>
  <% Movie.each do |movie| %>
    <li><%= movie.title.titleize %></li>
  <% end %>
</ul>
{% endhighlight %}

While this absolutely does work, one design smell here is that the view reaches down to a model class, which ties the view directly to the database.
One way to improve that is by introducing a variable the view can use to iterate over the available movies.

{% highlight ruby %}
it "has a list of movies" do
  get :index

  expect(assigns(:movies)).to be_an(Array)
end
{% endhighlight %}

{% highlight ruby %}
class MoviesController < ApplicationController
  def index
    @movies = Movie.all.to_a
  end
end
{% endhighlight %}

{% highlight html+ruby %}
<ul>
  <% @movies.each do |movie| %>
    <li><%= movie.title.titleize %></li>
  <% end %>
</ul>
{% endhighlight %}

That is a small improvement over the original version, but it still bears some problems.
Now that we introduced an abstraction in form of an instance variable, what happens if we rename `@movies` to `@all_movies`?
The tests for the controller verify the existence of that variable and would catch that.
Not an ideal unit test, because it is not testing any real behaviour, but still a small safety net.
It does not tell us if the view actually uses the correct variable, though.
This way we would end up with a green test suite but a broken application.
Only by manually testing the view again could we catch that bug.

### Testing Views?

The issue just presented is a common one.
It's not a problem with Rails per se, but this happens often when we have a UI that we want to test.
There are many ways to solve it, and we're going to discuss two possible options.

We've just seen one way to solve that: verifying the existence of an instance variable.
As mentioned, this approach does not completely guard us against the issue of a renamed variable (or even a typo in the view's code).
Tests like that only verify that _something_ the view could use is _available_.
They don't verify that the view actually uses it.

Another way is to actually build the view in the tests.
With that, we can verify that the template can be created without errors.
Rails (more specifically `rspec-rails`) has a helper method for that called `render_views`.
Calling that method first will cause the controller under test to try building the final view that would be sent to the client.

With that we can write a test that will fail every time the view can not be created, e.g. because a used variable is not initialised.

{% highlight ruby %}
RSpec.describe MoviesController
  render_views

  it "renders the overview" do
    get :index

    expect(response).to render_template("index")
  end
end
{% endhighlight %}

But depending on how big the views become, there can be a downside to this approach: building the actual view can slow down our tests.
But it's worth mentioning that the increased reliability of the test suite can be worth the time it takes to run it.

Both ways are valid options, and both ways should be considered for the specific use case at hand.
A good middle ground could be to have a few tests that use `render_views` to make sure everything is wired up correctly, and all other tests exercise the controller without rendering the views.

It's a trade-off between test run time and reliability of the test suite.
This trade-off should be discussed and decided on a case by case basis.

