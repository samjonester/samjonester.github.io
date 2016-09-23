---
layout: post
title: "Cucumber: Don't Get Pickled"
description: "This post will be lessons learned from using cucumber on a very large, very complicated application. The things I talk about will be either things I’ve seen done well, or mistakes I wish had been avoided. Either way, they were valuable lessons I’d like to share. Let’s learn how to keep our cucumbers from pickling!"
date: 2016-09-23
tags:
  - testing
  - cucumber
  - tdd
comments: false
share: true
---

## Lessons Learned with Cucumber
This post will be lessons learned from using cucumber on a very large, very complicated application. The things I talk about will be either things I’ve seen done well, or mistakes I wish had been avoided. Either way, they were valuable lessons I’d like to share. Let’s learn how to keep our cucumbers from pickling!

## Cucumber’s Purpose
Cucumber is a collaboration tool that has the handy ability to execute code. Being a collaboration tool, it’s main purpose is to help developers work together with product owners to define user stories. The handy ability to execute code comes into play after the story has been defined. A developer can then test their application through the UI, through API’s, or even by directly executing production code. Each of these methods has their value, and I’ll be describing them in detail. Each also has their drawbacks, and I’ll be discussing the appropriate time to use each type of test.


## Start with Collaboration
Don’t forget about collaboration when using cucumber. The most powerful way to communicate functional intent is to define it together. Your cucumber can quickly become pickled if only one person/team is writing them.

When features are handed off from product owners to developers, the context around each specific scenario in each large feature is lost, and with it, gaps in knowledge form. These gaps can either be filled by having a discussion after the handoff, or by making assumptions and developing the wrong thing. Obviously you won’t do the second, but what’s so bad about asking questions later? If you’re going to have the conversation, why not just have it up front? If you have the discussion up front, the context around why the feature is needed, and how it is to behave can be **completely** shared by both the product owner and the developer. If you work together upfront, then neither party is hemmed in by constraints they don’t fully understand, and there’s no chance of forgetting why a features was defined in a specific way!

Harness the **REAL** power of cucumber and collaborate!

## Be Declarative
Long, imperative features pickle even the most collaborative cucumbers. If you have a Background with many steps, or givens to setup a testing situation, that’s briney. You should aim to define the necessary testing state with the smallest number steps possible. If you have to execute multiple when steps to perform an action, that’s briney. If your feature assumes a specific button in a specific place on a specific page needs to be clicked in a specific order, then your test suite will be fragile, and the weight of the functionality being defined is lost in imperative steps.

Be declarative about what you want. Only define things that are actually important to the user story you’re testing. If the specific username you’re logging in with isn’t important to what you’re testing, don’t include it. Hell, don’t even log in to begin with! Say something like “Given an authenticated user” instead. Even that may be too much! Only arrange an authenticated user if the functionality you’re defining has a different outcome between an authenticated user and a non authenticated user, making the authenticated user important input to the functionality.

Being declarative helps make it painfully obvious what’s being defined, and what is important to the feature. You’ll appreciate this when you look at the feature in 6 months and won’t need to scratch your head about whether that step is really necessary, and what it’s purpose is. You’ll also appreciate the fact that you won’t need to update an insane amount of features when something like authentication changes in the future.

_**Say exactly what you mean and only what you need.**_

## Avoid Over-Testing Like Vinegar and Dill
Your cucumber tests aren’t unit tests, so don’t treat them like they are! Your cucumber will become pickled if you test too much. I’ve had great success with high level user story type cucumber features, and I’ve had many problems with low level cucumber features that have endless scenarios covering every use case, edge case, and sky falling down case.

In my opinion, features should be used to define a happy path style user story. The tests backing the feature, then have the purpose of an integration test. That means that they test that each piece of your user story is integrated properly. The value in this type of test is to ensure that each layer collaborates with next in the way we expect.

I’m going to assume that you are test driving each component of the user story, covering each branch of logic at a micro level. Duplicating these micro tests at a feature level adds unnecessary, fragile, and unmaintainable coverage. It creates many annoying layers of tests to keep track of as your application evolves. Your test suite will become slow and break often. *You __WILL__ become a slave to your test suite, if you over test in cucumber.*

## Diligently Decide the Level of Integration for Every Feature
Will this test be a UI test, an API test, an integration test, or even a component test? Let’s define each layer appropriately so that we can examine their strengths and weaknesses. Misusing or overusing different levels will pickle even the best defined cucumber!

1. **UI Tests** – UI tests will cover your entire application. They interact with the application from the outside in. They click buttons, fill in forms, and examine data in the UI (but in a declarative manner!). This type of test is accomplished with a tool like capybara + selenium. These tests make up the tip of the testing pyramid. UI tests should be used mainly for high level journey tests. They can also be used to test complicated UI interactions.
  - **Pros:**
    - Cover much more of your application code than any other type of test.
    - Most closely resemble user interaction with your application.
    - Truly allow outside in testing.
  - **Cons:**
    - Much slower than any other type of test because server and client frameworks are both involved, including the client side testing framework used to drive UI interactions.
    - Very fragile, as any change in any layer under test has the potential to break this type of test.
    - Tests from the outside using a tool like selenium can easily become flaky, and failures can easily become ignorable noise due to false negatives.
    - The fact that your are testing the entire application stack has the potential to demotivate test driving difficult layers in the stack. This is especially true for hard to test UI code. Its very easy to fall into a trap of writing untested code, knowing that you’ll cover it with selenium, and *untested code becomes unmaintainable code*.
    - Application boundaries will typically not be mocked, meaning a live “test” database or live “test” environment for external API calls is almost always necessary.
