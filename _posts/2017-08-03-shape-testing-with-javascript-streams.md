---
layout: post
title: "Shape Testing with JavaScript Streams and Lodash FP"
description: "Leverage streams in node to test the shape of JSON results. Test against objects whose values change with functional programming!"
date: 2017-08-03
tags:
 - testing
 - code-design
 - JavaScript
 - function-programming
comments: false
share: true
---

## Shape test changing data

Have you ever written an [end-to-end test][test-pyramid] against a live API? I have, and it's hard. It's tough to do because the data returned from that live API is always changing. You can't rely on the values in your response because they change when real users actually use that thing you're testing against.

This example was created when I was writing a utility that requested information from a big data store, creating a set of JSON reports from the data. I wanted to write a test to ensure that my report was shaped correctly, aka, a **shape test**. The big data store was a clients internal API, so this **shape test** also protected me against changes to the API I'm integrating with.

## Testing with streams

Let's plan a test for a utility like I described above.

1. Execute `mvn install` (yes this utility happened to be a Java maven plugin) on an example project.
2. Recursively examine results directory.
3. Ensure that a valid JSON report exists.

Even though this utility was written in Java, node feels tailor made for steps 2 and 3. Asynchronously streaming the file system and working with JSON are things that node has specifically been designed to do well, so let's take advantage of that.

There are a number of streaming libraries to choose from (surprise!) because the NPM ecosystem emphasizes sharing and package creation. This is nice for improving the [streaming API in Node][node-streams]. I chose [Highland][highland-js] for managing my stream of data, though other notable libraries are [bacon][bacon-js], and [RxJS][rxjs]. I like the way that Highland builds on Node streams to create higher order actions like `map`, `filter`, `reduce`, etc. Let's use Highland to start converting our plan into a test. We're using the term test loosely below, as this really just logs the results, but this concept could easily be included within a test harness.

``` javascript
const stream = require('highland')
const remove = stream.wrapCallback(require('fs-extra').remove)
const exec = stream.wrapCallback(require('child_process').exec)
const validReports = require('./valid-reports')

const reportDirectory = 'my/stuff/here'
const success = filePath => console.log('Success! Valid file at', filePath)
const failure = () => console.error('Rats! No valid files :(') || process.exit(1)

// Start with a fresh output directory.
remove(reportDirectory)
  // Because we've wrapped exec in a stream we need to flatten it into our original stream.
  .flatMap(_ => exec('mvn install'))
  // Same idea for our actual validation, which we will implement next.
  .flatMap(_ => validReports(reportDirectory))
  // If the stream is empty, no valid reports were found. Fail!
  .otherwise(failure)
  // For each valid report, print a success message. Each will consume the stream.
  .each(report => success(report.filePath))
```

Now we've got the initial integration test in place and ready to validate our reports. Since we used `flatMap` to flatten the stream produced by `validReports` into our original stream, that function will need to return a stream of valid reports for a given directory. Let's build that out now!

## Finding valid reports

Now we need to recursively search a report directory for valid reports. We can use highland to stream the directory and perform the filtering.

``` javascript
const stream = require('highland')
const walk = require('klaw')
const readJson = stream.wrapCallback(require('fs-extra').readJson)
const endsWith = require('lodash/fp/endsWith')
const set = require('lodash/fp/set')

const validReport = require('./valid-report')

const read = filePath => readJson(filePath).map(set('filePath', filePath))

module.exports = function validReports(reportDirectory) {
  // Kick off the stream pipeline by walking the given directory,
  // converting the resulting node stream into a highland stream
  return stream(walk(reportDirectory))
    // Convert each file info object into the file path.
    .map(get('path'))
    // Filter to only files that have the .json extension.
    .filter(endsWith('.json'))
    // Read each json file.
    .flatMap(read)
    // Filter to only valid reports.
    .filter(validReport)
}
```

## Getting functional

The validation in the next step relies heavily on [Lodash FP][lodash-fp], so let's review some of the important concepts before diving in.

### Curried functions

