---
layout: post
title: "Legacy Code: Spying On Global Functions"
external-url: https://8thlight.com/blog/christoph-gockel/2017/10/23/legacy-code-spying-on-global-functions.html
---

As developers we often find ourselves in a situation where we first need to get an existing piece of functionality under test before we can safely add a new feature.

In a recent project we ran into exactly that issue with a large codebase that had low test coverage.
The project's language was PHP and the code we needed to extend relied on a global function provided by PHP: [`imagepng()`](http://php.net/manual/en/function.imagepng.php).
While all code examples in this post are written in PHP, the ideas and examples shown do apply to other languages, too.

This is a simplified version of the class under test we're going to discuss in this post:

{% highlight php %}
<?php
namespace Images;

class Writer
{
    public function write($filename)
    {
        $path = "/var/www/html/" . $filename;
        $image = imagecreate(100, 50);

        imagepng($image, $path, 9, 0);
    }
}
{% endhighlight %}

The tests we wanted to add did not need to verify the image itself, but that an image will be created.
When running the tests we wanted to avoid having to write the actual image to the file system.
Not only would it slow down the test suite, we would also need to make sure to create the directory structure `/var/www/images` beforehand.


## Legacy Code Testing Strategies

The two previous points beg an interesting question:
When testing legacy code, do we care about the very strict definition of a unit test (e.g. no disk operations)?

The universal answer to that is _&ldquo;it depends&rdquo;_.
We probably do care about that when we prioritize pure speed of a test suite in order to shorten the feedback loop cycle.
If the goal is to get complete confidence at an integration level, then it is probably okay to wait a little bit longer.

Maybe it's good enough at the beginning to write an actual image file, just not in the directory `/var/www/images`.
If it is possible for us to change the constructor of `Writer` to take the target directory name, we might have improved the code enough to be able to start with the actual feature we want to implement.

There are usually many ways to tackle a problem like the one just described, and in this post we're going to look at three more ways.


### Inject a Dependency

When something is hard to test, we can extract the part that is hard to test and inject it.
This way, we can substitute the extracted part with a double in our tests, so that we don't have to worry about any disc operations.

In the following code example the global `imagepng()` function is wrapped in a class called `GlobalFunctions`:

{% highlight php %}
<?php

namespace Images;

class GlobalFunctions
{
    public function imagepng($image, $path, $quality, $filters)
    {
        return imagepng($image, $path, $quality, $filters);
    }
}

class Writer
{
    public function write($filename, GlobalFunctions $globals = null)
    {
        if ($globals === null) {
            $globals = new GlobalFunctions();
        }

        $path = "/var/www/html/" . $filename;
        $image = imagecreate(100, 50);

        $globals->imagepng($image, $path, 9, 0);
    }
}
{% endhighlight %}

The `null` check is added to avoid having to change any existing clients of `Writer`.
As we've added a new optional argument to the method `write()`, the behaviour of the method shouldn't change for anyone using it.

With that in place, we can now create a spy in our tests to verify that the global `imagepng()` function has been invoked.
Technically, we're not verifying the invocation itself, but the invocation of _something_ that will do the right thing for us.

{% highlight php %}
<?php
namespace Images;

use PHPUnit\Framework\TestCase;
use Images\Writer;
use Images\GlobalFunctions;

class GlobalFunctionsSpy extends GlobalFunctions
{
    public $imagepngHasBeenCalled = false;

    public function imagepng($image, $path, $quality, $filters)
    {
        $this->imagepngHasBeenCalled = true;
    }
}

class WriterTest extends TestCase
{
    public function testWritesAnImage()
    {
        $globals = new GlobalFunctionsSpy();
        $writer = new Writer();

        $writer->write("filename", $globals);

        $this->assertTrue($globals->imagepngHasBeenCalled);
    }
}

{% endhighlight %}

There are no tests for the class `GlobalFunctions` itself.
The reason for that is that it acts as a plain delegator to the global function it wraps.
As long as we're careful enough calling the correct method and not adding any typos to the parameters, it is usually safe enough for us to assume it will work.

One drawback of this approach is that it's not always easy or even possible for us to change the public API of the class under test.
But injecting a dependency to ease testing is not the only option we have.


### Use a Test-Specific Subclass

