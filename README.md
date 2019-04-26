Burpless is a pun on Cucumber and Gherkin. It's a thinner-skinned cucumber
variety which is easier to digest.

# Purpose

It's a flexible and asynchronous alternative to Cucumber which addresses the
use case of running tests in parallel. It loosely supports the Gherkin syntax
with the [vinegar parser](https://github.com/jeromebelleman/vinegar). It makes
organising feature and definition files rather natural, too.

# Dependencies

Provided Node.js is installed, run:

```shell
npm i
```

# Running Burpless

There's a short online help:

```shell
node burpless -h
```

Running the command without arguments will cause it to search for `.feature`
and `.js` files recursively from the current directory on and attempt to run
the tests it finds there:

```shell
node burpless
```

Burpless also takes the `-f` and `-d` command-line options to respectively
specify one or more feature and step definition files, for instance:

```shell
node burpless -f examples/example.feature -d examples/example.js examples/example-async.js
```

Note that I chose here to haphazardly cast the files into an `examples/`
directory, but I didn't have to – Burpless just couldn't care less. There is no
need to match the names either. What matters is how the prose in the feature
file matches the code in the JavaScript step definition file. Step definitions
defined in files supplied last override those in files defined first.

# Writing Tests

In essence, feature (`.feature`) files are for natural language test
definitions, whereas step definition files (`.js`) are for their corresponding
JavaScript implementations. In what follows, suppose we want to use Selenium
for running browsers to load an example page, click on a link and check the
header is correct.

# Writing a Feature File

```gherkin
Feature: Example
  Scenario: I load the example.com homepage.
    Given I start 2 browsers
    When I set the URL to "http://example.org"
    And I set the link to click to "More information..."
    Then the header should read "IANA-managed Reserved Domains".
 
# vim: wrap linebreak breakindent
```

Burpless isn't interested in the `Feature` and `Scenario` keywords, which are
only for you to organise the tests to your liking. It handles `Given` and
`When` keywords identically but in order. The `Then` step is treated a bit more
specially in that it will collect assertion errors if any, or otherwise
consider the tests as successful is there weren't any. As per the Gherkin
syntax tradition, I singled out here parameters with double-quotes for strings
and without for numerals. But that's really entirely up to you as you get to
write your own regular expressions in the step definition files.  Lines
starting with a `#` act as comments and are ignored. (Top tip, Vim users:
combined with the `wrap` and `linebreak` options, the `breakindent` option
works rather wonderfully to maintain your indents with very long steps and keep
the lot readable.) Scenario outlines (i.e. repeating scenarios with
examples) are supported, too.

# Writing a Step Definition File

```javascript
const assert = require('assert')
const { Builder, By } = require('selenium-webdriver')

function startBrowsers (world, scene, number) {
  world.drvs = []
  for (let i = 0; i < number; i++) {
    world.drvs.push(new Builder().forBrowser('world').build())
  }
}

function setUrl (world, scene, url) {
  world.url = url
}

function setLink (world, scene, link) {
  world.link = By.xpath(`//a[text()="${link}"]`)
}

async function headerShouldBe (world, scene, header) {
  for (const drv of world.drvs) {
    await drv.get(world.url)
    await drv.findElement(world.link).click()
    assert.strictEqual(await drv.findElement(By.css('h1')).getText(), header)
  }
}

exports.lib = {
  'I start (\\d+) browsers': startBrowsers,
  'I set the URL to "(.+)"': setUrl,
  'I set the link to click to "(.+)"': setLink,
  'the header should read "(.+)"': headerShouldBe
}
```

Jump straight to the last block where the `lib` object is exported. It maps
each step regular expression to a step definition. The JavaScript step
definitions precede it: the `startBrowsers` function simply starts browsers,
the `setUrl` and `setLink` functions respectively set a URL to test and a link
to click on (which shows how Burpless treats `Given` and `When` identically)
and `headerShouldBe` does a bit more work related to making sure information on
the page is what we expect it to be.

Note how the last regular expression pattern in the `lib` object didn't include
a trailing full stop even though it does in the feature file. That's quite
alright, Burpless attempts to find a matching sub-string in the step, rather
than attempting to match the whole sentence. Use `^` and `$` if that's
inconvenient, e.g. if you need more stringent matching.  Crucially, Burpless
expects parenthesised matches which it maps to the parameters. For instance,
`startBrowsers` has one parenthesised match for the number of browsers which is
mapped to the `number` parameter of the `startBrowsers` function.

# From the Scene to the World

The `world` object is some sort of global variable Burpless makes available
throughout its run-time to conveniently share data everywhere. It is passed as
first parameter of each function, right before any other parameters which may
be mapped from parenthesised matches. In this example, it's used for keeping
track of browsers (*drivers*), the URL to be tested and the link to click on.
The `scene` object is to a scenario (scene/scenario, see?) what the `world`
object is to the whole Burpless run-time. It is for using in more restrictive
scopes, when it is inconvenient to store objects in `world`, objects which can
for instance only be referred to from context.

# Asynchronicity

If you run the test as implemented above, you will notice that the 2 browsers
will each browse the pages in succession, not so much in parallel. This is due
of course to the `get()`, `click()` and `getText()` functions which are made to
run synchronously with `await`. This is necessary to ensure that you interact
with elements whose pages have finished loading. Burpless was designed with
asynchronicity in mind and it would therefore be sensible to rewrite this
function for the 2 browsers to do their browsing simultaneously.

## Making a Test Asynchronous

How to reorganise code to run things in parallel depends of course on the
original algorithm. In this particular one, instead of using a `for` loop, it
was as simple as turning the `get()-findElement()-getText()` block which was
synchronous (because of the `await` operator) into an asynchronous callback
function passed to `map()`, whose each promise we then `await`:

```javascript
const assert = require('assert')
const { By } = require('selenium-webdriver')

