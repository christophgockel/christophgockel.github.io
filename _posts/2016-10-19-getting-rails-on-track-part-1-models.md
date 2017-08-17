---
layout: default
title: "Getting Rails on Track - Part 1: Models"
external-url: https://8thlight.com/blog/christoph-gockel/2016/10/19/getting-rails-on-track-part-1-models.html
---

# Getting Rails on Track - Part 1: Models

_In this multi-series blog post, we're going hands-on through the steps that can help transform the design of a default Rails application into one with clearer responsibilities and that is easier to test._

_Have a look at the other parts, too!_

- _[Getting Rails on Track - Part 2: Views]({{ site.baseurl }}/posts/getting-rails-on-track-part-2-views)_
- _[Getting Rails on Track - Part 3: Controllers]({{ site.baseurl }}/posts/getting-rails-on-track-part-3-controllers)_

_You can follow along with a [repository available on GitHub](https://github.com/christophgockel/rail-track) that contains all the code discussed here._

Active Record is the default persistency framework that comes with Rails.
Using it makes it easier for us to design the data model of our applications.
Reading and writing data is relatively simple, but the simplicity of Active Record's extended features shadow some of its pitfalls.
These pitfalls do not surface for smaller applications, but can have an impact on our productivity as our application grows bigger.

For example, we can add custom validations whenever we want to save a record to the database.
What starts as an innocent use of a feature can turn into a high-cost refactoring if we need to change the definition of what it means to be a _valid_ record.

For that, let's have a look at the `Movie` class from the [example application](https://github.com/christophgockel/rail-track).

{% highlight ruby %}
class Movie < ActiveRecord::Base
  validates :title, presence: true
  validate  :release_date_is_after_1895

  private

  def release_date_is_after_1895
    if release_date.nil?
      errors.add(:release_date, "must be be provided.")
    elsif release_date.year < 1895
      errors.add(:release_date, "is older than the world's first motion picture.")
    end
  end
end
{% endhighlight %}

Whenever a new movie gets persisted, Active Record will ensure that it has at least a name and that the release date is not before 1895.
So far so good.
What are the issues with the above code?

The simplicity of `validates :title` is a great feature of Active Record, as it allows a simple and concise way of expressing the need of a title.
For those models whose requirements change frequently, this can cause trouble, though.
Especially in a test suite that creates a lot of movies.
Every time we change the definition of what it means to be a _valid_ movie, there's a chance it has a rippling effect across our code base.
This is also what the code smell &ldquo;shotgun surgery&rdquo; identifies.

To a certain extent, gems like [`factory_girl`](https://github.com/thoughtbot/factory_girl) can help alleviate these issues.
But before we go on and pull in more dependencies to fight symptoms, we should think about what the actual cause of the problem is.

The problem just described is not only visible in the test suite of our application.
The same behaviour would be visible if a user wants to edit a program and save it without making any changes.
All of a sudden a validation message could appear telling the user that the existing program she just edited is not valid anymore.
While it is good that the validations work as expected, it's surprising and unexpected from a user experience perspective.


## Alternatives

The tension between ease of use of a framework feature and maintainability stems from tightly coupled concerns about persistency and data validations.
To find an alternative to that, we can ask ourselves what we usually do when things are too coupled together: we separate them.

Once we have one _thing_ that deals with persisting a movie and another _thing_ to validate a movie, we have done more than created better separated concerns.
We also end up with two new classes that adhere better to the single responsibility principle.
As a side effect, we get the benefit of better testability.
Any test that's not specifically concerned about a valid movie can just create a new movie and use it.


### Code Example

Using the previous `Movie` example, what we want to do is move all code that is about validating a movie's data into a new class.

Let's start with the first test for our new `MovieValidator`.

{% highlight ruby %}
RSpec.describe MovieValidator do
  let(:valid_movie) {Movie.new(name: "the name", release_date: DateTime.parse("08 June 1984")}

  it "validates a movie" do
    expect(MovieValidator.valid?(valid_movie)).to be_truthy
  end
end
{% endhighlight %}

`MovieValidator` has a method `valid?` that returns `true` or `false`, depending on whether the given movie is valid or not.

A solution we come up with to make that test pass might look like this in the end:

{% highlight ruby %}
class MovieValidator
  def self.valid?(movie)
    has_title?(movie)
  end

  private

  def self.has_title?(movie)
    movie.title.present?
  end
end
{% endhighlight %}

But what happens when a movie is not valid?
We need a way to return validation messages to clients of `MovieValidator`.
Later, these messages could be displayed to an end-user, for example.
Let's capture that with new tests that verify these messages.

{% highlight ruby %}
RSpec.describe MovieValidator do
  let(:valid_movie)   {Movie.new(name: "the name", release_date: DateTime.parse("08 June 1984")}
  let(:invalid_movie) {Movie.new}

  def validation_of(movie)
    MovieValidator.validate(movie)
  end

  it "validates a movie" do
    expect(validation_of(valid_movie)).to be_okay
  end

  it "does not contain any messages" do
    expect(validation_of(valid_movie).messages).to be_empty
  end

  it "does not validate an invalid movie" do
    expect(validation_of(invalid_movie)).not_to be_okay
  end

  it "has messages for invalid movies" do
    expect(validation_of(invalid_movie).messages).to contain("Name cannot be empty.")
  end
end
{% endhighlight %}

To make these tests pass, we introduce the concept of a validation result.

{% highlight ruby %}
class MovieValidator
  def self.validate(movie)
    new(movie).validate
  end

  def initialize(movie)
    @movie = movie
  end

  def validate
    result = Result.new
    result << validate_title
    result
  end

  private

  attr_reader :movie

  def validate_title
    "Title can't be empty." if title_empty?
  end

  def title_empty?
    movie.title.blank?
  end

  class Result
    attr_reader :messages

    def initialize
      @messages = []
    end

    def okay?
      messages.empty?
    end

    def <<(message)
      @messages << message if message
    end
  end
end
{% endhighlight %}

Whenever a new requirement for the validity of a movie comes up, we can implement it isolated from all other uses of `Movie` centralised in `MovieValidator`.
The final version of the [test](https://github.com/christophgockel/rail-track/blob/part-1-models/spec/movie_validator_spec.rb) and the [production code](https://github.com/christophgockel/rail-track/blob/part-1-models/lib/movie_validator.rb) are available on GitHub.

Instead of rolling our own validations, there's also the option of including [ActiveModel::Validations](http://api.rubyonrails.org/classes/ActiveModel/Validations.html).
This is a viable alternative depending on the use case at hand, as we can make use of the already existing implementation almost without change.

{% highlight ruby %}
class MovieValidator
  include ActiveModel::Validations

  validates_presence_of :title
  validate              :release_year_not_before_1895

  def initialize(movie)
    @title        = movie.title
    @release_date = movie.release_date
  end

  private

  attr_reader :title, :release_date

  def release_year_not_before_1895
    if release_date.nil?
      errors.add(:release_date, "must be provided.")
    elsif release_date.year < 1895
      errors.add(:release_date, "is older than the world's first motion picture.")
    end
  end
end
{% endhighlight %}

After extracting the validations, how does the `Movie` class look after these changes?

{% highlight ruby %}
class Movie < ActiveRecord::Base
end
{% endhighlight %}

There's some truth to [the best code is no code at all.](https://blog.codinghorror.com/the-best-code-is-no-code-at-all/)
We managed to get rid of all custom logic in our model.
This leaves us with `Movie` only being our gateway to persistency, and nothing else.


## Which tests for Active Record do we keep then?

Ideally none.
But let me explain that.

No tests at all goes a bit too far.
In the end we still need to make sure that a Movie gets persisted at some point.
But we shouldn't verify that Active Record works as expected.
That's a dependency we expect to function properly.

What we do want to verify, though, is that the part of our code that interacts with Active Record does the right thing.
This is currently specified in the [tests](https://github.com/christophgockel/rail-track/blob/part-1-models/spec/controllers/movies_controller_spec.rb#L37) and [implementation](https://github.com/christophgockel/rail-track/blob/part-1-models/app/controllers/movies_controller.rb#L16) of `MovieController`.

In an upcoming part of this series we will go into more detail of whether that's the right place for it, or if there are better alternatives.