Another way to go about it is to use a [Test-Specific Subclass](http://xunitpatterns.com/Test-Specific%20Subclass.html).
For that, we're going to introduce a seam with an untested, but relatively safe, refactoring.
In essence, it looks like this:

{% highlight diff %}
- imagepng($image, $path, 9, 0);
+ $this->imagepng($image, $path, 9, 0);
{% endhighlight %}

Here, the global function is getting wrapped with a protected method that delegates all its arguments to the global function.

{% highlight php %}
<?php
namespace Images;

class Writer
{
    public function write($filename)
    {
        $path = "/var/www/html/" . $filename;
        $image = imagecreate(100, 50);

        $this->imagepng($image, $path, 9, 0);
    }

    protected function imagepng($image, $path, $quality, $filters)
    {
        return imagepng($image, $path, $quality, $filters);
    }
}
{% endhighlight %}

With this seam in place, we can now subclass `Writer` to verify the correct usage of `imagepng()` by spying through the protected method.

{% highlight php %}
<?php
namespace Images;

use Images\Writer;

class TestableWriter extends Writer
{
    public $writtenImage;
    public $writtenImagePath;

    protected function imagepng($image, $path, $quality, $filters)
    {
        $this->writtenImage = $image;
        $this->writtenImagePath = $path;
    }
}
{% endhighlight %}

The unit test for that can look like this:

{% highlight php %}
<?php
namespace Images;

use PHPUnit\Framework\TestCase;

class WriterTest extends TestCase
{
    public function testWritesAnImage()
    {
        $writer = new TestableWriter();

        $writer->write("filename");

        $this->assertNotNull($writer->writtenImage);
        $this->assertEquals("/var/www/html/filename", $writer->writtenImagePath);
    }
}
{% endhighlight %}

If we want to be thorough, we can also spy on the `$quality` and `$filters` arguments, which have been left out here for brevity reasons.

One notable difference is that we're not verifying the class under test directly.
Instead, it is tested indirectly through the subclass we just created.
Even though this approach blurs the line between production and test code, it offers a viable alternative to injecting a dependency.


### Use Language Specific Features

Sometimes the language we use provides features that help us tackle problems from a completely different angle.
In this case, we can use PHP's namespaces to shadow the global `imagepng()` function from within the test file.

{% highlight php %}
<?php
namespace Images;

use PHPUnit\Framework\TestCase;

$USED_IMAGEPNG_PATH = "";

function imagepng($image, $path, $quality, $filters)
{
    global $USED_IMAGEPNG_PATH;

    $USED_IMAGEPNG_PATH = $path;
}

class WriterTest extends TestCase
{
    public function setUp()
    {
        global $USED_IMAGEPNG_PATH;

        $USED_IMAGEPNG_PATH = "";
    }

    public function testWritesAnImage()
    {
        $writer = new Writer();
        $writer->write("filename");

        $this->assertEquals("/var/www/html/filename", $this->writtenImagePath());
    }

    private function writtenImagePath()
    {
        global $USED_IMAGEPNG_PATH;

        return $USED_IMAGEPNG_PATH;
    }
}
{% endhighlight %}

While this uses global variables for testing, it enables the verification of the production code without having to modify it at all.
Which adds an interesting characteristic to this technique the other two didn't provide.
The fewer modifications we need to do without tests as a safety net, the better.

When dealing with legacy code, we often have to put aside our stylistic and idealistic views on unit tests.
It's not an excuse to abandon them completely, but they do not always apply or help in a legacy codebase.
We need to keep a pragmatic view on the goal to achieve.

It's helpful to have a repertoire of techniques available.
As we've seen in this blog post, we can use dependency injection to isolate a _hard to test_ dependency from the code that is easier to test.
Sometimes, it already helps to inject a string to aid our testing.

If we can't change the public API of a class, we might be able introduce seams to support testability.
More information about seams can also be found in [Working Effectively With Legacy Code](http://www.informit.com/articles/article.aspx?p=359417&seqNum=3), as well as [Mike Knepper's post about Framework Seams](https://8thlight.com/blog/mike-knepper/2017/09/20/framework-seams.html).

Language-specific features can and should be considered, too.
There are &ldquo;standard&rdquo; ways to solve a problem that can be found in literature, but we don't have to follow them to the letter.
Implementing and using the ideas is more important than which syntax we use.