async function headerShouldBe (world, scene, header) {
  await Promise.all(world.drvs.map(async drv => {
    await drv.get(world.url)
    await drv.findElement(world.link).click()
    assert.strictEqual(await drv.findElement(By.css('h1')).getText(), header)
  }))
}

exports.lib = {
  'the header should read "(.+)"': headerShouldBe
}
```

Note how this snippet of code could easily live in a file of its own which,
when passed as the second `-d` argument to Burpless, overrides just the
`headerShouldBe()` step definition of the first file.

## Synchronising Asynchronous Functions

In the example, `startBrowsers()`, `setUrl()` and `setLink()` are synchronous
and only `headerShouldBe()` is asynchronous. This, as far as synchronicity is
concerned, is straightforward since their execution order is guaranteed to be:

1. `startBrowsers()`
1. `setUrl()`
1. `setLink()`
1. `headerShouldBe()`

That's because all the code in synchronous functions is executed in the order
they're called, whereas in asynchronous functions it's called upon the next
even loop – right at the end, in this case. Suppose now that you have several
asynchronous functions, for instance because they need to await asynchronous
functions themselves. And suppose you need to nonetheless ensure they are run
in a given sequence. Burpless keeps track in the `scene.functions` object of
all the promises which asynchronous functions return in a scenario. As such,
it's easy for a given function to wait for another function to complete before
starting its payload:

```javascript
async function foo (world, scene) {
  await somethingAsynchronous()
}
 
async function bar (world, scene) {
  await scene.functions.foo
  await somethingAsynchronous()
}
 
async function baz (world, scene) {
  await scene.functions.bar
  await somethingAsynchronous()
}
 
exports.lib = {
  'Foo': foo,
  'Bar': bar,
  'Baz': baz
}
```

## Assertion Errors and Asynchronicity

One last considerable advantage of asynchronicity, is that Burpless prints out
a report for each and every assertion errors that a given function may throw
– not just the first one.

## Results from Synchronous and Asynchronous Functions

Burpless runs each step definition value in the order dictated by the feature
files, collects their return values, but doesn't expect them to bear any
particular meaning. Throw `AssertionError`s or other exceptions if you want to
report problems. If the function is asynchronous, it will collect the promise
it returns without further ado. If the function is synchronous and completes,
it will collect `true`. If it is synchronous and throws an exception halfway,
Burpless will catch it, print it out and collect `false`.

Near the end of its run, Burpless runs through the results to report on the
success rate. It ignores `false` values, adds each `true` value to the number of
successes, and awaits promises. If a promise resolves, this adds to the number
of successes, too. If it throws an exception, it will print out its failure.

# Teardowns

They are functions which are meant for tidying up after running the steps,
e.g. for quitting any browsers. They run after all tests completed, including
those running from asynchronous step definition functions. Each step definition
file is expected to have at most one `teardown()` function exported as
`teardown`. It will have access to the `world` variable as first parameter (not
the `scene`, however, for historical reasons, but I guess I could change that
if needs be).  There is no reference to the `teardown()` function in the
feature file. In our example, next to the other step definition functions, we
could have:

```javascript
exports.teardown = function (world) {
  world.drvs.map(drv => drv.quit())
}
```

# The Background

This is the most significant departure from the traditional
Cucumber/Gherkin implementation: a `Background` is normally a `Given` that's
repeated before each scenario. However, Burpless treats a `Background` as a
`Given` that's executed only once, before anything else. It can therefore even
be used as a configuration file. The feature/step definition files parity
remains and the code is written much the same way. Here's an example of how to
set the browser instead of hard-coding it as was previously done:

```gherkin
Feature: Set browser
  Background:
    Given I set the browser to "chrome"

# vim: wrap linebreak breakindent
```

These are the corresponding step definitions in JavaScript – one notable
difference is the `teardown()` function which is expected to be exported
`bgTeardown`:

```gherkin
function setBrowser (world, browser) {
  world.browser = browser
}

exports.bgTeardown = function (world) {
  world.drvs.map(drv => drv.quit())
}

exports.lib = {
  'I set the browser to "(.+)"': setBrowser
}
```

Background teardown functions are run after regular scenario teardown
functions. Again, the file names and locations for step definition and feature
files aren't relevant, here. The regular expressions in the `exports.lib`
object are. Note how the `world` variable becomes useful here, to share
information with other scenarios.

# References

- [Gherkin Syntax](https://docs.cucumber.io/gherkin)
- [Cucumber Guide](https://docs.cucumber.io/guides), bearing in mind that
  Burpless behaves a bit differently. But it's useful to get familiar with the
  terminology (e.g. background, steps, scenario, etc.)
- [Vinegar](https://github.com/jeromebelleman/vinegar)
- JavaScript [Regular
  Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)
  discussed in MDN
