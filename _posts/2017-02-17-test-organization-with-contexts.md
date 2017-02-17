---
layout: post
title: "Test Organization With Contexts"
description: "When testing we create a test file for each class. This practice recognizes that classes group common behavior and ideas.  We can extend this idea further by using test contexts around the natural shared contexts of a subject under test."
date: 2017-02-17
tags:
 - testing
 - code-design
 - Java
 - C#
comments: false
share: true
---

Many of my favorite languages have testing libraries that support specs. Specs allow us to define testing contexts that help isolate and focus our unit tests. Many of my favorite languages have [testing libraries that support specs][spec-frameworks-list]. Two distinct standouts that do not support contexts are C# and Java; luckily, their language features still allow the same goals to be achieved. Before we look at how to use them, let's talk about why contexts are awesome!

## Defining Contexts For Test Separation
It’s beneficial to separate the setup and the “context” around a method call within a class. This allows a single test or set of tests to use similar setup or default values. The setup will be focused on one entry into the method under test, and extraneous noise can be eliminated.

Separating the contexts allows the scope of setup for one method or set of tests to be isolated from another. This is beneficial as tests change over time. Tests always change, and this technique will help prevent unintended changes to affect the outcome of other test contexts.


## Goals
Let’s clarify the goals that we would like to accomplish by separating test contexts.

### Focus
Most importantly we want to improve the focus of each of our tests. We want a single test to arrange only the setup that is unique, and therefore interesting to that specific test case.

### Isolation
We want to ensure that the setup for one test doesn’t accidentally affect another. We also want a way to group common setup between related tests without polluting the scope of unrelated tests.

### Naming Conventions
We want our naming conventions to help identify the level of abstraction for each context within our tests. We want the top level to describe the class under test, the next level to describe the method under test, subsequent levels to describe setup as it becomes refined, and finally the test to describe it’s unique inputs and outputs.

## An Example
These benefits are a little abstract, so let’s analyze them with an example. We have a class `FancyClass` with two related public methods: `ExecuteOnReversedResults` and `SortFilteredResults`. Both methods need to be tested, and it makes sense to group them together into a single test class. Here’s an example of what that class may look like. This simplistic class let’s us focus on how we would break up tests contexts with more focus. This example, and the tests defined for it are available in an [example-repo][example-repo].

``` c-sharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace Fancy
{
    public class FancyClass
    {
        private readonly IQuery querier;
        private readonly ICommand commander;

        public FancyClass(IQuery querier, ICommand commander)
        {
            this.querier = querier;
            this.commander = commander;
        }

        public void ExecuteOnReversedResults(string param)
        {
            var results = this.querier.Find(param).ToList();
            if (results.Any())
            {
                this.commander.Execute(results.OrderByDescending(v => v));
            }

        }

        public IEnumerable<string> SortFilteredResults(string param, Func<string, bool> predicate)
        {
            return this.querier.Find(param).Where(predicate).OrderBy(v => v);
        }
    }
}
```

This is a fairly common pattern. The class is defined with a couple collaborators, and has related public methods that interact with those collaborators. When we’re testing a class like this, it can be helpful to follow a script when testing a class like this:

1. Arrange
  - Instantiate mocks for our `subject`’s dependencies..
  - Instantiate the `subject` with those dependencies.
  - Setup test data for a specific test.
  - Setup stubs for query dependencies.
1. Act
  - Call the `subject` with our test data.
1. Assert
  - Verify that results are what we expect.
  - Verify that command objects are invoked as we expect.

### Problem
The problem that we face for this class is one of setup data. A test for `ExecuteOnReversedResults` than a test for `SortFilteredResults`. Therefore, ti should have different test setup.

Unless we define separate test contexts for each method under test, we only have two options

1. Each test method needs to be long to segregate the setup. This means that our test method contains unnecessary noise and duplication.
1. Values needed for only one test have to pollute the scope of other tests.

## Creating Contexts

The following code showcases the technique used to create test contexts; notice how it refines the test as we move in. It isolates one context from another and uses names to describe more specific levels within the tests.


``` c-sharp
using Xunit;

namespace FancyTest
{
    public class FancyClassFixture
    {

        public class ExecuteOnReversedResults : FancyClassFixture
        {

            public class WhenResultsFound : ExecuteOnSortedResults
            {

                [Fact]
                public void Executes_WithResults_SortedReverseAlphabetically()
                {
                    // Test
                }
            }

            public class WhenNoResultsFound : ExecuteOnSortedResults
            {

                [Fact]
                public void SkipsExecution()
                {
                    // Test
                }
            }
        }

        public class SortFilteredResults : FancyClassFixture
        {

            [Fact]
            public void ReturnsResults_Filtered_ReverseAlphabetized()
            {
                // Test
            }
        }

    }
}
```

Running our tests creates the following descriptions:

<img src="/assets/images/test-contexts/test-setup-with-contexts.png">

That’s pretty cool right! As we refine our tests, our names reflect the current context and it reads really well. It’s super easy to understand what’s being tested for a given context. Tests can be further nested to group shared setup as desired.

## Isolating and Refining Setup
We’ve named our test contexts to describe the way that we’re refining our tests. Now it’s time to create the shared setup for this class.