A curred function is a function that takes some of its arguments now, and then waits for the rest to come later. For example, `map` accepts 2 arguments, first a transformation function, then objects to transform. Because it's curried, we can give `map` a transformation function and create a new function that's waiting for the rest, the objects to transform.

``` javascript
const map = require('lodash/fp/map')

const allAtOnce = map(n => n+1, [1, 2, 3])

// => new partially applied function waiting for data
const plusOne = map(n => n+1)
const waitedForData = plusOne([1, 2, 3])

// allAtOnce === waitedForData
```

### Composing functions

A composed function is just multiple functions glued together. The trick is to compose functions that have been partially applied, or are still waiting on their thing to operate on. The entire composition will then be a pipeline, waiting on the thing that the first function is also waiting on.

``` javascript
const compose = require('lodash/fp/compose')

const transformX = x => x+1
const transformY = y => y*2

// transformX first, then transformY
const traditionalFunc = x => transformY(transformX(x))


// same thing, but since transformX is waiting on x, so is the composition.
const composedTransformation = compose(transformY, transformX)
```

## The shape of a valid report.

The two functional programming concepts from above can both be used to perform the validation on a report. We're building on small units of functionality create more complex validations for a report. Let's start by defining the validations that we care about.

``` javascript
const allPass = require('lodash/fp/allPass')
const { hasFields, anyAt } = require('./validation-utils')

const reportFields = [
  'name',
  'id',
  'findings.violations'
]
const violationsPath = 'findings.violations'
const violationFields = [
  'id',
  'description',
  'helpUrl',
]


module.exports = function validReport(report) {
  return allPass([
      hasFields(reportFields),
      anyAt(violationsPath, hasFields(violationFields))
    ], report);
}
```

Something really cool has happened by writing our validations this way. To understand the purpose of `validReport` we can just read the function names, we don't need to translate "how the functions operate" into "what is the purpose of this code". That's the most powerful concept to understand when getting started with functional programming. We've declared what the shape of a valid report is without getting distracted by the details of how to validate.

## Building validation utilities

Based on the validations that we need to perform, we need a few functional utilities. We will compose them from small utilities provided by [Lodash FP][lodash-fp].

``` javascript
const any = require('lodash/fp/any')
const compose = require('lodash/fp/compose')
const get = require('lodash/fp/get')
const has = require('lodash/fp/has')
const map = require('lodash/fp/map')

module.exports.hasfields = compose(allPass, map(has))
module.exports.anyAt = (path, check) => compose(any(check), get(path))
```
What we've done is use function composition to glue together smaller things into utilities that check that an object has values for given fields with `hasFields`, and that an object passes a validation check for a certain leaf node.

# Conclusion

The goal of this post was to test the shape of JSON data produced by a command line utility, in this case a maven plugin. The **shape test** was necessary because we can't guarantee the values within the objects that are produced, we just need to ensure that we're populating data into all the fields we need. We have specific unit tests to ensure each piece of data is transformed properly, so this is an [integration level test][test-pyramid].

To perform our **shape test**, we need to execute our utility, then validate that we've created results with the right shape. This is where the fun started! We chose node for our shape test because of it's ability to stream io tasks, and it's strength in working with JSON data. 

There were 3 main steps in within our test: test preperation, code execution, and walking the results for validation. Each of these steps is an I/O task which could be managed with [Highland][highland-js] streams. For validation, we walked the filesystem, adding a report to the stream for validation, then we filtered valid reports. This is where [Lodash FP][lodash-fp] entered the picture. The validation steps for the reports could be represented clearly, with great purpose-focused names by using functional programming concepts like `currying`, & `function composition`. *This allowed us to express what shape a valid report has without getting distracted by how to perform the validation!*

[test-pyramid]: https://github.com/testdouble/contributing-tests/wiki/Testing-Pyramid
[node-streams]: https://nodejs.org/api/stream.html
[highland-js]: http://highlandjs.org/
[bacon-js]: https://baconjs.github.io/
[rxjs]: http://reactivex.io/
[lodash-fp]: https://github.com/lodash/lodash/wiki/FP-Guide

