---
layout: default
title: API Assumption Tests
external-url: https://8thlight.com/blog/christoph-gockel/2017/06/14/api-assumption-tests.html
---

# API Assumption Tests

In any non-trivial codebase we rely on third-party dependencies.
These dependencies can be libraries or frameworks, but can also be external services that provide an API for us to use.
Relying on those APIs makes our code subject to breakage&mdash;even though our application code might not have changed.


## Assumptions

The moment we decide to use an API, we agree on the contract that API provides _today_.
The emphasis is on &ldquo;today&rdquo; because it's more or less out of our control whether the API keeps that contract or not.
Here we need to assume that API is working as its documentation said it would.

In an ideal world API providers will give their clients a heads up before any (breaking) changes get introduced.
Sometimes they have a good versioning strategy that enables clients to upgrade independently from the API itself.

Unfortunately we don't always work with APIs like that.
No matter what kind we work with, we can guard ourselves against these unforeseen changes.

In this post, we are going to discuss two techniques we've used successfully in previous projects:

1. Automatically verify assumptions of an API.
2. Use those assumptions to provide a strict conversion into domain objects.


## Verifying Assumptions

The technique described here is using a JavaScript testing framework like Jasmine or Mocha, but the ideas are language and framework agnostic.

Our unit-test suite automatically ran all files with a `.spec.js` extension.
We've created these API tests with files that ended with `.api-spec.js`.
This allowed us to separate the longer-running API tests from the unit tests.

We ran the API tests every now and then on our local machine to make sure everything was still as expected before creating a pull request, for example.
They were also enabled on our continuous integration server.

In the tests themselves, we issue one or more HTTP requests to the API we want to verify, take the response, and check that it still contains all the fields we expect.
This is not a semantic verification that the fields contain specific values, but rather a structural verification.

With test runners that have a documentation style output, these tests also read nice (here for a fictitious product search service):

{% highlight shell %}
$ mocha
  Product Search Service
    returns a list of products
    a product
      has property id
      has property description
      has property price
      has property image_url
{% endhighlight %}

To reduce the amount of code needed, we can leverage the power of JavaScript and iterate over the list of expected properties.
For each property we then call the test framework's `it` function.

{% highlight javascript %}
describe("Product Search Service", () => {
  let response, product

  before(() => {
    response = requestProducts()    // issues an HTTP request
    product  = response.products[0]
  })

  it("returns a list of products", () => {
    expect(response.products).to.be.an("array")
  })

  describe("a product", () => {
    [
      "id",
      "description",
      "price",
      "image_url"
    ].forEach((name) => {
      it(`has property ${name}`, () => {
        expect(product).to.have.property(name)
      })
    })
  })
})
{% endhighlight %}

As mentioned earlier these tests do not verify the actual data a service returns, they are only verifying the structure of it.

If, for example, the service adds a new field we don't expect yet, everything still works the same.
That would have been the case even without these tests, so we didn't gain much in that scenario.
The real benefit shows once the service removes or renames a field that was expected before.
Then the tests will fail and report with exactly the field name that is missing!

This at least gives us a way to know where something is not as expected.
It doesn't harden our code itself against a change like the one just described.
A previously expected field now all of a sudden is `undefined`, which can lead to our application crashing.

In order to keep having a working application in a scenario like that, we need something we called a &ldquo;conversion layer.&rdquo;


## Convert Service Responses into Domain Objects

Adding tests that verify the data we get from a service was the first step to add more resilience to our codebase.

One pattern we've seen in several codebases is that the structure of a service leaks into the core of an application.
For example, let's say the previously mentioned product service returns a product with the following structure:

{% highlight json %}
{
  "PRODUCT_ID": 9304390,
  "PRODUCT_DESC": "A description for an amazing product"
}
{% endhighlight %}

By simply issuing an HTTP request to that service and passing the data on to the next part in our application, it's possible that we see code like this to get all product IDs:

{% highlight javascript %}
const productIds = products.map((product) => product.PRODUCT_ID)
{% endhighlight %}

Or, every time we need the ID of a product, we get it as `product.PRODUCT_ID`.
That is not only redundant, but also violates common JavaScript naming conventions.

Another problem with an approach like this is that it ties our codebase directly to that one service.
It would be nicer if we could get an ID of a product by using `product.id`.

To help define a common language across our codebase, we added a conversion between the response of a service call and our code.
With this in place, we are in control of what a product looks like for us

That conversion can sometimes be as simple as copying the values from one object to another.
Other times, it gives more meaning to the fields:

{% highlight javascript %}
export function convertProduct(productFromService) {
  return {
    id:          productFromService.PRODUCT_ID,
    description: productFromService.PRODUCT_DESC,
    ...
  }
}
{% endhighlight %}

Whenever a service changes its structure, we can now focus the work needed to update our application to the `convertProduct()` function.
Even better: if we decide to use a different service, we can focus the effort of integrating that new service to this conversion function.

Now what happens if the service removes a field we expected before?
We can use this conversion layer to add sane defaults to properties:

{% highlight javascript %}
export function convertProduct(productFromService) {
  return {
    id:          contentOf(productFromService, "PRODUCT_ID"),
    description: contentOf(productFromService, "PRODUCT_DESC")
    ...
  }
}

function contentOf(object, property) {
  return object.hasOwnProperty(property) ? object[property] : ""
}
{% endhighlight %}

Sometimes an empty string instead of an `undefined` can make or break an application.
This way, we make sure that we at least display _nothing_ instead of having our application break because of an unforeseen `undefined` property.
Depending on the property we can also have differently typed `contentOf()` functions that return a default value of a specific type.
Like `integerContentOf()` would return `0` as a default, `booleanContentOf()` returns `false`, and so on and so forth.

Converting data from a third-party dependency is not bound to be used for external service calls only.
It can also be used to isolate our code from a library we use internally.
Switching to a new version or a completely different library becomes much easier with that, because we isolated the library's usage to the conversion layer.