1. **General test fixture context** - We’ll start by defining the setup that is shared across contexts. This is the instantiation of our mocked dependencies and our `subject`.
2. **Method context** - In the context for the method `ExecuteOnReversedResults`, we will define the test input used during our `Act` phase.
3. **Specific context** - In the context for `WhenResultsFound`, we will define values to be used for our stubbing and verifications. We can use names that represent business requirements here, since they will not pollute the scope of other test contexts!
4. **The test** - Finally, in our test `Executes_WithResults_SortedReverseAlphabetically`, we will arrange act and assert the values that make this specific test unique and interesting.

 This test is available in an [example-repo][example-repo].

``` c-sharp
using System.Collections.Generic;
using Xunit;
using Moq;
using Fancy;

namespace FancyTest
{
    public class FancyClassFixture
    {
        private readonly FancyClass subject;
        private readonly IQuery querier;
        private readonly ICommand commander;

        /* 1. General test fixture context
         * ---------------------------
         * Instantiate mocked dependencies, the subject, and any uninteresting default data here.
         */
        public FancyClassFixture()
        {
            this.querier = Mock.Of<IQuery>();
            this.commander = Mock.Of<ICommand>();
            this.subject = new FancyClass(this.querier, this.commander);
        }

        /* 2. Method context
         * ---------------------------
         * Instantiate test input, stubs, and/or values to aide in verification here.
         * All values will be shared within all lower, more specific contexts.
         */
        public class ExecuteOnReversedResults : FancyClassFixture
        {
            private readonly string param;

            public ExecuteOnSortedResults()
            {
                this.param = "search-text";
            }

            /* 3. Specific context
             * ---------------------------
             * Refined test input, stubs, and/or values to aide in verification can be defined here.
             * This section is not always necessary. Alternatively, a couple layers of specific context may be necessary.
             */
            public class WhenResultsFound : ExecuteOnSortedResults
            {
                private readonly List<string> unsortedResults;
                private readonly List<string> sortedResults;

                public WhenResultsFound()
                {
                    this.unsortedResults = new List<string> { "abc", "xyz", "hij" };
                    this.sortedResults = new List<string> { "xyz", "hij", "abc" };
                }

                /* 4. The test
                 * --------------------------
                 * Only setup values and stubs that are unique and interesting to the specific test here.
                 */
                [Fact]
                public void Executes_WithResults_SortedReverseAlphabetically()
                {
                    Mock.Get(this.querier).Setup(q => q.Find(this.param)).Returns(this.unsortedResults);

                    this.subject.ExecuteOnReversedResults(this.param);

                    Mock.Get(this.commander).Verify(c => c.Execute(this.sortedResults));

                }

            }

            public class WhenNoResultsFound : ExecuteOnSortedResults
            {
                // ...
            }
        }

        public class SortFilteredResults : FancyClassFixture
        {
            // ...
        }
    }
}
```

## Summary

By defining contexts around this test, we allow the test to focus on the things that control the flow through this method. We aren’t distracted by noise, making it easy to see what is in charge of the logical branching through the method. We're able to use the Arrange/Act/Assert pattern for the test with a lot of clarity around what’s being tested. The contexts also work as great documentation to show how this method functions.

## Additional Notes

This technique works in C# and Java by creating a nested subclass to the parent context. Inheritance rules ensure that parent constructors are invoked before the current context's constructor. This allows a context to further refine what was defined by a parent. See examples [here][example-repo].

**Details in C#**

Be sure to declare the nested subclass as `public`, and each constructor as `public`. C#'s inheritance rules allow `private` fields to be accessed by nested classes.

**Details in Java**

Be sure to declare the nested subclass as `public static` and each constructor as `public`. Java's inheritance rules allow `protected` fields to be accessed by subclasses. The constructor acts similarly a typical `@Before` method in JUnit while allowing the inheritance rules to work as described above.

### Spec Frameworks

These language's testing frameworks support test contexts. If you're already using one of these languages or testing frameworks, I suggest you try out the technique described above! This list is not extensive, so please don't be offended if your favorite is missing :)

**Ruby**: [RSpec](http://rspec.info/), [minitest](https://github.com/seattlerb/minitest#specs) (Yes you read right).
**JavaScript**: [Jest](https://facebook.github.io/jest/), [Mocha](http://mochajs.org/).
**Elm**: [elm-test](http://package.elm-lang.org/packages/elm-community/elm-test/latest).
**Python**: [Mamba](http://nestorsalceda.github.io/mamba/).
**Haskell**: [Hspec](http://hspec.github.io/).

## Additional Reading
- [Arrange Act Assert - C2 Wiki](http://wiki.c2.com/?ArrangeActAssert)
- [Arrange Act Assert - Test Double Wiki](https://github.com/testdouble/contributing-tests/wiki/Arrange-Act-Assert)
- [Mock Objects in Discovery Tests](http://blog.testdouble.com/posts/2014-05-14-mock-objects-in-discovery-tests.html)

[spec-frameworks-list]: #spec-frameworks
[example-repo]: https://github.com/samjonester/test-contexts-example