2. **API Tests** – API tests will cover each interaction with the application as a stateless API call. Like UI tests, they interact with the application from the outside in. This type of test is typically accomplished with an HTTP request library like Rest Client. These tests are very close to the tip of the testing pyramid. API tests should be used mainly for high level feature testing. API tests are good for testing high level happy path user stories defined by our declarative features.
  - **Pros:**
    - Cover integration of all server side application code.
    - API calls can be defined to match user interactions from user stories.
    - Allows for outside in testing of server code.
    - Faster than UI tests.
    - Less fragile than UI tests.
  - **Cons:**
    - Slower than all testing types except UI tests because the entire server side framework is involved.
    - Does not test client side code.
    - It’s hard to mock application boundaries with outside in API tests, so you will generally be relying on a live “test” database, or a live “test” environment for external API calls.
    - Integration Tests – Integration tests will execute production code and test the integration between all components within a feature. These tests are generally smaller and more specific than user stories, but still large enough to define an internal API. I’ve had success using this type of testing to drive development of large units of development. Integration tests should make up the bulk of your cucumber features, falling in middle of the testing pyramid. They should should be used much more sparingly than unit/component tests in a testing framework like RSpec or MiniTest. Following top down TDD, these types of tests will allow you to keep your unit tests green, while having a red integration test to drive development. They should be leveraged to test component integration over the happy path. This type of test can also be used sparingly to test non-happy path, and complicated use cases that should be collaborated on between developer and product owner.
  - **Pros:**
    - Useful to drive development of large units of development, where it’s hard to keep all pieces assembled in your head.
    - Internal API can be formed from the outside in with a YAGNI approach.
    - Allows for collaboration at a smaller level than user stories.
    - Much less fragile than API and UI tests.
    - Much faster, since the framework is not involved and production code is directly called.
    - Application boundaries are generally easier to mock or stub, since production code is directly called.
  - **Cons:**
    - Application boundaries can become hard to mock or stub depending on the number of external dependencies, meaning that reliance on a live “test” database, or a live “test” environment for external API calls could become necessary.
    - Most of the framework is typically eliminated in this type of test, meaning that there is no coverage of the integrations between your code and framework code.
    - Component Tests – Component tests act as unit tests written as cucumber features. They are usually a wrapper around one or a small number of components that also have unit coverage. Because of the size, there isn’t much benefit to this type of testing in most situations. This means that they should be used very sparingly to prevent unnecessary test maintenance. I’ll go into more detail about successful strategies for using this type of cucumber test below in “Domain Specific Exceptions.”
  - **Pros:**
    - Allow collaboration on complicated domains.
    - Allow a human readable unit testing framework for product owners.
  - **Cons:**
    - Essentially becomes duplicated coverage of use cases defined in unit tests.
    - Because of their small size, do not provide much value in testing integration between components.

## Drive Development with Cucumber
We’ve collaborated with the product owner to define our feature, so we understand it well. We’ve got our short, declarative feature that will be maintainable and easy to understand in the future. We’re sure that the value we’ll receive from our tests will be in the integration between the complicated moving pieces of our application. We’ve also decided which level of integration is appropriate for testing our feature. *Now What?*

**Run your feature!**

Let the feature tell you what to develop first. Don’t start developing, guessing what you’ll need along the way. Don’t write glue code backing your feature last. Writing glue code last is sure to pickle your cucumber. *Drive development with the feature!* Try it and you’ll feel the real power in cucumber. Coupled with top down TDD, driving your development with your feature is fun! Your feature will tell you what to develop next, then you’ll write a test defining interactions with collaborators, or component logic. Then you’ll code it and everything will turn green!

You can follow red + green + refactor with your unit test suite, while having a red integration test guiding you through development. You can also be ensured that your components will integrate properly, since you’ll be testing along the way. Your external and internal API’s will be developed naturally from the top down, and you’ll be more motivated to keep components small. You’ll never write code you don’t need, and refactoring won’t be as necessary as your patterns will emerge from your desired interactions with components.

## Domain Specific Exemptions
Sometimes cucumber can become a powerful tool to help define and test very small pieces of functionality. I’ve seen this be used very successfully when the feature describes functionality within complicated domain, like one that requires a PHD to understand. This is the only time that I would agree with defining all use cases, and even allowing edge cases into my features. Basically this is treating a feature like large unit test. Let me explain why, and maybe you’ll agree with me.

Cucumber is being used as a collaboration tool in this situation. Since the domain is so complicated, it helps to be able to use english to describe it. It also acts as a bridge (aka collaboration enhancer) between the developer and the expert in the domain.

The domain expert can see and verify every input into and output from the component. This allows more confidence that the developer has developed what they should have. It also means that the expert can define as many crazy edge cases, or very important regression tests as needed with little technical expertise.

The only downside here is duplicated coverage, but since this is such a complicated area of code, I’m not convinced that this con outweighs the other pros.

## Summary
Cucumber is a fantastic collaboration tool, so work together to define features. Features are best at defining high level user stories in a declarative manner, so say what you mean, and only what you need. Always be conscious about the level of integration needed for each test. The higher the level of integration, the more fragile and cumbersome maintenance becomes, but the more fully your application is covered by the test. The lower the level of integration, the more your features become duplicated unit tests, but the more resilient your tests are for future code changes. Finally, be sure to drive development with your feature to embrace the true power of the cucumber collaboration framework!
